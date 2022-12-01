Xdebug Update: November 2022
============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2022-12-06 09:06 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-22nov

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

In the last month, I spend 30 hours on Xdebug, with 24 hours
funded. Sponsorships are declining, which makes it harder for me to dedicate
time for maintenance and development.

Xdebug 3.2
----------

I spend most of November fixing outstanding bugs for Xdebug 3.2, so that
it is ready to be released when PHP 8.2.0 comes out at the start of
December. 

I also a fair amount of time triaging a crash bug with a segfault when
code coverage in use, which is likely related to generators, but I
haven't managed to fully check out yet. I did also find a use-after-free
error that I have now fixed.

As part of this, I have released Xdebug 3.1.6 to address a compressed
file writing bug on Windows, and Xdebug 3.2.0RC2 so that you are able to
test Xdebug 3.2 with PHP 8.2 with all the outstanding bugs addressed.

Once Xdebug 3.2 gets released next week, support for PHP 7 and Xdebug
3.1 (and lower) will no longer be available. This of course does not
mean that older versions of Xdebug are no longer available for download
to use with legacy PHP versions.

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

I have published one new videos:

- `Xdebug 3: xdebug_connect_to_client() <https://youtu.be/N-Hw12bBGDE>`_

I have continued writing scripts for videos about Xdebug 3.2's features,
and am also intending to make a video about "Running Xdebug in
Production".

You can find all previous videos on my `YouTube channel
<https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_.

Business Supporter Scheme and Funding
-------------------------------------

In November, no new business supporters signed up.

If you, or your company, would also like to support Xdebug, head over to
the `support <https://xdebug.org/support>`_ page!

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page, a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_, as well as an `OpenCollective
<https://opencollective.com/xdebug>`_ organisation.
