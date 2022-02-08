Xdebug Update: January 2022
===========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2022-02-10 09:47 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-22jan

In this monthly update I explain what happened with Xdebug development in this
past month. These are normally published on the first Tuesday after the 5th of
each month. I am late this month. Sorry.

`Patreon <https://www.patreon.com/derickr>`_ and `GitHub
<https://github.com/sponsors/derickr/>`_ supporters will get it earlier,
around the first of each month.

You can `become a patron <https://www.patreon.com/bePatron?u=7864328>`_ or
support me through `GitHub Sponsors <https://github.com/sponsors/derickr>`_.
I am currently 45% towards my $2,500 per month goal.
If you are leading a team or company, then it is also possible to support
Xdebug through `a subscription <https://xdebug.org/support>`_.

In January, I spend my time triaging issues, and planning for this year.

2022 Plans
----------

I spend most of my time reflecting on what I can do to make Xdebug even better
in 2022, and I have come to the conclusion that this is going to be done
through multiple improvements.

1. Creating an Xdebug Course: explaining in great detail how Xdebug
   works, how you use it, how you can get the most out of it, and many
   scenarios on how to set-up debugging in different environments. This needs
   to go beyond referential documentation pages, and will hence become a
   combined set of videos and tutorials, with examples and work-along
   exercises.

1. Developing an set-up-free debugger: A new tool that can be used through
   Xdebug Cloud, that would allow you to debug without IDE.

1. Xdebug Recorder and Player: A new feature in Xdebug which would
   allow for a full request to be stored in a file, including every
   intermediate state. Combined with a player, which would allow for replaying
   the request and interrogating every variable at every point during the
   execution of said script, through the debugging protocol and interacting
   with your IDE. The recorded files would be self contained, without needing
   access to the (original) source code. Besides "step over" it would also
   have a "step back", and perhaps even a slider to slide back and forwards
   through time.

1. Rewriting Xdebug's Profiler: so that it is more lightweight, and
   so that it can be enabled for specific parts of an application/request. In
   addition to this I am looking at sending the profiling data over the
   debugging protocol, so that visualisation tools do not need to find and
   read files.

1. Creating a profile analysis tool: To automatically analyse profiling files
   and apply logic so that it can point to the most likely cause of
   bottlenecks.

Let me know which one of these interests you most, and whether you would be
willing to pay for such a tool.


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

I did not create any new Xdebug videos this month on my `YouTube channel
<https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_.
But as I mentioned earlier, I am working on a more comprehensive course. Stay
tuned!

Business Supporter Scheme and Funding
-------------------------------------

In January, one new business supporter signed up:

- `VEMA eG <https://www.vema-eg.de>`_

Thank you!

If you, or your company, would also like to support Xdebug, head over to the
`support <https://xdebug.org/support>`_ page!

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page and a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_.
