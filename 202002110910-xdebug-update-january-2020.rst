Xdebug Update: January 2020
===========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-02-11 09:10 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-20jan

Another month, another monthly update where I explain what happened with
Xdebug development in this past month. It will be published on the first
Tuesday after the 5th of each month. Patreon_ supporters will get it earlier,
on the first of each month. You can become a patron here_ to support my work
on Xdebug. If you are leading a team or company, then it is also possible to
support Xdebug through a subscription_.

.. _Patreon: https://www.patreon.com/derickr
.. _here: https://www.patreon.com/bePatron?u=7864328
.. _subscription: https://xdebug.org/support

In January, I worked on Xdebug for just over 90 hours, on the following things:

Xdebug 2.9.1 and Xdebug 2.9.2
-----------------------------

This month brought two releases. The Xdebug `2.9.1 release`_ restores step
debugging performance as it was in Xdebug 2.7.2, while still maintaining the
`resolved breakpoint`_ feature. Beyond improved performance for debugging, it
also addresses a whole range of smaller issues.

.. _`2.9.1 release`: https://xdebug.org/announcements/2020-01-16
.. _`resolved breakpoint`: https://derickrethans.nl/breakpoints.html

The `2.9.2 release`_ is a normal bug fix release, which addresses three
relatively minor issue.

.. _`2.9.2 release`: https://xdebug.org/announcements/2020-01-31

With the latest 2.9.2 I also updated the Xdebug web site to have a `latest
release download page`_ and a `historical release download page`_. On top of
that, the code behind the web site no longer computers SHA256 checksums of the
download files, but instead they are now `committed to GIT`_. The
separation makes the general download page much leaner, which allows me to put
other related downloads on that same page too. Which brings me to the next
topic.

.. _`latest release download page`: https://xdebug.org/download
.. _`historical release download page`: https://xdebug.org/download/historical
.. _`committed to GIT`: https://github.com/xdebug/xdebug.org/blob/master/html/files/xdebug-2.9.2.tgz.sha256.txt

debugclient and DBGp Proxy
--------------------------

I have continued working on a command line debugging client in Go, and am
making more progress for a DBGp proxy implementation as well. The proxy
currently knows how to register IDEs with their IDE key, and can accept
connections from debugging engines. It does not yet know how to route these
incoming connections to IDEs yet.

However, I have released the command line debugging client "debugclient" for
Linux, macOS and Windows. You can find downloads on the `restructured
downloads`_ page. At the moment the debugging client is not yet Open Source,
but I am intending to do that once I've cleaned things up and know Go well
enough to not be embarrassed by it.

.. _`restructured downloads`: https://xdebug.org/download#debugclient

I have written some basic documentation_ for the debugclient. More extensive,
but harder to read, documentation is available in the description of the `DBGp
protocol`_. You can find a "teaser GIF" below:

.. _documentation: https://xdebug.org/docs/debugclient
.. _`DBGp protocol`: https://xdebug.org/docs/dbgp

.. image:: /images/content/debugclient.gif

Business Supporter Scheme and Funding
-------------------------------------

I have moved the supporters in the `Business Supporter Scheme`_ to a more
prominent place, right on the front page of https://xdebug.org

In January, no new supporters signed up.

If you, or your company, would also like to support Xdebug, head over to the
support_ page!

.. _`Business Supporter Scheme`: https://derickrethans.nl/xdebug-update-september-2019.html#a_business_supporter_scheme
.. _support: https://xdebug.org/support

Besides business support, I also maintain a Patreon_ page and a profile on
`GitHub sponsors <https://github.com/sponsors/derickr>`_.

Podcast
-------

The `PHP Internals News <https://phpinternals.news>`_ has returned with the
second season.  In this weekly podcast, I discuss in 15-30 minutes, proposed
new features to the PHP language with fellow PHP internals developers. It is
available on Spotify_ and iTunes_, and through an `RSS Feed`_. In the first
episode_ I spoke with Nikita Popov about Preloading and WeakMaps.

.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2
.. _`RSS Feed`: https://phpinternals.news/feed.rss
.. _episode: https://phpinternals.news/38
