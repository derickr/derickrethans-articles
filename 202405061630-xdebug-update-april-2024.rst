Xdebug Update: April 2024
=========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2024-05-06 16:30 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-24apr

I have not written an update like this for a while. I am sorry.

In the last months I have not spent a lot of time on Xdebug due to a set of
other commitments.

Since my last update in November a few things have happened though.

Xdebug 3.3
----------

I released Xdebug 3.3, and the patch releases 3.3.1 and 3.3.2.

`Xdebug 3.3 brings <https://xdebug.org/announcements/2023-11-30>`_ a bunch of
new features into Xdebug, such as `flamegraphs
<https://derickrethans.nl/flamboyant-flamegraphs.html>`_.

The debugger has significant performance improvements in relation to
breakpoints. And it can now also show the contents of ArrayIterator,
SplDoublyLinkedList, SplPriorityQueue objects, and information about thrown
exceptions.

A few bugs were present in 3.3.0, which have been addressed in 3.3.1 and
3.3.2. There is currently still an outstanding issue (or more than one), where
Xdebug crashes. There are a lot of confusing reports about this, and I have
not yet managed to reproduce any of them.

If you're running into a crash bug, please reach out to me.

There is also a new experimental feature: control sockets. These allow a
client to instruct Xdebug to either initiate a debugging connection, or
instigate a breakpoint out of band: i.e., when no debugging session is active.
More about this in a later update.

Funding Platform
----------------

Last year, I made a `prototype
<https://github.com/xdebug/xdebug/compare/master...derickr:xdebug:custom-map>`_
as part of a talk that I gave at `NeosCon.io
<https://derickrethans.nl/talks/xdebug-neoscon23>`_. In this talk I
demonstrated native path mapping — configuring path mapping in/through Xdebug,
without an IDE's assistance.

In collaboration with Robert from
`NEOS <https://www.neos.io/>`_, and Luca from `theAverageDev
<https://theaveragedev.com/>`_, we defined a `project plan
<https://xdebug.org/funding/001-native-path-mapping>`_ that explains all the
necessary functionality and work.

Adding this to Xdebug is a huge effort, and therefore I decided to set up a
way how projects like this could be funded.

There is now a dedicated `Projects <https://xdebug.org/funding>`_ section
linked from the home page, with a list of all the projects. The page itself
lists a short description for each project.

For each project, there is a full description and a list of its generous
contributors. The `Native Xdebug path Mapping
<https://xdebug.org/funding/001-native-path-mapping>`_ project is currently
85% funded. Once it is fully done, I will start the work to get this included
in Xdebug 3.4. `You could be part of this too
<https://xdebug.org/support/buy/001-native-path-mapping>`_!

Xdebug Videos
-------------

I have created several videos since November.

Two for `Xdebug <https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_:

- `Xdebug 3.3: Flamegraphs <https://youtu.be/4EocpeKxI0k>`_
- `Xdebug 3.3: New Features in xdebug_get_function_stack() <https://youtu.be/3FLdpMLBqMk>`_

And several for writing PHP extensions, as part of a new `series <https://www.youtube.com/playlist?list=PLg9Kjjye-m1hW4z0J-546qaFpysjlo27x>`_:

- `1. Writing PHP Extensions: Creating a Skeleton <https://youtu.be/WjbKYHzoKM0>`_
- `2. Writing PHP Extensions: Importing the Library, and … <https://youtu.be/8N8Rk-BE0d4>`_
- `3. Writing PHP Extensions: Implementing the rdp_simplify … <https://youtu.be/-Tb_f55fc0k>`_

If you have any suggestions, feel free to reach out to
`me on Mastodon <https://phpc.social/@derickr>`_ or via `email
<https://derickrethans.nl/who.html>`_.

Business Supporter Scheme and Funding
-------------------------------------

In the last month, no new business supporters signed up.

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page, a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_, as well as an `OpenCollective
<https://opencollective.com/xdebug>`_ organisation.

If you want to contribute to specific projects, you can find those on the
`Projects <https://xdebug.org/funding>`_ page.

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
