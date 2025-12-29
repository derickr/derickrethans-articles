Xdebug Update: January 2025
===========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2025-02-09 10:10 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-25jan

In this monthly update I explain what happened with Xdebug development.

`GitHub <https://github.com/sponsors/derickr/>`_ and `Pro/Business supporters
<https://xdebug.org/support>`_ will get it earlier, around the first of each
month.

In the last month, I spend around 18 hours on Xdebug, with 25 hours funded.

Xdebug 3.4
----------

At the start of the month I released Xdebug 3.4.1, which fixes a few crash
bugs when deciding whether to activate a specific feature.

During the rest of the month, I fixed an issue where PHP 8.4's new property
hooks would `not show their content
<https://bugs.xdebug.org/view.php?id=2314>`_ in the debugger.

Foreach Woes
------------

Besides the Xdebug 3.4 general bug fixes, I have also been looking at
addressing a long-standing issue where using ``foreach`` produces some
unexpected results when doing path and branch coverage. I wrote an article
called `Figuring Out Foreach
<https://derickrethans.nl/figuring-out-foreach.html>`_ to explain the problem.

The fix however is trickier, and I am inclined to roll that up into a big
patch that refactors Code Coverage into an analysis pass, and a collection
pass. The patch sits in a `branch
<https://github.com/derickr/xdebug/tree/improve-code-coverage>`_ and addresses
some other long `outstanding inaccuracies <https://bugs.xdebug.org/1799>`_.

I recently have rebranched this patch on Xdebug's master again, so it is fully
functional (minus the foreach changes), but unfortunately it is a little
slower than what currently is available in Xdebug. I might decide to release
it with this slight performance degradation in Xdebug 3.5 or 3.6 regardless.

Xdebug Videos
-------------

I have created one new video in the last two months:

- `Xdebug 3.4: Control Socket <https://youtu.be/jBvrVpNHOCw>`_

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
