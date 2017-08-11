New Date/Time Support in MongoDB
================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2017-08-15 10:22 Europe/London
   :Tags: blog, php, mongodb
   :Short: mongotimelib

In the past few months I have been working on adding time zone support to
`MongoDB's`_ `Aggregation Framework`_. This support brings in the timelib_
library that is also used in PHP_ and HHVM_ to do time zone calculations.

.. _`MongoDB's`: https://www.mongodb.com
.. _`Aggregation Framework`: https://docs.mongodb.com/manual/aggregation/
.. _timelib: https://github.com/derickr/timelib
.. _PHP: https://php.net
.. _HHVM: http://hhvm.com/

Time Zone Support for Date Extraction Operators
-----------------------------------------------

MongoDB's Aggregation Framework already has several operators to extract
date/time information *from* a Date value. For example, ``$month`` would
retrieve the month from a Date value. Considering we have the document::

    { _id: 'derickr', dateOfBirth: ISODate("1978-12-22T08:15:00Z") }

The following PHP code would extract the month ``12`` from the ``dateOfBirth``
field::

    <?php
    include 'vendor/autoload.php';

    $collection = (new MongoDB\Client)->demo->birthdays;

    $cursor = $collection->aggregate([
        [ '$project' => [
            'birthMonth' => [ '$month' => '$dateOfBirth' ]
        ] ]
    ]);

    foreach ($cursor as $person) {
        echo $person->birthMonth;
    }

However, it was not possible to extract this information in the local time
zone. So although my birth-time is given at 08:15:00, in reality it was 09:15
in the Netherlands.

The new functionality adds the ``timezone`` argument to the date extraction
functions. Currently the operators only take a single value::

    { "$month" : ISODateValue }

Because it is not possible to add another argument to this, the syntax has
been extended to::

    { "$month" : { "date" : ISODateValue } }

And then with the time zone argument added, it becomes::

    { "$month" : {
        "date" : ISODateValue,
        "timezone": timeZoneIdentifier
    } }

For example::

    { $hour: {
        date: ISODate("1978-12-22T08:15:00Z"),
        timezone: "Europe/Amsterdam"
    } }

Would return ``09`` (One hour later than UTC).

The ``timezone`` argument is optional, and must evaluate to either an Olson
Time Zone Identifier such as ``Europe/London`` or ``America/New_York``, or, an
UTC offset string in the forms: ``+03``, ``-0530``, and ``+04:45``. If you
specify a ``timezone`` argument it means that the ``dateString`` that you provided
will be interpreted as it was in that time zone.

In PHP, this example extracts the hour, and expresses the value in the
``Europe/Amsterdam`` time zone::

    <?php
    include 'vendor/autoload.php';

    $collection = (new MongoDB\Client)->demo->birthdays;

    $cursor = $collection->aggregate([
        [ '$project' => [
            'birthHour' => [
                '$hour' => [
                    "date" => '$dateOfBirth',
                    "timezone" => "Europe/Amsterdam",
                ]
            ]
        ] ]
    ]);

    foreach ($cursor as $person) {
        echo $person->birthHour, "\n";
    }

The $dateToParts Operator
-------------------------

Extracting a single value from a date can easily be done with one of the Date
Extraction operators, but it becomes cumbersome to do this for all the fields
one by one, especially when considering the application of time zones as
well::

    { "$project" : {
        "year": { "$year" : { "date" : '$dob', "timezone" : "Europe/Amsterdam" } },
        "month": { "$month" : { "date" : '$dob', "timezone" : "Europe/Amsterdam" } },
        "day": { "$day" : { "date" : '$dob', "timezone" : "Europe/Amsterdam" } },
        "hour": { "$hour" : { "date" : '$dob', "timezone" : "Europe/Amsterdam" } },
        "minute": { "$minute" : { "date" : '$dob', "timezone" : "Europe/Amsterdam" } },
    } }

The new ``$dateToParts`` operator simplifies having multiple single date
value extraction operators into a single one. Its syntax is::

    { "$project" : {
        "parts" : {
            "$dateToParts" : {
                "date" : ISODateValue,
                "timezone" : timeZoneIdentifier,
                "iso8601" : boolean
            }
        }
    } }

The ``timezone`` argument is optional, and is interpreted in the same as the
``timezone`` argument in the Date Extraction functions as explained above.

The result of the operator is a sub-document with the broken down parts,
expressed in the (optionally) given time zone::

    "parts" : {
        "year" : 1978, "month" : 12, "day" : 22,
        "hour" : 9, "minute" : 15, "second" : 0, "millisecond" : 0
    }

``$dateToParts`` also supports a third boolean argument, ``iso8601``. If set
to ``true``, instead of ``year``, ``month``, and ``day``, it returns the ISO
8601 ``isoYear``, ``isoWeekYear``, and  ``isoDayOfWeek`` fields representing
an `ISO Week Date`_. With the same date, the example is represented as::

    "parts" : {
        "isoYear" : 1978, "isoWeekYear" : 51, "isoDayOfWeek" : 5,
        "hour" : 9, "minute" : 15, "second" : 0, "millisecond" : 0
    }

