Debugging Connections with the MongoDB PHP driver
=================================================

.. articleMetaData::
   :Where: Tokyo, Japan
   :Date: 2012-12-11 10:13 Asia/Tokyo
   :Tags: blog, php, mongodb
   :Short: mongodbg

In a previous article__ I already mentioned that the 1.3 version of the
MongoDB driver has improved logging functionality to aid with debugging
connection issues. I've already briefly introduced
MongoClient::`getConnections()`_, but it provides a bit more information
than I have already shown. The other improvement are changes to the `MongoLog`_
class.

__ /mongodb-connection-handling.html

-----------------------
Mongo::getConnections()
-----------------------

Mongo::``getConnections()`` returns information about all the connections
that the connection manager knows about. Besides basic information
such as the connection's hash (``$m->getConnections()[0]['hash']``) it
also returns the following information:

 - ``hash``: the connection's identifier; f.e.: ``string(22)
   "whisky:27017;-;X;21093"``.
 - ``server``: contains an element for each of the elements in the hash, but
   broken up.
 - ``connection``: contains information about the connection and server it is
   connected to. It is an array with the following fields:

   - ``last_ping``: timestamp that indicates when the server was last pinged;
     for example: ``int(1352479219)``.
   - ``last_ismaster``: timestamp that indicates when the topology of a
     replicaset was last checked for; in the case of a standalone connection,
     this is: ``int(0)``.
   - ``ping_ms``: the time it takes to get to the server and back again in
     milliseconds, this is very often just ``0``.
   - ``connection_type``: the type of connection. The value is: ``1`` for a
     standalone connection, ``2`` for multiple "mongos" connections and ``3``
     for a replica set connection.
   - ``connection_type_desc``: a string representation of the above types.
   - ``max_bson_size``: the maximum size of a document, right now, that is
     ``int(16777216)`` (bytes).
   - ``tag_count``: the number of tags associated with this mongod server. I
     will explain what tags are and how you can use them in a future article on
     *Read Preferences*.
   - ``tags``: an array containing all the tags.

As an example, this is what is returned for my replica set test setup::

    <?php
    $m = new Mongo("mongodb://whisky:13000/?replicaSet=seta");
    print_r( $m->getConnections() );
    ?>

Which returns::

    Array
    (
      [0] => Array
        (
          [hash] => whisky:13000;seta;X;3955
          [server] => Array                           ← 'exploded' hash
            (
              [host] => whisky
              [port] => 13000
              [repl_set_name] => seta
              [pid] => 3955
            )

          [connection] => Array
            (
              [last_ping] => 1353515082
              [last_ismaster] => 1353515082
              [ping_ms] => 0                          ← ping time
              [connection_type] => 4
              [connection_type_desc] => SECONDARY     ← secondary node
              [max_bson_size] => 16777216             ← 16MB document limit
              [tag_count] => 2
              [tags] => Array                         ← tags
                (
                  [0] => dc:west
                  [1] => use:accounting
                )
            )
        )

      [1] => Array
        (
          [hash] => whisky:13001;seta;X;3955
          [server] => Array
            (
              [host] => whisky
              [port] => 13001
              [repl_set_name] => seta
              [pid] => 3955
            )

          [connection] => Array
            (
              [last_ping] => 1353515082
              [last_ismaster] => 1353515082
              [ping_ms] => 0
              [connection_type] => 2
              [connection_type_desc] => PRIMARY
              [max_bson_size] => 16777216
              [tag_count] => 2
              [tags] => Array
                (
                  [0] => dc:east
                  [1] => use:reporting
                )
            )
        )
    )

--------
MongoLog
--------

The MongoLog_ class already existed in earlier versions of the driver, but
the debugging *information* it provided was not really useful. While we were
implementing the connection handling, we also overhauled the logging through
``MongoLog``. We tried to improve the categorisation of specific into groups,
and also classify them according to *importantness*. There is also a new
MongoLog::`setCallback()`_ static method that can be used to catch all messages
that pass through ``MongoLog``.  In that case, they will no longer show up as
PHP Notices.

First, we show the most basic usage where we send all of the drivers messages
as PHP Notices::

    <?php
    /* Configure logging */
    MongoLog::setModule( MongoLog::ALL );
    MongoLog::setLevel( MongoLog::ALL );

    /* Make connection to see some log messages */
    $m = new MongoClient();

