squid config changes
====================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20070816 1551 CEST
   :Tags: php, work

Since we launched the `eZ Components`_ almost two years ago we've been using squid to
accelerate our `PEAR`_  `channel server`_ . Until today that never caused any problems. As part of
normal routines I updated all the Debian packages on this machine and
with that came now a new version of Squid. Unfortunately, this new Squid
(2.6) doesn't accept the old syntax (for Squid 2.5) for HTTP
acceleration mode anymore and things stopped working. After some digging
I found out that:

::

	httpd_accel_host 127.0.0.1
	httpd_accel_port 8080
	httpd_accel_single_host on
	httpd_accel_uses_host_header on

should now be:

::

	http_port 80 defaultsite=components.ez.no
	cache_peer 127.0.0.1 parent 8080 0 no-query originserver
	acl oursites dstdomain components.ez.no
	http_access allow oursites

Now it all works fine again.


.. _`eZ Components`: http://components.ez.no
.. _`PEAR`: http://pear.php.net
.. _`channel server`: http://pear.php.net/manual/en/guide.migrating.channels.php

