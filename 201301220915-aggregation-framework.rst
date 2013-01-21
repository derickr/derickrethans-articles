MongoDB's aggregation framework
===============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2013-01-22 09:15 Europe/London
   :Tags: blog, php, mongodb
   :Short: mdbaggr

As part of my preparations for my MongoDB_ workshop at `PHP Benelux`_, I ran
into a nice use case for MongoDB's aggregation framework. As I have already
promised to write about it, this seems to be a good time to actually write the
article.

The dataset that I am using for the workshop contains of
restaurants and other points of interest, extracted from OpenStreetMap_ data.
As an example, one of the documents that I am storing is::

	{
		"_id" : "n558797601",
		"type" : NumberLong(1),
		"loc" : [ 4.4577708, 51.1611465 ],
		"tags" : [
			"addr:city=Edegem",
			"addr:country=BE",
			"addr:housenumber=398",
			"addr:postcode=2650",
			"addr:street=Mechelsesteenweg",
			"amenity=restaurant",
			"cuisine=regional",
			"name=La Rosa",
		]
	}

I wanted to find out which cuisines are used in all of the documents of my
dataset. Described in a different way: I want all the different
``cuisine=…`` tags as used in my dataset. Traditionally you would write a
really complex™ Map/Reduce job, but since MongoDB 2.2, there is a new feature
called the `aggregation framework`_. The aggregation framework is meant to be
an easy way to do fairly complex aggregation jobs.

The idea behind it is that each aggregation job is defined by a pipeline of
operators. Each operator does a specific task. There is for example `$match`_,
which allows you to restrict which documents pass through. If the document
matches the predicate contained in the ``$match`` operator, then it is allowed
through, and otherwise it is dropped from the pipeline.

On the MongoDB_ shell, you use the `aggregate()`_ shell helper to execute a
pipeline of operators, and with PHP you use the `MongoCollection::aggregate()`_
method.

In order to let pass all the documents that have the tag
``amenity=restaurant``, you would run on the shell::

	db.poi.aggregate( { $match: { tags: "amenity=restaurant" } } );

And in PHP you would use the following script::

	<?php
	$m = new MongoClient;
	$c = $m->demo->poi;
	$result = $c->aggregate(
		array(
			array( '$match' => array( 'tags' => 'amenity=restaurant' ) )
		)
	);

	var_dump( $result['result'] );
	?>

If everything goes well, the return value of the ``aggregate()`` helper method
is an array with two elements: *ok* with a value of ``double(1)`` as well as a
*result* element containing an array of all the documents that made it through
the whole pipeline. Because the aggregation framework returns all of its
results as one document over the network, the full result is limited to 16MB.
There are also memory limits internally, so it is always wise to restrict the
data coming through the pipeline with an operator as soon as you can.

Let's try to construct a pipeline to get a list of all the different
``cuisine=…`` tags *including* how often they appear.

**$match**

The first thing to do is to match all documents that have the ``cuisine=…``
tag in the first place. We use the ``$match`` operator for that. If a
``$match`` operator is the first operator in a pipeline than it can make use
of indexes. It is therefore important that you have indexes in place. The
``$match`` operator definition that we need is::

	$allWithCuisine = array(
		'$match' => array( 'tags' => new MongoRegex( '/^cuisine=/' ) )
	);

To fit that into our script, we change it to::

	<?php
	$m = new MongoClient;
	$c = $m->demo->poi;

	$allWithCuisine = array( 
		'$match' => array( 'tags' => new MongoRegex( '/^cuisine=/' ) )
	);  

	$result = $c->aggregate(
		array( $allWithCuisine )
	);

	var_dump( $result['result'] );
	?>

In my case, this returns an array of 117 documents in the ``result`` element.

For each pipeline step, we create a variable such as ``$allWithCuisine`` to
define the operator, and then add those to the array that is passed to the
``aggregate()`` method.

**$project**

