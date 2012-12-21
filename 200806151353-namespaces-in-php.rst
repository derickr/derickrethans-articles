Namespaces in PHP
=================

.. articleMetaData::
   :Where: Dieren, the Netherlands
   :Date: 20080615 1353 CEST
   :Tags: blog, conference, php, work

During Stefan Priebsch' session at the `Dutch PHP Conference`_ on `PHP 5.3 and PHP 6 - A look ahead`_ a discussion popped up about PHP's namespace support. One of
the things that came up is the conflicts that can arise with internal
classes.

Take for example this code:

::

	<?php
	use PEAR::Date::Interval as Interval;
	?>

In PHP 5.3 this would alias the class Interval in the namespace
PEAR::Date to the class Interval. For now, this code would work just
fine. However, if PHP would introduce a class "Interval" at
some point in the future (and PHP can do this as it `owns the global namespace`_ ) then the above code would suddenly stop working.
So in order to use namespaces properly, you always need to have at least *two* elements left, like:

::

	<?php
	use PEAR::Date as pd;
	$interval = new pd::Interval;
	?>

You need to make sure of course that the short name that you pick is
something that does not sound to much like a class name otherwise you'll
have exactly the same issue. It's not very likely that PHP will
introduce a class *pd* though.


.. _`Dutch PHP Conference`: http://phpconference.nl
.. _`PHP 5.3 and PHP 6 - A look ahead`: http://phpconference.nl/schedule/php6
.. _`owns the global namespace`: http://www.php.net/manual/en/userlandnaming.rules.php

