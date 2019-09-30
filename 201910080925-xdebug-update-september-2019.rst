Xdebug Update: September 2019
=============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2019-10-08 09:25 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-19sep

Another month, another monthly update where I explain what happened with
Xdebug development in this past month. It will be published on the first
Tuesday after the 5th of each month. Patreon_ supporters will get it earlier,
on the first of each month. You can become a patron here_ to support my work
on Xdebug. More supporters, means that I can dedicate more of my time to
improving Xdebug.

.. _Patreon: https://www.patreon.com/derickr
.. _here: https://www.patreon.com/bePatron?u=7864328

In September, I worked on Xdebug for about 30 hours, on the following things:

Bug fixes
---------

I alluded in last month's `report
</xdebug-update-august-2019.html#towards_release_candidate_1>`_ that there
were still a few bugs to fix before I can release Xdebug 2.8.0 RC1. I have now
fixed the main outstanding one that had to do with garbage collection.
Additionally, I addressed two issues related to external extensions that do
things in a "boutique" way (`phalcon <https://phalcon.io/en-us>`_ and
ionCube). I am expecting that the RC1 release will come in the middle of
October.

Xdebug 3 development
--------------------

I started with the development of Xdebug 3, and more specifically, refactoring
its different features into their own separate modules. Xdebug's main features
(code coverage, debugger, profiler, function tracing) all now live in their
own separate directory. At the moment I am refactoring the initialisations of
each of the functions so that each "module" is responsible for its own
initialisation, instead of *everything* being done in one big function for
PHP's 5 life cycle events (process start/stop, request start/stop/post stop).
This work is still ongoing, and not going to be ready for quite some time.

Some of the tedious work—moving the test files into their respective module's
directory—has been finished, but there remains plenty to do. My first
milestone is reorganising all the code so that it is easy to see where things
can be improved. I suspect that will take at least until December.

Videos
------

When I set-up my `Patreon tiers <https://www.patreon.com/bePatron?u=7864328>`_
I included *"this tier will also give you access to podcasts and videos
showing upcoming Xdebug features or coding/problem solving sessions"* with the
Cricket tier and higher. I have now started creating screen casts of my Xdebug
3 work, and made them available to the Cricket and higher tiers. The first one
is also available on Vimeo. I plan to release these screen casts through
`Patreon <https://www.patreon.com/bePatron?u=7864328>`_ continuously, and also
make some of them available for non patrons on Vimeo in the future. In the
near future, I would also like to experiment by doing live session through
Twitch.

I have embedded the first one here too:

.. vimeo::
   :ID: 361512188
   :Width: 800
   :Height: 450
   :Title: Ep 001 — Breaking up the globals

Once Xdebug 3 gets to a usable state and I've redone settings and
functionality, I will also be producing tutorials and training sessions.

A Business Supporter Scheme
---------------------------

Although I have been going on about Patreon for half a year now, it isn't
really quite a solution for companies that want to help out. Patreon doesn't
really do invoices correctly, which might be required for a company to be
able to help to fund my work on Xdebug. Patreon is also mostly focussed on
individual developers, and being able to get "perks", which also doesn't
really match with the needs of companies.

With that in mind, I have launched a `Business Supporter Scheme
<https://xdebug.org/support>`_.

To differentiate this from the "Open Source
side", I've decided that I will only do community support through
`Stack Overflow <https://stackoverflow.com/questions/tagged/xdebug>`_. And
henceforth I will close down my support mailing list—it gets hardly any
traffic anyway. Pro and Business subscribers **will** be able to get support
through email. If a company signs up for a *Business* package, I will also
include them in the list of supporters on a new "transparency" page. In the
future, other perks will also only be made available to *Pro* and/or
*Business* supporters.

I believe that when funding is involved, it is also important to show where
the funding comes from, and how it is being used. On this `page
<https://xdebug.org/log>`_ you can see exactly that information. I will update
this at least monthly and when there is a new supporter.

Let me know if you have any questions or comments about this scheme, or if you
want to sign up for it! You can find the contact details on the `Supporting
Xdebug <https://xdebug.org/support>`_ page.

	This month's new supporters are 
	`Intracto <https://www.intracto.com/nl-be>`_,
	`TYPO3 GmbH <https://typo3.com>`_, and
	`Tideways <https://tideways.com/>`_. Thanks!


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
