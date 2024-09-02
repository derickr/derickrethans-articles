Xdebug Update: August 2024
==========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2024-09-03 14:35 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-24aug

In this monthly update I explain what happened with Xdebug development

`GitHub <https://github.com/sponsors/derickr/>`_ and `Pro/Business supporters
<https://xdebug.org/support>`_ will get it earlier, around the first of each
month.

On GitHub sponsors, I am currently 32% towards my $2,500 per month goal, which
is set to allow continued **maintenance** of Xdebug. This is again less than
last month.

If you are leading a team or company, then it is also possible to
support Xdebug through `a subscription <https://xdebug.org/support>`_.

In the last month, I spend around 8 hours on Xdebug, with 20 hours funded.
I also spent 8.5 hours on the Native Path Mapping project.

PHP 8.4
-------

The large change for the `exit-as-a-function
<https://wiki.php.net/rfc/exit-as-function>`_ proposal has been merged into
PHP, which means I had to spend some time finishing and tidying up a Pull
Request that Tim Düsterhus had provided.

I also improved Xdebug's `CI <https://xdebug.org/ci>`_ so that it can run as a
separate user.

Right now, there don't seem to be any outstanding issues with PHP 8.4 so I
will soon create another alpha (or beta) release so that you can try out
Xdebug with the latest beta versions of PHP 8.4's.

Native Xdebug Path Mapping
--------------------------

I continued with the `Native Xdebug Path Mapping
<https://xdebug.org/funding/001-native-path-mapping>`_ project.

This is a separately funded project, I don't classify the hours that I
worked on this in "Xdebug Hours". I also publish a separate report on the
`project page <https://xdebug.org/funding/001-native-path-mapping>`_.

Again, I am including it here:

I continued with the parser to parse the path mapping files. The parsing is
now finished, although I do still need to modify how the parsed information is
stored. Right now, the data structures are not optimised to be able to do line
mapping as well.

Because the parser parsers user input, it is important that the parser is
rubust. It should be able to handle correctly formatted files, but also files
with errors.

It is not always possible to come up with all the failure situations by
thinking, and therefore a common technique is to use a fuzzer. For PHP there
is `Infection PHP <https://infection.github.io/>`_ for mutation testing for
example. For C, and C++, a commonly used tool is `AFL++
<https://github.com/AFLplusplus/AFLplusplus>`_. This provides a compiler
wrapper and a run-time to fuzz the input to your application. You first
provide a template, which it then modifies to try to break your code.

The template that I used in this case was a minimal map file::

	remote_prefix: /usr/local/www
	local_prefix: /home/derick/project
	/projects/example.php:5-17 = /example.php:8
	/example.php:5-17 = /example.php:8-20
	/example.php:17 = /example.php:20
	/projects/php-web/ = /php-web/

In addition to the template you also need to provide a shim — the program that
in my case takes the argument given to it and then parses that as a file. The
AFL++ tool's compiler wrapper adds some magic to it to be able to catch
errors.

When you then run the fuzzer, such as with::

	AFL_SKIP_CPUFREQ=1 afl-fuzz -b 7 -i fuzz-seeds -o fuzz-output -- ./afl-test @@

It then runs your program with your template files (in fuzz-seeds in my
example). And then also with loads of variants.

The fuzzer found a few errors in my parser, which ended up crashing it. One
such examples that I hadn't thought of is of a line starting with a =::

	remote_prefix: /usr/local/www
	local_prefix: /home/derick/project
	=/example.php:42-5 = /example.php

You can find the other cases it found in a `dedicated test file <https://github.com/derickr/xdebug/blob/001-native-path-mapping/tests/ctest/fuzz-cases.cpp#L93>`_.

In September I hope to finish the parser (by storing things more efficiently
internally) as well as creating APIs to do the mapping.


Xdebug Videos
-------------

I have created no new videos since last month, but you can see all the
previous ones on my `channel
<https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_.

If you have any suggestions, feel free to reach out to
`me on Mastodon <https://phpc.social/@derickr>`_ or via `email
<http://derickrethans/who.html>`_.

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
