The Confused C-Driver
=====================

.. articleMetaData::
   :Where: London, UK
   :Date: 2018-12-21 09:21 Europe/London
   :Tags: blog, php, mongodb
   :Short: c-endian

Over the past few months I have been adding the MongoDB_ driver for PHP to
Evergreen_, MongoDB's CI system. Although we already test on Travis_ and
AppVeyor_, adding the driver to Evergreen_ also allows us to test on more
esoteric platforms, such as ARM64_, Power8_, and zSeries_.

.. _MongoDB: https://www.mongodb.com/
.. _Evergreen: https://github.com/evergreen-ci/evergreen
.. _Travis: https://travis-ci.org/
.. _AppVeyor: https://www.appveyor.com/
.. _ARM64: https://en.wikipedia.org/wiki/ARM_architecture#64/32-bit_architecture
.. _Power8: https://en.wikipedia.org/wiki/POWER8
.. _zSeries: https://en.wikipedia.org/wiki/IBM_Z

While adding the zSeries architecture to our test matrix, we noticed that one
specific group of tests was failing_. In these tests we would do an insert
with a write concern of ``{w: 0}`` followed by an insert with ``{w: 1}``.

.. _failing: https://evergreen.mongodb.com/task_log_raw/mongo_php_driver_tests_php7__versions~4.0_php_versions~7.0_os_php7~rhel74_zseries_test_replicaset_auth_cdec9691b67f57a76f079981b960dbd257832b74_18_12_14_14_41_39/0?type=T#L1560

A `write concern`_ is used to enforce specific guarantees on how many MongoDB
servers a write needs to be replicated to, before the primary responds to the
client that the write has been acknowledged. A write concern of ``{w: 0}`` means
that the client does not care about the result of the write operation, and
henceforth does not need a reply from the server. These are also referred to as
unacknowledged writes.

The test fails in such a way that the result for the second insert (with
``{w: 1}``) seemed empty.

.. _write concern: https://docs.mongodb.com/manual/reference/write-concern/#w-option

Figuring out what was going wrong was not particularly easy in this case.
Single-stepping through a lot of connection and socket handling code with a
fairly complicated protocol takes time. After several hours it was still
unclear what was going wrong.

I tend tried to find out whether the data was sent correctly over the wire. As
we only have a single shared zSeries development server without a GUI, I could
not use Wireshark_. Moreover, I could not use tcpdump_ either, as I didn't
have any ``sudo`` rights::

	$ tcpdump -i lo -nnXSs 0 'port 27017'
	tcpdump: lo: You don't have permission to capture on that device
	(socket: Operation not permitted)

.. _Wireshark: https://www.wireshark.org/
.. _tcpdump: https://www.tcpdump.org/

After some time I figured out that you can also use strace_ for finding out
what goes over the wire, albeit in a slightly annoying format. I used the
following invocation::

	strace -f -e trace=network -s 10000 -p 22506

.. _strace: https://en.wikipedia.org/wiki/Strace

The ``-f`` makes strace follow forked processes, the ``-e trace=network``
shows the system calls of all network related operations, the ``-s 10000``
makes sure we see 10 000 bytes of data in strings, and the ``22506`` value for
the ``-p`` argument is the process ID.

