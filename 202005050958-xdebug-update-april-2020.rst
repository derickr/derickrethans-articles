Xdebug Update: April 2020
=========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-05-05 09:58 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-20apr

Another monthly update where I explain what happened with Xdebug development
in this past month. These will be published on the first Tuesday after the 5th
of each month. `Patreon <https://www.patreon.com/derickr>`_ supporters will
get it earlier, on the first of each month. You can `become a patron
<https://www.patreon.com/bePatron?u=7864328>`_ to support my work on Xdebug.
If you are leading a team or company, then it is also possible to support
Xdebug through `a subscription <https://xdebug.org/support>`_.

In March, I worked on Xdebug for about 60 hours, on the following things:

Xdebug 2.9.5
------------

The 2.9.5 release addresses a few bugs. One of them was a follow on from the
issue where Xdebug would crash when another extension would run code in PHP's
Request Init stage, but only on a second or later request in the same PHP
process. As this is not something that's easy to catch with PHP's testing
framework that Xdebug uses, this issue slipped through the cracks.

The release fixes another bug, where throwing an exception from within a
destructor would crash Xdebug. The fix for this was merely making sure that
PHP's internal state is still available::

	- if (!(ZEND_CALL_INFO(EG(current_execute_data)) & ZEND_CALL_HAS_SYMBOL_TABLE)) {
	+ if (EG(current_execute_data) && !(ZEND_CALL_INFO(EG(current_execute_data)) & ZEND_CALL_HAS_SYMBOL_TABLE)) {

Beyond these two crashes, the release also addressed an issue where Xdebug did
not always correct catch where executable code could exist for code coverage
analyses. Over the last decade, PHP has been getting more and more optimised,
with more internal engine instructions. Unfortunately that sometimes means
that these are not hooked into by Xdebug, to see whether there could be a line
of code that would make use of these opcodes. As this is often very dependent
on how developers lay out their code, these issues are often found by them. Luckily,
these issues are trivially fixed, as long as I have access to just the file
containing that code. I then analyse it with `vld
<https://derickrethans.nl/projects.html#vld>`_ to see which opcode (PHP engine
instruction) I have missed.

Xdebug 3 and Xdebug Cloud
-------------------------

Most of my time was spend on getting `Xdebug Cloud
<https://cloud.xdebug.com>`_ to a state where I can invite select developers
to alpha test it. This includes allowing for Xdebug to connect to Xdebug
Cloud. There is currently a `branch
<https://github.com/derickr/xdebug/tree/cloud>`_ available, but it still lacks
the addition of SSL encryption, which is a requirement for allowing safe
transport of debug information.

The communications between an IDE and Xdebug through Xdebug Cloud is working,
with a few things related to detecting disconnections more reliably still
outstanding.

As Xdebug Cloud needs integration in debugging clients (such as PhpStorm, and
other IDEs), I have been extending the `dbgpProxy
<https://xdebug.org/docs/dbgpProxy>`_ tool to act as intermediate link between
existing IDEs and Xdebug Cloud without IDEs having to change anything. This
work is still ongoing, and is not documented yet, but I hope to finish that in
the next week. Once that and SSL support in the Xdebug to Xdebug Cloud
communication has been finalized, I will reach out to subscribers of the
`Xdebug Cloud newsletter <https://cloud.xdebug.com/>`_ to see if anybody is
interested in trying it out.


Business Supporter Scheme and Funding
-------------------------------------

In April, no new supporters signed up.

If you, or your company, would also like to support Xdebug, head over to the
`support <https://xdebug.org/support>`_ page!

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page and a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_.

Podcast
-------

The `PHP Internals News <https://phpinternals.news>`_ continues its second
season. Episodes in the last month included a discussion on PHP 8's `JIT
engine and increasing complexity <https://phpinternals.news/48>`_, `PHP's RFC
process <https://phpinternals.news/50>`_, and a discussion with Larry Garfield
about how to improve PHP's `Object Ergonomics
<https://phpinternals.news/51>`_. It is available on `Spotify
<https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB>`_, `iTunes
<https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2>`_,
and through an `RSS Feed <https://phpinternals.news/feed.rss>`_.
