Xdebug Update: November 2020
=============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-12-08 09:03 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-20nov

Another monthly update where I explain what happened with Xdebug development
in this past month. These will be published on the first Tuesday after the 5th
of each month.

`Patreon <https://www.patreon.com/derickr>`_ and `GitHub
<https://github.com/sponsors/derickr/>`_ supporters will get it earlier, on
the first of each month.

**I am currently looking for more funding, especially now some companies have
dropped out, and that GitHub sponsors is no longer matching supporters.**

You can `become a patron <https://www.patreon.com/bePatron?u=7864328>`_ or
support me through `GitHub Sponsors <https://github.com/sponsors/derickr>`_.
I am currently 73% towards my $1,000 per month goal.

If you are leading a team or company, then it is also possible to support
Xdebug through `a subscription <https://xdebug.org/support>`_.

In November, I worked on Xdebug for about 72 hours, with funding being around
30 hours. I worked mostly on the following things:

Xdebug 3
--------

The biggest thing this month is the `release of Xdebug 3
<https://xdebug.org/announcements/2020-11-25>`_!

After about a year and a half after starting with this massive undertaking, it
is finally ready. I released it the day before PHP 8 came out. Of course
Xdebug 3 also supports PHP 8. It however drops support for PHP 7.1 as per the
`support policy <https://xdebug.org/docs/compat>`_.

As is custom with a x.0.0 release, a few bugs did occur. I am currently
working at addressing then. I plan to release bug release versions weekly
throughout December, as long as it makes sense to do so.

Xdebug 3 should be a lot faster, as it is a lot more clever on when it hooks
into things it needs to do. That does come with changes as to how Xdebug needs
to be configured. Xdebug 3's `upgrade guide
<https://xdebug.org/docs/upgrade_guide>`_ lists all the changes.

The https://xdebug.org web site now only contains Xdebug 3 documentation, with
the old site `archived at https://2.xdebug.org <https://2.xdebug.org>`_ until
the end of 2021.

I will be recording some videos about the ideas behind the changes, and how to
use Xdebug 3. I am also playing with the idea of hosting "Office Hours" for an
hour a week where users can drop-in with questions and problems. If that is
something that you're interested in, please `let me know
<https://derickrethans.nl/who.php>`_.


Xdebug Cloud
------------

I have been continuing to test `Xdebug Cloud <https://cloud.xdebug.com>`_, and
I am working with a few private alpha testers. They're putting the hosted
Cloud service to its paces with the latest PhpStorm 2020.3 release candidate.
As I suspected, the alpha testers found some minor issues which I will be
addressing during December.

The web site for Xdebug Cloud does not have a design yet, but this is coming
this month as well.

If you want to be kept up to date with Xdebug Cloud, please sign up to the
`mailinglist <http://cloud.xdebug.com>`_, which I will use to send out an
update not more than once a month.


Business Supporter Scheme and Funding
-------------------------------------

In November, two new supporters signed up.

**Thank you `Find My Electric <https://www.findmyelectric.com/>`_ and `Edmonds
Commerce <https://www.edmondscommerce.co.uk/>`_!**

If you, or your company, would also like to support Xdebug, head over to the
`support <https://xdebug.org/support>`_ page!

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page and a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_.
