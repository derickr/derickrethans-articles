Read Preferences wth the MongoDB PHP driver
===========================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2012-12-18 09:41 Europe/London
   :Tags: blog, php, mongodb
   :Short: rp

Read Preferences are a new Replica Set and Sharding feature implemented by most
MongoDB_ drivers that are supported by 10gen_. This functionality requires
MongoDB 2.2. In short, Read Preferences allow you to configure from which nodes
you prefer the driver **reads** data from. In a Replica Set environment it is
the driver that does the selection of the preferred node, and in a Sharded
environment it is the ``mongos`` process that routes queries according to the
defined Read Preferences.

There are a few different Read Preference types. There is *primary* (the
default) which will make sure the driver (or *mongos*) only reads from 
primary nodes. This is the default and guarantees consistent reads. The other
types can all possibly returned outdated documents as replication is
asynchronous in MongoDB. Other types include *primaryPreferred*, which reads
from a primary, unless no primary is available; *secondary*, which runs
queries on secondaries **only**; *secondaryPreferred*, which reads from
secondaries first, and if none are available, from the primary node; and the
last one is *nearest* which does not prefer a specific type of node (primary
or secondary) but solely selects one of the "nearest" nodes - within 15ms of
the closest node.

The PHP driver checks ping times to nodes in a Replica Set about every 5
seconds and keeps this information with the connection manager, about which I
wrote in an earlier article__.

__ /mongodb-connection-handling.html

As an example, let us assume that we have a Replica Set with four nodes
and that the driver has established the following ping times:

 - Primary node *a*: 2ms
 - Secondary node *b*: 23ms
 - Secondary node *c*: 1ms
 - Secondary node *d*: 7ms
 - Secondary node *e*: 3ms

The manager searches the manager for all connections and returns all
connections for which a connection matches the Read Preference criteria. For
*primary* or *secondary* it **only** includes nodes with that property, for
the other three Read Preference types it will return **all** the connections.
The set that is returned for *secondaryPreferred* is ``[ a, b, c, d, e ]``.
The manager then sorts this list according to the Read Preference and the ping
time. As we are more interested in secondaries with *secondaryPreferred*, the
sorted list of nodes becomes ``{ c, e, d, b, a }``. Node ``c`` is first because it has
the lowest ping time (1ms). In the next step, all nodes that are more than
15ms slower than the fastest node are eliminated. In our example, we are now
left with ``{ c, e, d, a }``. In the last step, we select a node from this
set.  With a Read Preference of *secondaryPreferred* we eliminate the last
node from the list if there is more than one node in the list, and the last
node in the list is a primary. The final list to pick a node from is now ``{ c,
e, a }``. From this remaining list, a random node is picked.

With the PHP driver, you can choose a Read Preference in a few different ways.
The simplest way is probably directly in the connection string::

    new MongoClient( 'mongodb://a,d/?replicaSet=demoset&readPreference=secondaryPreferred' );

It is also possible to set the Read Preference on a connection, database or
collection object. When a read preference is set, it is inherited by every
database or connection object that is created afterwards::

    <?php
    $m = new MongoClient( 'mongodb://a,b,d/?replicaSet=demoset' );
    $m->setReadPreference( Mongo::RP_SECONDARY_PREFERRED );

    $d = $m->demoDb;
    /* The Read Preference for $d is 'secondaryPreferred' */
    $d->setReadPreference( Mongo::RP_PRIMARY_PREFERRED );
    /* The Read Preference for $d has been changed to 'primaryPreferred' */

    $c = $d->myCollection;
    /* The Read Preference for $c is 'primaryPreferred' */

    $c = $m->demoDb->myCollection;
    /* The Read Preference for $c is 'secondaryPreferred' */
    $d->setReadPreference( Mongo::RP_NEAREST );
    /* The Read Preference for $c is now 'nearest' */
    ?>

In future versions of the driver, we will also allow setting a Read Preference
on a by-query base.

**Tags**

Each node in a Replica Set can be annotated with zero or more tags. These tags
are for example ``dc=us-east``, ``use=reporting`` or ``systemtype=big``. Tags
can be added to an already existing Replica Set configuration as follows::

    cfg = rs.conf();
    cfg.members[0].tags = { "dc" : "west", "use" : "website" };
    cfg.members[1].tags = { "dc" : "east", "use" : "reporting" };
    rs.reconfig(cfg);

In this example we add the tags ``dc`` and ``use`` to the first and second
node in the Replica Set. In combination with Read Preferences, those tags
allow you to restrict from which node you want to read. Let's go back to our
previous example with five nodes, but add tags as well:

 - Primary node *a*: 2ms, tags: dc=west, use=website
 - Secondary node *b*: 23ms, tags: dc=west, use=website
 - Secondary node *c*: 1ms, tags: dc=west, use=analysis
 - Secondary node *d*: 7ms, tags: dc=east, use=analysis
 - Secondary node *e*: 3ms, tags: dc=east, use=website

When we now combine the Read Preference *secondaryPreferred* with the Tag Set
``dc=west``, nodes ``c`` and ``d`` are removed from the candidate list after
all possible candidates are found and before the candidate servers are sorted
according to their node type (*primary* or *secondary*) and their ping time.
With the nodes not having the tag ``dc=west`` set eliminated, we are left with
the set ``[ a, b, c ]``. This set then gets sorted into ``{ c, b, a }``. Node
``b`` is eliminated because it is too slow and finally node ``c`` is picked
because ``a`` is primary and we prefer a secondary instead.

If **none** of the nodes matches the giving Tag Set, then a connection can not
be found, and the query fails. In order to prevent this, it is possible to
specify multiple lists of Tag Sets. For example. In this example we prefer the
west data centre to be used over the *east* data centre. If none of the
nodes destined for powering the website are available, then we are fine using
any available node.

 - dc=west, use=website
 - dc=east, use=website
 - *empty*

With the PHP driver, you would configure this in the connection string like::

    new MongoClient( 'mongodb://a,b,d/?replicaSet=demoset&readPreference=secondaryPreferred&readPreferenceTags=dc:west,use:website&readPreferenceTags=dc:east,use:website&readPreferenceTags=' );

As you can see, this becomes a really long string, so you can 
alternatively set options as an array through the second argument to the
constructor::

    new MongoClient(
        'mongodb://a,b,c',
        array(
            'replicaSet' => 'demoset',
            'readPreference' => 'secondaryPreferred',
            'readPreferenceTags' => array(
                'dc=west,use=website',
                'dc=east,use=website',
                ''
            )
        )
    );

The `MongoClient::setReadPreference`_, `MongoDb::setReadPreference`_ and
`MongoCollection::setReadPreference`_ methods accept an array of Tag Sets as
second argument::

    $m = new MongoClient( 'mongodb://a,c,e/?replicaSet=demoset' );
    $c = $m->demoDb->myCollection;
    $c->setReadPreference(
        Mongo::RP_SECONDARY_PREFERRED,
        array( 'dc=west,use=website', 'dc=east,use=website', '' )
    );

With this I conclude the introduction of *Read Preferences*. I will write
about the new *Aggregation Framework* in MongoDB 2.2 in an upcoming article.

.. _MongoDB: http://mongodb.org
.. _10gen: http://10gen.com
.. _`MongoClient::setReadPreference`: http://www.php.net/manual/en/mongoclient.setreadpreference.php
.. _`MongoDb::setReadPreference`: http://www.php.net/manual/en/mongodb.setreadpreference.php
.. _`MongoCollection::setReadPreference`: http://www.php.net/manual/en/mongocollection.setreadpreference.php