To reduce the amount of data going through the pipeline, our next step is to
remove all the fields from the documents that we are not interested in. In
fact, we are actually only interested in the ``tags`` field. In order to
"re-shape" a document into a different structure, we use the `$project`_
operator. In its most basic form, it works the same as the `$fields`_ argument
to `MongoCollection::find()`_. It is a lot more powerful that that, as it
supports changing the whole structure of a document, as well as computed
fields. Have a look at the `$project`_ documentation for some more
inspiration. 

As we are only interested in the ``tags`` field of the documents, we just put that in
the projection::

	$justTheTags = array(
		'$project' => array( 'tags' => 1 )
	);

and modify the ``aggregate()`` call::

	$result = $c->aggregate(
		array( $allWithCuisine, $justTheTags )
	);

**$unwind**

In order to be able to do some work on individual tags, we need to split up
the ``tags`` array. The `$unwind`_ operator does just that. It is a rather
tricky operator to explain, so I will try with an example. Take for example
this document::

	{
		_id: "n478547159",
		related_ids: [ "n516583937", "n401309937" ]
	}

Using the ``$unwind`` operator on ``related_ids`` removes each document from
the pipeline and introduces **two** new ones. One for each of the
``related_ids`` elements. At the same time, it replaces the ``related_ids``
array with one of the values. Running ``{ $unwind: '$related_ids' }`` turns
the above document into the following two::

	{
		_id: "n478547159",
		related_ids: "n516583937"
	}
	{
		_id: "n478547159",
		related_ids: "n401309937"
	}

In our case, we want a document for each of the elements in the ``tags`` array
so that we can group on this field later. We introduce our ``$unwind``
operator::

	$unwindTags = array(
		'$unwind' => '$tags'
	);

and add it to our list of pipeline operators::

	$result = $c->aggregate(
		array( $allWithCuisine, $justTheTags, $unwindTags )
	);

When we run the script now, we get 554 documents in the following form::

	…
	array (
		'_id' => 'n470071537',
		'tags' => 'amenity=fast_food',
	),
	array (
		'_id' => 'n470071537',
		'tags' => 'cuisine=burger',
	),
	array (
		'_id' => 'n470071537',
		'tags' => 'name=C&Ms',
	),
	…

Because we are only interested in the ``cuisine=…`` tag, we use our previously
defined ``$match`` operator to filter out all the documents that don't have
this tag::

	$result = $c->aggregate(
		array( $allWithCuisine, $justTheTags, $unwindTags, $allWithCuisine )
	);

Which leaves us with 117 documents again.

**$group**

Now that we have extracted and massaged our data, we are ready to group the
documents by their ``cuisine=…`` key. The `$group`_ operator groups all
documents in the pipeline by a key, and allows for computed fields. In our
case we want to group by the ``tags`` field::

	$groupByTags = array(
		'$group' => array( '_id' => '$tags' )
	);

Then we add it to our list of pipeline operators::

	$result = $c->aggregate(
		array(
			$allWithCuisine, $justTheTags, $unwindTags, $allWithCuisine,
			$groupByTags,
		)
	);

Our results includes one document for each distinct ``$tags`` value. A small
excerpt::

	…
	array (
		'_id' => 'cuisine=kebab;turkish',
	),
	array (
		'_id' => 'cuisine=pizza',
	),
	array (
		'_id' => 'cuisine=fine_dining',
	),
	…

In order to also have a count for each of the distinct values in an extra
``count`` field, we need to modify the ``$group`` operator in the pipeline. I
have already mentioned that you can have computed fields, and that's what we
need here. A computed field attaches an expression to a field name. In this
case, we want the ``count`` field to increment by ``1`` each time we find a
document with this field—for this we use the `$sum`_ operator::

	$groupByTags = array(
		'$group' => array( 
			'_id' => '$tags',
			'count' => array( '$sum' => 1 )
		)
	);

