Using the Right Debugging Tools
===============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2018-10-09 10:26 Europe/London
   :Tags: blog, php, mongodb
   :Short: dbgtools

A while ago, we updated the MongoDB PHP driver's embedded C library to a new
version. The C library handles most of the connection management and other low
level tasks that the PHP driver needs to successfully talk to MongoDB
deployments, especially in replicated environments where servers might
disappear for maintenance or hardware failures.

After upgrading the C library from 1.12 to `1.13`_ we noticed that one of the
PHP driver's tests was failing::

	derick@singlemalt:~/dev/php/derickr-mongo-php-driver $ make test TESTS=tests/atlas.phpt
	…
	====================================================================
	FAILED TEST SUMMARY
	---------------------------------------------------------------------
	Atlas Connectivity Tests [tests/atlas.phpt]
	=====================================================================

When running the test manually, we get::


	derick@singlemalt:~/dev/php/mongodb-mongo-php-driver $ php tests/atlas.phpt 
	--TEST--
	Atlas Connectivity Tests
	--SKIPIF--
	skip Atlas tests not wanted
	--FILE--
	PASS
	mongo-php-driver/src/libmongoc/src/libmongoc/src/mongoc/mongoc-cluster.c:1852 mongoc_cluster_fetch_stream_single():
		precondition failed: scanner_node && !scanner_node->retired
	Aborted

That was not good news.

.. _`1.13`: https://github.com/mongodb/mongo-c-driver/releases/tag/1.13.0

The ``atlas.phpt`` test tests whether the PHP driver (through the C driver)
can connect to a set of different deployments of Atlas_, MongoDB's Database as a
Service platform. The test makes sure we can talk to an Atlas_ replica set, a
sharded cluster, a `free tier`_ replica set, as well as TLS 1.1 and TLS 1.2
specific deployments. The test started failing when connecting to the second
provided URI (the sharded cluster).

.. _Atlas: https://www.mongodb.com/cloud/atlas
.. _`free tier`: https://docs.mongodb.com/manual/tutorial/atlas-free-tier-setup/

At first I thought this was caused by the upgrade from version 1.12 to 1.13 of
the C driver, but that didn't end up being the case. Let's see how we got to
finding and fixing this bug.

First of all, I started GDB to see where it was failing::

	derick@singlemalt:~/dev/php/mongodb-mongo-php-driver $ gdb --args php tests/atlas.phpt 
	GNU gdb (Debian 8.1-4+b1) 8.1
	…
	Reading symbols from php...done.
	(gdb) run
	Starting program: /usr/local/php/7.3dev/bin/php tests/atlas.phpt
	[Thread debugging using libthread_db enabled]
	Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
	--TEST--
	Atlas Connectivity Tests
	--FILE--
	PASS
	mongo-php-driver/src/libmongoc/src/libmongoc/src/mongoc/mongoc-cluster.c:1852 mongoc_cluster_fetch_stream_single():
		precondition failed: scanner_node && !scanner_node->retired

	Program received signal SIGABRT, Aborted.
	__GI_raise (sig=sig@entry=6) at ../sysdeps/unix/sysv/linux/raise.c:51
	51	../sysdeps/unix/sysv/linux/raise.c: No such file or directory.

This shows that the C driver bailed out due to a specific assertion on line
``1852`` of ``mongoc-cluster.c``. This assertion reads::

	BSON_ASSERT (scanner_node && !scanner_node->retired);

Which didn't really say a lot. The next thing to try is then to make a
**backtrace** in GDB with the ``bt`` command. This revealed::

	(gdb) bt
	#0  __GI_raise (sig=sig@entry=6) at ../sysdeps/unix/sysv/linux/raise.c:51
	#1  0x00007ffff41472f1 in __GI_abort () at abort.c:79
	#2  0x00007ffff32cdcf1 in mongoc_cluster_fetch_stream_single (
		cluster=0x555556ba8e68, server_id=2, reconnect_ok=true, error=0x555556b95858)
		at mongo-php-driver/src/libmongoc/src/libmongoc/src/mongoc/mongoc-cluster.c:1852
	#3  0x00007ffff32cd9df in _mongoc_cluster_stream_for_server (
		cluster=0x555556ba8e68, server_id=2, reconnect_ok=true, cs=0x0, reply=0x7fffffff9ee0, error=0x555556b95858)
		at mongo-php-driver/src/libmongoc/src/libmongoc/src/mongoc/mongoc-cluster.c:1762
	#4  0x00007ffff32cdbe6 in mongoc_cluster_stream_for_server (
		cluster=0x555556ba8e68, server_id=2, reconnect_ok=true, cs=0x0, reply=0x7fffffff9ee0, error=0x555556b95858)
		at mongo-php-driver/src/libmongoc/src/libmongoc/src/mongoc/mongoc-cluster.c:1826
	#5  0x00007ffff32ddb0a in _mongoc_cursor_fetch_stream (cursor=0x555556b95700)
		at mongo-php-driver/src/libmongoc/src/libmongoc/src/mongoc/mongoc-cursor.c:647
	#6  0x00007ffff32e12a0 in _prime (cursor=0x555556b95700)
		at mongo-php-driver/src/libmongoc/src/libmongoc/src/mongoc/mongoc-cursor-find.c:40
	#7  0x00007ffff32df22c in _call_transition (cursor=0x555556b95700)
		at mongo-php-driver/src/libmongoc/src/libmongoc/src/mongoc/mongoc-cursor.c:1146
	#8  0x00007ffff32df54b in mongoc_cursor_next (cursor=0x555556b95700, bson=0x7fffffffa038)
		at mongo-php-driver/src/libmongoc/src/libmongoc/src/mongoc/mongoc-cursor.c:1214
	#9  0x00007ffff332511c in phongo_cursor_advance_and_check_for_error (cursor=0x555556b95700)
		at mongo-php-driver/php_phongo.c:742
	#10 0x00007ffff33253d9 in phongo_execute_query (
		client=0x555556ba8e60, namespace=0x7fffeaeb5d58 "test.test", zquery=0x7ffff38201c0, options=0x0,
		server_id=1, return_value=0x7ffff38200f0, return_value_used=1)
		at mongo-php-driver/php_phongo.c:810
	#11 0x00007ffff3342cc3 in zim_Manager_executeQuery (execute_data=0x7ffff3820160, return_value=0x7ffff38200f0)
		at mongo-php-driver/src/MongoDB/Manager.c:492
	#12 0x0000555555e41d16 in execute_internal (execute_data=0x7ffff3820160, return_value=0x7ffff38200f0)
		at /home/derick/dev/php/php-src.git/Zend/zend_execute.c:2328
	…

