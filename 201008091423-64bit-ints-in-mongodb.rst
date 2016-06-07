64-bit integers in MongoDB
==========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2010-08-09 14:23 Europe/London
   :Tags: blog, php, mongodb, extensions

The current project that I'm working on relies heavily on MongoDB_, a bridge
between key-value stores and traditional RDBMS systems. Users in this project
are identified by their Facebook UserID_, which is a "64-bit int datatype".
Unfortunately, the `MongoDB PHP Driver`_ only had support for 32-bit integers
causing problems for newer users of Facebook. For those users, their nice
long UserID was truncated to only 32 bits which didn't quite make the
application work.

MongoDB stores documents internally in something called BSON_ (Binary JSON_).
BSON has two integer types, a 32-bit signed integer type called ``INT`` and a 64-bit signed
integer type called ``LONG``. The documentation of the MongoDB PHP driver on types_ says (or used to
say, depending on when you're reading this) that only the 32-bit signed integer
type is supported because "PHP does not support 8 byte integers". That's not
quite true. PHP's integer type supports 64-bit on the platforms where the
C-data type ``long`` is 64 bits. That is generally on every 64-bit platform
(where PHP is compiled for 64 bits); except on Windows, where the C-data type
``long`` is always only 32 bits. 

Whenever a PHP integer is sent to MongoDB, the driver would use the 32 least
significant bits to store the number as part of the document. The example
here shows what happens (on a 64-bit platform)::

	<?php
	$m = new Mongo();
	$c = $m->selectCollection('test', 'inttest');
	$c->remove(array());

	$c->insert(array('number' => 1234567890123456));

	$r = $c->findOne();
	echo $r['number'], "\n";
	?>

shows::

	int(1015724736)

In binary::

	1234567890123456 = 100011000101101010100111100100010101011101011000000
	      1015724736 =                      111100100010101011101011000000

Truncating data is obviously not a very good idea. In order to address this
issue we could just allow for the native PHP integer type to be used when
storing data from PHP into MongoDB. But instead of changing how the MongoDB
driver works by default I've added the new setting ``mongo.native_long`` —
simply because otherwise we might be breaking applications. With the
``mongo.native_long`` setting enabled, we see the following result instead of
the outcome of the script above::

	<?php
	ini_set('mongo.native_long', 1);
	$c->insert(array('number' => 1234567890123456));

	$r = $c->findOne();
	var_dump($r['number']);
	?>

This script shows::

	int(1234567890123456)

On **64-bit platforms**, the ``mongo.native_long`` setting allows for 64-bit
integers to be stored in MongoDB. The MongoDB data type that is used in this
case is the BSON LONG, instead of the BSON INT that is used if this setting
is turned off. The setting *also* changes the way how BSON LONGs behave when
they are read back from MongoDB. Without ``mongo.native_long`` enabled, the
driver would convert every BSON LONG to a PHP double which results in the loss
of precision. You can see that in the following example::

	<?php
	ini_set('mongo.native_long', 1);
	$c->insert(array('number' => 12345678901234567));

	ini_set('mongo.native_long', 0);
	$r = $c->findOne();
	var_dump($r['number']);
	?>

This script shows::

	float(1.2345678901235E+16)

On **32-bit platforms**, the ``mongo.native_log`` setting changes nothing for
storing integers in MongoDB: the integer is stored as a BSON INT as before.
However, when the setting is enabled and a BSON LONG is read from MongoDB a
MongoCursorException_ is thrown alerting you that the data could not be read
back without losing precision::

	MongoCursorException: Can not natively represent the long 1234567890123456 on this platform

If the setting is not enabled, a BSON LONG is converted to a PHP float in
order to avoid breaking backwards compatibility with the current behaviour.

Although the ``mongo.native_long`` settings allows for 64-bit support on
64-bit platforms, it doesn't provide much for 32-bit platforms except 
preventing loss of precision while reading BSON LONGs—and then by just throwing
an exception.

As part of making 64-bit integers work reliably with MongoDB, I've also added
two new classes: ``MongoInt32`` and ``MongoInt64``. These two classes are
simple wrappers around a string representation of a number. They are instantiated
like this::

	<?php
	$int32 = new MongoInt32("32091231");
	$int64 = new MongoInt64("1234567980123456");
	?>

You can use those objects in normal insert and update queries just like
normal numbers::

	<?php
	$m = new Mongo();
	$c = $m->selectCollection('test', 'inttest');
	$c->remove(array());

	$c->insert(array(
		'int32' => new MongoInt32("1234567890"),
		'int64' => new MongoInt64("12345678901234567"),
	)); 

	$r = $c->findOne();
	var_dump($r['int32']);
	var_dump($r['int64']);
	?>

Which shows::

	int(1234567890)
	float(1.2345678901235E+16)

As you can see, nothing is changed with how values are returned. A BSON INT is
still returned as an integer, and a BSON LONG as a double. If we turn on the
``mongo.native_long`` setting then the BSON LONG that was stored through the
``MongoInt64`` class is returned as a PHP integer on 64-bit platforms, and a
MongoCursorException is thrown on 32-bit platforms.

In order to received 64-bit integers back from MongoDB on 32-bit platforms,
I've added another setting: ``mongo.long_as_object``. This will (on any
platform) return a BSON LONG as stored in MongoDB as a ``MongoInt64`` object.
The following script demonstrates that::

	<?php
	$m = new Mongo();
	$c = $m->selectCollection('test', 'inttest');
	$c->remove(array());

	$c->insert(array(
		'int64' => new MongoInt64("12345678901234567"),
	)); 

	ini_set('mongo.long_as_object', 1);
	$r = $c->findOne();
	var_dump($r['int64']);
	echo $r['int64'], "\n";
	echo $r['int64']->value, "\n";
	?>

