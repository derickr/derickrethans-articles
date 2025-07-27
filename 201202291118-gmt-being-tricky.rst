To GMT or not to GMT
====================

.. articleMetaData::
   :Where: Montréal, Canada
   :Date: 2012-02-29 11:18 America/Montreal
   :Tags: blog, php, datetime
   :Short: gmt

A leap day with a post on Date/Time issues seems fitting...

Earlier today, on twitter_, `@skoop`_ asked: *"dear #lazyweb, when I use
DateTimeZone('GMT'), why does format('e') output UTC?"* What he means is that::

	$date = new DateTime('now', new DateTimeZone('GMT'));
	echo $date->format(DateTime::RFC2822 . ' e' );

which shows::

	Wed, 29 Feb 2012 16:26:23 +0000 UTC

As you can see that has ``UTC`` and not ``GMT`` as you might expect.

If you look closely at the documentation_ for the "Other" group of timezones,
it lists with the ``GMT`` timezone as warning: *"Please do not use any of the
timezones listed here (besides UTC), they only exist for backward compatible
reasons."* If you use ``GMT`` as timezone identifier in the constructor to
``DateTimeZone``, PHP will instead use the correct ``UTC`` in output. When
you create a ``DateTimeZone`` object like this, you will always get a "type 3"
DateTimeZone object::

	$date = new DateTime('now', new DateTimeZone('GMT'));
	var_dump($date);

which shows::

	object(DateTime)#1 (3) {
	  ["date"]=>
	  string(19) "2012-02-29 16:30:51"
	  ["timezone_type"]=>
	  int(3)
	  ["timezone"]=>
	  string(3) "UTC"
	}

Now apparently some systems *\*cough\*Silverlight\*cough\** require ``GMT`` to be
used.  ``GMT`` is **not** a timezone, but just a timezone abbreviation meant
for **output only**.  Read more about that in the article
`"Leap Seconds and What To Do With Them"`_.  However, if it is necessary you
can create a ``DateTime`` object with a different timezone type. In this
case you want a "type 2" timezone associated with the ``DateTime`` object.
You do that by simply forcing that timezone abbreviation when instantiating
a ``DateTime`` object::

	$date = new DateTime('today GMT');
	var_dump( $date );

which shows::

	object(DateTime)#1 (3) {
	  ["date"]=>
	  string(19) "2012-02-29 16:32:16"
	  ["timezone_type"]=>
	  int(2)
	  ["timezone"]=>
	  string(3) "GMT"
	}

Things like this also work::

	$date = new DateTime( "GMT" );
	$date->setDate( 2012, 2, 19 );
	var_dump( $date );

And of course, this is not limited to ``GMT`` only::

	$date = new DateTime( "EST" );
	$date->setDate( 2012, 2, 19 );
	var_dump( $date );

which shows::

	object(DateTime)#1 (3) {
	  ["date"]=>
	  string(19) "2012-02-19 16:37:58"
	  ["timezone_type"]=>
	  int(2)
	  ["timezone"]=>
	  string(3) "EST"
	}


As a reminder of the three different types of timezones that can be attached
to ``DateTime`` objects:

 #. A UTC offset, such as in ``new DateTime( "2012-02-29 -0500" );``
 #. A timezone abbreviation, such as in ``new DateTime( "2012-02-29 EST" );``
 #. A timezone identifier, such as in ``new DateTime( "2012-02-29 America/Montreal" );``

Please also be aware that *only* ``DateTime`` objects with "type 3" timezones
attached to them will calculate correctly over Daylight Saving Time boundaries.

If you want to learn more about Dates and Times, and how to use them with PHP,
please get a copy of my book
`"php|architect's Guide to Date and Time Programming"`_.

.. _twitter: https://twitter.com/skoop/status/174794576312287232
.. _`@skoop`: https://twitter.com/skoop
.. _documentation: http://ca.php.net/manual/en/timezones.others.php
.. _`"Leap Seconds and What To Do With Them"`: http://drck.me/lsawtdwt-6ye
.. _`"php|architect's Guide to Date and Time Programming"`: http://phpdatebook.com/

