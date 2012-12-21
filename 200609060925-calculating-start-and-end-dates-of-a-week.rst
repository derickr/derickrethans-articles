Calculating start and end dates of a week.
==========================================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20060906 0925 CEST
   :Tags: php, datetime

A friend asked "How do I calculate start (monday) and end (sunday)
dates from a given week number for a specified year?" Instead of
having to come up with your own algorithm you can simply do the
following in PHP 5.1 and higher:

::

	<?php
	// Monday
	echo date(
	    datetime::ISO8601,
	    strtotime("2006W37"));
	
	// Sunday
	echo date(
	    datetime::ISO8601,
	    strtotime("2006W377"));
	?>

The format is " *year* 'W' *weeknr* ( *daynr* )?" where the default *daynr* is 1 being the Monday of that
week. The *daynr* can be in the range 0 to 7. The *weeknr* is
the `ISO week number`_ . Please note that the " *year* '-W' *weeknr* ( '-' *daynr* )?" format is only supported in PHP 5.2 and
higher.


.. _`ISO week number`: http://en.wikipedia.org/wiki/ISO_week_date

