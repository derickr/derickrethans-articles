Typehints for scalar types
==========================

.. articleMetaData::
   :Where: Sandefjord, Norway
   :Date: 20060617 1603 CEST
   :Tags: conference, php, work

An attendee at the `PHP New York Conference`_ asked me on some tips on how to implement typehinting
for scalar types (integer, float, string, boolean...). In about 20
minutes I explained how it could be done, but I got a bit bored at my
flight back so I finished the `patch`_ already. The patch is most likely not the best way of implementing this
either, and I am also not sure if this patch should be applied to PHP's
CVS at all. It might be useful for people that have more control over
their PHP setup so I decided to publish it here instead.

Besides the four scalar types it also allows the "object" type
hint which will allow objects of any class to be passed. A small example
is here:

::

	<?php
	class blah {
	}
	function foo1( object $a )
	{
	}
	$b = new blah;
	foo1( $b );
	function foo2( integer $i, float $f, bool $b )
	{
	}
	foo2( 'string', 42, 49.9 );

The example above should show:

Catchable fatal error: Argument 1 passed to foo2() must be of type
integer, string given, called in /tmp/test.php on line 16 and defined
in /tmp/test.php on line 12


.. _`PHP New York Conference`: http://phpnycon.com
.. _`patch`: http://files.derickrethans.nl/patches/ze-type-hint-2006-06-17.diff.txt

