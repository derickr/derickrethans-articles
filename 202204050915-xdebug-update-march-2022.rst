Xdebug Update: March 2022
=========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2022-04-05 09:15 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-22mar

In this monthly update I explain what happened with Xdebug development in this
past month. These are normally published on the first Tuesday on or after the
5th of each month.

`Patreon <https://www.patreon.com/derickr>`_ and `GitHub
<https://github.com/sponsors/derickr/>`_ supporters will get it earlier,
around the first of each month.

You can `become a patron <https://www.patreon.com/bePatron?u=7864328>`_ or
support me through `GitHub Sponsors <https://github.com/sponsors/derickr>`_.
I am currently 46% towards my $2,500 per month goal.
If you are leading a team or company, then it is also possible to support
Xdebug through `a subscription <https://xdebug.org/support>`_.

In March, I spend 39 hours on Xdebug, with 25 hours funded.

Development
-----------

Most of my time in March was spent on investigating and fixing bugs. Ranging
from a crash during debugging of closures with static properties, support for
showing ArrayObject elements while debugging, improvements to code coverage,
and bugs with Xdebug's step-debugger triggering mechanism.

I also improved Xdebug's diagnostics by adding warnings. I added one for when
systemd PrivateTmp directories are used. And Xdebug now also warns if zlib
compression is enabled in situations, such as with the profiler's append
feature, where it can not be used.

I spend a considerable amount of time `cataloguing
<https://docs.google.com/document/d/1W-NzNtExf5C4eOu3rRQm1WlWnbW44u3ANDDA49d3FD4/edit?usp=sharing>`_
the different set-ups that developers use, and how to configure Xdebug's
debugger. This work will result in better documentation and perhaps a flow
chart. The research is also useful for the *Xdebug Course* that I have
mentioned before.

I have not figured out how to do it will all different set-ups, so if you have
extra information, or if I am still missing set-ups, feel free to comment on
the `Google Doc
<https://docs.google.com/document/d/1W-NzNtExf5C4eOu3rRQm1WlWnbW44u3ANDDA49d3FD4/edit?usp=sharing>`_.

In the more complicated set-ups, perhaps it would be easier to use `Xdebug
Cloud <https://xdebug.cloud>`_ as it has none of these networking
complications.

Xdebug Recorder
---------------

I have been making steady progress with a first version, which can now play
back recording information and display it in the likeness of an Xdebug trace
file. There are still a few bugs outstanding, and I also have not added
support for objects yet.

I will be recording a video with a demo once I have something to show. You can
follow the development in the `recorder
<https://github.com/derickr/xdebug/tree/recorder>`_ branch.

Xdebug Cloud
------------

Xdebug Cloud is the *Proxy As A Service* platform to allow for debugging in
more scenarios, where it is hard, or impossible, to have Xdebug make a
connection to the IDE. It is continuing to operate as Beta release.
Packages start at £49/month.

If you want to be kept up to date with Xdebug Cloud, please sign up to the
`mailinglist <https://xdebug.cloud/newsletter>`_, which I will use to send out
an update not more than once a month.

Xdebug Videos
-------------

I have published no new videos this month. You can find the existing ones on
my `YouTube channel
<https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_.

Business Supporter Scheme and Funding
-------------------------------------

In March, no new business supporters signed up.

If you, or your company, would also like to support Xdebug, head over to the
`support <https://xdebug.org/support>`_ page!

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page and a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_.
