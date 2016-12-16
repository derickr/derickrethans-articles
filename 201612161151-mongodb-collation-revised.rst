Natural Language Sorting with MongoDB 3.4
=========================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2016-12-16 11:51 Europe/London
   :Tags: blog, php, mongodb
   :Short: mdbcoll34

Arranging English words in order is simple—most of the time. You simply
arrange them in alphabetical order. Sorting a set of German words, or French
words with all of their accents, or Chinese with their different characters is
a *lot* harder than it looks. Sorting rules are specified through
*locales*, which determine how accents are sorted, in which order the
characters are in, and how to do case-insensitive sorting. There is a good set
of those sorting rules available through CLDR_, and there is a neat example to
play with all kinds of sorting at `ICU's demo site`_. If you want to know how
the algorithms work, have a look at the Unicode Consortium's report on the
`Unicode Collation Algorithm`_.

.. _`ICU's demo site`: http://demo.icu-project.org/icu-bin/locexp?_=en_UK&d_=en&x=col
.. _CLDR: http://cldr.unicode.org/index/cldr-spec/collation-guidelines
.. _`Unicode Collation Algorithm`: http://www.unicode.org/reports/tr10/

Years ago I wrote_ about collation and MongoDB. There is an old issue in
MongoDB's JIRA tracker, `SERVER-1920`_, to implement collation so that sorting
and indexing could work depending on the different sorting orders as described
for each language (locale).

Support for these collations have finally landed in MongoDB 3.4 and in this
article we are going to have a look at how they work.

.. _wrote: /mongodb-collation.html
.. _`SERVER-1920`: https://jira.mongodb.org/browse/SERVER-1920

How Unicode Collation Works
---------------------------

Many computer languages have their own implementation of the Unicode Collation
Algorithm, often implemented through ICU. PHP has an ICU based implementation
as part of the intl_ extension, in the form of the Collator_ class.

.. _intl: http://php.net/manual/en/book.intl.php
.. _Collator: http://php.net/manual/en/class.collator.php

The Collator class encapsulates the Unicode Collation Algorithm to allow you
to sort an array of text yourself. It also allows you to visualise the
"sort key" to see how the algorithm works:

Take for example the following array of words::

    $dictionary = [
        'boffey', 'bøhm', 'brown',
    ];

Which we can turn into sort keys, and sort using the ``en`` locale (English)::

    $collator = new Collator( 'en' );
    foreach ( $dictionary as $word )
    {
        $sortKey = $collator->getSortKey( $word );
        $dictionaryWithKey[ bin2hex( $sortKey ) ] = $word;
    }

    ksort( $dictionaryWithKey );
    print_r( $dictionaryWithKey );

Which outputs::

    Array
    (
        [2b4533333159010a010a] => boffey
        [2b453741014496060109] => bøhm
        [2b4b45554301090109] => brown
    )

If we would do this according to the ``nb`` (Norwegian) locale, the output
would have ``brown`` and ``bøhm`` reversed::

    Array
    (
        [2b4533333159010a010a] => boffey
        [2b4b45554301090109] => brown
        [2b5c6703374101080108] => bøhm
    )

The sort key for ``bøhm`` has now changed, so that its numerical value now
makes it sort after ``brown`` instead of before ``brown``. In Norwegian, the
``ö`` is a distinct letter that sorts after ``z``.

MongoDB 3.4
-----------

Before the release of MongoDB 3.4, it was not possible to do a locale based
search. As case-insensitivity is just another property of a locale, that was
not supported either. Many users worked around this by storing a lower case
version of the value in separate field just to do a case-insensitive search.
But this has now changed with the implementation of `SERVER-1920`_.

In MongoDB 3.4 you may attach a *default locale* to a collection::

    db.createCollection( 'dictionary', { collation: { locale: 'nb' } } );

A *default* locale is used for any query without a different locale being
specified with the query. Compare the default (``nb``) locale::

    > db.dictionary.find().sort( { word: 1 } );
    { "_id" : ObjectId("5846d65210d52027a50725f0"), "word" : "boffey" }
    { "_id" : ObjectId("5846d65210d52027a50725f1"), "word" : "brown" }
    { "_id" : ObjectId("5846d65210d52027a50725f2"), "word" : "bøhm" }

