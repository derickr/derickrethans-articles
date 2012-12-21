Distributions: Please Don't Cripple PHP or Red Hat: Stop Fucking Around
=======================================================================

.. articleMetaData::
   :Where: Barcelona, Spain
   :Date: 20090204 1640 CET
   :Tags: blog, php

A friend of me pointed me to an issue in the Agavi issue tracker titled `"Fucking Debian fucking ruined their fucking PHP package once again, and now we need to waste fucking time to fucking fix it)"`_ . The issue is quite
related to earlier discussions with the Red Hat folks about bundling a
timezone database with PHP. Red Hat thought it'd be wise to create a
patch to use the system provided timezone database instead. We (the PHP
development team) thought that to be a bad idea because of several
reasons. Among them is that it removes control from PHP's users about
which database is, decreased performance, and some missing
functionality. On top of that I `mentioned`_ that although the bundled timezone database is a concatenation of the
system provided timezone database files, that might not necessarily be
like that in the future. PHP does of course support means of upgrading
the timezone database. For every release of the `Olson`_ database, there is now a
bundled timezone database available through `PECL`_ .

Because of these reasons, we didn't want to accept Red Hat's patch to
disable the bundled timezone database. The most *important* thing
however for users of PHP that *PHP works the same on every system* . We, the PHP developers, see this as a much more important thing as a
tiny bit of annoyance by distributions' maintainers to have to package
the timezone database for PHP separately. Unfortunately the Red Hat PHP
package maintainer proceeded to deploy this patch in RHEL and Fedora.
SuSE, Debian and Ubuntu have since then picked up on it as well. So
instead the Agavi issue's title should be " *Fucking Red Hat* fucking ruined their fucking PHP package and every other distribution
once again, and now we need to waste fucking time to fucking fix
it)". Unfortunately, this is not where it stops.

But first we go back on how PHP determines which timezone to use and
what a timezone is. For PHP, a timezone is an identifier that describes
UTC offsets and daylight savings times for a specific area, such as
"Europe/Oslo" or "America/New_York". This identifier
provides more information as just a timezone abbreviation such as
"CET" or "EST", because abbreviations are not
unique.

PHP determines which timezone to use with the following algorithm: First
of all, if a call to `date_default_timezone_set()`_ is made, that's the timezone identifier that PHP uses.
Secondly—although this is deprecated behavior—the "TZ"
environment variable is checked. If it contains a valid timezone
identifier, this is used. Thirdly it checks for a php.ini setting
date.timezone. If the value is a valid timezone identifier, this is
used. If neither of the above three methods succeed, PHP issues an
E_STRICT warning and has to *guess* which timezone the current
system is using. Although the guessing algorithm works reasonably well,
it's not 100% accurate. This is also why the E_STRICT warning is thrown.
In PHP 5.3 this has be increased to an E_WARNING type error instead to
be more apparent that you are *required* to set a default timezone
with php.ini's date.timezone setting. That the guessing algorithm isn't
100% correct can be demonstrated by setting your system's timezone to
"Europe/London" and running:

::

	php -n -r "echo @date('e');"

This will output "UTC" instead of the correct value
"Europe/London". Because `people`_ do not set
their date.timezone setting and have E_STRICT turned off, they never a
see the warning. Instead of trying to understand the issue, they file a
bug with their `distribution`_ .
This distribution then adds a `klugde`_ instead of doing the `proper thing`_ . So now RedHat (and other distributions such as RedHat,
Debian and Ubuntu) don't only cause performance issues, they also change
the behavior of PHP. This of course is not making the developer's lives
easier. This is bad for PHP's name and in my opinion a violation of our
license term "3. The name "PHP" must not be used to
endorse or promote products derived from this software without prior
written permission.". I do not have problems with build fixes and
packaging modifications because they are IMO not derivations of PHP. I
do however have problems with people using the name PHP for crippled and
intentionally broken derivations like the ones RedHat and Debian (and
others) bundle. The whole idea behind this license clause is just to
prevent exactly the mess that RedHat thought would be a good thing.

In the near future, when PHP 5.3 comes out, the timezone database that
is bundled with PHP will no longer be compatible with the system version
of the timezone database. I've mentioned that this could happen `before`_ .
This change is made to provide extra functionality, and as a side effect
breaks those stupid crippling patches that RedHat thought was a good
idea!


.. _`"Fucking Debian fucking ruined their fucking PHP package once again, and now we need to waste fucking time to fucking fix it)"`: http://trac.agavi.org/ticket/1008
.. _`mentioned`: http://thread.gmane.org/gmane.comp.php.devel/47609/focus=47681
.. _`Olson`: ftp://elsie.nci.nih.gov/pub/
.. _`PECL`: http://pecl.php.net/package/timezonedb
.. _`date_default_timezone_set()`: http://php.net/date_default_timezone_set
.. _`people`: http://bugs.php.net/bug.php?id=46459
.. _`distribution`: https://bugzilla.redhat.com/show_bug.cgi?id=469532
.. _`klugde`: https://bugzilla.redhat.com/show_bug.cgi?id=469532#c8
.. _`proper thing`: https://bugzilla.redhat.com/show_bug.cgi?id=469532#c6
.. _`before`: http://thread.gmane.org/gmane.comp.php.devel/47609/focus=47681

