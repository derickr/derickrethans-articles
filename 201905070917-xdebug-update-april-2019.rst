Xdebug Update: April 2019
=========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2019-05-07 09:17 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-19apr

This is another of the monthly update reports in which I explain what happened
with Xdebug development in this past month. It will be published on the first
Tuesday after the 5th of each month. Patreon_ supporters will get it earlier,
on the first of each month. You can become a patron here_ to support my work
on Xdebug. More supporters, means that I can dedicate more of my time to
improving Xdebug.

.. _Patreon: https://www.patreon.com/derickr
.. _here: https://www.patreon.com/bePatron?u=7864328

In April, I worked on the following things:

2.7.1 Release
-------------

The `2.7.1 release <https://xdebug.org/#2019_04_05>`_ came just at the start
of the month, and addressed three bugs:

- Issue `#1641 <https://bugs.xdebug.org/1641>`_: Performance degradation with
  getpid syscall, contributed by `Kees Hoekzema
  <https://github.com/keeshoekzema>`_.
- Issue `#1646 <https://bugs.xdebug.org/1646>`_: Missing newline in error
  message.
- Issue `#1647 <https://bugs.xdebug.org/1647>`_: Memory corruption when a
  conditional breakpoint is used.

The second bug in the list was more than just a missing newline. Xdebug's
handling of connections that were aborted by the IDE was not as good as they
could be. For this to be tested, I updated the test harness for remote
debugger tests to be able to test for ``stdout`` and ``stderr`` as well.

I would recommend that everybody to update to this version as the last of
these bugs can cause PHP with Xdebug to crash.

Bugfixes
--------

Since the Xdebug 2.7.1 release I have worked on a few outstanding bugs that
did not make the earlier 2.7 releases. 

The remote step debugger allows you to change a variable's value through an
IDE. Previously, this would use PHP's internal ``eval`` functionality to set
the value, except in the cases where the IDE would also associate a new type
with the value. I finally switched all code paths of setting new values to use
``eval``. Due to changes introduced in PHP 7, this is now the only reliable
way of doing this.

The second issue (`#1656 <https://bugs.xdebug.org/1656>`_) fixes an issue with
the step debugger's connect-back functionality. This functionality looks at
HTTP headers to find out which IP address to connect to. In some cases, a
proxy might inject a second IP address, and Xdebug could not handle this. I
improved upon a prototype patch by `Florian Dorn
<https://gist.github.com/derflocki>`_ to make Xdebug now pick the first IP
address in the header that it finds.

Through the implementation of issue (`#1615 <https://bugs.xdebug.org/1615>`_),
Xdebug now will turn off Zend OPcache's optimiser when the step debugger is
active. The optimiser is really good making crappy code run fast, but it also
optimises things away that normal users wouldn't expect, and that then
confuses developers single-stepping through their code. Which brings me onto
the last point of this update:

Resolving Breakpoints
---------------------

In last month's update_ I introduced a new feature that Jetbrains_ is
sponsoring. This issue, `#1388: Support 'resolved' flag
for breakpoints <https://bugs.xdebug.org/view.php?id=1388>`_, is now
implemented pending additional testing with a fresh (and experimental) build
of PhpStorm_. As part of the testing I'm interested in knowing how many
breakpoints are usually in use in PhpStorm projects. I would be interested in
knowing how many breakpoints, and which types of breakpoints (location,
exception), your project has defined. Please leave a note in the comment
section if you care to help me out with this.

.. _JetBrains: https://www.jetbrains.com/
.. _PhpStorm: https://www.jetbrains.com/phpstorm/
.. _update: /xdebug-update-march-2019.html

Podcast
-------

I have also been continuing with the `PHP Internals News
<https://phpinternals.news>`_ podcast. This is a weekly podcast where in 15-30
minutes I discuss new proposed features to the PHP language with fellow PHP
internals developers. It is available on Spotify_ and iTunes_, and through an
`RSS Feed`_. Let me know if you are a listener!

.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2
.. _`RSS Feed`: https://derickrethans.nl/feed-phpinternalsnews.xml
