New MongoDB Drivers for PHP and HHVM: Cursor Behaviour
======================================================

.. articleMetaData:: :Where: London, UK
   :Date: 2016-02-24 09:07 Europe/London
   :Tags: blog, php, mongodb
   :Short: newmongo3

We released a new version of the MongoDB driver for PHP (the
``mongodb`` extension) near the end of last year. In a `previous blog post`_,
I covered the back story of the how and why we undertook this effort. And in
`another post`_, I spoke about the architecture of the new driver. In this
episode I am discussion changes in cursor behaviour.

.. _`previous blog post`: /new-drivers.html
.. _`another post`: /new-drivers-part2.html

I noticed that a `comment`_ had been added to PHP's documentation, which
reads:

	I noticed that  ->sort is missing from the cursor.  Seems like the old
	driver has more functionality.

.. _comment: http://php.net/manual/en/class.mongodb-driver-cursor.php#118856

The new driver of course also allows for sorting of results of queries, but
no longer by calling the ``sort()`` method on the cursor object. This is
because the way how cursors are created is different between the drivers.

Legacy Driver
-------------

In the old driver, the cursor would not really be created until after the
first ``rewind()`` call on the iterator::

	<?php
	$m = new MongoClient;

	// Select 'demo' database and 'example' collection
	$collection = $m->demo->example;

	// Create the cursor
	$cursor = $collection->find();

At this moment, although the cursor has been created, the *query* has not been
executed and sent to the server. The query would only be executed until you
either use ``foreach ( $cursor as $result )`` or by calling
``$cursor->rewind()``. This gives you the chance to configure the cursor with
``sort()``, ``limit()``, and ``skip()`` before it is executed by the server::

	// Add sort, and limit
	$cursor->sort( [ 'name' => 1 ] )->limit( 40 );

After the cursor started iteration (through ``foreach`` or ``->rewind()``),
you can no longer call these three methods.

New Driver
----------

In the new driver, as soon as you have a `\\MongoDB\\Driver\\Cursor`_ object, it
has already been processed by the server. Because sort (and limit and skip)
parameters need to be send to the server before the query is executed, you can
not retroactively call them on the already created ``Cursor`` object.

.. _`\\MongoDB\\Driver\\Cursor`: http://php.net/manual/en/class.mongodb-driver-cursor.php

You can use sort (and limit and skip) with the new driver as well, by
specifying them as options to the `\\MongoDB\\Driver\\Query`_ object before
you pass the query to `\\MongoDB\\Driver\\Manager::executeQuery()`_::

	<?php
	$m = new \MongoDB\Driver\Manager();
	$ns = 'demo.example';

	// Create query object with all options:
	$query = new \MongoDB\Driver\Query(
		[], // query (empty: select all)
		[ 'sort' => [ 'name' => 1 ], 'limit' => 40 ] // options
	);

	// Execute query and obtain cursor:
	$cursor = $manager->executeQuery( $ns, $query );

.. _`\\MongoDB\\Driver\\Query`: http://php.net/manual/en/class.mongodb-driver-query.php
.. _`\\MongoDB\\Driver\\Manager::executeQuery()`: http://php.net/manual/en/mongodb-driver-manager.executequery.php

What's Next?
------------

In the next installment of this series on the new MongoDB driver, we will focus
on a new feature in the new MongoDB driver: ODS - the Object Data Store.
