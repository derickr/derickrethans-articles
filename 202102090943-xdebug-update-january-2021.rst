Xdebug Update: January 2021
===========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-02-09 09:43 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-21jan

Another monthly update where I explain what happened with Xdebug development
in this past month. These will be published on the first Tuesday after the 5th
of each month.

`Patreon <https://www.patreon.com/derickr>`_ and `GitHub
<https://github.com/sponsors/derickr/>`_ supporters will get it earlier,
around the first of each month.

You can `become a patron <https://www.patreon.com/bePatron?u=7864328>`_ or
support me through `GitHub Sponsors <https://github.com/sponsors/derickr>`_.
I am currently 94% towards my $1,000 per month goal.

If you are leading a team or company, then it is also possible to support
Xdebug through `a subscription <https://xdebug.org/support>`_.

In January, I worked on Xdebug for about 60 hours, with funding being around
35 hours. I worked mostly on the following things:

Releases
--------

The start of the month saw the release of Xdebug 3.0.2 which addresses further
issues that are present in Xdebug 3.0. `Xdebug 3.0.2
<https://xdebug.org/announcements/2021-01-04>`_ mainly addresses issues
related to triggering features in Xdebug through Xdebug's new modes, and a few
issues related to code coverage were fixed too. Expect another bug fix release
in February, as I am intending to make one release per month.

Now that the GitHub repository of the `VS Code Plugin
<https://github.com/xdebug/vscode-php-debug>`_ has been moved to the `Xdebug
organisation <https://github.com/xdebug>`_, there is going to be progress here
too.

DBGp Client and DBGp Proxy
--------------------------

I have had some expert reviews of the Go code for `DBGp Client <https://xdebug.org/docs/dbgpClient>`_ and `DBGp
Proxy <https://xdebug.org/docs/dbgpProxy>`_, by JetBrains developers working
on their `GoLand IDE <https://www.jetbrains.com/go/>`_. Once I have reviewed
and integrated their comments I am intending to release the source code of
these two tools under an open source license.

Documentation
-------------

I spent some time adding anchors throughout the documentation to make it
easier to link to specific sections. The `upgrade guide <https://xdebug.org/docs/upgrade_guide>`_ now also has a
`Japanese translation <https://xdebug.org/docs/upgrade_guide/ja>`_.
Xdebug.org's code has been updated to allow for translations of the
documentation, although I do not have any plans to translate the rest of it at
the moment. The main issue with having translations of the documentation that
the translations also need updating when the original in English changes.

As part of Xdebug's documentation efforts I created another video in my series
on `Xdebug 3
<https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_.
This new video explains Xdebug 3's `diagnostics features <https://www.youtube.com/watch?v=IN6ihpJSFDw>`_ to help finding
issues with Xdebug's configuration and settings.

The next video that I am going to release will deal with how to trigger
Xdebug's features, such as single step debugging, profiling, and tracing.

Xdebug Cloud
------------

The `Xdebug Cloud <https://cloud.xdebug.com>`_ now has a design, created by
`Simon Collison <https://colly.com/work>`_. With the design in place, I am
currently putting the dots on the i's related to signing up and billing. I
expect to launch Xdebug Cloud for beta customers during this month.

If you want to be kept up to date with Xdebug Cloud, please sign up to the
`mailinglist <http://cloud.xdebug.com>`_, which I will use to send out an
update not more than once a month.


Business Supporter Scheme and Funding
-------------------------------------

In January, no new supporters signed up.

If you, or your company, would also like to support Xdebug, head over to the
`support <https://xdebug.org/support>`_ page!

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page and a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_.
