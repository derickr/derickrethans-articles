Parsing non-wellformed XML
==========================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20040825 0958 CEST
   :Tags: php, work

I just found out that `Christian`_ added a `patch`_ to `PHP`_ 5.1 that allows the parsing of
non-welformed XML documents.

The thing is that a non-welformed XML document is no XML document
at all and therefore such thing should never be possible. Adding
those features to PHP promotes the use of using bad XML over
trying to fix the XML documents in the first place. This might
have a huge impact as PHP is soo widely used that people just care
less about non-wellformed XML. This feature should not be in PHP,
and if you ask me `Daniel`_ shouldn't have added this to `libxml2`_ in the first place.

I would urge Christian to restrict the use of this function
drastically, perhaps with forcing the display of the warnings, or
with a configure switch to allow this silly feature to be used.


.. _`Christian`: http://blog.bitflux.ch/
.. _`patch`: http://news.php.net/php.cvs/28328
.. _`PHP`: http://php.net
.. _`Daniel`: http://veillard.com/
.. _`libxml2`: http://xmlsoft.org/

