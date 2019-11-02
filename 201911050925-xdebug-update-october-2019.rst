Xdebug Update: October 2019
===========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2019-11-05 09:25 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-19oct

Another month, another monthly update where I explain what happened with
Xdebug development in this past month. It will be published on the first
Tuesday after the 5th of each month. Patreon_ supporters will get it earlier,
on the first of each month. You can become a patron here_ to support my work
on Xdebug. More supporters, means that I can dedicate more of my time to
improving Xdebug.

.. _Patreon: https://www.patreon.com/derickr
.. _here: https://www.patreon.com/bePatron?u=7864328

In October, I worked on Xdebug for about 35 hours, on the following things:

Xdebug 2.8.0 Release
--------------------

With all the outstanding tickets for `2.8.0dev` out of the way, and the
release of PHP 7.4 coming at the end of next month, I felt it was right to
release `Xdebug 2.8.0 <https://xdebug.org/updates#x_2_8_0>`_ on Halloween,
yay! 👻 

I did not feel the need to make a Release Candidate, as my experience tells me
that nobody tests these anyway. I therefore expect to have to make a bug fix
release (2.8.1) not too long after PHP 7.4 comes out, as I suspect that some
minor bugs and/or inconsistencies with PHP 7.4 will come to light.

Xdebug 3 development
--------------------

I have continued with the development of Xdebug 3, and am now about 70% done
splitting out the different features into more separated modules. When that is
done, I can begin to work on improving the initialisation of each module to
make sure only the least amount of work is done by Xdebug. During this phase,
I expect that some configurations settings are going to change, to prepare
the way for having to put Xdebug into a specific mode. However, I do not think
that I will start working on this before December, as November is full with
travel to conferences. Come and say hi if you're at any of `them
<https://derickrethans.nl/talks.html>`_!

Business Supporter Scheme and Funding
-------------------------------------

I have had minimal interest for the Business Supporter Scheme that I launched
last month. I am a little bit disappointed in this, as I believe that in order
to be able to continue to work on Xdebug (instead of having to take a *real*
job), the support from many businesses is really important.

Having said that, during the month, both JetBrains (purveyors of fine IDEs
such as PhpStorm) and Automattic (purveyors of WordPress.com) have agreed to
fund part of my work on Xdebug. JetBrains has agreed to do this for three
months, and their funding is reflected in my
`log <https://xdebug.org/log>`_. Automattic agree to fund part of my work for
a year, and this will show up from next month's log. Thanks!

	This month's new supporters are 
	`JetBrains <https://jetbrains.com>`_, and
	`Automattic <https://automattic.com>`_. Thanks!

Besides business support, and in addition to Patreon_, I was also accepted for
`GitHub sponsors <https://github.com/sponsors/derickr>`_. The functionality
that they provide is not up to par with Patreon, so I will continue to use the
latter for exclusive treats like videos and earlier access to information
(like this report). However, if you are currently considering chipping in,
then GitHub sponsors provides another way of doing so. As they are not paying
out anything until after 3 months, this source does not yet feature in the
`log <https://xdebug.org/log>`_.

Podcast
-------

I have been continuing with the `PHP Internals News
<https://phpinternals.news>`_ podcast. In this weekly podcast, I discuss in
15-30 minutes, proposed new features to the PHP language with fellow PHP
internals developers. It is available on Spotify_ and iTunes_, and through an
`RSS Feed`_. Let me know if you are a listener!

.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2
.. _`RSS Feed`: https://phpinternals.news/feed.rss