.. _`ISO Week Date`: https://en.wikipedia.org/wiki/ISO_week_date

In PHP::

    <?php
    include 'vendor/autoload.php';

    $collection = (new MongoDB\Client)->demo->birthdays;

    $cursor = $collection->aggregate([
        [ '$project' => [
            'parts' => [
                '$dateToParts' => [
                    "date" => '$dateOfBirth',
                    "timezone" => "Europe/Amsterdam",
                ]
            ]
        ] ]
    ]);

    foreach ($cursor as $person) {
        var_dump( $person->parts );
    }

Which outputs, with formatting::

    class MongoDB\Model\BSONDocument#5 (1) {
      private $storage =>
      array(7) {
        'year' => int(1978)
        'month' => int(12)
        'day' => int(22)
        'hour' => int(9)
        'minute' => int(15)
        'second' => int(0)
        'millisecond' => int(0)
      }
    }

The $dateFromParts Operator
---------------------------

The new ``$dateFromParts`` operator does the opposite of the ``$dateToParts``
operator and constructs a new Date value from its constituent parts,
with the possibility of interpreting the given values in a different time zone.

Its syntax is either::

    { "$project" : {
        "date" : {
            "$dateFromParts": {
                "year" : yearExpression,
                "month" : monthExpression,
                "day" : dayExpression,
                "hour" : hourExpression,
                "minute" : minuteExpression,
                "second" : secondExpression,
                "millisecond" : millisecondExpression,
                "timezone" : timezoneExpression
            }
        }
    } }

or::

    { "$project" : {
        "date" : {
            "$dateFromParts": {
                "isoYear" : isoYearExpression,
                "isoWeekYear" : isoWeekYearExpression,
                "isoDayOfWeek" : isoDayOfWeekExpression,
                "hour" : hourExpression,
                "minute" : minuteExpression,
                "second" : secondExpression,
                "millisecond" : millisecondExpression,
                "timezone" : timezoneExpression
            }
        }
    } }

Each argument's expression needs to evaluate to a number. This means the
source can be either double, NumberInt, NumberLong, or Decimal. Decimal and
double values are only supported if they convert to a NumberLong without any
data loss.

Every argument is optional, except for ``year`` or ``isoYear``, depending on
which variant is used. If ``month``, ``day``, ``isoWeekYear``, or
``isoDayOfWeek`` are not given, they default to ``1``. The ``hour``,
``minute``, ``second`` and ``millisecond`` values default to ``0`` if not
present.

The ``timezone`` argument is interpreted in the same as the ``timezone`` argument
in the Date Extraction functions as explained above.

In PHP, an example looks like::

    <?php
    include 'vendor/autoload.php';

    $collection = (new MongoDB\Client)->demo->birthdays;

    $cursor = $collection->aggregate([
        [ '$project' => [
            'date' => [
                '$dateFromParts' => [
                    "year" => 1978, "month" => 12, "day" => 22,
                    "hour" => 9, "minute" => 15, "second" => 0,
                    "millisecond" => 0,
                    "timezone" => "Europe/Amsterdam",
                ]
            ]
        ] ]
    ]);

    foreach ($cursor as $person) {
        var_dump( $person->date->toDateTime() );
    }

Which outputs::

    class DateTime#12 (3) {
      public $date => string(26) "1978-12-22 08:15:00.000000"
      public $timezone_type => int(1)
      public $timezone => string(6) "+00:00"
    }

Changes to the $dateToString Operator
-------------------------------------

The ``$dateToString`` operator is extended with the ``timezone`` argument.
Its full new syntax is now::

    { $dateToString: {
        format: formatString,
        date: dateExpression,
        timezone: timeZoneIdentifier
    } }

The ``timezone`` argument is optional. If present, it formats the string
according to the given time zone, otherwise it uses UTC.

The ``$dateToString`` format arguments have also been expanded. With the
addition of the ``timezone`` argument came the ``%z`` and ``%Z`` format
specifiers:

%z
  The ``+hhmm`` or ``-hhmm`` numeric time zone as a string (that is, the hour
  and minute offset from UTC). Example: ``+0445``, ``-0500``

%Z
  The minutes offset from UTC as a number. Example (following the ``+0445`` and
  ``-0500`` from ``%z``): ``+285``, ``-300``

Once SERVER-29627_ gets merged, the following new format specifiers will also
be available:

%a
  The abbreviated English name of the day of the week.

%b
  The abbreviated English name of the month.

%e
  The day of the month as a decimal number, but unlike ``%d``, pre-padded with
  space instead of a ``0``.

.. _SERVER-29627: https://jira.mongodb.org/browse/SERVER-29627

