Xdebug Update: December 2021
=============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2022-01-11 08:54 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-21dec

In this monthly update I explain what happened with Xdebug development
in this past month. These will be published on the first Tuesday after the 5th
of each month.

`Patreon <https://www.patreon.com/derickr>`_ and `GitHub
<https://github.com/sponsors/derickr/>`_ supporters will get it earlier,
around the first of each month.

You can `become a patron <https://www.patreon.com/bePatron?u=7864328>`_ or
support me through `GitHub Sponsors <https://github.com/sponsors/derickr>`_.
I am currently 46% towards my $2,500 per month goal.
If you are leading a team or company, then it is also possible to support
Xdebug through `a subscription <https://xdebug.org/support>`_.

In December, I worked on Xdebug directly for only about 26 hours, with funding
being around 21 hours. Please become a supporter of Xdebug through Patreon or
GitHub.

Xdebug 3.1 and further
----------------------

On the first of the month, I released Xdebug 3.1.2.

It addresses a few crash bugs related to PHP 8.1 fibers, a crash bug when
Xdebug can't write a profiler file, and an issue with Xdebug's var_dump() not
using the magic __debugInfo method. 

The full list of changes can be found on the `updates
<https://xdebug.org/updates#x_3_1_2>`_ page on the Xdebug website. 

Since Xdebug 3.1.2 I have fixed a few more bugs, which are not yet in a
released version. One fix pertains to the debugger generating not-well-formed
XML, and another one improves performance with long strings in the debugger.

I spend most of my time in December to investigate issues that have not yet
been solved. One of them turned to be a change in PHP behaviour between PHP
8.0 and 8.1. In PHP 8.0 and earlier, the ``$_GLOBALS[]`` superglobal also had
a key ``GLOBALS``, which PHP 8.1 no longer has. PhpStorm was reading values
for its **watch** feature, from the ``GLOBALS`` context, but also added the extra (unnecessary) ``GLOBALS``
array element to read out the real variables. Fixing this in Xdebug is
complex, so hopefully this will be addressed in the next version of PhpStorm
itself.

The second issue turned out to be an issue with PHP-FPM, which is not strictly
following PHP's processing model. This can cause a discrepancy between
PHP-FPM's control and worker processes, where they do not agree what the value
of the ``xdebug.mode`` INI setting is. Ideally this should get fixed in
PHP-FPM, but there are further issues that both PHP-FPM and Xdebug probably
need changes for.

Xdebug Cloud
------------

To help with funding my work on Xdebug, I have a paid-for-service, called
Xdebug Cloud.

Xdebug Cloud is the *Proxy As A Service* platform to allow for debugging in
more scenarios, where it is hard, or impossible, to have Xdebug make a
connection to the IDE. It is continuing to operate as Beta release.

Packages start at £49/month, and revenue will be used to further the
development of Xdebug.

If you want to be kept up to date with Xdebug Cloud, please sign up to the
`mailinglist <https://xdebug.cloud/newsletter>`_, which I will use to send out
an update not more than once a month.

Xdebug Videos
-------------

I did not create any new Xdebug videos this month on my `YouTube channel
<https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_.
But I am working on a more thought out set of instructional videos. Stay
Tuned!

If you would like to suggest a topic for a 5 to 15 minute long video, feel
free to request them through this `Google Form
<https://forms.gle/ugjGbxs6ZhiTyvCSA>`_.

Business Supporter Scheme and Funding
-------------------------------------

In December, two new business supporter signed up:

- `Seiden Group <http://www.seidengroup.com>`_
- `Digitell, Inc. <https://digitellinc.com/>`_

Thank you!

If you, or your company, would also like to support Xdebug, head over to the
`support <https://xdebug.org/support>`_ page!

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page and a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_.