Each document that now comes out of the pipeline looks like::

	…
	array (
		'_id' => 'cuisine=turkish',
		'count' => 2,
	),
	array (
		'_id' => 'cuisine=japanese',
		'count' => 7,
	),
	array (
		'_id' => 'cuisine=italian',
		'count' => 10,
	),
	…

Other `computed fields`_ are also possible. If we want for example to also
record which original ``_id`` field had a ``cuisine=…`` tag, we modify the
group operator to add this field as well::

	$groupByTags = array(
		'$group' => array(
			'_id' => '$tags',
			'count' => array( '$sum' => 1 ),
			'ids' => array( '$addToSet' => '$_id' )
		)
	);

The `$addToSet`_ operator adds the original ``_id`` value as a new value to the
``ids`` array for each grouped ``cuisine=…`` tag. When we run the full script
with the modified ``$group`` operator, we now get documents in the form::

	array (
		'_id' => 'cuisine=friture',
		'count' => 3,
		'ids' => array (
			0 => 'n2040116467',
			1 => 'n1701471939',
			2 => 'n1701465430',
		),
	),

Because our requirement didn't really want this ``ids`` array, I have removed
it from future examples.

**$sort**

The only thing left to do is now sort our ``cuisine=…`` tags with the most
used tags first.  For this we use the `$sort`_ operator. The sort operator
works in the same way as the `MongoCursor::sort()`_ method and accepts the
same arguments. In order to sort by the ``count`` field in descending order,
we create the pipeline operator as follows::

	$sort = array(
		'$sort' => array( 'count' => -1 )
	);

And add it to our pipeline::

	$result = $c->aggregate(
		array(
			$allWithCuisine, $justTheTags, $unwindTags, $allWithCuisine,
			$groupByTags, $sort,
		)
	);

When running our script now, we get a list of all distinct ``cuisine=…`` tags
ordered by their occurrence::

	array (
		'_id' => 'cuisine=regional',
		'count' => 19,
	),
	array (
		'_id' => 'cuisine=burger',
		'count' => 15,
	),
	array (
		'_id' => 'cuisine=chinese',
		'count' => 14,
	),
	…

**Conclusion**

With this I conclude my introduction to the aggregation framework. You can
find the final script here_. The documentation_ is extensive so I would suggest
to give it a good read.

I'm going back to preparing my PHP Benelux workshop now!

.. _MongoDB: http://mongodb.org
.. _`PHP Benelux`: http://conference.phpbenelux.eu/2013/
.. _`aggregation framework`: http://docs.mongodb.org/manual/aggregation/
.. _documentation: http://docs.mongodb.org/manual/aggregation/
.. _`$match`: http://docs.mongodb.org/manual/reference/aggregation/#_S_match
.. _`$unwind`: http://docs.mongodb.org/manual/reference/aggregation/#_S_unwind
.. _`$group`: http://docs.mongodb.org/manual/reference/aggregation/#_S_group
.. _`aggregate()`: http://docs.mongodb.org/manual/applications/aggregation/#use
.. _`MongoCollection::aggregate()`: http://docs.php.net/manual/en/mongocollection.aggregate.php
.. _`$project`: http://docs.mongodb.org/manual/reference/aggregation/#_S_project
.. _`$sum`: http://docs.mongodb.org/manual/reference/aggregation/sum/#_S_sum
.. _`$addToSet`: http://docs.mongodb.org/manual/reference/operator/addToSet/
.. _`$fields`: http://docs.php.net/manual/en/mongocollection.find.php#refsect1-mongocollection.find-parameters
.. _`MongoCollection::find`: http://docs.php.net/manual/en/mongocollection.find.php
.. _here: /files/aggregation.php.txt
.. _`computed fields`: http://docs.mongodb.org/manual/reference/aggregation/#group-operators
.. _`$sort`: http://docs.mongodb.org/manual/reference/aggregation/sort/#_S_sort
.. _`MongoCursor::sort`: http://docs.php.net/manual/en/mongocursor.sort.php
.. _OpenStreetMap: http://openstreetmap.org
