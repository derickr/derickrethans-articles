DateTimeImmutable
=================

.. articleMetaData::
   :Where: Montreal, Canada
   :Date: 2014-02-25 09:41 America/Montreal
   :Tags: blog, php, datetime
   :Short: dti

The first time that my improved DateTime support made its way into PHP was
officially in PHP 5.1, although the more advanced features such as the DateTime
class only made it appearance in PHP 5.2. Since its introduction the
DateTime_ class implementation suffered from one design mistake —
arguably not something that even an RFC would have highlighted.

In PHP, if you do the following::

	<?php
	function formatNextMondayFromNow( DateTime $dt )
	{
		return $dt->modify( 'next monday' )->format( 'Y-m-d' );
	}

	$d = new DateTime();
	echo formatNextMondayFromNow( $d ), "\n";
	echo $d->format( 'Y-m-d' ), "\n";
	?>

It displays::

	2014-02-17
	2014-02-17

The ``modify()`` method does not only return the modified DateTime object, but
also changes the DateTime object it was called on. In an API like above, this
is of course totally unexpected and the only way to avoid this behaviour is to
do the following instead::

	echo formatNextMondayFromNow( clone $d ), "\n";

This **mutability** property that all modifying methods of the DateTime class
have is highly annoying, and something that I would now rather remove. But of
course we cannot as that would break backwards compatibility.

So in PHP 5.5, after a few stumbles, I finally managed to rectify this.
I did not change the original class's behaviour, but instead, I added a new
class. The new ``DateTimeImmutable`` class which does not display this
"mutable" behaviour, and only **returns** the modified object::

	<?php
	function formatNextMondayFromNow( DateTimeImmutable $dt )
	{
		return $dt->modify( 'next monday' )->format( 'Y-m-d' );
	}

	$d = new DateTimeImmutable();
	echo formatNextMondayFromNow( $d ), "\n";
	echo $d->format( 'Y-m-d' ), "\n";
	?>

Which displays::

	2014-02-17
	2014-02-11

Both the old DateTime_ and the new DateTimeImmutable_ classes implement the
DateTimeInterface_ interface. This interface defines all the methods that
both classes implement. Of course, they can not be methods that change the
object. That is why the interface is restricted to formatting and other
read-only methods, such as getTimezone_.

Perhaps in the future (PHP 6), we can replace the mutable DateTime class with
the new DateTimeImmutable_ variant. Until then, you will have to take care of
this yourself!

.. _DateTime: http://php.net/datetime
.. _DateTimeImmutable: http://php.net/datetimeimmutable
.. _DateTimeInterface: http://php.net/datetimeinterface
.. _getTimezone: http://uk1.php.net/manual/en/datetime.gettimezone.php
