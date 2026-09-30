Xdebug Update: September 2026
=============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2026-10-06 15:10 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-25sep

In this update I explain what happened with Xdebug development in the last
months. Support through Patreon and GitHub has been continuing to decline,
and now covers about 13 hours a month only. If you can help, that'd
be greatly appreciated.

In the last few months I have been working on getting Xdebug ready for PHP
8.6. However, there have also been a few notable changes that warrants
highlighting.

Code Coverage
-------------

First, I have rewritten Xdebug's code coverage. In earlier versions, Xdebug
would analyse PHP code during execution, and at the same time, and in the same
data structures, collect code coverage information. Now, these tasks are
split.

This results in more accurate coverage information, fixing bugs such as
`Inconsistent output of branch/path data when running under Opcache
<https://bugs.xdebug.org/1799>`_. It also means that `some behaviour changed
<https://bugs.xdebug.org/2438>`_. I would welcome further testing during the
alpha and beta releases of Xdebug 3.6.

Unfortunately, the changes also has a cost associated with it. The new
approach uses more memory and adds an additional performance penalty.

To alleviate some of this impact, I have started a branch implementing a few
optimisations. This has not been merged yet, but you can track the changes in
this `experimental GitHub branch
<https://github.com/xdebug/xdebug/compare/master...derickr:xdebug:improve-code-coverage-3.6>`_.
Right now, these two changes reduce the number of instructions by about 6%,
but I have only just started this project.

Windows
-------

Secondly, failing to make a debugging connection on Windows now fails faster.
I wrote a blog post about this, `Fail Faster
<https://derickrethans.nl/fail-faster.html>`_, where I explain in detail what
the cause is, how I found out about it, and what I changed. In short, on
Windows, if Xdebug now tries to make a connection to an IDE that isn't
listening on the port, the connection attempt will break off immediately,
instead of waiting for the 200ms time-out. This was not a bug in Xdebug, but
rather something that Windows does in an odd way.

FrankenPHP
----------

Lastly, a contribution by Xavier Leune makes Xdebug work better with
`FrankenPHP's worker mode <https://bugs.xdebug.org/2403>`_. This feature makes
sure that upon the first function call in a new worker request, Xdebug tries
to connect to the IDE, just as it would do for normal PHP requests. Unlike
"normal" PHP, FrankenPHP's worker mode reuses the same PHP "request" multiple
times.

Contributions
-------------

Starting with Xavier's contribution and Pull Request, I have decided to
approach contributions in a more personal way. Any substantial contribution
now not only needs a ticket and pull request, but I will insist on having a
face-to-face (online) conversation with you where you explain me how the new
feature works. This is both to teach *me*, as I need to maintain it, but it
also is useful for the contributors as they will learn how I approach
development. While going over the feature with Xavier, we found several
problems that are now addressed. I think this was a really good experiment
which I will now put into practise.

Dealing with Bots
-----------------

Scraper bots have also been a problem, and it is unfortunately now necessary
that you need to ask me before you can get access to *signing up for the issue
tracker*, and applying filters.

There were two different attacks in fact. The first one had bots signing up a
lot of new accounts. A second badly behaved AI scraping bot was trashing the
server with so many new expensive search requests (making use of filters),
that it was taking the server down constantly. I don't like having to do this,
but I can also not see a way out.

What's Next?
------------

Before the next release, I would still like to make a few additions for
`native path mapping <https://xdebug.org/funding/001-native-path-mapping>`_,
namely issues `#2399 <https://bugs.xdebug.org/2399>`_ and `#2400
<https://bugs.xdebug.org/2400>`_.

For now, I would hope that you could try out Xdebug 3.6.0alpha1, with the
latest version of PHP (8.6). Let me know if you find any issues, or if you
have any questions.

Xdebug Cloud
------------

I have reworking `Xdebug Cloud <https://xdebug.cloud>`_, the *Proxy As
A Service* platform to allow for debugging in complex networking scenarios.

Packages will start at £16/month for one-developer companies.
