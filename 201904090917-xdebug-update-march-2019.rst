Xdebug Update: March 2019
=========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2019-04-09 14:15 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-19mar

This is the first of the monthly update reports in what happened with Xdebug
development in this past month. In these reports I will outline what I've been
working on the month that has passed. It will be published on every second
Tuesday of each month. Patreon_ supporters will get it earlier, on the first
of each month. You can become a patron here_.

.. _Patreon: https://www.patreon.com/derickr
.. _here: https://www.patreon.com/bePatron?u=7864328

In March, I worked on the following things:

CI
--

Xdebug needs testing on many PHP versions, from PHP 7.0.0 to PHP 7.3.3, and
the tip of the PHP 7.0 to 7.4 and master branches. Each patch version can
introduce changes, and so they all need to be tested against. There are also
two different configuration axis: thread-safe vs. non-thread-safe, and 32-bit
vs 64-bit. There are currently 37 versions, and with all 4 combinations, that
makes 148 configurations to test. If I would test this with `Travis CI`_
this would take at least an hour, and with Appveyor_ it would take at least
16 hours. This is clearly not something that I would like to wait on.

.. _`Travis CI`: https://travis-ci.org/xdebug/xdebug
.. _Appveyor: https://ci.appveyor.com/project/derickr/xdebug

So instead I set up a local CI system, where after each commit I can
kick off a script that recompiles and tests all of the configurations. I
test the following three variants for each PHP version: 64-bit and non-thread-safe
(that's the normal one most people use), 64-bit with thread-safe, and 32-bit
with non-thread-safe. My local machine can run 24 tasks in parallel, and the
full run of recompiling and testing takes less than 5 minutes.

The results are imported in a MongoDB_ database that I run on their Atlas_
hosted server, and the results are read and published by the Xdebug website,
where you can see the result_ of the latest 8 runs.

.. _MongoDB: https://mongodb.com
.. _Atlas: https://www.mongodb.com/cloud/atlas
.. _result: https://xdebug.org/ci.php

I still need to add the with-OPcache/without-OPcache access; right now, I
only test with OPcache present and loaded. I would also like to add
configurations of daily versions of the latest PHP-7.4 and master branches to
be able to get PHP API changes, and functional changes before they start
breaking PHP.

Bugs
----

I created stable xdebug_2_7_ branch so that I can work on both bug fixes (in
``xdebug_2_7``) and features (in ``master``). There is currently a persistent
bug that I discovered while analysing `#1646: Excessive error messages for
broken pipes <https://bugs.xdebug.org/view.php?id=1646>`_. I noticed that
although my tests would check for this happening, it would never actually make
sure that there were no messages sent to ``stderr``. When enabling_ this, I
encountered issues in the step-debugger tests where I wouldn't correctly
detach the debugging session. 

These tests are now fixed_, but there is now one that produces an elusive
crash bug that I haven't managed to track down now.

.. _`xdebug_2_7`: https://github.com/xdebug/xdebug/commits/xdebug_2_7
.. _enabling: https://github.com/xdebug/xdebug/pull/462/commits/c48476c328fabeba55c9876eb876f415482bcdda
.. _fixed: https://github.com/xdebug/xdebug/pull/462/commits/f8171ea26cfd7bf4967195ed57f6e5bcefb13f37

Beyond the broken-pipe step debugger connection issue, I have also started to
make a little bit of progress in addressing `#1388: Support 'resolved' flag
for breakpoints <https://bugs.xdebug.org/view.php?id=1388>`_ in cooperation
with Jetbrains_ and the PhpStorm_ team, who are sponsoring this work.

.. _JetBrains: https://www.jetbrains.com/
.. _PhpStorm: https://www.jetbrains.com/phpstorm/

Podcast
-------

The last thing I wanted to draw your attention to is a new podcast that I
started: **PHP Internals News**. This is a weekly-ish 15 minutes-ish long
podcast, where I talk with PHP internals developers about a new RFC,
implementations, and upcoming features. It is available on Spotify_ and
iTunes_, and an `RSS Feed`_.

So far I have spoken with Nikita Popov on the `Saner string to number
comparisons RFC <https://drck.me/pin001-eqp>`_, Anthony Ferrara on his `PHP
Compiler and FFI <https://drck.me/pin002-er8>`_, and Joe Watkins on the
`Abolish Narrow Margins and Weak References RFCs
<https://drck.me/pin003-eri>`_. Why not give it a listen too?

.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2
.. _`RSS Feed`: https://derickrethans.nl/feed-phpinternalsnews.xml
