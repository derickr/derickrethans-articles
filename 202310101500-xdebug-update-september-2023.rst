Xdebug Update: September 2023
=============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2023-10-10 15:00 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-23sep

In this monthly update I explain what happened with Xdebug development
in the past month. These are normally published on the first
Tuesday on or after the 5th of each month.

`Patreon <https://www.patreon.com/derickr>`_ and `GitHub
<https://github.com/sponsors/derickr/>`_ supporters will get it earlier,
around the first of each month.

You can `become a patron <https://www.patreon.com/bePatron?u=7864328>`_
or support me through `GitHub Sponsors
<https://github.com/sponsors/derickr>`_. I am currently 37% towards my $2,500
per month goal, which is set to allow continued maintenance of Xdebug.

If you are leading a team or company, then it is also possible to
support Xdebug through `a subscription <https://xdebug.org/support>`_.

In the last month, I spend around 28 hours on Xdebug, with 25 hours funded.

Towards Xdebug 3.3
------------------

In September I released a first alpha release of Xdebug 3.3 so that people
trying out PHP 8.3 Release Candidates have a compatible Xdebug to test with.

This quickly followed by a second alpha version due to issues with the PECL
website. Instead of mangling UTF-8 characters, it stopped accepting them
altogether.

I have reintroduced the ``xdebug.collect_params`` setting, which I had removed
in Xdebug 3.0. Instead of the setting, Xdebug would just always collect
functions' and methods' arguments while tracing. However, some users were
suggesting that this created too much information which was not always needed.
With the setting restored, you can now again hide these function arguments
from trace files.

As frameworks are getting more complicated, they are more likely to hit
Xdebug's default ``xdebug.max_nesting_level`` limit of ``256``. In Xdebug 3.3,
this will now be ``512``.

The maximum nesting level setting is now less important that it was all these
years ago. The PHP engine now uses stack in a more economic way. This feature
unfortunately is negated when extensions override PHP's internal execution
method, which is what Xdebug has to do to capture function calls for tracing,
profiling, and certain breakpoints.

In PHP 8.1 a new Observer API was added, which would allow extensions to
observe user-land function calls without having to override the internal
execution method. This means that the stack is used more sparingly again. It
also would allow for these extensions to work better with opcache enabled.

I am currently in the process of investigating whether Xdebug can make use of this
Observer API as well, while maintaining all its functionality and without BC
breaks. I will keep you updated in the next monthly update.

Beyond this, I will continue to work on the features and issues on the `3.3
roadmap <https://bugs.xdebug.org/roadmap_page.php?version_id=101>`_, without
any guarantees these tickets will be implemented.

Xdebug Videos
-------------

I have published one new videos in the last month:

- `Xdebug 3: Using the DBGp Proxy <https://www.youtube.com/watch?v=_3RkGZK-UC8>`_

Let me know what you'd like to see!

You can find all previous videos on my `YouTube channel
<https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_.

Business Supporter Scheme and Funding
-------------------------------------

In the last month, no new business supporters signed up.

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page, a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_, as well as an `OpenCollective
<https://opencollective.com/xdebug>`_ organisation.

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