On the zSeries platform, the strace dump looks like the following. The first
group is the insert with ``{w: 0}``, and the second group the insert with
``{w: 1}}``::

	recvmsg(36, {msg_name(0)=NULL, msg_iov(1)=[{"\252\0\0\0\2\0\0\0\0\0\0\0\335\7\0\0", 16}], msg_controllen=0, msg_flags=0}, 0) = 16
	recvmsg(36, {msg_name(0)=NULL, msg_iov(1)=[{"\0\0\0\2\0h\0\0\0\2insert\0#\0\0\0server_server_executeBulkWrite_002\0\10ordered\0\1\2$db\0\7\0\0\0phongo\0\3writeConcern\0\f\0\0\0\20w\0\0\0\0\0\0\0\1,\0\0\0documents\0\36\0\0\0\20wc\0\0\0\0\0\7_id\0\\\31\32\27\r\0200G%yF\322\0", 154}], msg_controllen=0, msg_flags=0}, 0) = 154
	sendmsg(36, {msg_name(0)=NULL, msg_iov(1)=[{"&\0\0\0v\0\0\0\2\0\0\0\335\7\0\0\0\0\0\0\0\21\0\0\0\1ok\0\0\0\0\0\0\0\360?\0", 38}], msg_controllen=0, msg_flags=0}, MSG_NOSIGNAL) = 38

	recvmsg(36, {msg_name(0)=NULL, msg_iov(1)=[{"\316\0\0\0\4\0\0\0\0\0\0\0\335\7\0\0", 16}], msg_controllen=0, msg_flags=0}, 0) = 16
	recvmsg(36, {msg_name(0)=NULL, msg_iov(1)=[{"\0\0\0\0\0\214\0\0\0\2insert\0#\0\0\0server_server_executeBulkWrite_002\0\10ordered\0\1\2$db\0\7\0\0\0phongo\0\3lsid\0\36\0\0\0\5id\0\20\0\0\0\4\337{QQi\312A\245\213\270\210\376&+\247\260\0\3writeConcern\0\f\0\0\0\20w\0\1\0\0\0\0\0\1,\0\0\0documents\0\36\0\0\0\20wc\0\1\0\0\0\7_id\0\\\31\32\27\r\0200G%yF\323\0", 190}], msg_controllen=0, msg_flags=0}, 0) = 190
	sendmsg(36, {msg_name(0)=NULL, msg_iov(1)=[{"-\0\0\0x\0\0\0\4\0\0\0\335\7\0\0\0\0\0\0\0\30\0\0\0\20n\0\1\0\0\0\1ok\0\0\0\0\0\0\0\360?\0", 45}], msg_controllen=0, msg_flags=0}, MSG_NOSIGNAL) = 45

The first line is the header, with the last four bytes (``\335\7\0\0``) being
the opcode ``2013`` (OP_MSG_). After the opcode, on the second line follow the
flagBits_: ``\0\0\0\2``. The flagBits_ field is ``\0\0\0\0`` for the second
insert.

.. _OP_MSG: https://github.com/mongodb/specifications/blob/master/source/message/OP_MSG.rst#id1
.. _flagBits: https://github.com/mongodb/specifications/blob/master/source/message/OP_MSG.rst#flagbits

