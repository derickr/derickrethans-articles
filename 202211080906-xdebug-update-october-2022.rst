Xdebug Update: August, September, and October 2022
==================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2022-11-08 09:06 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-22oct

In this monthly update I explain what happened with Xdebug development
in this past month. These are normally published on the first Tuesday on
or after the 5th of each month.

`Patreon <https://www.patreon.com/derickr>`_ and `GitHub
<https://github.com/sponsors/derickr/>`_ supporters will get it earlier,
around the first of each month.

You can `become a patron <https://www.patreon.com/bePatron?u=7864328>`_
or support me through `GitHub Sponsors
<https://github.com/sponsors/derickr>`_. I am currently 45% towards my
$2,500 per month goal. If you are leading a team or company, then it is
also possible to support Xdebug through `a subscription
<https://xdebug.org/support>`_.

I did not find the time to write this report in the last two months,
sorry for that. So today I present you with a report for the last
quarter.

In the last three months, I spend 63 hours on Xdebug, with 77 hours
funded.

Xdebug 3.2
----------

In the preceding months I worked on a few more specific PHP 8.2 issues,
and generic bugs.

PHP 8.2 now presents ``__debugInfo()`` for closures, which means that
Xdebug now no longer needs to emulate those itself. This does result in
minor changes in the output that IDEs present for closures, but not with
less information.

I also fixed nearly a dozen bugs. Some were related to the new return
value debugging feature, or specific PHP 8.2 support.

A few more bugs are outstanding, and I intend to release Xdebug 3.2 when
PHP 8.2.0 comes out at the end of November. I will also create one more
Xdebug 3.1 release (3.1.6), to address a specific crash bug. Xdebug 3.2
will only support PHP 8.0 and later.

PhpStorm's latest `2022.3 EAP #5
<https://blog.jetbrains.com/phpstorm/2022/11/phpstorm-2022-3-early-access-5/>`_
release now has support for Xdebug 3.2's return value debugging, as well
as a bunch of other new features to make it easier to set up Xdebug.

I have also decided to no longer actively test on 32-bit platforms on
CI, which also means I will no longer be providing 32-bit DLLs of Xdebug
on Windows.


Xdebug Cloud
------------

Xdebug Cloud is the *Proxy As A Service* platform to allow for debugging
in more scenarios, where it is hard, or impossible, to have Xdebug make
a connection to the IDE. It is continuing to operate as Beta release.

Previously Xdebug Cloud was only supported by PhpStorm, but the PHP
Debug Adaptor for Visual Studio Code now also supports it.

Packages start at £49/month, and I have recently introduced a package
for larger companies. This has a larger initial set of tokens, and
discounted extra tokens.

If you want to be kept up to date with Xdebug Cloud, please sign up to
the `mailinglist <https://xdebug.cloud/newsletter>`_, which I will use
to send out an update not more than once a month.

Xdebug Videos
-------------

I have published two new videos:

- `Xdebug 3: Debugging with VIM and Vdebug <https://www.youtube.com/watch?v=6CIOxguEQao>`_
- `Xdebug 3: Start Upon an Error <https://www.youtube.com/watch?v=jEjo6wyTggA>`_

I have started writing scripts for videos about Xdebug 3.2's features,
and am also intending to make videos about "Running Xdebug in
Production" and "Debugging Worker Tasks with
xdebug_connect_to_client()".

You can find all previous videos on my `YouTube channel
<https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_.

Business Supporter Scheme and Funding
-------------------------------------

In August, September, and October, no new business supporters signed up.

If you, or your company, would also like to support Xdebug, head over to
the `support <https://xdebug.org/support>`_ page!

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page, a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_, as well as an `OpenCollective
<https://opencollective.com/xdebug>`_ organisation.