At first glance, I couldn't really see anything wrong with this backtrace,
and was still puzzled as to why it would abort. I decided to go for a lunch time
walk and have a look at it again. I always find that walks are good for
clearing my mind.

After the walk, and a cuppa tea, I looked at the backtrace again, and noticed
the following curiosity::

	#4  0x00007ffff32cdbe6 in mongoc_cluster_stream_for_server (
		cluster=0x555556ba8e68, server_id=2, reconnect_ok=true, cs=0x0, reply=0x7fffffff9ee0, error=0x555556b95858)
		at mongo-php-driver/src/libmongoc/src/libmongoc/src/mongoc/mongoc-cluster.c:1826

vs::

	#10 0x00007ffff33253d9 in phongo_execute_query (
		client=0x555556ba8e60, namespace=0x7fffeaeb5d58 "test.test", zquery=0x7ffff38201c0, options=0x0,
		server_id=1, return_value=0x7ffff38200f0, return_value_used=1)
		at mongo-php-driver/php_phongo.c:810

In frame ``#10`` the ``server_id`` variable is ``1``, whereas in frame ``#4``
later on, the ``server_id`` variable is ``2``. These should have been the same.

The server ID is determined by the C driver when selecting a server to send a
read or write operation to, and refers to a specific server's connection ID.
The PHP driver uses this server ID when executing the query through the
``phongo_execute_query`` function, which calls the C driver's
``mongoc_collection_find_with_opts``. The latter accepts as its 3rd argument a
``bson_t`` value with options to use while executing a query. These options
include the pre-selected server ID, so that the C driver does not attempt to
reselect a server::

	cursor = mongoc_collection_find_with_opts(collection, query->filter, query->opts,
		phongo_read_preference_from_zval(zreadPreference TSRMLS_CC));

I decided to investigate which options the PHP driver was sending to
``mongoc_collection_find_with_opts``. A while ago I developed a `GDB helper
function`_, about which I wrote in `pretty-printing BSON`_. I sourced this
helper within my GDB instance, and switched to frame ``#10`` to inspect the
value of the query options::

	(gdb) source ~/dev/php/mongodb-mongo-php-driver/src/libmongoc/.gdbinit 
	(gdb) frame 10

The function call uses the options from the query struct ``query->opts``, so I
used the ``printbson`` helper function to display its contents::

	(gdb) printbson query->opts

Which showed::

	$11 = "!\000\000\000\020serverId\000\002\000\000\000\020serverId\000\001", '\000' <repeats 90 times>
	INLINE (len=33)
	{
		'serverId' : NumberInt("2"),
		'serverId' : NumberInt("1")
	}

.. _`GDB helper function`: https://github.com/mongodb/mongo-c-driver/blob/5e76b2244032d1eb9d3610753504fd7cd9ad56ed/.gdbinit
.. _`pretty-printing BSON`: /gdb-bson.html

There are not supposed to be two conflicting ``serverId`` elements. Unlike
PHP's arrays, ``bson_t`` values can have the same key appear multiple times.
Although the C driver had selected server ID ``1`` for this query, server
``2`` was used because it was the first ``serverId`` element in the options
struct. But why were there two values in the first place?

If you look at the PHP test, you see the following::

	<?php
	$urls = explode("\n", file_get_contents('.travis.scripts/atlas-uris.txt'));

	…
	$query = new \MongoDB\Driver\Query([]);

	foreach ($urls as $url) {
		…

		try {
			$m = new \MongoDB\Driver\Manager($url);
			…
			iterator_to_array($m->executeQuery('test.test', $query));
			…
		} catch(Exception $e) {
			…
		}
	}
	?>

From this follows that we create the ``Query`` object, assign it to
``$query``, and then use the same variable **for each iteration**. Somehow, we
were not resetting the query options back to default before we used them,
resulting in a duplicate ``serverId`` field. Once we figured out the problem,
creating `the fix`_ was easy enough: Make sure we use a clean ``query->opts``
struct before we pass it to the ``mongoc_collection_find_with_opts`` function.

.. _`the fix`: https://github.com/mongodb/mongo-php-driver/commit/3624e5acfd1d64db5a636880c41f1a88aa480a25#diff-c06c6e1c9374aecabcf544157f9d0c26

Debugging this issue was made a lot easier by having the right debugging tools,
and this case shows that spending time on writing the GDB helper function
``printbson`` earlier in the year paid off. With this bug fixed, we could
release a `new patch version`_ of the MongoDB Driver for PHP.

Happy hacking!

.. _`new patch version`: https://github.com/mongodb/mongo-php-driver/releases/tag/1.5.3
