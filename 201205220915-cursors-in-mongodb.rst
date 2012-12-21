MongoDB Cursors with PHP
========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2012-05-22 09:15 Europe/London
   :Tags: blog, php, mongodb
   :Short: mongocur

Recently I was asked to improve the `MongoCursor::batchSize`_
documentation. This began an indepth investigation in how the PHP
driver_ for MongoDB_ handles pulling data that's been queried from the
MongoDB server. Here are my findings.

A MongoCursor_ is created as soon as you run the `find()`_ method on a
MongoCollection_ object, like in::

	$m = new Mongo();
	$collection = $m->demoDb->demoCollection;
	$cursor = $collection->find();

.. image:: /images/content/cursor.gif
   :align: right

Just calling ``find()`` will only create a cursor object, and does not
immediately send the query to the server for processing. That is only
done as soon as you start reading from the cursor for the first time.
Because of this, you can call additional methods on the newly created
cursor object that still influence how the query is run on the server.
One of such examples is the `sort()`_ method that makes the result sort
according to its arguments (in this example, by name)::

	$cursor->sort( array( 'name' => 1 ) );
	$result = $cursor->getNext();

When you then call `getNext()`_ on ``$cursor`` the driver sends to the
server the query, and requests to return a default number of documents
in the first batch. The default *Batch Size* is 101. Let's have a look
on what's get send on the wire in our simple query for all documents,
sorted by name:

.. image:: /images/content/01-find-sort-name.jpg

The *Number to Return* is 0, which means to use the default. So even
although we only want to fetch one result (``getNext()`` asks the cursor
for the next document **only**), the server returns 101 documents:

.. image:: /images/content/02-find-sort-name.jpg

The driver stores all 101 documents locally and during the next 100 calls to
``getNext()`` the driver will simply return the documents from the local
memory. Once ``getNext()`` gets called for the 102th time, the driver connects
back to the server to request more documents::

	// skip the other 100 docs
	for ($i = 0; $i < 100; $i++) { $cursor->getNext(); }
	// request document 102:
	$result = $cursor->getNext();

When the driver asks for more documents separately (i.e., not at the
same time it is issuing a query) without a specific batch size, the
server fills up 4MB of documents. On the wire, the request for *Get More*
looks like:

.. image:: /images/content/03-find-sort-name.jpg

and the reply like:

.. image:: /images/content/04-find-sort-name.jpg

As you can see, the returned data is ``4194378`` bytes, and the *Number
Returned* is ``34673``.

**Setting your own batch size**

You can instruct the driver to use different batch sizes, by using the 
`batchSize()`_ method on the ``$cursor``. In this new example, we use
the ``batchSize()`` method to request ``25`` documents per round trip 
to the server::

	$cursor = $collection->find()->sort( array( 'name' => 1 ) );
	$cursor->batchSize(25);
	$result = $cursor->getNext();

When we run this script, we will see the following on the wire:

.. image:: /images/content/05-batch25.jpg

As expected, the *Number to Return* is now 25. During iteration, all
query results are returned from the server to the driver in batches of 25
documents::

	// retrieve another 25 documents to trigger the getMore
	for ($i = 0; $i < 25; $i++) { $cursor->getNext(); }

Which creates this query:

.. image:: /images/content/06-batch25.jpg

And this reply:

.. image:: /images/content/07-batch25.jpg

As you can see, another 25 documents are returned, starting from the 25th
document.

**Using limit**

Besides ``batchSize()``, the driver also supports `limit()`_. ``limit()`` is
something that exclusively happens on the client side. It is just a counter
that restricts how often the cursor can iterate over the data. However,
the driver also uses the value passed to the function to ask for only the
amount of documents that it still needs to fetch. This means that if we run
this script, the driver will request 50000 documents::

	$cursor = $c->find()->sort( array( 'name' => 1 ) );
	$cursor->limit( 50000 );
	$res = $cursor->getNext();

On the wire we'll see:

.. image:: /images/content/09-limit50000.jpg

Sadly, not all 50000 documents fit in the first reply (as every reply is
limited to 4MB) and the server replies that it has only returned ``34678``
documents:

.. image:: /images/content/10-limit50000.jpg

The driver now calculates how many documents it still needs to fullfill it's
limit of 50000: ``50000 - 34678 = 15322``. It then requests those images with a
*Get More* query:

.. image:: /images/content/11-limit50000.jpg

It is also possible to combine ``limit()`` and ``batchSize()``.

**Combining limit and batchSize**