With the English (``en``) locale::

    > db.dictionary.find().collation( { locale: 'en'} ).sort( { word: 1 } );
    { "_id" : ObjectId("5846d65210d52027a50725f0"), "word" : "boffey" }
    { "_id" : ObjectId("5846d65210d52027a50725f2"), "word" : "bøhm" }
    { "_id" : ObjectId("5846d65210d52027a50725f1"), "word" : "brown" }

The *default* locale of a collection is also inherited by an index when you
create one::

    db.dictionary.createIndex( { word: 1 } );

    db.dictionary.getIndexes();
    [
        …
        {
            "v" : 2,
            "key" : { "word" : 1 },
            "name" : "word_1",
            "ns" : "demo.dictionary",
            "collation" : {
                "locale" : "nb",
                "caseLevel" : false,
                "caseFirst" : "off",
                "strength" : 3,
                "numericOrdering" : false,
                "alternate" : "non-ignorable",
                "maxVariable" : "punct",
                "normalization" : false,
                "backwards" : false,
                "version" : "57.1"
            }
        }
    ]


From PHP
--------

All the examples below are using the PHP driver_ for MongoDB (1.2.0) and
the accompanying library_ (1.1.0). These are the minimum versions to work with
locales.

.. _driver: https://pecl.php.net/mongodb
.. _library: https://packagist.org/packages/mongodb/mongodb

To use the MongoDB PHP Library, you need to use Composer_ to install it, and
include the Composer-generated autoloader to make the library available
to the script. In short, that is::

    php composer require mongodb/mongodb=^1.1.0

And at the start of your script::

    <?php
    require 'vendor/autoload.php';

.. _Composer: https://getcomposer.org/

In this first example, we are going to drop the collection ``dictionary`` from
the ``demo`` database, and create a collection with the default collation
``en``. We also create an index on the ``word`` field and insert a couple of
words.

First the set-up and assigning of the database handle (``$demo``)::

    $client = new \MongoDB\Client();
    $demo = $client->demo;

Then we drop the ``dictionary`` collection::

    $demo->dropCollection( 'dictionary' );

We create a new collection ``dictionary`` and set the default collation for this
collection to the ``en`` locale::

    $demo->createCollection(
        'dictionary',
        [
            'collation' => [ 'locale' => 'en' ],
        ]
    );
    $dictionary = $demo->dictionary;

We create the index, and we also give the index the name ``dictionary_en``.
MongoDB supports multiple indexes with the same field pattern, as long as they
have a different name **and** have different collations (e.g. locale, or
locale options)::

    $dictionary->createIndex( 
        [ 'word' => 1 ],
        [ 'name' => 'dictionary_en' ]
    );  

And then we insert some words::

    $dictionary->insertMany( [
        [ 'word' => 'beer' ],
        [ 'word' => 'Beer' ],
        [ 'word' => 'côte' ],
        [ 'word' => 'coté' ],
        [ 'word' => 'høme' ],
        [ 'word' => 'id_12' ],
        [ 'word' => 'id_4' ],
        [ 'word' => 'Home' ],
    ] );

When doing a query, you can specify the locale for that operation. Only one
locale can be used for a single operation, which means that MongoDB uses
the same locale for the ``find`` and the ``sort`` parts of a query.
We do intent to support more granular support for using collations on
different parts of an operation. This is tracked in `SERVER-25954`_.

.. _`SERVER-25954`: https://jira.mongodb.org/browse/SERVER-25954

Using the Default Locale
------------------------

Let's do a query while sorting with the ``en`` locale. Because this is the
default locale for this collection, we don't have to specify it. We also
define a helper function to show the result of this query, and further
queries::

    function showResults( string $name, \MongoDB\Driver\Cursor $results )
    {
        echo $name, ":\n";
        foreach( $results as $result )
        {
            echo $result->word, " ";
        }
        echo "\n\n";
    }

    showResults( 
        "Sort with default locale",
        $dictionary->find( [], [ 'sort' => [ 'word' => 1 ] ] )
    );  

This outputs::

    Sort with default locale:
    beer Beer coté côte Home høme id_12 id_4 


Only the Base Character
-----------------------

There are many variants of locales. The *strength* option defines the number of
levels that are used to perform a comparison of characters. At *strength=1*,
only base characters are compared. This means that with the ``en`` locale:
``beer == Beer``, ``coté == côte``, and ``Home == høme``.

