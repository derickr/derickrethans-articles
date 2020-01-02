Xdebug Update: December 2019
============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-01-07 09:52 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-19nov

Another month, another monthly update where I explain what happened with
Xdebug development in this past month. It will be published on the first
Tuesday after the 5th of each month. Patreon_ supporters will get it earlier,
on the first of each month. You can become a patron here_ to support my work
on Xdebug. If you are leading a team or company, then it is also possible to
support Xdebug through a subscription_.

.. _Patreon: https://www.patreon.com/derickr
.. _here: https://www.patreon.com/bePatron?u=7864328
.. _subscription: https://xdebug.org/support

In December, I worked on Xdebug for near 50 hours, on the following things:

Xdebug 2.9.0
------------

After releasing Xdebug 2.8.1, which I mentioned in last `month's`_ update, at
the start of the month, more users noticed that although I had improved code
coverage speed compared to Xdebug 2.8.0, it was still annoyingly slow.
`Nikita Popov`_, one of the PHP developers, provided me with a new idea on how
to approach trying to find out which classes and functions still had to be
analysed. He mentioned that classes and functions are always added to the end
of the class/function tables, and that they are never removed either. This
resulted in a patch, where the algorithm to find out whether a class/function
still needs to be analysed changed from from O(n²) to approximately O(n). You
can read more about this in the `article <https://drck.me/ccc-f1p>`_ that I
wrote about it. A few other issues were addressed in `Xdebug 2.9.0
<https://xdebug.org/announcements/2019-12-09>`_ as well.

.. _`month's`: https://derickrethans.nl/xdebug-update-november-2019.html
.. _`Nikita Popov`: https://github.com/nikic/

Breakpoint Resolving
--------------------

In the May update I wrote about `resolving breakpoints
<https://derickrethans.nl/xdebug-update-may-2019.html#resolving_breakpoints>`_.
This feature will try to make sure that whenever you set a breakpoint, Xdebug
makes sure that it also breaks. However, there are currently two issues with
this: 1. breaks happen more often than expected, and 2. the algorithm to find
lines is really slow. I am addressing both these problems by using a similar
trick to the one Nikita suggested for speeding up code coverage analysis. This
work requires quite a bit of rewrites of the breakpoint resolving function,
and hence this is ongoing. I expect this to cumulate in an Xdebug 2.9.1
release during January.

debugclient and DBGp Proxy
--------------------------

I have wanted to learn Go_ for a while, and in order to get my feet wet I
started implementing Xdebug's bundled debugclient_ in Go, and at the same time
create a library to handle the DBGp protocol.

.. _Go: https://golang.org
.. _debugclient: https://github.com/xdebug/xdebug/tree/f2b1f40d126c59d783155bd79b2d1bfe4fc4e742/debugclient

The main reason why a rewrite is useful, is that the debugclient as bundled
with Xdebug no longer seems to work with ``libedit`` any more. This makes
using debugclient really annoying, as I can't simply use the *up* and *down*
arrows to scroll through my command history. I primarily use the debugclient
to test the DBGp protocol, without an IDE "in the way".

The reason to write a DGBp *library* is that there are several implementations
of a `DBGp proxy`_. It is unclear as to whether they actually implement the
protocol, or just do something that "works". I will try to make the DBGp proxy
that I will be working on stick to the protocol exactly, which might require
changes to IDEs who implement it against an non compliant one (Komodo's
pydbgpproxy seems to be one of these).

.. _`DBGp proxy`: https://xdebug.org/docs/dbgp#just-in-time-debugging-and-debugger-proxies

This code is currently not yet open source, mostly because I am still finding
my feet with Go. I expect to release parts of this on the way to Xdebug 3.0.

Business Supporter Scheme and Funding
-------------------------------------

Support through the `Business Supporter Scheme`_ continues to trickle in.

	This month's new supporter is `Strategery <https://strategery.io/>`_.
	Thanks!

If you, or your company, would also like to support Xdebug, head over to the
support_ page!

.. _`Business Supporter Scheme`: https://derickrethans.nl/xdebug-update-september-2019.html#a_business_supporter_scheme
.. _support: https://xdebug.org/support

Besides business support, I also maintain a Patreon_ page and a profile on
`GitHub sponsors <https://github.com/sponsors/derickr>`_.

Podcast
-------

The `PHP Internals News <https://phpinternals.news>`_ has been on hiatus since
the PHP 7.4 release, but it will return in January.

.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2
.. _`RSS Feed`: https://phpinternals.news/feed.rss