If we just use ``limit()`` then the driver will always try to fetch as many
documents it can to full-fill the limit. This sometimes means it will fill up
all 4MB of the maximum allowed reply packet, especially if your limit is set
high enough (in our example, that's more than ``34678`` documents). That's not
often what you want, and you can use a combination of ``limit()`` and
``batchSize()`` to fix it. If we want to query at most 128 documents with at
most 50 documents per batch, we can specify that as::

	$cursor = $c->find()->sort( array( 'name' => 1 ) );
	$cursor->limit( 128 )->batchSize( 50 );
	$res = $cursor->getNext();
	// retrieve the other 127 documents that we still want
	for ($i = 0; $i < 127; $i++) { $cursor->getNext(); }

On the wire, we'll see the following exchange:

Initial query:

.. image:: /images/content/12-limitbatch.jpg

First 50 documents:

.. image:: /images/content/13-limitbatch.jpg

First *Get More* for the second batch of 50:

.. image:: /images/content/14-limitbatch.jpg

The second batch of 50 documents:

.. image:: /images/content/15-limitbatch.jpg

The second and last *Get More* for a batch of 28 (``128 - 50 - 50 = 28``):

.. image:: /images/content/16-limitbatch.jpg

And the last batch of 28 documents returned:

.. image:: /images/content/17-limitbatch.jpg

I wouldn't quite suggest you use a small batch such as 50 though as it would
incur lots of round trips from and to the server. In some cases a small
batch size makes sense. 

Take for example a situation where you need to process 100.000s of documents
and the processing time for each item is 2 seconds. In the first batch, the
driver will pull 101 documents, which in total will take 202 seconds. When the
driver attempts to fetch the next batch with *Get More*, it fails because the
default cursor time-out (on the client size) is only 30 seconds. Setting your
batch size to 5 in this case avoids your cursor from timing out. You can of
course also `change the cursor timeout`__. Do not use the `immortal`__ flag
though, as that means something else (see *NoCursorTimeout* at the `MongoDB
Wire Protocol`__ documentation).

**One More Thing**

Setting the batch size to 1, as well as setting a negative batch size has
a special meaning for the MongoDB server. In both cases, it instructs the
server to return up to the absolute value of the requested and then terminate
the cursor, allowing no further documents to be fetched. That means that this
script doesn't do what you expect it to::

	$cursor = $c->find()->sort( array( 'name' => 1 ) );
	$cursor->batchSize( 1 )->limit( 10 );
	$cursor->getNext();
	var_dump( $cursor->getNext() );

The ``var_dump()`` for the second ``getNext()`` will always return ``NULL``.
However, if you set a batch size of 2, it works just like you would expect::

	$cursor = $c->find()->sort( array( 'name' => 1 ) );
	$cursor->batchSize( 2 )->limit( 10 );
	$cursor->getNext(); // item 1
	$cursor->getNext(); // item 2
	var_dump( $cursor->getNext() ); // item 3

Now let's set a batch size of ``-2`` and see what happens on the wire. The
request is:

.. image:: /images/content/18-neg-batch.jpg

As you can see, the driver really sends ``-2`` to the server, indicating that
negative batch sizes are handled on the server side.

And the reply is:

.. image:: /images/content/19-neg-batch.jpg

The *Cursor ID* is ``0`` in the reply, meaning that there is no cursor to
fetch further documents.

We have now come to the end of the article on MongoDB's cursors. All the
screenshots of the packets on the wire were made with WireShark, which has
support for the MongoDB wire protocol. You can read more about the wire
protocol at http://www.mongodb.org/display/DOCS/Mongo+Wire+Protocol; but this
is often not something you should have to be concerned about. Just to sum
things up:

 - the **Batch Size** controls how many documents the server sends in a reply
   packet.
 - the server will never send more documents than fit in 4MB.
 - the **Limit** option controls how many documents the driver will (try to)
   fetch from the server.
 - by default the first batch of documents without any limits specified is 101
   documents.
 - a negative **Batch Size** or a Batch Size of 1, will terminate the cursor
   immediately after returning the next batch of documents.

.. _MongoDB: http://mongodb.org
.. _driver: http://php.net/mongodb
.. _MongoCursor`: http://docs.php.net/manual/en/mongocursor.php
.. _MongoCollection`: http://docs.php.net/manual/en/mongocollection.php
.. _`find()`: http://docs.php.net/manual/en/mongocursor.find.php
.. _`sort()`: http://docs.php.net/manual/en/mongocursor.sort.php
.. _`getNext()`: http://docs.php.net/manual/en/mongocursor.getnext.php
.. _`batchSize()`: http://docs.php.net/manual/en/mongocursor.batchsize.php
.. _`limit()`: http://docs.php.net/manual/en/mongocursor.limit.php
.. _`MongoCursor::batchSize`: http://docs.php.net/manual/en/mongocursor.batchsize.php
__ http://php.net/manual/en/mongocursor.timeout.php
__ http://www.php.net/manual/en/mongocursor.immortal.php
__ http://www.mongodb.org/display/DOCS/Mongo+Wire+Protocol#MongoWireProtocol-OPQUERY