You can specify the strength while doing each query. First we use the ``en``
locale and strength ``1``. This is equivalent to a case insensitive match::

    showResults(
        "Match on base character only",
        $dictionary->find( 
            [ 'word' => 'beer' ],
            [ 'collation' => [ 'locale' => 'en', 'strength' => 1 ] ]
        )
    );

Which outputs::

    Match on base character only:
    beer Beer

Strength 1 also ignores accents on characters, such as in::

    showResults(
        "Match on base character only, ignoring accents",
        $dictionary->find(
            [ 'word' => 'home' ],
            [ 'collation' => [ 'locale' => 'en', 'strength' => 1 ] ]
        )
    );

Which outputs::

    Match on base character only, ignoring accents:
    høme Home 

As *strength*, or any of the other options we will see later, changes the
sort key for a string, it is important that you realise that because of this,
an index in MongoDB will only be used **if it is created with the exact same
locale options as the query**.

Because we only have an index on ``word`` with the default ``en`` locale, all
other examples do **not** make use of an index while matching or sorting.
If you want to make an indexed lookup for the ``en``/``strength=1`` example,
you need to create an index with::

    $dictionary->createIndex( 
        [ 'word' => 1 ],
        [
            'name' => 'word_en_strength1',
            'collation' => [
                'locale' => 'en',
                'strength' => 1
            ],
        ]
    );  

Different Locales, Different Letters
------------------------------------

Not every language considers an accented character a variant of the original
base character. If we run the last example with the Norwegian Bokmål (``nb``)
locale we get a different result::

    showResults(
        "Match on base character only (nb locale)",
        $dictionary->find(
            [ 'word' => 'home' ],
            [ 'collation' => [ 'locale' => 'nb', 'strength' => 1 ] ]
        )
    );

Which outputs::

    Match on base character only (nb locale), ignoring accents:
    Home 

In Norwegian, the ``ø`` sorts as a distinct letter after ``z``, where the
alphabet ends with: ``y`` ``z`` ``æ`` ``ø`` ``å``.

Sorting Accents
---------------

*Strength 2* takes into account accents on letters while matching and
sorting. If we run the match on ``home`` in the English locale with strength
2, we get::

    showResults(
        "Match on base character with accents",
        $dictionary->find(
            [ 'word' => 'home' ],
            [ 'collation' => [ 'locale' => 'en', 'strength' => 2 ] ]
        )
    );

Which outputs::

    Match on base character with accents:
    Home 

The word ``høme`` is no longer included. However, the case of characters is
still not considered::

    showResults(
        "Match on base character with accents (and not case sensitive)",
        $dictionary->find(
            [ 'word' => 'beer' ],
            [ 'collation' => [ 'locale' => 'en', 'strength' => 2 ] ]
        )
    );

Which outputs::

    Match on base character with accents (and not case sensitive):
    beer Beer 

Again, more fun can be had while sorting with accents, because languages
do things differently. If we take the words ``cøte`` and ``coté``, we see a
difference in sorting between the ``fr`` (French) and ``fr_CA`` (Canadian
French) locales::

    showResults(
        "Sorting accents in French (France)",
        $dictionary->find(
            [ 'word' => new \MongoDB\BSON\Regex( '^c' ) ],
            [ 
                'collation' => [ 'locale' => 'fr', 'strength' => 2 ],
                'sort' => [ 'word' => 1 ],
            ]
        )
    );

    showResults(
        "Sorting accents in Canadian French",
        $dictionary->find(
            [ 'word' => new \MongoDB\BSON\Regex( '^c' ) ],
            [
                'collation' => [ 'locale' => 'fr_CA', 'strength' => 2 ],
                'sort' => [ 'word' => 1 ],
            ]
        )
    );

Which outputs::

    Sorting accents in French (France):
    coté côte 

    Sorting accents in Canadian French:
    côte coté 

In Canadian French, the accents sort from back to front. This is called
`Backward Secondary Sorting`_ sorting, and is an option you can set on any
locale-based query. Some language locales have different `default values`_ for
options. To make the French Canadian sort the "wrong" way, we can specify the
additional ``backwards`` option::

    showResults(
        "Sorting accents in Canadian French, the 'wrong' way",
        $dictionary->find(
            [ 'word' => new \MongoDB\BSON\Regex( '^c' ) ],
            [
                'collation' => [ 'locale' => 'fr_CA', 'strength' => 2, 'backwards' => false ],
                'sort' => [ 'word' => 1 ],
            ]
        )
    );

