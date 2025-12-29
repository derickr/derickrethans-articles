Sponsoring Xdebug
=================

.. articleMetaData::
   :Where: London, UK
   :Date: 2011-03-22 18:30 Europe/London
   :Tags: blog, php, xdebug
   :Short: sponsor-xdebug

Over the past 7 years I've spend countless hours making Xdebug_ an awesome
development and debugging tool. I love working on it; it is a good
way to get familiar with PHP's "interesting" internals, and it also helps
PHP developers finding (potential) issues within their code base faster.

With the 2.0 release announcement I asked_ users of Xdebug to send me a
postcard if they liked it; and since the 2.1 announcement I mention_ that
it is possible to give donations_ through PayPal. I have received quite
few postcards and donations.

Recently, an issue_ with KCacheGrind_ and Xdebug's profiler_ functionality
became known. As an experiment_ Sebastian suggested to see whether it would be
a good idea to set-up a pledge-like system to arrange some funding so that I
can dedicate my "work time" to working on Xdebug issues and features.  I set-up
a campaign on Pledgie (now defunct) for this purpose. Both Sebastian and I expected Pledgie
to work a bit different than it actually did. Instead of holding on to the
pledged money until the issue was implemented, it transferred the pledges
directly to my PayPal account. 

Twenty-four pledges were made, in about two weeks; matching the goal. Thanks
Sebastian, Jan, Michael, `Pale Purple`_, Jeff, Christoph, Karel, Yannick, Jake,
Venakis, Brian, Simon, Kenneth, DM Baker, "gizmola", Ladislav, Volker, React_,
Michal and three anonymous supporters!

Last night I committed_ several patches to the Xdebug repository_, including
the one that sparked the idea to set-up a pledge to fund Xdebug's profiler
files. These patches, and a few others, now form part of the upcoming Xdebug
2.1.1 release. I've published the source_ package for the first (and most
likely only) release candidate: `Xdebug 2.1.1RC1`_. Windows binaries will
follow shortly. You can also install/upgrade the release candidate through
PECL: ``pecl install xdebug-beta`` or ``pecl upgrade xdebug-beta``.

The Pledgie campaign has now been closed, but I am intending to set-up new
ones for more of the elaborate features—most likely starting with one related
to Xdebug's `code coverage`_ functionality. In the meanwhile, please test
`Xdebug 2.1.1RC1`_, and let me know your comments about sponsoring specific
Xdebug features. I plan on releasing to write up some more thoughts about that
soon. I expect Xdebug 2.1.1 to come out before the end of the month. If
you think Xdebug is useful, feel free to donate_ as well!

.. _Xdebug: http://xdebug.org
.. _asked: http://derickrethans.nl/xdebug-2-released.html
.. _mention: https://drck.me/xdebug-2.1-7x2
.. _donations: http://xdebug.org/donate.php
.. _issue: https://bugs.kde.org/show_bug.cgi?id=256425#c10
.. _KCacheGrind: http://kcachegrind.sourceforge.net/html/Home.html
.. _profiler: http://xdebug.org/docs/profiler
.. _experiment: http://sebastian-bergmann.de/archives/909-On-Sponsored-Open-Source-Development.html
.. _`Pale Purple`: http://www.palepurple.co.uk/
.. _`React`: http://www.react.nl/
.. _committed: http://xdebug.org/archives/xdebug-dev/1740.html
.. _repository: http://svn.xdebug.org/cgi-bin/viewvc.cgi/xdebug/?root=xdebug
.. _source: http://xdebug.org/download.php#releases
.. _`Xdebug 2.1.1RC1`: http://xdebug.org/updates.php#x_2_1_0rc1
.. _`code coverage`: http://xdebug.org/docs/code_coverage
.. _donate: http://xdebug.org/donate.php
