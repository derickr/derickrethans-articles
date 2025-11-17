Leap Seconds and What To Do With Them
=====================================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20090101 2303 CET
   :Tags: blog, cms, php, work, datetime
   :Short: leapsec

.. image:: images/clock.jpg
   :align: right

The start
of this new year started with some buzz about a `leap second`_ being
introduced between Dec 31st 2008, 23:59:59 and Jan 1st 2009, 00:00:00.
I've had people ask where this leap second actually comes from, and
whether you need to worry about it in your applications. To understand
leap seconds means, unfortunately, understanding how time is actually
kept.

There are many different time keeping scales. They are either defined as
an arbitrary value, or obtained from astronomical observations. First of
all, there are the variants of `Universal Time`_. UT1 is the principal form of Universal Time. UT1 is the same
all over the world, and is related to the Earths rotation. The rotation
speed of the Earth is not precise, and is slowed down by the tidal
friction of the Moon among things. Obviously, this is a bad base to
build a stable and precise timekeeping scale on.

An (SI) second is nowadays defined as "the duration of 9 192 631
770 periods of the radiation corresponding to the transition between the
two hyperfine levels of the ground state of the caesium 133 atom".
This can be measured very accurately by atomic clocks and is the base
for `International Atomic Time`_ (TAI, from Temps Atomique International). This scale
however, does not take into account the gradual but increasing slowing
down of the Earth's rotation and will therefore not reflect the
approximation to "mean solar time" over longer periods of
time. This makes it useless as a civil timekeeping scale.

For the latter purpose, `Coordinated Universal Time`_ (UTC, from Universal Time, Coordinated) has been
established. UTC uses SI seconds but is adjusted for the slowing down of
Earths rotation with leap seconds. Because it's not possible to predict
the rotation of the Earth upfront, leap seconds are only added when the
difference between UTC and UT1 approaches 0.6 SI seconds in order to
keep UTC and UT1 not differ from each other for more than 0.9 seconds.
UTC was defined (in the latest adjustment of its definition) as being 10
seconds different from TAI making 1972-01-01T00:00:00 UTC and
1972-01-01T00:00:10 TAI the same instant. Since 1972, 24 leap seconds have been
introduced, making TAI differ from UTC with 34 seconds currently. As the
Earth's rotation is slowing down at an ever increasing rate, the use of leap
seconds will get more and more annoying. After the 25th century, it is likely
that more than 4 leap seconds would be required every year. This is of course
highly unpractical and a solution needs to be found for this.

There are two other timescales based on TAI. `Terrestial Time`_ (TT) as the modern astronomical standard for the passage of
time. TT is defined as TAI + 32.184 seconds. `GPS Time`_, as used by GPS satellites and receivers, was set to match
UTC in 1980, but now deviates with 15 seconds due to the addition of
leap seconds in UTC. GPS Time is defined as TAI - 19 seconds.

Greenwich Mean Time (GMT) is often used as synonym for UTC, although
that is strictly not correct. GMT is an astronomical concept and
therefore linked to UT1. Greenwich Mean Time is *also* used as
timezone name for a timezone with zero seconds offset from UTC. It is
for example used in winter in the UK and Ireland. During the summer,
when the UK and Ireland switch to daylight savings time, GMT is not used
at all. British Summer Time (BST) and Ireland Standard Time (IST) are then
used instead. [1]_

A thing to realize is that leap seconds are added only at the end of
June 30th and December 31st. The seconds are added at 23:59:59 *UTC* . This means that there is no need to add an extra second
while counting down to the new year unless you're in a timezone that is
UTC +0 seconds (such as GMT, or WET). Of course, in other locations the
leap second is inserted as well at the same time. In Norway, the leap
second was added at Jan 1st, 2009, at 00:59:59 for example.

`Unix time`_ (also
called POSIX time) is defined as the number of seconds since Jan 1st
1970, 00:00 UTC, but without leap seconds. Although it is called *Unix
time* , this timekeeping standard is used widely in many other
operating systems and computing devices. Unix time simply counts the
number of seconds since the epoch. A date/time expressed in those
seconds is often called a Unix time stamp. Not supporting leap seconds
means that Unix time does not have any way to represent the leap second
in the form of 23:59:60. Instead it just uses the same second twice. In
practical terms, that means that Unix time progressed like this during
the year transition: 1230767999, 1230768000, 1230768000 and 1230768001.
The first 1230768000 represents 2008-12-31 23:59:60 UTC, and the second
1230768000 represents 2009-01-01 00:00:00 UTC. For applications there is
no way to differentiate between the two distinct times, although many
applications will accept both as input, like this PHP example shows:

::

	<?php
	echo strtotime("2008-12-31 23:59:59 UTC"), "\n";
	echo strtotime("2008-12-31 23:59:60 UTC"), "\n";
	echo strtotime("2009-01-01 00:00:00 UTC"), "\n";
	echo strtotime("2009-01-01 00:00:01 UTC"), "\n";
	?>

In order to synchronize the system time the `Network Time Protocol`_ (NTP) is widely used. NTP can use different sources of
time information in an hierarchical fashion. The protocol knows how to
deal with leap seconds to provide the correct Unix time stamp.

To recap the above information: UT1 is based on the actual rotational
speed of the Earth and is therefore never accurate. TAI is a time scale
based on the actual count of SI seconds. UTC also uses SI seconds, but
requires a leap second in order not to deviate from the unstable UT1
which is in sync with the average time of when the Sun is due South
(mean solar time). Unix time counts SI seconds, and resembles UTC except
that instead of leap seconds, the Unix time stamp is simply used twice.
GMT is both a different name for UT1 *and* a name for a timezone
with an UTC offset of 0 seconds. PHP uses Unix time, and does not care
about leap seconds.

Happy New Year!


.. _`leap second`: http://en.wikipedia.org/wiki/Leap_second
.. _`Universal Time`: http://en.wikipedia.org/wiki/Universal_Time#Versions
.. _`International Atomic Time`: http://en.wikipedia.org/wiki/International_Atomic_Time
.. _`Coordinated Universal Time`: http://en.wikipedia.org/wiki/Coordinated_Universal_Time
.. _`Terrestial Time`: http://en.wikipedia.org/wiki/Terrestrial_Time
.. _`GPS Time`: http://en.wikipedia.org/wiki/Global_Positioning_System#Timekeeping
.. _`Unix time`: http://en.wikipedia.org/wiki/Unix_time
.. _`Network Time Protocol`: http://en.wikipedia.org/wiki/Network_Time_Protocol

.. [1] In Ireland, "Standard Time" refers to the summer period, with the winter period defined as "Winter Time". See http://www.irishstatutebook.ie/1968/en/act/pub/0023/print.html and  http://www.irishstatutebook.ie/1971/en/act/pub/0017/index.html
