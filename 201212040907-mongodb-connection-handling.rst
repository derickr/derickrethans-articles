Connection Handling with the MongoDB PHP driver
===============================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2012-12-04 09:07 Europe/London
   :Tags: blog, php, mongodb
   :Short: mongocon

The 1.3 release series of the PHP MongoDB_ driver features a rewritten
connection handling library. This is quite a large change and changes how
the PHP driver deals with persistent connections and connection pooling.

**Connections in the 1.2 series**

The 1.2 release introduced connection pooling into the driver. This means
that any time the driver needs to use a connection to run queries, it would
request a connection from the pool. It would then later return the connection
once it was done with it. Being done is defined as "the variable holding
the connection object goes out of scope". To illustrate that, take for
example the following scripts.

The simplest version::

	<?php
	$m = new MongoClient();    // ← connection is requested from the pool
	$c = $m->demo->test;
	$c->insert( array( 'test' => 'yes' ) );
	?>
	← connection is returned to the pool as $m goes out of scope

Inside a function::

	<?php
	function doQuery()
	{
		$m = new MongoClient();    // ← connection is requested from the pool
		$c = $m->demo->test;
		$c->insert( array( 'test' => 'yes' ) );
	} // ← connection is returned to the pool as $m goes out of scope
	?>

Now, in a few cases, this can result in lots of connections being made. One
common occurence is trying to use connections deep inside a model layer-which
typically happens if you use ORMs/ODMs with complex structures having a link
to your connection object. A simple variant of how this happens is::

	<?php
	for ( $i = 0; $i < 5; $i++ )
	{
		$conns[] = new MongoClient();
		}
		// ← now we have 5 connections open
	?>

**Connections in the 1.3 series**

In the 1.3 series the connection management is done different. Per worker
process (thread or PHP-FPM or Apache worker) the driver now manages
connections in the C-part of driver *separately* from the ``Mongo*`` objects.
This greatly reduces complexity in the driver.
Let's have a look at how the driver handles connection for a basic MongoDB
setups with just one node.  

.. image:: /images/content/sad-manager.png
   :align: right
   :alt: Sad manager, it has no connections yet.

When a worker process starts up, the MongoDB driver instantiates a *manager* to
manage your connections. By default of course, this manager is rather sad, as
it contains no connections yet.

Upon the first request doing ``new MongoClient();``, the driver creates
a new connection, and with this connection it creates a hash to identify this
connection. Parameters for this hash includes: the
*hostname* and the *port*, the process ID (*pid*), if available the *replica
set name* and in case of authenticated connections, the *database name*, the
*username* and a hash of the password. We will come back to authenticated
connections later. You can quite easily see which hash is created, by calling
the MongoClient::`getConnections()`_ method:

.. include:: ../files/mongo-connect-1.php.txt
   :literal:

Which outputs::

	string(22) "whisky:27017;-;X;22835"

The ``-`` in this output denotes that the connection does not belong to a
replica set and the ``X`` in this output is a place holder in case no username,
database and password are given. ``22835`` is the process ID of the running
script.

This newly created connection is then registered with the manager:

.. image:: /images/content/manager-1con.png
   :alt: One connection, to the whisky!

Any time a connection is needed, for either an *insert*, *delete*, *update*,
*find* or *command* query, the driver asks the manager to find a fitting
connection to perform this query. It uses the information from the arguments
to ``new MongoClient()`` as well as the PID of the current process to find this
connection.  Because the manager has a connection list per worker
process/thread, and PHP's workers all run one request at a time, there is no
need to have more than one connection per host. The connection gets reused and
reused until the PHP worker ends, or you close the connection forcefully with
MongoClient::`close()`_.

**Replica sets**

In a replicaset environment, this works all a bit different. In the connection
string to ``new MongoClient()`` you need to specify a few hosts, preferrably all
that you are aware of, as well as an annotation that you are using a replica
set, for example::

	$m = new MongoClient("mongodb://whisky:13000,whisky:13001/?replicaSet=seta");

It is **important** that you specify the ``replicaSet`` argument, as without
it the driver will assume you are connecting to three different ``mongos``
processes.

Upon instantiation, the driver will discover the replica set's topology. The
following output shows that all visible data nodes of the replica set will have
a connection registered in the manager after the ``new MongoClient()`` call:

.. include:: ../files/mongo-connect-2.php.txt
   :literal:

Which outputs::

	whisky:13001;seta;X;32315
	whisky:13000;seta;X;32315

Notice that the connection string made no reference to the ``whisky:13000``
node. The manager has now registered two connections:

.. image:: /images/content/manager-replcon.png
   :alt: More whisky!

The manager contains a bit more information than just the connection hash and
the TCP/IP socket. It also knows which nodes are *primary* nodes, and how "far
away" a specific node is. This script shows how to retrieve this
additional information:

.. include:: ../files/mongo-connect-3.php.txt
   :literal:

Which outputs::

	whisky:13001;seta;X;5776:
	 - SECONDARY, 1 ms
	whisky:13000;seta;X;5776:
	 - PRIMARY, 0 ms

The driver knows about two types of queries. Write queries, which
involve *insert*, *update*, *remove* and *commands*, and read queries,
which involves *find* and *findOne*. By default, the manager will always
return a connection to a  *primary* node, unless you use the new *read
preferences*. I will be writing a separate post about read preferences,
but the basic idea is that they can control from which replica set nodes
data is read from. For now, we will only use the ``setSlaveOkay()``
equivalent directly in the connection string::

	$m = new MongoClient("mongodb://whisky:13000,whisky:13001/?replicaSet=seta&readPreference=secondaryPreferred");

As you can see, this connection string is rather long, which is why the PHP
driver also allows you to pass in an array of options as second argument::

	$options = array(
		'replicaSet' => 'seta',
		'readPreference' => 'secondaryPreferred',
	);
	$m = new MongoClient("mongodb://whisky:13000,whisky:13001/", $options);

Per query, the driver will ask the manager for a fitting connection. 
For write queries, it will always pick a *primary* node, and for read
queries it will pick a *secondary* node if they are available and not too
"far away" (what that means, I'll again explain in my upcoming read
preferences article).

**Authenticated connections**

If authentication is enabled on MongoDB, then the connection hash will
start to include an authentication hash. This is to make sure that different
scripts that use the same MongoDB server, but different
user name/password/database combinations, do not inadvertently use a wrongly
authenticated connection. Let's see what the hash turns into if we connect
to the *admin* database with the *admin* user:

.. include:: ../files/mongo-connect-4.php.txt
   :literal:

Which outputs::

	string(64) "whisky:27017;-;admin/admin/bda5cc70cd5c23f7ffa1fda978ecbd30;8697"

The ``X`` that we previously saw, has now been replaced by a string containing
the database name ``admin``, the user name ``admin`` and the hash
``bda5cc70cd5c23f7ffa1fda978ecbd30``. This hash is created from the user name
database and hashed password.

In order for authentication to work the driver needs to have the database name
in the connection string (in our
case ``admin``), otherwise, it will default to ``admin``. 

To use a database after you have established a connection, need to select the
database again though. For example when you want to run a query::

	$collection = $m->demoDb->collection;
	$collection->findOne();

If this database matches the database name in the connection string, or when
the database name in the connection string was ``admin``, everything is
well. However, if the database name accessed as property of the connection
object is *different* then the one specified in the connection string then the
driver has to create a new connection to prevent leaking of authentication
between multiple requests. The following example illustrates this:

.. include:: ../files/mongo-connect-5.php.txt
   :literal:

Which outputs::

	Fatal error: Uncaught exception 'MongoCursorException' with message
	'whisky:27017: unauthorized db:test2 ns:test2.collection lock type:0 client:127.0.0.1'
	in …/mongo-connect-5.php.txt:6

This is of course we have not actually authenticated for the ``test2``
database. If we authenticate, then it works (even though ``findOne()``
does not actually find anything), and we can see all the 
created connections:

.. include:: ../files/mongo-connect-6.php.txt
   :literal:

Which outputs::

	whisky:27017;-;test/user/602b672e2fdcda7b58a042aeeb034376;26983
	whisky:27017;-;test2/user2/984b6b4fd6c33f49b73f026f8b47c0de;26983

And we have two authenticated connections in our manager:

.. image:: /images/content/manager-auth.png
   :alt: Safe whisky!

But with one snag. If you have ``E_DEPRECATED`` notices turned on, you
will see::

	Deprecated: Function MongoDB::authenticate() is deprecated in …/mongo-connect-6.php.txt on line 5

The driver can do things much more optimised if you create *two*
``MongoClient`` objects instead:

.. include:: ../files/mongo-connect-7.php.txt
   :literal:

With this, I conclude this post on the MongoDB driver's new connection
handling. In future posts I will write about the 1.3 release providing (a
lot) better debugging information, read preference functionality, and
the new aggregation framework that comes with MongoDB 2.2.

.. credit::
   :Type: photo
   :Description: Sad T-rex
   :Link: http://www.flickr.com/photos/ijammin/5456757507/

.. credit::
   :Type: photo
   :Description: Whisky glass
   :Link: http://www.flickr.com/photos/feastguru_kirti/5396934673

.. credit::
   :Type: photo
   :Description: Lock
   :Link: http://www.flickr.com/photos/leehaywood/4141478822

.. _MongoDB: http://mongodb.org
.. _`getConnections()`: http://www.php.net/manual/en/mongoclient.getconnections.php
.. _`close()`: http://docs.php.net/manual/en/mongoclient.close.php
