Xdebug Update: December 2022
============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2023-01-10 09:06 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-22dec

In this monthly update I explain what happened with Xdebug development
in this past month. These are normally published on the first Tuesday on
or after the 5th of each month.

`Patreon <https://www.patreon.com/derickr>`_ and `GitHub
<https://github.com/sponsors/derickr/>`_ supporters will get it earlier,
around the first of each month.

You can `become a patron <https://www.patreon.com/bePatron?u=7864328>`_
or support me through `GitHub Sponsors
<https://github.com/sponsors/derickr>`_. I am currently 45% towards my
$2,500 per month goal. If you are leading a team or company, then it is
also possible to support Xdebug through `a subscription
<https://xdebug.org/support>`_.

In the last month, I spend 25 hours on Xdebug, with 21 hours funded.
Sponsorships are continuing to decline, which makes it harder for me to
dedicate time for maintenance and development.

Xdebug 3.2
----------

Xdebug 3.2.0 got released at the start of December, to coincide with the
release of PHP 8.2 which it supports, after fixing a last crash with code
coverage. Since then a few bugs were reported, which I have started to triage.
A particularly complicated one seems to revolve on Windows with PHP loaded in
Apache, where suddenly all modes are turned on without them having been
activated through the `xdebug.mode` setting. This is a complicated issue that
I hope to figure out and fix during January, resulting in the first patch
release later this month.

Plans for the Year
------------------

Beyond that, I have spend some time away from the computer in the Dutch
country side to recharge my battery. I hope to focus on redoing the profiler
this year, as well as getting the "recorder" feature to a releasable state.

Smaller feature wise, I hope to implement file/path mappings on the Xdebug
side to aide the debugging of generated files containing PHP code.

Xdebug Cloud
------------

Xdebug Cloud is the *Proxy As A Service* platform to allow for debugging
in more scenarios, where it is hard, or impossible, to have Xdebug make
a connection to the IDE. It is continuing to operate as Beta release.

Packages start at £49/month, and I have recently introduced a package
for larger companies. This has a larger initial set of tokens, and
discounted extra tokens.

If you want to be kept up to date with Xdebug Cloud, please sign up to
the `mailinglist <https://xdebug.cloud/newsletter>`_, which I will use
to send out an update not more than once a month.

Xdebug Videos
-------------

I have published two new videos:

- `Xdebug 3.2: Return Value Debugging with PhpStorm <https://www.youtube.com/watch?v=TNOGhUgY6Sc>`_
- `Xdebug 3: Skipping Files when Debugging <https://www.youtube.com/watch?v=FMysmRePbb8>`_

I have continued writing scripts for videos about Xdebug 3.2's features,
and am also intending to make a video about "Running Xdebug in
Production", as well as one on using the updated
"xdebug.client_discovery_header" feature (from Xdebug 3.1).

You can find all previous videos on my `YouTube channel
<https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_.

Business Supporter Scheme and Funding
-------------------------------------

In December, no new business supporters signed up.

If you, or your company, would also like to support Xdebug, head over to
the `support <https://xdebug.org/support>`_ page!

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page, a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_, as well as an `OpenCollective
<https://opencollective.com/xdebug>`_ organisation.
