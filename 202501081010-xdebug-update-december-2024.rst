Xdebug Update: December 2024
============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2025-01-08 10:10 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-24dec

In this monthly update I explain what happened with Xdebug development.

`GitHub <https://github.com/sponsors/derickr/>`_ and `Pro/Business supporters
<https://xdebug.org/support>`_ will get it earlier, around the first of each
month.

In the last month, I spend around 8 hours on Xdebug, with 22 hours funded.

Xdebug 3.4.1
------------

When the 3.4.0 release was published, users found a few issues where Xdebug
would crash when it was deciding whether any feature needed to be activated
through a trigger.

In December, I spend a lot of time tracking this down, and made several
attempts at a fix. In the end, the real cause ended up being a problem where
applications had created a reference to a super-global. Xdebug did not
correctly dereference this special internal PHP variable information.

Over the holiday period, several users tested out this fix and confirmed that
it indeed worked. I didn't manage to release Xdebug 3.4.1 before the year was
over, but will come early in January.

Xdebug Videos
-------------

I have created no new Xdebug videos in the last month.

All Xdebug videos can be watched on my `channel
<https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_.

If you have any suggestions, feel free to reach out to
`me on Mastodon <https://phpc.social/@derickr>`_ or via `email
<https://derickrethans.nl/who.html>`_.

Business Supporter Scheme and Funding
-------------------------------------

On GitHub sponsors, I am currently 40% towards my $2,500 per month goal, which
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
the `mailinglist <https://xdebug.cloud/newsletter>`_, which I will use
to send out an update not more than once a month.
