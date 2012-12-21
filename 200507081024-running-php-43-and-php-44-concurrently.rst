Running PHP 4.3 and PHP 4.4 concurrently
========================================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20050708 1024 CEST
   :Tags: cms, php, work

As I wrote `before`_ ,
PHP 4.4 addresses a couple of very serious memory corruption problems
when dealing with references. Previously you had to work around these
problems. But with the workarounds in place you will now see that things
no longer with with PHP 4.4. This is a real problem if you have a large
application (like `eZ publish`_ ) as you
know might need to run PHP 4.4 for the sites with an upgraded eZ
publish, but also PHP 4.3 for the non-upgraded sites. Because of this I
wrote a little `HOWTO`_ with information how you can set this up.


.. _`before`: /php_440_release_candidate_1.php
.. _`eZ publish`: http://ez.no/
.. _`HOWTO`: http://ez.no/community/articles/multiple_apache_installations_howto

