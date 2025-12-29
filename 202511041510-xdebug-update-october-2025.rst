Xdebug Update: October 2025
===========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2025-11-04 15:10 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-25oct

In this update I explain what happened with Xdebug development in the last two
months.

`GitHub <https://github.com/sponsors/derickr/>`_ and `Pro/Business supporters
<https://xdebug.org/support>`_ will get it earlier, around the first of each
month.

In the last months, I spend around 35 hours on Xdebug, with 44 hours funded.

Xdebug 3.4
----------

In the last installment I wrote about a problem with Lazy Objects. Since then,
a user managed to provide a `short reproducible case
<https://bugs.xdebug.org/2375>`_ which illustrates the problem. With that, I
managed to fix this `specific bug <https://bugs.xdebug.org/2375>`_ and a
`related one <https://bugs.xdebug.org/2371>`_.

These bug fixes also made it into a new release, `Xdebug 3.4.7
<https://xdebug.org/announcements/2025-10-26>`_.

After this was released, a user reported two more bugs (`#2376
<https://bugs.xdebug.org/2376>`_, `#2377 <https://bugs.xdebug.org/2377>`_)
with regards to Lazy Objects, so I will have to do more work here for a next
bug fix release.


PHP 8.5
-------

Most of the time this month I spent on making Xdebug PHP 8.5 ready, and
finalising Native Path Mapping.

I have added PHP 8.5 to the Windows CI test runners, but I am still running
into some issue where `some tests don't seem to work
<https://github.com/xdebug/xdebug/actions/runs/18719433952/job/53387513527?pr=1040>`_.
I haven't quite figured out why these suddenly fail with PHP 8.5, but not with
8.4.

I did `fix a bug <https://bugs.xdebug.org/2372>`_ with regards to changes
in PHP 8.5's glob feature. With PHP 8.5 being released soon, I hope to have
this sorted before that release happens.

Native Path Mapping
-------------------

The Xdebug 3.5.0alpha2 release, that came out at the beginning of October, has
the first preview for PHP 8.5, with Native Path Mapping support. Documentation
is still missing, as well as a minor issue with mapping on Windows, due to its
different path separator (``/`` vs ``\\``).

Beyond that issue, the native path mapping's basic functionality seems to
work.

There are a few minor things to do when looking at the `project plan
<https://xdebug.org/funding/001-native-path-mapping>`_, but these might have
to wait for Xdebug 3.5.1 (or later).

Xdebug Videos
-------------

I have created no new videos in the last months.

All Xdebug videos can be watched on my `channel
<https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_.

If you have any suggestions, feel free to reach out to
`me on Mastodon <https://phpc.social/@derickr>`_ or via `email
<https://derickrethans.nl/who.html>`_.

Business Supporter Scheme and Funding
-------------------------------------

On GitHub sponsors, I am currently 41% towards my $2,500 per month goal, which
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
