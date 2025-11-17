Obtaining the next month in PHP
===============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2010-01-08 09:13 Europe/London
   :Tags: blog, php, xdebug, datetime

Over and over again PHP users complain that ``next month`` in PHP's
date-string parser doesn't go to the next month, but instead skips to the
one after next month; like in the following example::

	<?php
	$d = new DateTime( '2010-01-31' );
	$d->modify( 'next month' );
	echo $d->format( 'F' ), "\n";
	?>

The output of the little script will be ``March``. March obviously doesn't
follow January as February is in between. However, the current behavior **is**
correct. The following happens internally:

- ``next month`` increases the month number (originally 1) by one. This makes
  the date ``2010-02-31``.
- The second month (February) only has 28 days in 2010, so PHP auto-corrects
  this by just continuing to count days from February 1st. You then end up at
  March 3rd.
- The formatting strips off the year and day, resulting in the output
  ``March``.

This can easily be seen when echoing the date with a full date format, which
will output ``March 3rd, 2010``::

	<?php
	echo $d->format( 'F jS, Y' ), "\n";
	?>

To obtain the correct behavior, you can use some of PHP 5.3's new functionality
that introduces the relative time stanza ``first day of``. This stanza can be
used in combination with ``next month``, ``fifth month`` or ``+8 months`` to
go to the **first day** of the specified month. Instead of ``next month`` from
the previous example, we use ``first day of next month`` here::

	<?php
	$d = new DateTime( '2010-01-08' );
	$d->modify( 'first day of next month' );
	echo $d->format( 'F' ), "\n";
	?>

This script will correctly output ``February``. The following things
happen when PHP processes this ``first day of next month`` stanza:

- ``next month`` increases the month number (originally 1) by one. This makes
  the date ``2010-02-31``.
- ``first day of`` sets the day number to 1, resulting in the date
  ``2010-02-01``.
- The formatting strips off the year and day, resulting in the output
  ``February``.

Besides ``first day of``, there is an equivalent ``last day of`` to go to the
last day of a month. The following example demonstrates this::

	<?php
	$d = new DateTime( '2010-01-08' );
	$d->modify( 'last day of next month' );
	echo $d->format( 'F jS, Y' ), "\n";
	?>

This outputs ``February 28th, 2010``. Internally the following happens:

.. image:: images/datebook-cover.jpg
   :align: right
   :target: http://tinyurl.com/phpdatebookuk

- ``next month`` increases the month number (originally 1) by one. This makes
  the date ``2010-02-08``.
- ``last day of`` increases the month number by one, and sets the day number to
  0, resulting in the date ``2010-03-00``.
- PHP then auto-corrects the invalid day number 0 by removing one from the
  month and skipping to the last day of that month, resulting in
  ``2010-02-28``.

I hope this clears up some of the behaviour of PHP's Date/Time handling.
For more information on Date/Time Programming with PHP, please refer to
my book "`php|architect's Guide to Date and Time Programming`_" that is
available through Amazon_.

.. _`php|architect's Guide to Date and Time Programming`: http://tinyurl.com/phpdatebookuk
.. _Amazon: http://tinyurl.com/phpdatebookuk

