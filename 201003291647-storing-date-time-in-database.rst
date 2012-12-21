Storing Date/Times in Databases
===============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2010-03-29 16:47 Europe/London
   :Tags: blog, php, cms, work, datetime, extensions

After my talk during ConFoo_ on `Advanced Date/Time Handling`_ I received a
question about whether the UTC-offset, together with the date/time in years,
months, days, hours, minutes and seconds, was enough for storing a date/time in
a database and still being able to do calculations with this. The answer to
this question was no, but it lead to an even more interesting discussion about
what *would* be enough to store an accurate date/time in a database.

.. _ConFoo: http://confoo.ca/en
.. _`Advanced Date/Time Handling`: http://confoo.ca/en/2010/session/advanced-date-time-handling-with-php

Firstly let me explain why storing a UTC offset is not adequate at all
for doing any sort of calculations. Right now (March 25th, 2010),
Montreal is on the same time as Santiago (in Chile) with both a UTC
offset of -4 hours. This means that for the same date/time (``2010-03-25
19:03``) the same timestamp is generated (``1269558180``). However if we
want to add eight months (``2010-11-25 19:03``) to the current time for
each of those locations then the timestamps *should* be different —
``1290729780`` for Montreal and ``1290722580`` for Santiago. It turns
out that *neither* of them is exactly a full 24 hour difference from
``2010-03-25 19:03`` as is shown the following script::

    <?php
    $s = '2010-03-25 19:03';
    $montrealNow = date_create( "{$s} America/Montreal" )->format( 'U' );
    $santiagoNow = date_create( "{$s} America/Santiago" )->format( 'U' );
    $s = '2010-11-25 19:03';
    $montrealThen = date_create( "{$s} America/Montreal" )->format( 'U' );
    $santiagoThen = date_create( "{$s} America/Santiago" )->format( 'U' );

    $hour = 60 * 60;
    $day = 24 * $hour;

    echo "Montreal:\n",
        $montrealNow, " - ",
        $montrealThen, "\n",
        ( ( $montrealThen - $montrealNow ) % $day ) / $hour, "\n\n";

    echo "Santiago:\n",
        $santiagoNow, " - ",
        $santiagoThen, "\n",
        ( ( $santiagoThen - $santiagoNow ) % $day ) / $hour, "\n\n";

    ?>

which gives the output::

    Montreal:
    1269558180 - 1290729780
    1

    Santiago:
    1269558180 - 1290722580
    23

From this we see that Montreal has an extra hour and Santiago has an
hour less. This is because Montreal changed from Daylight Savings Time
to normal time and Santiago, being on the southern hemisphere, moved
from normal time to Daylight Savings time.

Now if we had only the timestamp (``1269558180``) and the UTC-offset, the
only thing we could do would be to advance a specific number of days. In
this case, there are 245 days, which makes (``245 * 24 * 60 * 60 =`` )
``21168000`` seconds. If we add this to the original timestamp, we end up at
``1290726180`` which corresponds to ``2010-11-25 18:03`` Montreal time, or
``2010-11-25 20:03`` Santiago time. Neither of them being the correct
``2010-11-25 19:03``. Because from the UTC offset of 4 hours we don't know
whether we're in Montreal or Santiago, we can conclude that with just
this UTC offset and the original timestamp we can't calculate the
timestamp of something that's 8 months in the future. Storing the
UTC-offset can not change this fact either.

In order to to proper calculations, you need to keep information about
the timezone itself. In PHP timezones are identified with identifiers
such as ``America/Montreal``. If we store those alongside the timestamps,
we can do the proper calculations. The following example demonstrates
that::

    <?php
    $timestamp = 1269558180;
    $tzid      = 'America/Montreal';

    $d = new DateTime( "@$timestamp" );
    $d->setTimeZone( new DateTimeZone( $tzid ) );
    $d->modify( '+8 months' );

    echo $d->format( 'Y-m-d H:i' ), "\n";
    ?>

which gives the output: ``2010-11-25 19:03``.

This seems to work, but unfortunately, even the approach of storing the
timestamp and timezone identifier is not going to work correctly under
certain circumstances. In order to find out why, we have to go back to
January this year.

Imagine that in January this year—January 15th to be precise—we run the following
script to determine the timestamp of a date/time two months in the future::

    <?php
    $tz = 'America/Santiago';
    $ts = date_create( "2010-01-15 10:41 $tz" )->format( 'U' );

    $d = new DateTime( "@$ts" );
    $d->setTimeZone( new DateTimeZone( $tz ) );
    $d->modify( '+2 months' );

    echo $d->format( 'U Y-m-d H:i' ), "\n";
    ?>

This returns the timestamp ``1268664060`` for the date/time ``2010-03-15
10:41:00``.

