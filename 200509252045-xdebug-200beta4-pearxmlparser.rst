Xdebug 2.0.0beta4 / PEAR::XMLParser
===================================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20050925 2045 CEST
   :Tags: php, work, xdebug

Last night I released version 2.0.0beta4 of `Xdebug`_ which fixes a small number of
bugs. In the next couple of weeks I hope to fix all other bugs, and
implement the features that I would like to have in Xdebug
2.0.0.

While getting the release ready I spend some time looking at the new
PEAR installer, and encountered some interesting problems with how the
PackageFile and XMLParser handle UTF-8 and XML. In PHP 5 the XML parsers
output always UTF-8, which was not taken into account and because of
misunderstandings they suggested to use entities for weird characters,
such as the norwegian "æ". But entities are *HTML* features, which do not exist in XML - so this didn't work properly
either. I cooked up a small `patch`_ which should make "pear convert" and "pear package"
deal with UTF-8 and non-ASCII characters in a more proper way.


.. _`Xdebug`: http://xdebug.org
.. _`patch`: http://files.derickrethans.nl/patches/pear-utf8-2005-09-24.diff.txt