This script outputs::

	object(MongoInt64)#7 (1) {
	  ["value"]=>
	  string(17) "12345678901234567"
	}
	12345678901234567
	12345678901234567

The ``MongoInt32`` and ``MongoInt64`` classes implement ``__toString()`` so
that their values can be echoed. You can only get their values out as strings.
Please be aware that MongoDB is type-sensitive, and will not treat a number
contained in a string the same way as a number that's just a number. This
script shows this (on a 64-bit platform)::

	<?php
	ini_set('mongo.native_long', 1);

	$m = new Mongo();
	$c = $m->selectCollection('test', 'inttest');
	$c->remove(array());

	$nr = "12345678901234567";
	$c->insert(array('int64' => new MongoInt64($nr)));

	$r = $c->findOne(array('int64' => $nr)); // $nr is a string here
	var_dump($r['int64']);
	$r = $c->findOne(array('int64' => (int) $nr));
	var_dump($r['int64']);
	?>

Which shows::

	NULL
	int(12345678901234567)

The following tables summarises all the different conversions regarding
integers, depending on which settings are enabled:

**PHP to MongoDB on 32-bit systems**:

+----------------------------+----------------------------------------+
|From PHP                    |Stored in Mongo                         |
|                            +--------------------+-------------------+
|                            |native_long=0       |native_long=1      |      
+============================+====================+===================+
|1234567                     |INT(1234567)        |INT(1234567)       |
+----------------------------+--------------------+-------------------+
|123456789012                |FLOAT(123456789012) |FLOAT(123456789012)|
+----------------------------+--------------------+-------------------+
|MongoInt32("1234567")       |INT(1234567)        |INT(1234567)       |
+----------------------------+--------------------+-------------------+
|MongoInt64("123456789012")  |LONG(123456789012)  |LONG(123456789012) |
+----------------------------+--------------------+-------------------+


**PHP to MongoDB on 64-bit systems**:

+----------------------------+----------------------------------------+
|From PHP                    |Stored in Mongo                         |
|                            +--------------------+-------------------+
|                            |native_long=0       |native_long=1      |
+============================+====================+===================+
|1234567                     |INT(1234567)        |LONG(1234567)      |
+----------------------------+--------------------+-------------------+
|123456789012                |*garbage*           |LONG(123456789012) |
+----------------------------+--------------------+-------------------+
|MongoInt32("1234567")       |INT(1234567)        |INT(1234567)       |
+----------------------------+--------------------+-------------------+
|MongoInt64("123456789012")  |LONG(123456789012)  |LONG(123456789012) |
+----------------------------+--------------------+-------------------+