Now skip forwards to the current date and time. If we used this
calculated timestamp and the timezone identifier ``America/Santiago``
today to generate a date/time string, we would however get a different
output. The following example shows this::

    <?php
    $tz = 'America/Santiago';
    $d = date_create( "@1268664060" );
    $d->setTimeZone( new DateTimeZone( $tz ) );

    echo $d->format( 'Y-m-d H:i:s' ), "\n";
    ?>

which gives the output: ``2010-03-15 11:41:00`` (and not ``2010-03-15
10:41:00``).

The difference in output is not a bug, but is caused because some
countries change Daylight Savings Time rules quite frequently. In this
case Chile decided (on March 4th) that instead of going forwards to DST at midnight
March 14th, they will delay that to April 4th. When we ran the code on
January 15th, the rules still thought that March 15th would already be
on normal time again, outside of DST. But running the code now, with
the updated rule set, we find that DST is still in effect until April
4th. The following example shows the transitions from/to DST for
Santiago in 2010::

    <?php
    $tzid = "America/Santiago";

    $tz = new DateTimeZone( $tzid );
    $transitions = $tz->getTransitions(
        strtotime( '2010-01-01 00:00 UTC' ),
        strtotime( '2010-12-31 00:00 UTC' )
    );

    foreach ( $transitions as $t )
    {
        echo $t['time'], ' ', $t['offset'] / 3600, ' ', $t['abbr'], "\n";
    }
    ?>

If we run this script with PHP 5.3.1 or 5.3.2—which still have the old
incorrect rule set—the output is::

    2010-01-01T00:00:00+0000 -3 CLST
    2010-03-14T03:00:00+0000 -4 CLT
    2010-10-10T04:00:00+0000 -3 CLST

In the SVN repository, the rule set has been updated, so snapshots of
PHP 5.3-dev have the correct rules and the script will show::

    2010-01-01T00:00:00+0000 -3 CLST
    2010-04-04T03:00:00+0000 -4 CLT
    2010-10-10T04:00:00+0000 -3 CLST

This leads to the conclusion that storing timestamps and timezone
identifiers is not good enough either, unless you want an *exact* point
in time, as opposed to the more expected date/time in a location. So how
*should* you store the latter then? Basically, in the same way that
DateTime objects are serialized in PHP 5.3. Let us imagine again, that
the following code is run on January 15th again::

    <?php
    $tzid = 'America/Santiago';
    $d = new DateTime( "2010-01-15 10:41 $tzid" );
    $d->modify( '+2 months' );
    $s = $d->format( 'Y-m-d H:i:s' );
    $ts = $d->format( 'U' );
    echo "$s (ts=$ts)\n";
    ?>

which gives the output ``2010-03-15 10:41:00 (ts=1268664060)``. The next
example is run with a recent rule set (such as in PHP SVN) today::

    <?php
    $tzid = 'America/Santiago';
    $d = new DateTime( "$s $tzid" );
    $s = $d->format( 'Y-m-d H:i:s' );
    $ts = $d->format( 'U' );
    echo "$s (ts=$ts)\n";
    ?>

which gives the output ``2010-03-15 10:41:00 (ts=1268660460)``. As you
can see, now the date/time itself is correct, although there is a
different timestamp. PHP's DateTime serialisation does something
similar::

    <?php
    $d = new DateTime();
    $d->setTimeZone( new DateTimeZone( 'America/Santiago' ) );
    var_dump( $d );
    ?>

which gives the output::

    object(DateTime)#1 (3) {
      ["date"]=>
      string(19) "2010-03-29 11:32:08"
      ["timezone_type"]=>
      int(3)
      ["timezone"]=>
      string(16) "America/Santiago"
    }

For things to work correctly, you need to have an up-to-date rule set.
PHP versions are not released as often as the timezonedb (sometimes more
than 20 times a year) and to address this issue you can install the pecl
extension timezonedb_ with ``pecl install timezonedb``. 

To store information in a database, I would use a ``char`` column-type
to store the whole ``America/Santiago`` timezone identifier and another
``char`` column-type to store the date/time (in the ``yyyy-mm-dd
hh:ii:ss`` format).  Alternatively, you can pick a 'datetime'
column-type, as long as that type **ignores** timezones altogether. For
MySQL that is the ``DATETIME`` column-type, and for PostgreSQL the
``TIMESTAMP`` or ``TIMESTAMP WITHOUT TIME ZONE`` column-types.

Databases rarely handle timezones, daylight savings time and rule
changes correctly, so avoid the database specific functionality all
together. Using either a ``char`` or a timezone-less 'datetime'
column-type would still allow you to sort and by using a 'datetime'
column-type you can even do calculations.

Happy summer time! 

.. _timezonedb: http://pecl.php.net/package/timezonedb