An example of this in PHP::

    <?php
    include 'vendor/autoload.php';

    $collection = (new MongoDB\Client)->demo->birthdays;

    $cursor = $collection->aggregate([
        [ '$project' => [
            'date' => [
                '$dateToString' => [
                    'date' => '$dateOfBirth',
                    'format' => '%Y-%m-%d %H:%M:%S %z',
                    'timezone' => 'Australia/Sydney',
                ]
            ]
        ] ]
    ]);

    foreach ($cursor as $person) {
        echo $person->date;
    }

Which outputs::

    1978-12-22 19:15:00 +1100

The $dateFromString Operator
----------------------------

Analogous to PHP's DateTimeImmutable_ constructor, this operator can be used
to create a Date value out of a string. It has the following syntax::

    { "$dateFromString": {
        "dateString": dateString,
        "timezone": timeZoneIdentifier
    } }

The *dateString* could be anything like:

- ``2017-08-04T17:02:51Z``
- ``August 4, 2017 17:10:27.812+0100``

In fact, it will accept everything that PHP's DateTimeImmutable_ constructor
accepts as under the hood, it uses the same library. MongoDB enforces though
that it is an absolute date/time string.

.. _DateTimeImmutable: http://php.net/datetimeimmutable

The ``timezone`` argument is optional, and is interpreted in the same as the
``timezone`` argument in the Date Extraction functions as explained above.

For example::

    { $dateFromString: {
        dateString: "2017-08-04T17:06:41.113",
        timezone: "Europe/London"
    } }

Would mean ``17:06`` local time in London, or ``16:06`` in UTC (as London right now is
at UTC+1).

It is not allowed to specify a time zone through the ``dateString`` (such as the
ending ``Z`` or ``+0400``) and **also** specify a time zone through the
timezone argument. In that case, an exception is thrown.

In PHP, this looks like::

    <?php
    include 'vendor/autoload.php';

    $collection = (new MongoDB\Client)->demo->birthdays;

    $cursor = $collection->aggregate([
        [ '$project' => [
            'date' => [
                '$dateFromString' => [
                    "dateString" => 'August 8th, 2017. 14:14:40',
                    "timezone" => "Europe/Amsterdam",
                ]
            ]
        ] ]
    ]);

    foreach ($cursor as $person) {
        var_dump( $person->date->toDateTime() );
    }

Which outputs::

    class DateTime#12 (3) {
      public $date =>
      string(26) "2017-08-08 12:14:40.000000"
      public $timezone_type =>
      int(1)
      public $timezone =>
      string(6) "+00:00"
    }

As you can see, the time zone information is lost when the data is transferred
between MongoDB and PHP as the BSON DateTime data type does not carry this
information.

Using Date Expressions in $match
--------------------------------

From MongoDB 3.5.12, it is also possible to use the new date expressions (and
other expressions) in the ``$match`` pipeline operator. For example, in order
to find all the documents before ``June 17th, 2017`` in the New York time
zone::

    db.dates.aggregate( [
        { $match: {
            date: { $gte: { $expr: {
                $dateFromString: {
                    dateString: "June 17th, 2017",
                    timezone: "America/New_York"
                }
            } } }
        } }
    ] );

Or from PHP::

    <?php
    include 'vendor/autoload.php';

    $collection = (new MongoDB\Client)->demo->dates;

    $date = "June 17th, 2017";

    $cursor = $collection->aggregate( [
        [ '$match' => [
            'date' => [ '$gte' => [ '$expr' => [
                '$dateFromString' => [
                    "dateString" => [ '$literal' => $date ],
                    "timezone" => "America/New_York",
                ]
            ] ] ]
        ] ]
    ]);

    foreach ($cursor as $person) {
        var_dump( $person->date->toDateTime() );
    }

Please note the use of the `$literal`_ operator here, which should be used for
any user input that might be able to sneak in an expression into the value.

.. _`$literal`: https://docs.mongodb.com/manual/reference/operator/aggregation/literal/

Notes
-----

The time zone support is currently only available in a development release of
MongoDB, and should be considered experimental. It is likely that some of it
will still change. In particular:

- Before MongoDB 3.5.12, the argument ``millisecond`` to ``dateFromParts``
  is incorrectly spelled ``milliseconds``.

- Until SERVER-30547_ is resolved, ``$dateFromParts`` does not accept an
  *sub-document* as argument, and instead requires each single field to be
  specified.

- Until SERVER-30523_ is resolved, the field values to ``dateFromParts``
  can not underflow or overflow their expected range. For example, the ``day``
  field's value needs to be in the range ``1..31`` and the ``hour`` field's
  value needs to be in the range ``0..23``.

.. _SERVER-30547: https://jira.mongodb.org/browse/SERVER-30547
.. _SERVER-30523: https://jira.mongodb.org/browse/SERVER-30523
.. _SERVER-30046: https://jira.mongodb.org/browse/SERVER-30046
