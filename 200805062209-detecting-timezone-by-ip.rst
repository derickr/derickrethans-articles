Detecting Timezone By IP
========================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20080506 2209 CEST
   :Tags: blog, cms, php, datetime

Through `Planet PHP`_ I found an
article on `Pre-populating forms with the timezone`_ . I'd normally add a comment instead, but
the comment would almost be larger then the original post, so I am
instead writing up an entry myself. The post describes several ways to
obtain the user's timezone and use that to pre-fill a form. None of them
are working properly though. I'll try to explain for each of them why
not, but first of all it is important to know what a timezone actually
is.

A timezone is a set of rules that determines the UTC offset for
different times around the year for a *specific* location. Because
of daylight savings time, a specific location can have two (or more)
different UTC offsets depending on what point in time you look at it.
Some areas might have had different rules in the past, even in the same
country. An example here is China, where *currently* there is only
one timezone, but previously there were multiple. Thus there are *different* timezones for the different locations in China, even
though the *current* UTC offset is the same for all of them.
Timezones are identified by Country/City or Country/Subcountry/City.

The offering by `MaxMind`_ allows you to
link a country/region combination to Timezone identifier. For the US it
subdivides this per state even. However, it misses the different
timezones for Russia, Australia and even Indiana, USA.

`IP2Location`_ only provides a single UTC offset per IP range, totally ignoring
daylight savings time.

The `Hostip.info`_ solution I can't access because phpclasses requires some stupid
registration.

As for the author's own solution, using JavaScript, is flawed at least
partly as well. In his example JavaScript only returns a UTC offset.
Luckily it is possible to detect the correct *timezone* quite a bit
better by using some of PHP's functionality. The following bit of code
uses both the UTC offset and the timezone abbreviation to find out the
user's timezone:

::

	<?php
	if (!isset($_GET['tzinfo'])) {
	?>
	<html>
	<script type="text/javascript">
	var d = new Date()
	var tza = d.toLocaleString().split(" ").slice(-1)
	var tzo = d.getTimezoneOffset()
	window.location = window.location + '?tzinfo=' + tza + '|' + tzo
	</script>
	</html>
	<?php
	} else {
	list( $abbr, $offset ) = explode( '|', $_GET['tzinfo']);
	echo timezone_name_from_abbr( $abbr, $offset * -60 );
	}
	?>

This is still not perfect of course, because it would for example give
the same result for most of Europe (Europe/Berlin).

Figuring out the user's timezone *can* be done by using a
high-accuracy database of IPs to lat/longitudes such as `hostip.info`_ and `MaxMind`_ offer, as well as a
mapping from location to timezone. The latter however, is not available
as far as I know. If however a data file that has proper definitions of
the different timezone boundaries exist (mostly, on a per-province level
would be enough), then such a tool can be easily build. I saw that `OpenStreetMap`_ has such a map,
but I can't really find the raw data for that unfortunately. It would
however, be awesome to have such a data file.


.. _`Planet PHP`: http://planet-php.org
.. _`Pre-populating forms with the timezone`: http://torrentialwebdev.com/blog/archives/152-Pre-populating-forms-with-the-timezone.html
.. _`MaxMind`: http://www.maxmind.com/app/city
.. _`IP2Location`: http://www.ip2location.com/ip-country-region-city-latitude-longitude-zipcode-timezone.aspx
.. _`Hostip.info`: http://www.hostip.info/dl/index.html
.. _`OpenStreetMap`: http://www.openstreetmap.org/

