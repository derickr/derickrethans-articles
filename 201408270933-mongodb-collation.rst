Natural Sorting with MongoDB
============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2014-08-27 09:33 Europe/London
   :Tags: blog, php, mongodb
   :Short: mdbcoll

Arranging English words in order is simple--well, most of the time. You simply
arrange them in alphabetical order. However sorting a set of German words, or
French words with all their accents, or Chinese with their different characters is a
*lot* harder than it looks. Sorting rules are specified through
"locales", which determine how accents are sorted, in which order the letters
are in and how to do case-insensitive sorts. There is a good set of those
sorting rules available through CLDR_, and there is a neat example to play
with all kinds of sorting at `ICU's demo site`_. If you really want to know
how the algorithms work, have a look at the Unicode Consortium's report on the
`Unicode Collation Algorithm`_.

.. _`ICU's demo site`: http://demo.icu-project.org/icu-bin/locexp?_=en_UK&d_=en&x=col
.. _CLDR: http://cldr.unicode.org/index/cldr-spec/collation-guidelines
.. _`Unicode Collation Algorithm`: http://www.unicode.org/reports/tr10/

Right now, MongoDB does not support indexes or sorting on anything but Unicode
Code Points. Basically, that means, that it can't sort anything but English.
There is a long standing issue, `SERVER-1920`_, that is at the top
of the priority list, but is not scheduled to be added to a future release. 
I expect this to be addressed at a point in the near future. 
However, with some tricks there is a way to solve the 
sorting problem manually.

Many languages, have their own implementation of the Unicode Collation
Algorithm, often implemented through ICU. PHP has an ICU based implementation
as part of the intl_ extension. And the class to use is the Collator_ class.

.. _intl: http://php.net/manual/en/book.intl.php
.. _Collator: http://php.net/manual/en/class.collator.php
.. _`SERVER-1920`: https://jira.mongodb.org/browse/SERVER-1920

The Collator class encapsulates the Collation Algorithm to allow you to sort
an array of text yourself, but it also allows you extract the "sort key". By
storing this generated sort key in a separate field in MongoDB, we can sort by
locale—and even multiple locales.

Take for example the following array of words::

	$words = [
		'bailey', 'boffey', 'böhm', 'brown', 'серге́й', 'сергій', 'swag',
		'svere' 
	];

Which we can turn into sort keys with a simple PHP script like::

	$collator = new Collator( 'en' );
	foreach ( $words as $word )
	{
		$sortKey = $collator->getSortKey( $word );
		echo $word, ': ', bin2hex( $sortKey ), "\n";
	}

We create a collator object for the ``en`` locale, which is generic English.
When running the script, the output is (after formatting)::

	bailey: 2927373d2f57010a010a
	boffey: 294331312f57010a010a
	böhm:   2943353f01859d060109
	brown:  294943534101090109
	серге́й: 5cba34b41a346601828d05010b
	сергій: 5cba34b41a6066010a010a
	swag:   4b53273301080108
	svere:  4b512f492f01090109

Those sort keys can be used to then sort the array of names. In PHP, that
would be::

	$collator->sort( $words );
	print_r( $words );

Which returns the following list::

	[0] => bailey
	[1] => boffey
	[2] => böhm
	[3] => brown
	[4] => svere
	[5] => swag
	[6] => серге́й
	[7] => сергій

We can extend this script, to use multiple collations, and import each word
including its sort keys into MongoDB.

Below, we define the words we want to sort on, and the collations we want to compare.
They are in order: English, German with phone book sorting, Norwegian, Russian
and two forms of Swedish: "default" and "standard"::

	<?php
	$words = [ 
		'bailey', 'boffey', 'böhm', 'brown', 'серге́й', 'сергій', 
		'swag', 'svere' 
	];
	$collations = [ 
		'en', 'de_DE@collation=phonebook', 'no', 'ru', 
		'sv', 'sv@collation=standard',
	];

Make the connection to MongoDB and clean out the collection::

	$m = new MongoClient;
	$d = $m->demo;
	$c = $d->collate;
	$c->drop();