On my local Linux machine, the strace dump looks a little more complicated as
the data is split into multiple IOV packets, but similar data is present. The
trace is also made with the client and server reversed, so ``sendmsg`` and
``recvmsg`` are also swapped::

	sendmsg(4, {msg_name=NULL, msg_namelen=0, msg_iov=[{iov_base="\252\0\0\0", iov_len=4}, {iov_base="\2\0\0\0", iov_len=4}, {iov_base="\0\0\0\0", iov_len=4}, {iov_base="\335\7\0\0", iov_len=4},
	           {iov_base="\2\0\0\0", iov_len=4}, {iov_base="\0", iov_len=1}, {iov_base="h\0\0\0\2insert\0#\0\0\0server_server_executeBulkWrite_002\0\10ordered\0\1\2$db\0\7\0\0\0phongo\0\3writeConcern\0\f\0\0\0\20w\0\0\0\0\0\0\0", iov_len=104}, {iov_base="\1", iov_len=1}, {iov_base=",\0\0\0", iov_len=4}, {iov_base="documents\0", iov_len=10}, {iov_base="\36\0\0\0\20wc\0\0\0\0\0\7_id\0\\\31\32=\343\231,\25\301\\\203r\0", iov_len=30}], msg_iovlen=11, msg_controllen=0, msg_flags=0}, MSG_NOSIGNAL) = 170

	sendmsg(4, {msg_name=NULL, msg_namelen=0, msg_iov=[{iov_base="\316\0\0\0", iov_len=4}, {iov_base="\4\0\0\0", iov_len=4}, {iov_base="\0\0\0\0", iov_len=4}, {iov_base="\335\7\0\0", iov_len=4},
	           {iov_base="\0\0\0\0", iov_len=4}, {iov_base="\0", iov_len=1}, {iov_base="\214\0\0\0\2insert\0#\0\0\0server_server_executeBulkWrite_002\0\10ordered\0\1\2$db\0\7\0\0\0phongo\0\3lsid\0\36\0\0\0\5id\0\20\0\0\0\4 \314rM\25\362L\7\203\36O\2157\344\201V\0\3writeConcern\0\f\0\0\0\20w\0\1\0\0\0\0\0", iov_len=140}, {iov_base="\1", iov_len=1}, {iov_base=",\0\0\0", iov_len=4}, {iov_base="documents\0", iov_len=10}, {iov_base="\36\0\0\0\20wc\0\1\0\0\0\7_id\0\\\31\32=\343\231,\25\301\\\203s\0", iov_len=30}], msg_iovlen=11, msg_controllen=0, msg_flags=0}, MSG_NOSIGNAL) = 206
	recvfrom(4, "-\0\0\0", 4, 0, NULL, NULL) = 4
	recvfrom(4, "i\0\0\0\4\0\0\0\335\7\0\0\0\0\0\0\0\30\0\0\0\20n\0\1\0\0\0\1ok\0\0\0\0\0\0\0\360?\0", 41, 0, NULL, NULL) = 41

The first *big* difference is that locally, there is no ``recvfrom`` for the
first ``sendmsg``, but there is one for the second insert. The other
difference is that the flagBits (the second line in each ``sendmsg``) is
``\2\0\0\0`` for the first insert. This is different in the zSeries trace,
where the value is ``\0\0\0\2``. 

The latest version of the MongoDB `wire protocol`_ uses `OP_MSG`_ for all
operations, which is a typical request/response API. This means that generally
every request is expected to generate a reply from the server. For
unacknowledged writes, the client does not need a reply from the server, and
the way to tell the server that is by setting the moreToCome_ flag in the
OP_MSG_ packet. It's `bit 1`_ in the 32 bit wide bit field.

.. _`wire protocol`: https://docs.mongodb.com/manual/reference/mongodb-wire-protocol/
.. _moreToCome: https://github.com/mongodb/specifications/blob/master/source/message/OP_MSG.rst#moretocome-on-requests
.. _`bit 1`: https://github.com/mongodb/specifications/blob/master/source/message/OP_MSG.rst#flagbits

The wire protocol requires the use of the little-endian_ byte order for
numbers. libmongoc_, which implements the connection aspects of the PHP
driver, accomplishes this by running a "swap" in case the driver is run on a
big-endian_ system such as zSeries.

.. _`little-endian`: https://en.wikipedia.org/wiki/Endianness#Little-endian
.. _`big-endian`: https://en.wikipedia.org/wiki/Endianness#Big-endian
.. _libmongoc: http://mongoc.org/

It turned out that although this swap happened for all normal integers (such
as the ``OP_MSG`` opcode ``2013``), it did **not** happen for the flagBits_.
This meant that instead of setting the moreToCome_ flag (bit 1), we set bit
25, which does absolutely nothing. Because of that, the server helpfully sent
a reply (the first ``sendmsg``) in the zSeries dump. Because the driver did
not expect such a packet, it did not **read** it from the socket either.

This meant that when the driver read information from the socket in response
to the *second* insert (the one with a write concern of ``{w: 1}``) it read
the response of the *first* insert. And then it got confused.

In the end, the fix_ was as easy as making sure that the flagBits field was
also correctly swapped between big- and little-endian.

.. _fix: https://github.com/mongodb/mongo-c-driver/pull/557

Confusion solved!
