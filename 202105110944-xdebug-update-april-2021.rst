Xdebug Update: April 2021
=========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-05-11 09:44 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-21apr

Another monthly update where I explain what happened with Xdebug development
in this past month. These will be published on the first Tuesday after the 5th
of each month.

`Patreon <https://www.patreon.com/derickr>`_ and `GitHub
<https://github.com/sponsors/derickr/>`_ supporters will get it earlier,
around the first of each month.

You can `become a patron <https://www.patreon.com/bePatron?u=7864328>`_ or
support me through `GitHub Sponsors <https://github.com/sponsors/derickr>`_.
I am currently 93% towards my $1,000 per month goal.
If you are leading a team or company, then it is also possible to support
Xdebug through `a subscription <https://xdebug.org/support>`_.

In April, I worked on Xdebug for about 35 hours, with funding being
around 28 hours.

Xdebug 3.1
----------

I have continued to work on the Xdebug 3.1
`tasks
<https://bugs.xdebug.org/roadmap_page.php?version_id=87>`_. I have added an
API to that tools can query which modes Xdebug has enabled, no matter how;
added compression for profiling file, which both PhpStorm and
KCacheGrind/QCacheGrind can read, and I've made some changes to how errors are
reported if the Xdebug log is active.

To enable `Xdebug Cloud <https://xdebug.cloud>`_ to be used in more
situations, it is now possible to replace the "session name" that browser
extensions use to activate Xdebug with your Cloud ID. By setting the Cloud ID
through the browser extension (or with the GET/POST) parameters, you can now
share a single development server with multiple developers all using their own
IDE through Xdebug Cloud.

Xdebug Videos
-------------

I also continued to produce a few more videos on how to use Xdebug.

The first of these is `Profiling with Xdebug in Docker
<https://www.youtube.com/watch?v=8yUY063WgDg>`_, and the second one is
`Debugging Unit Tests with PhpStorm on Linux
<https://www.youtube.com/watch?v=AsBLzqj3B2U>`_. I will continue to release
more videos on YouTube, where you can also find other Xdebug videos in the
`Xdebug 3 play list
<https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_.

If you would like to see a 5 to 10 minute long video on another specific
topic, feel free to email me at derick@xdebug.org.

Xdebug Cloud
------------

`Xdebug Cloud <https://xdebug.cloud>`_ is continuing to operate as Beta
release, and provides an easy way to debug your PHP applications with Xdebug,
without having to deal with complexities with regards to networking.

Packages start at £49/month, and revenue will also be used to further the
development of Xdebug.

If you want to be kept up to date with Xdebug Cloud, please sign up to the
`mailinglist <https://xdebug.cloud/newsletter>`_, which I will use to send out
an update not more than once a month.


Business Supporter Scheme and Funding
-------------------------------------

In April, no new supporters signed up.

If you, or your company, would also like to support Xdebug, head over to the
`support <https://xdebug.org/support>`_ page!

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page and a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_.
