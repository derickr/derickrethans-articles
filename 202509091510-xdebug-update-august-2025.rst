Xdebug Update: August 2025
==========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2025-09-09 15:10 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-25aug

In this monthly update I explain what happened with Xdebug development.

`GitHub <https://github.com/sponsors/derickr/>`_ and `Pro/Business supporters
<https://xdebug.org/support>`_ will get it earlier, around the first of each
month.

In the last month, I spend around 27 hours on Xdebug, with 29 hours funded.

Xdebug 3.4
----------

I spend a fair amount of time trying to triage `bug #2359
<https://bugs.xdebug.org/2359>`_ where Xdebug is interfering with Lazy
Objects, which were introduced in PHP 8.4. However, there is no small
reproducible case, and I have not managed to reproduce this myself at all.

However, I did manage to get to the bottom of `bug #2328
<https://bugs.xdebug.org/2328>`_. This ended up being reference counting in
PHP and resources (such as open file pointers) not being quite compatible.
Instead of holding on to these resources when I keep the stack traces when an
Exception occurs, I now instead ignore these.

These resource types are `being phased out in PHP
<https://github.com/php/php-tasks/issues/6>`_, as they are the source of many
other issues as well. But, file and stream resources have not been ported yet.

I also fixed a `bug <https://bugs.xdebug.org/2360>`_ where sometimes internal
PHP objects (such as ``DateTimeInterval``) would cause a crash when debugging.

I will make a release in early September to get these fixes out.

PHP 8.5
-------

Most of the time this month I spent on making Xdebug PHP 8.5 ready.

Last month
I wrote about a `patch for PHP <https://github.com/php/php-src/pull/19377>`_
that I created to introduce intermediate steps between pipe stages. With this,
`I discovered a parser issue with PHP's implementation of pipes
<https://news-web.php.net/php.internals/128473>`_, and especially
when closures were used. Due to the precedence order, something went awry.

This has now been `addressed in PHP
<https://github.com/php/php-src/pull/19533>`_, and my patch to introduce
intermediate steps has also been merged after adjusting it slightly. I have
also merged the Xdebug side of this feature into to the master branch.

The rest of the PHP 8.5 work was mostly due to Opcache now always being
enabled. This caused some churn in my test cases and CI workflows, where I
would load ``opcache.so`` conditionally. With PHP 8.5 this library no longer
exists as Opcache is now built-in.

Native Path Mapping
-------------------

I continued investigating how to implement "skip" for native path mapping, and
am planning to get this implemented in early September. It is a fair amount of
work, as I need to adjust my data structures and parser rules.

Xdebug Videos
-------------

I have created no new videos in the last months.

All Xdebug videos can be watched on my `channel
<https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_.

If you have any suggestions, feel free to reach out to
`me on Mastodon <https://phpc.social/@derickr>`_ or via `email
<http://derickrethans/who.html>`_.

Business Supporter Scheme and Funding
-------------------------------------

On GitHub sponsors, I am currently 42% towards my $2,500 per month goal, which
is set to allow continued **maintenance** of Xdebug.

If you are leading a team or company, then it is also possible to
support Xdebug through `a subscription <https://xdebug.org/support>`_.

In the last month, no new business supporters signed up.

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page, a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_, as well as an `OpenCollective
<https://opencollective.com/xdebug>`_ organisation.

If you want to contribute to specific projects, you can find those on the
`Projects <https://xdebug.org/funding>`_ page.

Xdebug Cloud
------------

`Xdebug Cloud <https://xdebug.cloud>`_ is the *Proxy As A Service* platform to
allow for debugging in more scenarios, where it is hard, or impossible, to
have Xdebug make a connection to the IDE. It is continuing to operate as Beta
release.

Packages start at £49/month, and I have recently introduced a package
for larger companies. This has a larger initial set of tokens, and
discounted extra tokens.

If you want to be kept up to date with Xdebug Cloud, please sign up to
the `mailing list <https://xdebug.cloud/newsletter>`_, which I will use
to send out an update not more than once a month.
