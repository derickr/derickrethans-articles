Xdebug Update: February 2022
============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2022-03-08 09:25 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-22feb

In this monthly update I explain what happened with Xdebug development in this
past month. These are normally published on the first Tuesday after the 5th of
each month.

`Patreon <https://www.patreon.com/derickr>`_ and `GitHub
<https://github.com/sponsors/derickr/>`_ supporters will get it earlier,
around the first of each month.

You can `become a patron <https://www.patreon.com/bePatron?u=7864328>`_ or
support me through `GitHub Sponsors <https://github.com/sponsors/derickr>`_.
I am currently 46% towards my $2,500 per month goal.
If you are leading a team or company, then it is also possible to support
Xdebug through `a subscription <https://xdebug.org/support>`_.

In February, I spend 26 hours on Xdebug, with 24 hours funded.

Releases and Development
------------------------

As first task in February I `released Xdebug 3.1.3
<https://xdebug.org/announcements/2022-02-01>`_ which addressed a bug with the
debugging protocol's ``eval`` command and exceptions, an issue with the
debugger generating invalid XML, and a memory leak and performance issue with
tracing.

Since then I have worked on triaging and fixing a few more issues. These have
currently not been merged due to changes in GitHub Action's Windows support. I
expect that to be resolved by the time you read this.

Fixes include a crash while using closures and static properties, something
that Symfony Cache seems to use a lot, some memory leaks, and an issue with
the legacy ``XDEBUG_SESSION_START`` trigger. I expect Xdebug 3.1.4 to be
released imminently.

Priorities
----------

In the previous `update </xdebug-update-january-2022.html>`_ I wrote about
what I had in mind for this coming year. From feedback to this communique, and
a highly scientific Twitter poll I've decided to prioritise the following
items:

1. Creating an *Xdebug Course*: explaining in great detail how Xdebug
   works, how you use it, how you can get the most out of it, and many
   scenarios on how to set-up debugging in different environments. This needs
   to go beyond referential documentation pages, and will hence become a
   combined set of videos and tutorials, with examples and work-along
   exercises.

2. *Xdebug Recorder and Player*: A new feature in Xdebug which would
   allow for a full request to be stored in a file, including every
   intermediate state. Combined with a player, which would allow for replaying
   the request and interrogating every variable at every point during the
   execution of said script, through the debugging protocol and interacting
   with your IDE. The recorded files would be self contained, without needing
   access to the (original) source code. Besides "step over" it would also
   have a "step back", and perhaps even a slider to slide back and forwards
   through time.

I have started making an outline for the *Course*, and a prototype for the
`Recorder <https://github.com/xdebug/xdebug/compare/master...derickr:recorder?expand=1>`_
and Player.

Let me know if you have ideas about which specific topics the Course should
offer.

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

I have published one more video on how to use Xdebug on my `YouTube channel
<https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_.

- `Debugging Unit Tests with PhpStorm
  <https://youtu.be/WMGYfgzoap0>`_ explains how to use PhpStorm to debug Unit
  Tests, following a question on Twitter.

Business Supporter Scheme and Funding
-------------------------------------

In February, no new business supporter signed up.

If you, or your company, would also like to support Xdebug, head over to the
`support <https://xdebug.org/support>`_ page!

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page and a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_.