Which outputs::

    Sorting accents in Canadian French, the 'wrong' way:
    coté côte

.. _`Backward Secondary Sorting`: http://userguide.icu-project.org/collation/concepts#TOC-Backward-Secondary-Sorting
.. _`default values`: https://docs.mongodb.com/manual/reference/collation-locales-defaults/#collation-default-parameters

Interesting Locales
-------------------

There are a few other interesting sorting and matching methods in different
locales.

- In Germany's phone book collation, the ``ö`` in ``böhm`` sorts like an
  ``oe``.
- In Russian, the Cyrillic letters sort before Latin letters.
- In Sweden's "standard" collation, the ``v`` and ``w`` are considered
  equivalent letters.

As an example::

    $demo->dropCollection( 'dictionary' );

    $dictionary->insertMany( [
        [ 'word' => 'swag' ],
        [ 'word' => 'Boden' ],
        [ 'word' => 'böse' ],
        [ 'word' => 'Bogen' ],
        [ 'word' => 'sverre' ],
        [ 'word' => 'Валенти́на' ],
        [ 'word' => 'Ю́рий' ],
    ] );

    $locales = [
        'de',
        'de@collation=phonebook',
        'ru',
        'sv@collation=standard',
    ];

    foreach( $locales as $locale )
    {
        showResults(
            "Sorting with the '$locale' locale",
            $dictionary->find(
                [],
                [
                    'collation' => [ 'locale' => $locale, 'strength' => 2 ],
                    'sort' => [ 'word' => 1 ]
                ]
            )
        );
    }

Which outputs::

    Sorting with the 'de' locale:
    Boden Bogen böse sverre swag Валенти́на Ю́рий 

    Sorting with the 'de@collation=phonebook' locale:
    Boden böse Bogen sverre swag Валенти́на Ю́рий 

    Sorting with the 'ru' locale:
    Валенти́на Ю́рий Boden Bogen böse sverre swag 

    Sorting with the 'sv@collation=standard' locale:
    Boden Bogen böse swag sverre Валенти́на Ю́рий 

Please also note that I had to set ``strength`` to ``2`` here, as Germans
like capitalizing their nouns as well as names!

Other Options
-------------

The default strength is 3, which besides base character and accents, also
takes the case into account. A search for ``beer`` will no longer find
``Beer`` (☹).

But there are a few other things you can configure with locales. If you paid
attention, you saw that my word list includes ``id_4`` and ``id_12``. If you
sort this in the normal default order, you will see the following::

    showResults(
        "Sorting with numbers in strings",
        $dictionary->find(
            [ 'word' => new \MongoDB\BSON\Regex( '^id_' ) ],
            [ 'sort' => [ 'word' => 1 ] ]
        )
    );

Which outputs::

    Sorting with numbers in strings:
    id_12 id_4 

In order to fix that, you can set the ``numericOrdering`` option on the
locale, as this done here::

    showResults(
        "Sorting with numbers in strings, properly",
        $dictionary->find(
            [ 'word' => new \MongoDB\BSON\Regex( '^id_' ) ],
            [
                'collation' => [ 'locale' => 'en', 'numericOrdering' => true ],
                'sort' => [ 'word' => 1 ],
            ]
        )
    );

Which then outputs::

    Sorting with numbers in strings, properly:
    id_4 id_12 

Other options are also available, and are documented in the Collation_ section of
the MongoDB manual.

.. _Collation: https://docs.mongodb.com/manual/reference/collation/#collation-document

Conclusion
----------

Languages and language sorting is complex. In the examples above I have only
shown collations with Western Latin and Cyrillic characters. Asian languages
make searching and sorting even more complicate. With Japanese and Chinese
characters, there are different ways of determining their sort order for
example. But getting sorting strings and matching search phrases right is very
important for the usability of applications. And because of that, the
implementation of `SERVER-1920`_ is a very welcome addition to MongoDB.
The implementation in MongoDB supports every locale and variant that ICU
supports. A list of these locales with their identifier can be found in
the documentation_.

Further work on collation support is also expected. To track issues and vote
for them, please refer to list on JIRA_.

.. _documentation: https://docs.mongodb.com/manual/reference/collation-locales-defaults/#supported-languages-and-locales
.. _JIRA: https://jira.mongodb.org/issues/?jql=project%20%3D%20SERVER%20AND%20text%20~%20%22collation%22%20and%20status%20%3D%20Open