Create the Collator objects for each of our collations::

	$collators = [];

	foreach ( $collations as $collation )
	{
		$c->createIndex( [ $collation => 1 ] );
		$collators[$collation] = new Collator( $collation );
	}

Loop over all the words, and for each collation we have define, use the
created Collator object to generate the sort key. We encode the sort key with
`bin2hex()`_ because sort keys are binary data, and MongoDB requires UTF-8 for
strings. My original plan of using MongoDB's BinData type did not work, as it
`sorts first according to the length of the data`_. Encoding with
`base64_encode()`_ also does not work, as it's encoding scheme does not keep
the original order. Encoding with `utf8_encode()`_ *does* work, but as it
creates some binary (but valid-for-MongoDB-UTF-8) data, it's not good to use as
an example.

::

	foreach ( $words as $word )
	{
		$doc = [ 'word' => $word ];
		foreach ( $collations as $collation )
		{
			$sortKey = $collators[$collation]->getSortKey( $word );
			$doc[$collation] = bin2hex( $sortKey );
		}
		$c->insert( $doc );
	}

.. _`bin2hex()`: http://docs.php.net/bin2hex
.. _`sorts first according to the length of the data`: http://docs.mongodb.org/manual/reference/bson-types/#comparison-sort-order
.. _`base64_encode()`: http://docs.php.net/base64_encode
.. _`utf8_encode()`: http://docs.php.net/utf8_encode

When we run the script, and see what's in the database, we find something like
the following for ``böhm``::

	> db.collate.find( { word: 'böhm' }).pretty();
	{
		"_id" : ObjectId("53fc721844670a35498b4569"),
		"word" : "böhm",
		"en" : "2943353f01859d060109",
		"de_DE@collation=phonebook" : "29432f353f0186870701848f06",
		"no" : "295aa105353f018687060108",
		"ru" : "2b45374101859d060109",
		"sv@collation=standard" : "295aa106353f01080108",
		"sv@collation=default" : "295aa106353f01080108"
	}

To see the sorting for the words in all the locales, I've added the following
to the end of the script::

	foreach ( $collations as $collation )
	{
		echo $collation, ":\n";
		
		$r = $c->find()->sort( [ $collation => 1 ] );
		foreach ( $r as $res )
		{
			echo $res['word'], ' ';
		}
		
		echo "\n\n";
	}

As you can see, we call `sort()`_ and specify which field to sort on. The
``$collation`` variable contains the name of the collation. In each stored
document, the field with the name of the collation, stores the sort key for
that collation as you saw in the previous MongoDB shell output.

.. _`sort()`: http://docs.php.net/MongoCursor.sort 

Running with this part of the code added, we get::

	en:
	bailey boffey böhm brown svere swag серге́й сергій 

	de_DE@collation=phonebook:
	bailey böhm boffey brown svere swag серге́й сергій 

	no:
	bailey boffey brown böhm svere swag серге́й сергій

	ru:
	серге́й сергій bailey boffey böhm brown svere swag 

	sv@collation=standard:
	bailey boffey brown böhm swag svere серге́й сергій 

	sv@collation=default:
	bailey boffey brown böhm svere swag серге́й сергій 

- In English, the ``ö`` in ``böhm`` sorts as an ``o``.
- In Germany's phone book collation, the ``ö`` in ``böhm`` sorts like an
  ``oe``.
- In Norwegian, the ``ö`` in ``böhm`` sorts as an extra letter after ``z``.
- In Russian, the Cyrillic letters sort before Latin letters.
- In Sweden's "standard" collation, the ``v`` and ``w`` are considered
  equivalent letters.

By generating a sort key for your data, you get to chose with which locale
MongoDB will do the sorting, but with the overhead of having to maintain an
index yourself. ICU, the library that lies underneath PHP's intl_ extension
supports a lot more customisations for collators, and even allows you to
define your own custom rules. In the future, we will likely see some of this
functionality make it into MongoDB as well. Until this implemented, generating
your own sort-key field for each document like this article shows, is your
best MongoDB-only approach. If you find collation sorting in MongoDB
important, feel free to vote on the `SERVER-1920`_ issue in Jira.
