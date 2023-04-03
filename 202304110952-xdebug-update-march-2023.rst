Xdebug Update: March 2023
===========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2023-04-11 09:52 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-23mar

In this monthly update I explain what happened with Xdebug development
in this past two months. These are normally published on the first
Tuesday on or after the 5th of each month.

`Patreon <https://www.patreon.com/derickr>`_ and `GitHub
<https://github.com/sponsors/derickr/>`_ supporters will get it earlier,
around the first of each month.

You can `become a patron <https://www.patreon.com/bePatron?u=7864328>`_
or support me through `GitHub Sponsors
<https://github.com/sponsors/derickr>`_. I am currently 34% (7% less
than two months ago) towards my $2,500 per month goal, which is set to
allow continued maintenance of Xdebug.

If you are leading a team or company, then it is also possible to
support Xdebug through `a subscription <https://xdebug.org/support>`_.

In the last month, I spend 18 hours on Xdebug, with 22 hours funded.
Sponsorships through GitHub sponsors have now also drastically declined.
Unless this is reversed, I would find it hard to spend the effort in
making sure Xdebug continues to be updated for newer PHP versions. It
certainly makes me think hard as to where to put my dedication towards.

This is also why I have not been as diligent with these update reports
and been as active in resolving issues and bugs.

Xdebug 3.2.1
------------

March however did see the release of Xdebug 3.2.1, which finally addressed the
longer running issue with Xdebug's ``xdebug.mode`` setting being read and
interpreted wrongly, sometimes also causing crashed due to mismatches in what
Xdebug thought it had enabled, and what was actually enabled.

This does mean, that since Xdebug 3.1, the ``xdebug.mode`` setting now can
only be set in ``php.ini``, and not through per-directory or per-FPM-pool
settings with ``php_admin_value``. This has delayed creating a video about
running Xdebug in a production environment as well, as originally I had
thought to do that through a specific PHP-FPM pool.

I did also work on starting to make Xdebug 3.3, the next version, to be
compatible with PHP 8.3 which has made some changes that mostly required
changes to Xdebug's test cases.

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

Xdebug Videos
-------------

I have published one new video in the last two months:

- `Xdebug 3: Debugging Remote Code with VS Code
  <https://www.youtube.com/watch?v=UvElBs37JLg>`_ — this is not a common way
  of debugging code on a remote machine, but it is nevertheless a feature that
  Xdebug in combination with the VS Code plugin provides.

I have continued writing scripts for videos about Xdebug 3.2's features,
and am also intending to make a video about "Running Xdebug in
Production", and the updated "xdebug.client_discovery_header" feature (from
Xdebug 3.1).

Let me know what you'd like to see!

You can find all previous videos on my `YouTube channel
<https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_.

Business Supporter Scheme and Funding
-------------------------------------

In February and March, no new business supporters signed up.

If you, or your company, would also like to support Xdebug, head over to
the `support <https://xdebug.org/support>`_ page!

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page, a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_, as well as an `OpenCollective
<https://opencollective.com/xdebug>`_ organisation.
