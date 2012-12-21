Why SVN still sucks
===================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20041028 0958 CEST
   :Tags: php, work

Because it still is very intuitive for basic things like reverting
a file back to a previous version.

::

	ez-3.5$ svn merge -r 9201:9200 kernel/classes/ezcontentobject.php
	svn: REPORT request failed on '/svn/nextgen/!svn/vcc/default'
	svn: Invalid editor anchoring; at least one of the input paths is not a directory and there was no source entry
	ez-3.5$ svn merge -r9201:9200 http://host/trunk/kernel/classes/ezcontentobject.php
	svn: REPORT request failed on '/svn/nextgen/!svn/vcc/default'
	svn: Invalid editor anchoring; at least one of the input paths is not a directory and there was no source entry
	ez-3.5$ cd kernel/classes
	ez-3.5/kernel/classes$ svn merge -r9201:9200 http://host/trunk/kernel/classes/ezcontentobject.php
	U  ezcontentobject.php
