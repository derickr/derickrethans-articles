Xdebug Update: October 2020
=============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-11-10 09:12 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-20oct

Another monthly update where I explain what happened with Xdebug development
in this past month. These will be published on the first Tuesday after the 5th
of each month.

`Patreon <https://www.patreon.com/derickr>`_ and `GitHub
<https://github.com/sponsors/derickr/>`_ supporters will
get it earlier, on the first of each month.

**I am currently looking for more funding, especially now some companies have
dropped out.**

You can `become a patron <https://www.patreon.com/bePatron?u=7864328>`_ or
support me through `GitHub Sponsors <https://github.com/sponsors/derickr>`_.
I am currently 66% towards my $1,000 per month goal.

If you are leading a team or company, then it is also possible to support
Xdebug through `a subscription <https://xdebug.org/support>`_.

In October, I worked on Xdebug for about 75 hours, with funding being around
45 hours. I worked mostly on the following things:

Xdebug 3
--------

Xdebug had its first beta release in October, `3.0.0beta1
<https://xdebug.org/announcements/2020-10-14>`_ so that uses can test out the
new speed improvements, new functionality, new ways of configuring things, as
well as support for PHP 8.

Some bugs were found, and I'm continuing to work together with JetBrains to
make sure all the new configuration methods are also supported with/through
PhpStorm.

There are also several anecdotal reports that Xdebug 3 is indeed a lot faster
in many environments. I have asked the community for benchmarks, but I have not
seen any write ups beyond a tweet (now deleted), and an `article
<https://php.watch/articles/php-code-coverage-comparison?rd>`_.

PHP 8
-----

Although PHP 8 is now in the Release Candidate phase, some changes are still
happening which requires remediation in Xdebug. I expect that to continue
until 8.0.0 is released, which means that Xdebug 3.0.0 can also not be
released until that happens, likely on November 26th. Xdebug 3.0.0 will
however follow quickly after that.

Xdebug Cloud
------------

I have been working on the web site for Xdebug Cloud, and have been testing
the integration with PhpStorm. It all seems to work well, although I'm only
the one user.

Once I've found somebody to design the site for me, I will open up Xdebug
Cloud through a private alpha.

If you want to be kept up to date with Xdebug Cloud, please sign up to the
`mailinglist <http://cloud.xdebug.com>`_ I'll let you know as soon as
something can be tried-out. 

Business Supporter Scheme and Funding
-------------------------------------

In October, no new supporters signed up, but a few existing supports renewed.

**Thank you Dennis Lassiter!**

If you, or your company, would also like to support Xdebug, head over to the
`support <https://xdebug.org/support>`_ page!

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page and a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_.
