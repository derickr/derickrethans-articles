HTML name attribute deprecated
==============================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20071020 1831 CEST
   :Tags: php

Just now somebody on IRC was claiming that the "name"
attribute in HTML - the one that is used to give form input fields a
name to be used in $_GET and _$POST in PHP is in fact deprecated.
Quoting `About.com`_ that seems indeed true - but you should always double check on the
internet, as nobody ever reads specifications right.

The W3C has also a section on this `available`_ , and if you
only skim you might only see:

"Note that in XHTML 1.0, the name attribute of these elements is
formally deprecated, and will be removed in a subsequent version of
XHTML."

But if you read correctly, it's only for the elements: a, applet, form,
frame, iframe, img, and map.

Today's lesson: always read the specs *carefully* .


.. _`About.com`: http://webdesign.about.com/od/htmltags/a/bltags_deprctag.htm
.. _`available`: http://www.w3.org/TR/xhtml1/#h-4.10

