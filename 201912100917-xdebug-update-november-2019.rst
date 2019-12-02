Xdebug Update: November 2019
============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2019-12-10 09:17 Europe/London
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

In November, I worked on Xdebug for about 30 hours, on the following things:

Website Redesign
----------------

`Matt Brown`_ contacted me a few months ago, suggesting that I should consider
cleaning up the design and content of Xdebug's website_. He spend countless
hours both redoing my atrocious (and old!) code that powers the website, and
creating a new design for it. During November we put this new design online,
and I upgraded the server to run `PHP 7.4`_ too.
There are still a few rough edges, and there are a few thins I still want to
improve, but I believe that the new design (and code!) are much cleaner.
Thanks Matt!

.. _`Matt Brown`: https://mattbrown.dev/
.. _`PHP 7.4`: https://www.php.net/archive/2019.php#2019-11-28-1
.. _website: https://xdebug.org

Xdebug 3 development
--------------------

I only spent a little time on Xdebug 3 this month, mostly due to travel to
speak at conferences_. I did finished the modularizing of the Xdebug code
base, and now have moved on to cleaning code up and refactoring it even more
to continue to make it more maintainable.

.. _conferences: /talks.html

Beyond that, I have started to remove a few things from Xdebug as well. I
removed the aggregated profiler feature, which was never documented, and
prepared an uncommitted patch to remove the ``xdebug.remote_handler`` setting.
This setting could only ever have one value (``dbgp``), and it seems very
unlikely that in the future Xdebug will support other debugging protocols. The
underlying code for being able to **have** more protocols continues to exist.
This is mainly because it enforces better design and less coupling between the
different parts of Xdebug.

Xdebug 2.8.1 Release
--------------------

I was right to think last month that it would be likely to have to make a bug
fix release. A user commented on Twitter that the code coverage functionality
was drastically slower. In Xdebug 2.8 I changed_ how the coverage functionality
remembers which classes and their methods it had already analysed. In 2.7 and
earlier, it sets a specific flag on the class entry, but that was always a
hack, which stopped working (again) with PHP 7.4. Instead of using that flag,
I now use a hash table to do so.

.. _changed: https://github.com/xdebug/xdebug/commit/ddecc4143

However, I had inadvertently negated the check, so instead of only analysing
classes and their methods on the first visit, Xdebug ended up analysing it
every single time. The fix_ for this was therefore small (and embarrassing).

.. _fix: https://github.com/xdebug/xdebug/commit/abd48292a

During the testing of this new "fix", I noticed that code coverage was still a
lot slower than in Xdebug 2.7.2, so I did some more research to improve this.
Instead of allocating memory to create the hash key, I use stack memory
instead_.

.. _instead: https://github.com/xdebug/xdebug/commit/9547f8378#diff-c23a49b975bde4591084a08c43ee6c45R1051

For Xdebug 3 I have a few further ideas to speed up code coverage.

The bug fix for the performance degradation is the only ticket that made it
into `Xdebug 2.8.1 <https://xdebug.org/updates#x_2_8_0>`_.

*Update:* Xdebug 2.8.1 was released on December 2nd (so not actually in
November).

Business Supporter Scheme and Funding
-------------------------------------

I have had further, but minimal, interest for the `Business Supporter Scheme`_
that I launched in September.

	This month's new supporter is 
	`e3N GmbH <e3N GmbH>`_. Thanks!

If you, or your company, would also like to support Xdebug, head over to the
support_ page!

.. _`Business Supporter Scheme`: https://derickrethans.nl/xdebug-update-september-2019.html#a_business_supporter_scheme
.. _support: https://xdebug.org/support

Besides business support, I also maintain a Patreon_ page and a profile on
`GitHub sponsors <https://github.com/sponsors/derickr>`_.

Podcast
-------

I have continued with the `PHP Internals News <https://phpinternals.news>`_
podcast. In this weekly podcast, I discuss in 15-30 minutes, proposed new
features to the PHP language with fellow PHP internals developers. It is
available on Spotify_ and iTunes_, and through an `RSS Feed`_. Let me know if
you are a listener! The podcast will be on hiatus until early next year.

.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2
.. _`RSS Feed`: https://phpinternals.news/feed.rss
