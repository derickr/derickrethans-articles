MongoDB and arbitrary key names
===============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2014-02-11 09:41 Europe/London
   :Tags: blog, php, mongodb
   :Short: mdb-arb

I hang out on the MongoDB IRC channel (Freenode/#mongodb) quite a bit, and an
often recurring question is inherently abour storing values as keys. 

One of the examples is storing timed data points like this::

	{
		person: "derickr",
		steps_made: [
			{"20140201": 10800},
			{"20140202":  5906}
		]
	}

The sub key under ``steps_made`` is the date on which the steps are made, and
the value is the amount of steps for that day. A schema like this prevents the
same key from being used multiple times—enforcing uniqueness, and it's also
possible to add steps from future walks atomically::

	<?php
	$m = new MongoClient;
	$m->demo->steps->update(
		[ 'person' => "derickr" ],
		[ '$inc' => [ "steps_made.20140202" => 712 ] ],
		[ 'upsert' => true ]
	);
	?>

Because of the ``upsert`` option, this update query would even work if there
was no document for this person yet in the collection, and it would also work
in case a specific date did not have an entry yet.

Although it seems sensible to store the data like this, there are a few
problems when storing data like this. 

The first problem is this schema makes it impossible to find the step counts
for a range of dates—both as a normal query or as with the aggregation
framework. This is because you can **not** query on key names. You
would have to request whole documents, and pick out the correct keys in the
application. 

You can also not easily find which dates had a step count larger than 10000,
because you would need to **know** the key name (date) for the comparison
first.

Another problem is indexes. Indexes can only be made on field names. To be
able to use an indexed lookup with the above schema, you would need to create
an index on ``steps_made.20140201`` and on ``steps_made.20140202``, etc. That
is of course not practical.

And lastly, with the schema above you can not create aggregates on years or
months as, again, you can only manipulate **values** with the aggregation
framework and not with keys.

So what is the major issue here? The schema uses a **value** as a key. This
is like having an XML schema that looks like this::

	<steps>
		<person>derickr</person>
		<steps_made>
			<20140201>10800</20140201>
			<20140202>712</20140202>
		</steps_made>
	</steps>

This hopefully makes your eyes bleed.

For very similar reasons as you wouldn't do this in XML (validation, XPath
queries), you should not do this with MongoDB. 

One alternative is to store the data like this::

	{
		person: "derickr",
		steps_made: [
			{ date: "20140201", steps: 10800 },
			{ date: "20140202", steps:  5906 },
		]
	}

With this schema some of the previous issues go away.

You can create an index on the ``date`` field::

	<?php
	$m = new MongoClient;
	$m->demo->steps->ensureIndex( [ 'date' => 1 ] );
	?>

And you can create an aggregation for the average per month::

	<?php
	$m = new MongoClient();
	$r = $m->demo->steps->aggregate( [
		[ '$match' => [ 'person' => 'derickr' ] ],
		[ '$unwind' => '$steps_made' ],
		[ '$project' => [
			'person' => 1,
			'steps_made'=> '$steps_made.steps',
			'month' => [ '$substr' => [ '$steps_made.date', 0, 6 ] ]
		] ],
		[ '$group' => [
			'_id' => '$month',
			'avg' => [ '$avg' => '$steps_made' ]
		] ],
	] );
	var_dump( $r['result'] );
	?>

It is not yet possible to find all the step count for a range of dates, as they
are collectively stored in one document and you would always get a whole
document back.

You can not easily find which dates had a step count larger than
10000 with a normal query. However you can do that with the aggregation
framework, albeit not in a very efficiant way::

	<?php
	$m = new MongoClient();
	$r = $m->demo->steps->aggregate( [
		[ '$match' => [ 'person' => 'derickr' ] ],
		[ '$unwind' => '$steps_made' ],
		[ '$match' => [ 'steps_made.steps' => [ '$gt' => 10000 ] ] ]
	] );
	foreach( $r['result'] as $record )
	{
		echo $record['steps_made']['date'], "\n";
	}
	?>

An additional problem with storing the step count for all the days in the same
document is that the documents keep growing and growing when new days are
added. It is unlikely to hit the 16MB document limit soon as it would take
about 1050 years worth of "step data", but in general the recommendation is to
avoid having such a data structure. Growing documents also mean that it will
need to be moved around on disk a lot, which is not good for performance.

In the last two aggregation framework queries you see a common theme: an
``$unwind``. This is to break up each document into a document that represents
a single day. If we store the data like that ourselves, these aggregation
framework queries, as well as other queries become easier.

In our **second alternative** we therefore store the data like::

	{
		person: "derickr",
		date: "20140201",
		steps: 10800,
	}
	{
		person: "derickr",
		date: "20140202",
		steps: 5906,
	}

Adding steps for a single walk (and creating a new document for a new day) is
mostly the same::

	<?php
	$m = new MongoClient;
	$m->demo->steps->update(
		[ 'person' => 'derickr', 'date' => "20140201" ],
		[ '$inc' => [ 'steps' => 712 ] ],
		[ 'upsert' => true ]
	);
	?>

Finding the step count for a range of dates is now possible, and rather
trivial::

	<?php
	$m = new MongoClient;
	$r = $m->demo->steps->find( [
		'date' => [ '$gte' => "20140201", '$lt' <= "20140301" ]
	] );
	?>

Compared to the first alternative, the application doesn't need to filter
anything out of the returned document either.

Because we don't have to unwind on the ``steps_made`` field while aggregating
per-month, calculating the average is now simpler as the following
aggregation framework query shows::

	<?php
	$m = new MongoClient();
	$r = $m->demo->steps->aggregate( [
		[ '$project' => [
			'person' => 1,
			'steps'=> 1,
			'month' => [ '$substr' => [ '$date', 0, 6 ] ]
		] ],
		[ '$group' => [
			'_id' => '$month',
			'avg' => [ '$avg' => '$steps' ]
		] ],
	] );
	var_dump( $r['result'] );
	?>

And finding which days saw more than 10000 steps is now done with a trivial
query::

	<?php
	$m = new MongoClient;
	$r = $m->demo->steps->find(
		'person' => 'derickr',
		'steps' => [ '$gt' => 10000 ]
	);
	?>

So unless you have a requirement where you need to show all the step counts of
one person, I would recommend the second alternative as it is the most
flexible, and will likely provide the best performance. There are however
(other) use cases where the first alternative option makes sense, but I will
get back to that in a future article.