This returns for example the following messages::

    Notice: PARSE   INFO: Parsing localhost:27017 in - on line 7

    Notice: PARSE   INFO: - Found node: localhost:27017 in - on line 7

    Notice: PARSE   INFO: - Connection type: STANDALONE in - on line 7

    Notice: CON     INFO: mongo_get_read_write_connection: finding a STANDALONE connection in - on line 7

    Notice: CON     INFO: connection_create: creating new connection for localhost:27017 in - on line 7

    Notice: CON     INFO: get_server_flags: start in - on line 7

    Notice: CON     FINE: send_packet: read from header: 36 in - on line 7

    Notice: CON     FINE: send_packet: data_size: 70 in - on line 7

    Notice: CON     FINE: get_server_flags: setting maxBsonObjectSize to 16777216 in - on line 7

    Notice: CON     FINE: is_ping: pinging localhost:27017;-;X;8830 in - on line 7

    Notice: CON     FINE: send_packet: read from header: 36 in - on line 7

    Notice: CON     FINE: send_packet: data_size: 17 in - on line 7

    Notice: CON     WARN: is_ping: last pinged at 1353519357; time: 0ms in - on line 7

    Notice: REPLSET FINE: finding candidate servers in - on line 7

    Notice: REPLSET FINE: - all servers in - on line 7

    Notice: REPLSET FINE: filter_connections: adding connections: in - on line 7

    Notice: REPLSET FINE: - connection: type: STANDALONE, socket: 3, ping: 0, hash: localhost:27017;-;X;8830 in - on line 7

    Notice: REPLSET FINE: filter_connections: done in - on line 7

    Notice: REPLSET FINE: limiting by seeded/discovered servers in - on line 7

    Notice: REPLSET FINE: - connection: type: STANDALONE, socket: 3, ping: 0, hash: localhost:27017;-;X;8830 in - on line 7

    Notice: REPLSET FINE: limiting by seeded/discovered servers: done in - on line 7

    Notice: REPLSET FINE: sorting servers by priority and ping time in - on line 7

    Notice: REPLSET FINE: - connection: type: STANDALONE, socket: 3, ping: 0, hash: localhost:27017;-;X;8830 in - on line 7

    Notice: REPLSET FINE: sorting servers: done in - on line 7

    Notice: REPLSET FINE: selecting near servers in - on line 7

    Notice: REPLSET FINE: selecting near servers: nearest is 0ms in - on line 7

    Notice: REPLSET FINE: - connection: type: STANDALONE, socket: 3, ping: 0, hash: localhost:27017;-;X;8830 in - on line 7

    Notice: REPLSET FINE: selecting near server: done in - on line 7

    Notice: REPLSET FINE: pick server: random element 0 in - on line 7

    Notice: REPLSET INFO: - connection: type: STANDALONE, socket: 3, ping: 0, hash: localhost:27017;-;X;8830 in - on line 7

    Notice: CON     FINE: mongo_connection_destroy: Closing socket for localhost:27017;-;X;8830. in Unknown on line 0

    Notice: CON     INFO: freeing connection localhost:27017;-;X;8830 in Unknown on line 0

As you can see, that is very verbose information—mostly useful when
there is really something going wrong. We might ask you for this when
submitting a bug report to the PHP driver's JIRA. In general though, you
probably want to restrict the amount of information to something more
manageable.

First of all, there are a few different levels that you can set through
``MongoLog::setLevel()``:

 - ``MongoLog::NONE``: *no* messages at all
 - ``MongoLog::WARN``: only show warnings, such as connection failures
   and misconfigurations
 - ``MongoLog::INFO``: informational messages, such as which options
   have been found during parsing, which hosts have been found, and
   which sort of connection the driver thinks it is using—this is in
   most cases the best level to pick
 - ``MongoLog::FINE``: diagnostic messages such as debugging for
   algorithms, and timing messages

The four constants can be used as a bit-field, so the following is
possible (but not very useful)::

    MongoLog::setLevel( MongoLog::WARN | MongoLog::FINE );

The messages are divided into different categories. You can configure
which categories are represented in the messages with the parameter to
``MongoLog:setModule()``:

 - ``MongoLog::CON``: Single connection issues and operations
 - ``MongoLog::IO``: Transferring queries, results and cursor related
   operations
 - ``MongoLog::PARSE``: Connection string parsing
 - ``MongoLog::RS``: Selecting which connection to use while using
   replica sets.
 - ``MongoLog::SERVER``: Not used at all

These categories can also be combined as a bit-field. For example, to
only show connection and connection selection messages, use::

    MongoLog::setModule( MongoLog::CON | MongoLog::RS );

The ``MongoLog`` class already existed, but new in 1.3 is the
MongoLog::`setCallback`_ method. This method allows you to set-up your
own callback function to deal with the MongoDB driver's log messages.
We use this in the test cases to check whether specific messages have
values we are looking for. It also allows you to filter, handle, and for
example log messages in various ways.

For example, in the replica set tests, we set the following::

    MongoLog::setCallback( function($a, $b, $message) { if (preg_match('/connection: type: ([A-Z]+),/', $message, $m )) { @$GLOBALS['mentions'][$m[1]]++; }; } );

And to make that a bit more readable and avoiding the closure::

    function logCallBack( $a, $b, $message )
    {
        if ( preg_match( '/connection: type: ([A-Z]+),/', $message, $m ) )
        {
            @$GLOBALS['mentions'][$m[1]]++;
        };
    }

    MongoLog::setCallback( 'logCallBack' );

The callback function accepts three arguments, the first one
representing the module for this message, the second the log level and
the third argument is the message. In the above example, we basically
only want to find out which connection types the log dumps, and we
compare that against what we expect. The values of the module constants
that you can compare the first two arguments against are described in
the documentation__.

__ http://docs.php.net/manual/en/class.mongolog.php#mongolog.synopsis

With this I conclude this post about new debugging functionality in the
driver. Still to come are posts about *Read Preferences* and the new
*Aggregation Framework* in MongoDB 2.2.

.. _`getConnections()`: http://www.php.net/manual/en/mongoclient.getconnections.php
.. _MongoLog: http://www.php.net/manual/en/class.mongolog.php
.. _`setCallback()`: http://www.php.net/manual/en/mongolog.setcallback.php
