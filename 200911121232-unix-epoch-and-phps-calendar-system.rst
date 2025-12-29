Unix Epoch and PHP's calendar system
====================================

.. articleMetaData::
   :Where: London, UK
   :Date: 20091112 1232 GMT
   :Tags: blog, php, work, datetime

I just saw a `commit`_ flying past as
a response to `PHP bug #50155`_. While *right* now it is proper to define the Unix
Epoch at "1970-01-01 00:00:00 UTC", UTC wasn't actually
defined until 1972. So it would be more correct to define the Unix Epoch
as "the number of seconds elapsed since midnight `proleptic`_ Coordinated Universal Time (UTC) of January 1, 1970, not counting leap
seconds." (from Wikipedia). What the bit "not counting leap
seconds" means, I've already explained before in `Leap Seconds and What To Do With Them`_.

Similarly, PHP's internal calendar is the `ISO 8601 calendar`_. This is a modification of the proleptic `Gregorian`_ calendar. The Gregorian calendar implements the current set of leap
years every 4 years, but not every 100 years, but again every 400 years
(to get to an average year length of 365.2425 days). Obviously this
calendar is only in use since 1582 (some countries adopted it as late as
the 1900s), so using days like 1066-10-14 in the Gregorian calendar
makes little sense because that calendar didn't exist back then. Now,
PHP's ISO 8601-based calendar even modifies the Gregorian calendar by
including the year 0. The Gregorian calendar goes straight from -1 to 1
which is a pain to do proper date calculations with. Therefore the ISO
8601 calendar uses `Astronomical year numbering`_.


.. _`commit`: http://news.php.net/php.doc.cvs/5325
.. _`PHP bug #50155`: http://bugs.php.net/bug.php?edit=1&id=50155
.. _`proleptic`: http://dictionary.reference.com/browse/proleptic
.. _`Leap Seconds and What To Do With Them`: /leap-seconds-and-what-to-do-with-them.html
.. _`ISO 8601 calendar`: http://en.wikipedia.org/wiki/ISO_8601#Years
.. _`Gregorian`: http://en.wikipedia.org/wiki/Gregorian_calendar
.. _`Astronomical year numbering`: http://en.wikipedia.org/wiki/Astronomical_year_numbering

