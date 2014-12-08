Parallelizing document retrieval
================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2014-12-09 09:31 Europe/London
   :Tags: blog, php, mongodb
   :Short: mongopcs

This is an article I wrote a while ago, but apparently hadn't posted.

MongoDB 2.6 has a new feature that allows you to read all the documents from
one collection with multiple cursors in parallel. This is done through a
database command called parallelCollectionScan_. The idea behind it is that it
is faster then reading all the documents in a collection sequentially.

Just like the `Aggregation Cursor`_, calling this command returns cursor
information. However, it returns an array of these structures. Let run this
example to see what it returns::

    <?php
    $m = new MongoClient;
    $d = $m->demo;

    $r = $d->command( [
        'parallelCollectionScan' => 'cities', 
        'numCursors' => 3 
    ] );

    var_dump( $r );
    ?>

And this outputs (after some formatting)::

    array(2) {
      ["cursors"]=>
      array(3) {
        [0]=>
        array(2) {
          ["cursor"]=>
          array(3) {
            ["firstBatch"]=> array(0) { }
            ["ns"]=> string(14) "demo.cities"
            ["id"]=>
            object(MongoInt64)#5 (1) {
              ["value"]=> string(12) "339843550291"
            }
          }
          ["ok"]=> bool(true)
        }
        [1]=>
        array(2) {
          ["cursor"]=>
          array(3) {
            ["firstBatch"]=> array(0) { }
            ["ns"]=> string(14) "demo.cities"
            ["id"]=>
            object(MongoInt64)#6 (1) {
              ["value"]=> string(12) "340949759620"
            }
          }
          ["ok"]=> bool(true)
        }
        [2]=>
        array(2) {
          ...
        }
      }
      ["ok"]=> float(1)
    }

With the `MongoCommandCursor::createFromDocument`_ from an earlier article_
you can create a `MongoCommandCursor`_ for each of the array elements::

	<?php
	$m = new MongoClient;
	$d = $m->demo;

	$r = $d->command( [
		'parallelCollectionScan' => 'cities', 
		'numCursors' => 3 
	], null, $hash );

	$cursors = [];
	foreach( $r['cursors'] as $cursorInfo )
	{
		$cursors[] = MongoCommandCursor::createFromDocument( $m, $hash, $cursorInfo );
	}
	?>

Instead of creating an array of cursors yourself, the driver implements the
`MongoCollection::parallelCollectionScan`_ method. Making the above a little
bit easier::

	<?php
	$m = new MongoClient;
	$c = $m->demo->cities;

	$cursors = $c->parallelCollectionScan( 3 );
	?>

The idea is that with multiple cursors you can iterate over each of the
segments in parallel, for example indifferent threads. Of course, PHP does
not have threads so that you can't really run things in parallel. However,
PHP does have a MultipleIterator_ class that allows you to iterate over
multiple cursors at the same time::

	<?php
	$m = new MongoClient;
	$c = $m->demo->cities;

	$cursors = $c->parallelCollectionScan( 3 );


	$multiple_it = new MultipleIterator( MultipleIterator::MIT_NEED_ANY );
	foreach ( $cursors as $cursor )
	{
		$multiple_it->attachIterator( $cursor );
	}


	foreach ( $multiple_it as $items )
	{
		foreach ( $items as $item )
		{
			if ( $item !== NULL )
			{
				echo $item['name'], "\n";
			}
		}
	}
	?>

There are three sections here. First we create the cursors with
`MongoCollection::parallelCollectionScan`_, then we collect the created
cursors into a MultipleIterator_ and lastly we iterate over the
``$multiple_it`` iterator to get our results. Each iteration gives us an
**array** of elements back. One element for each of the containing cursors (3
in our example). We need a second loop (``foreach``) to pick out the real
document.

Not every contained cursor will provide the same amount of items, it is up to
the MongoDB server to divide this. When a contained iterator is exhausted, the
MultipleIterator sets the value to ``NULL``. It is probably better to then
remove that specific contained iterator from the MultipleIterator, but that is
left as an excercise for the reader.

When running some benchmarks, I didn't actually see any performance benefit
with multiple cursors over just one cursor, but that is likely because the
cursors are still iterated over sequentially, and not in parallel. Perhaps
using the `pthreads PECL extension`_ allows for a better benchmark, but right
now, the PHP driver for MongoDB doesn't support threaded execution yet.


.. _parallelCollectionScan: http://docs.mongodb.org/master/reference/command/parallelCollectionScan/
.. _`MongoCommandCursor::createFromDocument`: http://php.net/mongocommandcursor.createfromdocument 
.. _`MongoCommandCursor`: http://php.net/mongocommandcursor
.. _MultipleIterator: http://php.net/multipleiterator
.. _article:  /aggregation-cursor.html
.. _`Aggregation Cursor`:  /aggregation-cursor.html
.. _`Aggregation Framework`: /aggregation-framework.html
.. _`MongoCollection::aggregate`: http://php.net/mongocollection.aggregate
.. _`MongoDB::command`: http://php.net/mongodb.command
.. _`MongoCollection::aggregateCursor`: http://php.net/mongocollection.aggregatecursor
.. _`MongoCommandCursor::batchSize`: http://php.net/mongocommandcursor.batchsize
.. _`release notes`: http://docs.mongodb.org/master/release-notes/2.6/#aggregation-enhancements
.. _`MongoCollection::parallelCollectionScan_`: http://php.net/manual/en/mongocollection.parallelcollectionscan.php
.. _`pthreads PECL extension`: http://pecl.php.net/package/pthreads