**Mongo to PHP on 32-bit systems**:

+------------------+-----------------------------------------------------------------------+
|Stored in Mongo   |Returned to PHP as                                                     |
|                  +--------------------------------------------+--------------------------+
|                  |long_as_object=0                            | long_as_object=1         |
|                  +-----------------------+--------------------+                          |
|                  |native_long=0          |native_long=1       |                          |
+==================+=======================+====================+==========================+
|INT(1234567)      |int(1234567)           |int(1234567)        |int(1234567)              |
+------------------+-----------------------+--------------------+--------------------------+
|LONG(123456789012)|float(123456789012)    |MongoCursorException|MongoInt64("123456789012")|
+------------------+-----------------------+--------------------+--------------------------+


**Mongo to PHP on 64-bit systems**:

+------------------+-----------------------------------------------------------------------+
|Stored in Mongo   |Returned to PHP as                                                     |
|                  +--------------------------------------------+--------------------------+
|                  |long_as_object=0                            | long_as_object=1         |
|                  +-----------------------+--------------------+                          |
|                  |native_long=0          |native_long=1       |                          |
+==================+=======================+====================+==========================+
|INT(1234567)      |int(1234567)           |int(1234567)        |int(1234567)              |
+------------------+-----------------------+--------------------+--------------------------+
|LONG(123456789012)|float(123456789012)    |int(123456789012)   |MongoInt64("123456789012")|
+------------------+-----------------------+--------------------+--------------------------+


**Conclusion**

Getting 64-bit support right with MongoDB can be tricky as we've seen. My
recommendations would be to use just ``mongo.native_long=1`` if you only deal
with 64-bit platforms for your code. In this case, every integer number that
you put into MongoDB will also come out as an integer number; with 64-bit
integers supported.

If you however have to deal with 32-bit platforms (remember, that includes a
64-bit Windows build of PHP) then you can not reliably use just PHP's integer
types and you **have** to use the ``MongoInt64`` class. This comes with the
restriction that you have to deal with numbers in strings for initialisation.
You also need to be aware that the MongoDB shell regards all numbers as
floating point numbers, and that it can not represent 64-bit integer values.
Instead they will show up as floating point numbers. Do **not** attempt to
modify those numbers on the shell, as that `could`_ change the type.

So with the script::

	<?php
	$m = new Mongo();
	$c = $m->selectCollection('test', 'inttest');
	$c->remove(array());

	$c->insert(array('int64' => new MongoInt64("123456789012345678")));

The MongoDB shell ``mongo`` behaves like::

	$ mongo
	MongoDB shell version: 1.4.4
	url: test
	connecting to: test
	type "help" for help
	> use test
	switched to db test
	> db.inttest.find()
	{ "_id" : ObjectId("4c5ea6d59a14ce1319000000"), "int64" : { "floatApprox" : 123456789012345680, "top" : 28744523, "bottom" : 2788225870 } }

Of course, when fetching data through a driver that supports 64-bit integers
you get the proper result::

	ini_set('mongo.long_as_object', 1);
	$r = $c->findOne();
	var_dump($r['int64']);
	?>

Which shows::

	object(MongoInt64)#7 (1) {
	  ["value"]=>
	  string(18) "123456789012345678"
	}

The new functionality as outlined in this article is part of the `1.0.9 mongo
release`_ that's available through PECL with ``pecl install mongo``. Good luck
with your 64-bit integers!


.. _`UserID`: http://wiki.developers.facebook.com/index.php/User_ID
.. _`MongoDB`: http://mongodb.org
.. _`MongoDB PHP Driver`: http://github.com/mongodb/mongo-php-driver
.. _BSON: http://bsonspec.org/
.. _JSON: http://www.json.org/
.. _types: http://uk2.php.net/manual/en/mongo.types.php
.. _MongoCursorException: http://php.net/manual/en/class.mongocursorexception.php
.. _could: http://www.mongodb.org/display/DOCS/mongo+-+The+Interactive+Shell#mongo-TheInteractiveShell-Numbers
.. _`1.0.9 mongo release`: http://pecl.php.net/package-info.php?package=mongo&version=1.0.9
