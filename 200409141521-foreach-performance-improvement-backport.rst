foreach() performance improvement backport
==========================================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20040914 1521 CEST
   :Tags: php, work

Today I `backported`_ Marcus' patch from PHP 5.1 back to PHP 4.3.x for inclusion in PHP
4.3.10. This patch improves performance with foreach() where the
key of the array element is not interesting (for example
foreach($array as $value); ). Benchmarks show that this gives
about a performance increase of 4% for `eZ publish`_ without using an accelerator. If an accelerator is
used, the performance increase will be even larger as this is a
pure execution speed patch.


.. _`backported`: 
.. _`eZ publish`: http://ez.no

