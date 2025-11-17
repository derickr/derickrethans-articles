Xdebug Update: February 2021
============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-03-09 09:11 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-21feb

Another monthly update where I explain what happened with Xdebug development
in this past month. These will be published on the first Tuesday after the 5th
of each month.

`Patreon <https://www.patreon.com/derickr>`_ and `GitHub
<https://github.com/sponsors/derickr/>`_ supporters will get it earlier,
around the first of each month.

You can `become a patron <https://www.patreon.com/bePatron?u=7864328>`_ or
support me through `GitHub Sponsors <https://github.com/sponsors/derickr>`_.
I am currently 90% towards my $1,000 per month goal (4% less than last month).
If you are leading a team or company, then it is also possible to support
Xdebug through `a subscription <https://xdebug.org/support>`_.

In February, I worked on Xdebug itself for about 16 hours, with funding being
around 32 hours. I am no longer including Xdebug Cloud time in this figure,
which is what I spend most of my time on. More on that later.

Releases
--------

The Xdebug 3.0.3 `release <https://xdebug.org/announcements/2021-02-22>`_ that
came out last week, addresses a few minor issues. One regarding missing
information in Code Coverage, another one where local variables are missing if
a debugging session was started through ``xdebug_break()``, and another one
addressed an issue with ``xdebug_info()`` output.

The next release will likely be Xdebug 3.1 where I will focus on making
``xdebug_info()`` show even more diagnostics information, and also implement
some additional features for Xdebug Cloud, which brings me to the last point
of this month's short newsletter.


Xdebug Cloud
------------

Xdebug Cloud is now released in Beta, and has a new web site address at
https://xdebug.cloud — which means that it is now possible to sign up!
Packages start at £49/month, and revenue will also be used to further the
development of Xdebug.

.. image:: images/xdebug-cloud-reveal.gif

If you want to be kept up to date with Xdebug Cloud, please sign up to the
`mailinglist <https://xdebug.cloud/newsletter>`_, which I will use to send out
an update not more than once a month.


Business Supporter Scheme and Funding
-------------------------------------

In February, no new supporters signed up.

If you, or your company, would also like to support Xdebug, head over to the
`support <https://xdebug.org/support>`_ page!

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page and a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_.
