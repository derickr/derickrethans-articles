Cursors and the Aggregation Framework
=====================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2014-04-09 09:29 Europe/London
   :Tags: blog, php, mongodb
   :Short: aggrcur

With MongoDB 2.6 about to be released, the PHP driver for MongoDB has also
seen many updates to support the features in the new MongoDB release. In this
series of articles, I will illustrate some of those.

In this article, I am introducing cursor aggregations. Last year, I already
wrote about the `Aggregation Framework`_, but since then it has received a
lot of updates and improvements. One of the improvements relates to how the
Aggregation Framework (A/F) returns results. Up to MongoDB 2.6, the A/F only
could return *one* document, with all the results stored under the
``results`` key::

     <?php
     $m = new MongoClient;
     $c = $m->demo->cities;

     $pipeline = [
          [ '$group' => [
               '_id' => '$country_code',
               'timezones' => [ '$addToSet' => '$timezone' ]
          ] ],
          [ '$sort' => [ '_id' => 1 ] ],
     ];

     $r = $c->aggregate( $pipeline );
     var_dump( $r['result'] );
     ?>

Would output something like::

     array(242) {
       [0] =>
       array(2) {
          '_id' => string(2) "AD"
          'timezones' => array(1) { [0] => string(14) "Europe/Andorra" }
       }
       [1] =>
       array(2) {
          '_id' => string(2) "AE"
          'timezones' => array(1) { [0] => string(10) "Asia/Dubai" }
       }
       [2] =>
       array(2) {
          '_id' => string(2) "AF"
          'timezones' => array(1) { [0] => string(10) "Asia/Kabul" }
       }
       …

`MongoCollection::aggregate()`_ is implemented under the hood as a database
command. The method in the PHP driver merely wraps this, but you can also
call A/F through the `MongoDB::command()`_ method::

     <?php
     $m = new MongoClient;
     $d = $m->demo;

     $pipeline = [
          [ '$group' => [
               '_id' => '$country_code',
               'timezones' => [ '$addToSet' => '$timezone' ]
          ] ],
          [ '$sort' => [ '_id' => 1 ] ],
     ];

     $r = $d->command( [
          'aggregate' => 'cities',
          'pipeline' => $pipeline,
     ] );
     var_dump( $r['result'] );
     ?>

Because a database command only returns *one* document, the result is limited
to a maximum of 16MB. In my example this is not a problem, but it could be
for other A/F queries.

MongoDB 2.6 adds support for returning a cursor for an aggregation command.
With the raw command interface, you simply add the extra ``cursor`` element::

     $r = $d->command( [
          'aggregate' => 'cities',
          'pipeline' => $pipeline,
          'cursor' => [ 'batchSize' => 1 ],
     ] );
     var_dump( $r );

Instead of returning the results now, you get a cursor definition back::

     array(2) {
       'cursor' =>
       array(3) {
          'id' => class MongoInt64#5 (1) {
               public $value => string(12) "392201189815" 
          }
          'ns' => string(11) "demo.cities"
          'firstBatch' => array(1) {
            [0] =>
            array(2) {
               '_id' => string(2) "AD"
               'timezones' => array(1) { [0] => string(14) "Europe/Andorra" }
            }
          }
       }
       'ok' => double(1)
     }

The cursor definition contains the cursor ID (in ``id``), the namespace
(``ns``), and whether the command succeeded (in ``ok``).
The definition also includes one result. The number of items in
``firstBatch`` is configured by the value given to ``batchSize`` in the
command.

To create a cursor that you can iterate over in PHP, you need to convert this
cursor definition to a `MongoCommandCursor`_ object. You can do that with the
`MongoCommandCursor::createFromDocument()`_ factory method. However, this
factory method takes three arguments: The ``MongoClient`` object (``$m`` in
my example), the *connection hash*, and the cursor definition that was
returned. The hash is required so that we can fetch new results from the
same connection as where we sent the command to. 

To obtain the connection hash, we need to have a by-ref variable as third
argument to ``MongoCollection::command()``::

     <?php
     $m = new MongoClient;
     $d = $m->demo;

     $pipeline = [
          [ '$group' => [
               '_id' => '$country_code',
               'timezones' => [ '$addToSet' => '$timezone' ]
          ] ],
          [ '$sort' => [ '_id' => 1 ] ],
     ];

     $r = $d->command(
          [
               'aggregate' => 'cities',
               'pipeline' => $pipeline,
               'cursor' => [ 'batchSize' => 1 ],
          ],
          null,
          $hash
     );
     var_dump( $hash );

The hash looks like ``localhost:27017;-;.;26415``. Together with the result,
you can now construct a ``MongoCommandCursor``::

     $cursor = MongoCommandCursor::createFromDocument( $m, $hash, $r );

And iterate over it::

     foreach ( $cursor as $result )
     {
          echo $result['_id'], ': ', join( ', ', $result['timezones'] ), "\n";
     }
     ?>

As this is all a bit cumbersome, we have also added a helper method for this:
`MongoCollection::aggregateCursor`_. This internally does the whole
MongoCommandCursor_ creation dance, and simplifies the previous example to::

     <?php
     $m = new MongoClient;
     $c = $m->demo->cities;

     $pipeline = [
          [ '$group' => [
               '_id' => '$country_code',
               'timezones' => [ '$addToSet' => '$timezone' ]
          ] ],
          [ '$sort' => [ '_id' => 1 ] ],
     ];

     $r = $c->aggregateCursor( $pipeline );

     foreach ( $r as $result )
     {
          echo $result['_id'], ': ', join( ', ', $result['timezones'] ), "\n";
     }
     ?>

This helper also automatically sets the initial batch size to 101. You can
change the batchSize for subsequent batches by using the
`MongoCommandCursor::batchSize()`_ method, and for the initial batch by
specifying an option to ``MongoCollection::aggregateCursor``::

     $options = [ 'cursor' => [ 'batchSize' => 5 ] ];

     $r = $d->cities->aggregateCursor( $pipeline, $options );
     $r->batchSize( 25 );

In general, you probably should not change the default batch sizes.

The Aggregation Framework has some other new features in MongoDB 2.6 as well.
Please refer to the `release notes`_ for more information. I might write
another post on this later too.

.. _`Aggregation Framework`: /aggregation-framework.html
.. _`MongoCollection::aggregate`: http://php.net/mongocollection.aggregate
.. _`MongoDB::command`: http://php.net/mongodb.command
.. _`MongoCommandCursor`: http://php.net/mongocommandcursor
.. _`MongoCommandCursor::createFromDocument`: http://php.net/mongocommandcursor.createfromdocument 
.. _`MongoCollection::aggregateCursor`: http://php.net/mongocollection.aggregatecursor
.. _`MongoCommandCursor::batchSize`: http://php.net/mongocommandcursor.batchsize
.. _`release notes`: http://docs.mongodb.org/master/release-notes/2.6/#aggregation-enhancements
