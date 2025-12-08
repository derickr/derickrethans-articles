Xdebug Update: November 2025
============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2025-12-09 15:10 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-25oct

In this update I explain what happened with Xdebug development in the last
month.

In the last month, I spent around 28 hours on Xdebug, with 24 hours funded.

Xdebug 3.5
----------

Most of this month I spent on making Xdebug 3.5 ready, to coincide with the
release of PHP 8.5.

First there were a few bug reports from the alpha releases, and I found some
issues with the new Native Path Mapping myself. I also fixed a crash bug, and
released the last pre-release version: 3.5.0alpha3.

After that release, I worked with the author of the `PHP Debug Adapter for
Visual Studio Code
<https://marketplace.visualstudio.com/items?itemName=xdebug.php-debug>`_ to
bring Xdebug's control sockets to Windows. This held up the release of Xdebug
3.5 a little, as we wanted to get this right.

With this, I also created `documentation for the Xdebug Control
<https://xdebug.org/docs/xdebugctl>`_ tool, which can be used to talk to
Xdebug to initiate debugging sessions, and to force pausing running code
without setting a breakpoint beforehand. This is only available on Linux and
Windows.

Documentation Upgrades
----------------------

While working on the documentation for the Xdebug Control tool, I went down
the rabbit hole, and revamped the layout of the `documentation's front page
<https://xdebug.org/docs/>`_ too. Instead of a single list with bullet points,
there are now sections, and some images depicting the different features.

I also added a section on `Flame Graphs
<https://xdebug.org/docs/flamegraphs>`_. This feature that I added in Xdebug
3.3 was not yet documented.

PIE and PECL
------------

I also documented how to install Xdebug with `PIE
<https://xdebug.org/docs/install#pie>`_, the new installer for PHP extensions
based around the `packagist <https://packagist.org/packages/xdebug/xdebug>`_
ecosystem.

PIE is replacing the legacy PECL tool and website. Installing Xdebug with PECL
is no longer the preferred installation method. Using PIE is strongly
recommended.

In the future, there will no longer be new versions of Xdebug uploaded to the
PECL website. In the mean time, releases through PECL will be delayed.

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
