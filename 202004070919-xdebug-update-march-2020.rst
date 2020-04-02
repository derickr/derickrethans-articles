Xdebug Update: March 2020
============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-04-07 09:19 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-20mar

Another month, another monthly update where I explain what happened with
Xdebug development in this past month. It will be published on the first
Tuesday after the 5th of each month. Patreon_ supporters will get it earlier,
on the first of each month. You can become a patron here_ to support my work
on Xdebug. If you are leading a team or company, then it is also possible to
support Xdebug through a subscription_.

.. _Patreon: https://www.patreon.com/derickr
.. _here: https://www.patreon.com/bePatron?u=7864328
.. _subscription: https://xdebug.org/support

In March, I worked on Xdebug for about 75 hours, on the following things:

Xdebug 2.9.3 and 2.9.4
----------------------

The last month saw two releases. In Xdebug 2.9.3 I fixed an `issue
<https://bugs.xdebug.org/1753>`_ with breakpoint resolving. In files with a
class that inherits from another class, the line start/end information from
the *inherited* methods were incorrectly added to the lines map for the file
with the extending class. This caused Xdebug to stop at confusing lines in
some cases.

Xdebug overloads PHP's internal error handler. As the hooks in the PHP engine
aren't great, Xdebug reimplements most of this. This code is liable for
getting out of sync with how PHP itself handles errors. In Xdebug 2.9.3 I
fixed such an `issue <https://bugs.xdebug.org/1758>`_, where a behavioural
change in PHP 7.2 was not propagated to Xdebug's reimplementation of the error
handler.

Through a discussion with other PHP contributors I found out that Xdebug's way
of handling the overriding of opcodes (PHP Engine's "instructions") was not
optimal. Other extensions also overload opcodes, such as Nikita's `scalar objects
<https://github.com/nikic/scalar_objects>`_, or Xinchen's `taint
<https://github.com/laruence/taint>`_. When Xdebug and one of these other
opcode-overloading extensions are loaded at the same time, none of them would
check whether they were also overloaded by another extension. In Xdebug 2.9.3
I fixed that, and this is now also resolved in `taint
<https://github.com/laruence/taint/commit/0edecddad8bd41bb86a7dfa4fdbebb7cbb1cc911>`_,
although the issue for `scalar objects
<https://github.com/nikic/scalar_objects/issues/40>`_ is still open.

Unfortunately this fixed introduced a `crash <https://bugs.xdebug.org/1763>`_
for thread safe builds of PHP. I quickly released `Xdebug 2.9.4
<https://xdebug.org/announcements/2020-03-23>`_ to rectify this problem after
a number of reports.

Last month I mentioned that I merged a patch for `Asynchronous Debugging
Support
<https://derickrethans.nl/xdebug-update-february-2020.html#asynchronous_debugging_support>`_
into Xdebug's master branch (which will become Xdebug 3.0). While doing some
more work on this, in particularly towards making it less of a performance
impact, I found a bug that was present in Xdebug for a long time: When an IDE
uses the `detach <https://xdebug.org/docs/dbgp#continuation-commands>`_
command, Xdebug would disable the remote debugger for the entire life time of
the PHP process in use. This potentially explains lots of weird situations
where debugger suddenly stopped working. This bug is also fixed in Xdebug
2.9.4.

Xdebug 3
--------

I've been continuing to work on little improvements for Xdebug 3, such as
adding Units to profiler output's categories. This is a feature that was
thought up when I did an Xdebug workshop last year at CHECK24. I expect to
start with the refactoring of `php.ini` settings in April, as I am Staying
Safe at Home, and pretty much have nothing else going on.

I've also continue to improve the dbgpClient and dbgpProxy tools, and made
further progress on `Xdebug Cloud <https://cloud.xdebug.com>`_.


Business Supporter Scheme and Funding
-------------------------------------

In March, no new supporters signed up.

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
available on Spotify_ and iTunes_, and through an `RSS Feed`_.

.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2
.. _`RSS Feed`: https://phpinternals.news/feed.rss
.. _episode: https://phpinternals.news/38
