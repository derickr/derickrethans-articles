The Xdebug Experience
=====================

.. articleMetaData::
   :Where: London, UK
   :Date: 2019-01-22 09:23 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-experience

Xdebug_ development is currently in its `17th year`_. I started working on it
when `PHP 4.2.0`_ had just been released. The `first CVS commit`_ added the
(in)famous "maximum nesting level" functionality, to prevent PHP from crashing
when your script would infinitely recurse. I had forgotten this, but
apparently Xdebug has some origins in VL-SRM_, a naive attempt to create an
application server running PHP bananas_ (think of: Java beans). The VL prefix
still lives on as part of my PHP internals debugging extension VLD_.

.. _Xdebug: https://xdebug.org
.. _`17th year`: https://github.com/xdebug/xdebug/graphs/contributors
.. _`PHP 4.2.0`: http://php.net/ChangeLog-4.php#4.2.0
.. _`first CVS commit`: https://github.com/xdebug/xdebug/commit/78749ee4113003cc3391f7d468d1a9ddb320f963
.. _`VL-SRM`: https://derickrethans.nl/projects.html#srm
.. _bananas: https://github.com/xdebug/xdebug/commit/78749ee4113003cc3391f7d468d1a9ddb320f963#diff-50b7f832915ee1d6491a14aa16a90786R363
.. _VLD: https://derickrethans.nl/projects.html#vld

Where Are We Now?
-----------------

As with any legacy project, Xdebug has technical debt. Some of it I managed to
address during the years by dropping support for older PHP versions. There is
a fair amount of sub-optimal code, algorithmic, and design issues, which
impact ease of configuration, usability, and performance. 

**Configuration Confusion**

On the configuration side, take for example the 65 `configuration settings`_.
There are a few pointless ones (`xdebug.extended_info`_,
`xdebug.remote_handler`_), a few that duplicate functionality
(`xdebug.trace_output_dir`_ and `xdebug.profiler_output_dir`_), and a few that
should never be used together (`xdebug.remote_enable`_ and
`xdebug.profiler_enable`_).

.. _`configuration settings`: https://xdebug.org/docs/all_settings
.. _`xdebug.extended_info`: https://xdebug.org/docs/all_settings#extended_info
.. _`xdebug.remote_handler`: https://xdebug.org/docs/all_settings#remote_handler
.. _`xdebug.trace_output_dir`: https://xdebug.org/docs/all_settings#trace_output_dir
.. _`xdebug.profiler_output_dir`: https://xdebug.org/docs/all_settings#profiler_output_dir
.. _`xdebug.remote_enable`: https://xdebug.org/docs/all_settings#remote_enable
.. _`xdebug.profiler_enable`: https://xdebug.org/docs/all_settings#profiler_enable

**Unproductive Usability**

If we look at usability, there are situations where breakpoints don't break.
Some of these I can likely address, whereas others I can not due to the way
how PHP works internally (See also: `The Mystery of the Missing
Breakpoints`_). Having OPcache_ in the mix does not help either as it can
create another set of problems where visible code has been optimised out.

.. _`The Mystery of the Missing Breakpoints`: /breakpoints.html
.. _OPcache: http://php.net/manual/en/intro.opcache.php

To solve a third group of issues with breakpoints, the DBGp_ "breakpoint
resolved" notification_ needs to be implemented in both Xdebug and IDEs.

.. _DBGP: https://xdebug.org/docs-dbgp.php
.. _notification: https://xdebug.org/docs-dbgp.php#standard-notifications

Another issue is that both Xdebug and PHP-FPM use port 9000 as their default,
although Xdebug's use of port 9000 precedes PHP-FPM's by about five years
(2004_ vs 2009). 

.. _2004: https://github.com/xdebug/xdebug/commit/d837fe99fcaf2e34f34acd5088615f2b6376854b

And lastly, (potential) users often cite that it is "hard" to set Xdebug up.
Although after talking to some of these users, it often becomes clear that the
problem is not so much on the Xdebug side of things, but rather other tools:
their IDE, Docker, SELinux_, firewalls, etc. Improving this is a matter of
education (and better error messages).

.. _SELinux: https://en.wikipedia.org/wiki/Security-Enhanced_Linux

**Poor Performance**

On the performance side, there are some places with major issues, and some
others with minor issues. Code coverage isn't particularly fast, and that
needs an in-depth investigation. I have several ideas on where improvements in
code coverage performance can be made—nonetheless, I would continue to put
correctness over speed.

Even with Xdebug just loaded and enabled, there is too much of a performance
impact. This is because Xdebug usually has every feature ready to go, even
though you don't necessarily want to use all of these different features. For
example, it is ludicrous to use single step debugging with the profiler turned
on.

Although it's often possible to reduce the impact of feature sets by setting
configuration options correctly, a better way to put Xdebug in a specific
mode would be welcome.

What Needs to Be Done?
----------------------

In order to address many of these problems, Xdebug must undergo a massive
refactoring. As part of that effort, the refactoring must primarily focus
on solving the above mentioned three categories, although improvements in code
layout are also desirable.

As part of the refactoring process, the following primary tasks need to be
completed, in preferably the listed order:

- Finalize PHP 7.3 support (there is a nasty bug_ where Xdebug and a specific
  OPcache optimisation setting conflict).
- Code needs to be reorganised, as the current location of source files and
  functions is detrimental to improvements.
- The number of configuration options needs to be reduced.
- *Modes* needs to be introduced, so that it is easier to turn off and on
  (internal) code to support different features. This will very likely improve
  general performance already.
- Specific code paths and features need to be optimised. This includes
  primarily the basic features (stack traces, `var_dump()` improvements, etc.),
  as well as Code Coverage.

.. _bug: https://bugs.xdebug.org/view.php?id=1583

What is Needed to Get This Done?
--------------------------------

I would love to be able to continue to work on Xdebug; to keep it in sync with
changes in PHP itself, to implement the ideas from this article to improve the
Xdebug Experience, and to produce educational material such as improved
documentation, tutorials, and videos. (Dare I say a book?)

Therefore I am currently looking for ways that I can be funded for doing all
the required work. It will take a lot of time to get Xdebug 3 out in a
tip-top shape. Possibly, if not more, than 6 months. Although I have plenty of
time now that `I've left MongoDB`_, the bills and mortgage do need to get paid
too. I can think of a few options:

- Get more users to sign up to my Patreon_ account. There is a group of loyal
  users and companies that contribute towards the upkeep of the server. But in
  order to make this sustainable, the patron count need to increase about 30
  fold. I struggle with setting up rewards to nudge people to support me.
- A fundraiser (through a crowd funding site) for specific tasks and/or
  features from the plan.
- Some functionality that would only be available under a commercial license.
  One of the ideas here is to add a recording feature that records the full
  execution of a script, which then later can be replayed back through an
  interactive single step debugging session in an IDE.
- Work with IDE manufacturers to implement some of their requested features
  (such as the before mentioned "resolved breakpoint" notification), and to
  come up with new features to make the debugging experience better.

I am interested to hear whether you have a specific preference, or perhaps
some additional suggestions for me to consider. I would absolutely want to
give Xdebug all the love it still deserves after 17 years. Let me know what
you think! Either in a comment, or by email_.

cheers,
Derick

.. _`I've left MongoDB`: /moving-on-from-mongodb.html
.. _`bottle of whisky`: https://www.amazon.co.uk/gp/registry/wishlist/SLCB276UZU8B
.. _Patreon: https://www.patreon.com/derickr
.. _email: /who.html#email
.. _`issue tracker`: https://bugs.xdebug.org
