Xdebug Update: February 2020
============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-03-10 09:34 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-20feb

Another month, another monthly update where I explain what happened with
Xdebug development in this past month. It will be published on the first
Tuesday after the 5th of each month. Patreon_ supporters will get it earlier,
on the first of each month. You can become a patron here_ to support my work
on Xdebug. If you are leading a team or company, then it is also possible to
support Xdebug through a subscription_.

.. _Patreon: https://www.patreon.com/derickr
.. _here: https://www.patreon.com/bePatron?u=7864328
.. _subscription: https://xdebug.org/support

In February, I worked on Xdebug for just over 75 hours, on the following things:

PHP 8 Compatibility
-------------------

Although PHP 8 is no where near being released, I am keeping Xdebug's master
branch in line with PHP's master branch. A few features have been added to PHP
8 that ended up breaking some code and tests, and some additions to the PHP
DOM extension required work on upgrading test cases. When PHP 8 comes out, I
am intending to have Xdebug ready to go with it. Hopefully, that will be
Xdebug 3.0.

dbgpClient and dbgpProxy
------------------------

I have now finished work on early-release versions of a DBGp Client
(dbgpClient_) and DBGp Proxy (dbgpProxy_). These tools are written in Go, and
binaries can be downloaded from the `download page`_. Source code is not yet
available, but I am intending to open that at some point in the near future.

.. _dbgpClient: https://xdebug.org/docs/dbgpClient
.. _dbgpProxy: https://xdebug.org/docs/dbgpProxy
.. _`download page`: https://xdebug.org/download#dbgpClient

For Xdebug Cloud I can no longer have the debugging protocol going over the
network unencrypted. I am therefore working at including SSL support in
Xdebug, as well as in the dbgpClient and dbgpProxy tools. The tools already
have support as it was easy to add that in Go, but Xdebug does not yet. I've
done research and selected BearSSL_ to provide this.

.. _BearSSL: https://bearssl.org/

Asynchronous Debugging Support
------------------------------

`Robert Lu <https://www.robberphex.com/>`_ has been working on asynchronous
support in Xdebug's debugging protocol for a while, the `#1016: Support for
pause-execution <https://bugs.xdebug.org/1016>`_ issue. After extensive
testing, review, and reworking of the patch by Robert and I, this patch
finally landed in Xdebug. The asynchronous support allows you to pause scripts
while they are running without a breakpoint to have been hit. This feature
needs support in a debugging client, which actively needs to opt into this
behaviour. The new `dbgpClient` from the previous section supports this.

I will be working with purveyors of IDE to also include "pause-exection" in
their feature set.

Business Supporter Scheme and Funding
-------------------------------------

In February, a few new supporters signed up. They are:

	`Supermetrics <https://supermetrics.com>`_, `Private Packagist
	<https://packagist.com/?utm_source=xdebug&utm_medium=homepage&utm_campaign=sponsorlogo>`_,
	and `CHECK24
	<https://jobs.check24.de/aufgabenbereich/it-web-development?agid=371>`_.

Thanks!

If you, or your company, would also like to support Xdebug, head over to the
support_ page!

.. _`Business Supporter Scheme`: https://derickrethans.nl/xdebug-update-september-2019.html#a_business_supporter_scheme
.. _support: https://xdebug.org/support

Besides business support, I also maintain a Patreon_ page and a profile on
`GitHub sponsors <https://github.com/sponsors/derickr>`_.

Podcast
-------

The `PHP Internals News <https://phpinternals.news>`_ continues its
second season. In this weekly podcast, I discuss in 15-30 minutes, proposed
new features to the PHP language with fellow PHP internals developers. It is
available on Spotify_ and iTunes_, and through an `RSS Feed`_. In the first
episode_ I spoke with Nikita Popov about Preloading and WeakMaps.

.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2
.. _`RSS Feed`: https://phpinternals.news/feed.rss
.. _episode: https://phpinternals.news/38
