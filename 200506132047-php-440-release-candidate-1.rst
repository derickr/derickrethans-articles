PHP 4.4.0 Release Candidate 1
=============================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20050613 2047 CEST
   :Tags: blog, cms, php, work

I just packaged PHP 4.4.0RC1, the release announcement goes as
follows:

The PHP Team just released it's first release candidate for PHP 4.4.0.
This is solely a bug-fix only release, the increased middle digit is
needed because this release changes PHP's Internal API that causes
third-party binary extensions to be incompatible with PHP
4.3.x.

This release address a major problem within PHP concerning references.
If references where used in a wrong way, PHP would often create memory
corruptions which would not always surface and be visible. In other
cases it can cause variables and objects to change type or class. If you
encountered strange behavior like this, this release might fix
it.

Besides addressing this reference related bug, 46 other bugs are fixed.
Please test this release and report any bugs or problems in our
bugsystem (after searching first).

If all goes well I plan to roll a release of 4.4.0 on June 27th. If
there are problems a new RC will follow. Only critical bugs are going to
be fixed between RC1 and RC2.

For users of `eZ publish`_ , this release
will make the CMS run much more stable, because of the reference bug
fixes.You can find 4.4.0 RC1 at `http://qa.php.net/~derick/`_ .


.. _`eZ publish`: http://ez.no
.. _`http://qa.php.net/~derick/`: http://qa.php.net/~derick/

