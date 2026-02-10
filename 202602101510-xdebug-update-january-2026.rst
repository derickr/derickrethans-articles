Xdebug Update: January 2026
===========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2026-02-10 15:10 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-25oct

In this update I explain what happened with Xdebug development in the last
month.

In the last month, I spent only around 9 hours on Xdebug, with 24 hours funded.
The rest of the time, I spend on building out `Xdebug Cloud
<https://xdebug.cloud/>`_ version 2.

Xdebug 3.5
----------

Most of this month I spent on a few bug and performance reports
from the Xdebug 3.5 release — most notable a performance degradation
on Windows due to the new experimental control sockets.

I also spent time on my large better code coverage patch, which still
isn't quite as performant as the current feature; although it does
give more precise results.

Native Path Mapping
-------------------

Fabian Potencier, from Symfony fame, has created an exploratory
`patch <https://github.com/twigphp/Twig/pull/4733>`_ for Twig to make use of
Xdebug's new `Native Path Mapping functionality <https://xdebug.org/funding/001-native-path-mapping>`_.

From the initial patch, it became clear that a few things need to be improved
on the Xdebug side for this to be a complete feature. For that reason, I have
created a few issues to work on:

- `Invent better way of configuring lots of line-to-line mappings <https://bugs.xdebug.org/view.php?id=2398>`_
- `Create xdebug_add_source_map_directory() function to dynamically add directories where Xdebug will load path mapping files from <https://bugs.xdebug.org/view.php?id=2399>`_
- `Add EOF marker/stanza to native path file mapper <https://bugs.xdebug.org/view.php?id=2400>`_

My `PhpStorm issue
<https://youtrack.jetbrains.com/issue/WI-81344/Cant-Set-Breakpoints-in-Template-Files-Even-Though-I-Added-The-File-Name-Pattern-to-PHP>`_
to allow for the setting of breakpoints in template file has now been merged,
and is scheduled for 2026.1 EAP 2. At the time of writing this isn't quite out
yet, so I will keep you posted.

Xdebug Videos
-------------

I have created one new videos in the last month:

- `Xdebug 3: Interrupting Long Running Scripts with VS Code for Debugging
  <https://phpc.tv/w/5P3TbVPDWxjnhXejky13FK>`_

All Xdebug videos are now available on the `phpc.tv PeerTube instance
<https://phpc.tv/c/xdebug/videos>`_. This will be the primary location
for new videos, although I also still post them to 
my YouTube `channel
<https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_.

If you have any suggestions, feel free to reach out to
`me on Mastodon <https://phpc.social/@derickr>`_ or via `email
<https://derickrethans.nl/who.html>`_.

Xdebug Cloud
------------

I am currently reworking `Xdebug Cloud <https://xdebug.cloud>`_, the *Proxy As
A Service* platform to allow for debugging in complex networking scenarios.

The new version will allow for automatic subscriptions. 

Packages will start at £16/month for one-developer companies.

If you want to be kept up to date with Xdebug Cloud, please sign up to
the `mailing list <https://xdebug.cloud/newsletter>`_, which I will use
to send out an update not more than once a month.
