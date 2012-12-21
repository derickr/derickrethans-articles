Long Time No Blog
=================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20050815 0004 CEST
   :Tags: php

It has been a while since I blogged. I've been busy with several things
such as `ApacheCon Europe`_ and a week
holiday at the `Lofoten`_ . But
new exiting things are on the program. First of all PHP is finally
getting ready for Unicode support--with brand new major version number
"6". It is going to take quite some time to convert all the
extensions to make use of this in a proper way. For me that means that
I've to check the mcrypt extension (as that uses binary data and not
strings), add locale support with ICU's new locale functions to the date
extension. I will also be working on porting the other PHP functions
that make use of the system locale to make use of the ICU locale system
instead. Besides that I drafted a `proposal`_ for a new filter extension to safely deal with input variables. Stay
tuned!


.. _`ApacheCon Europe`: /apachecon.php
.. _`Lofoten`: /travel_report_lofoten.php
.. _`proposal`: http://files.derickrethans.nl/filter_extension.html

