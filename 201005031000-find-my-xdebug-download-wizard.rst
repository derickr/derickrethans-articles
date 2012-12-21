Find my Xdebug download wizard
==============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2010-05-03 10:00 Europe/London
   :Tags: blog, php, xdebug, extensions

Installing Xdebug_ seems to be a problem for some users. The main trouble is
for Windows users that don't know which file to download. Over the past few
days I've worked on a wizard_ that analyses `phpinfo()'s`_ output and
for Windows users suggests:

- which file to download
- where to put the file
- which php.ini file to modify, and how

For Unix users, it explains:

- which source tarball to download
- how to compile
- which php.ini file to modify, and how

I hope this makes it a bit easier to get going with Xdebug. Next up (hopefully
this week) is release candidate 2 for Xdebug 2.1.0.

.. _Xdebug: http://xdebug.org
.. _wizard: http://xdebug.org/wizard.php
.. _`phpinfo()'s`: http://php.net/phpinfo
