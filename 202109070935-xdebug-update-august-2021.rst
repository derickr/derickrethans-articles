Xdebug Update: August 2021
==========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-09-07 09:35 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-21aug

In this monthly update I explain what happened with Xdebug development
in this past month. These will be published on the first Tuesday after the 5th
of each month.

`Patreon <https://www.patreon.com/derickr>`_ and `GitHub
<https://github.com/sponsors/derickr/>`_ supporters will get it earlier,
around the first of each month.

You can `become a patron <https://www.patreon.com/bePatron?u=7864328>`_ or
support me through `GitHub Sponsors <https://github.com/sponsors/derickr>`_.
I am currently 58% towards my $2,000 per month goal.
If you are leading a team or company, then it is also possible to support
Xdebug through `a subscription <https://xdebug.org/support>`_.

In August, I worked on Xdebug for about 50 hours, with funding being
around 25 hours, which is only **half**.

Xdebug 3.1
----------

I have continued to work on the task list for Xdebug 3.1, with now just one
issue outstanding. This means it's nearing a first beta release, which I am
hoping to release close to when PHP 8.1 RC1 gets released.

The main thing that I worked on is compression support for profiling and trace
files. Previously, I already had added preliminary support for compressed
profiling files, but it turned out that QCacheGrind could not read them.
Because Xdebug would always use compressed files, if the zlib compression
library was available, it would generate non-consumable files.

With the improved feature it is now possible to disable the compression with
the `xdebug.use_compression=0` setting. And additionally I have also support
to compressed trace files. The zlib/gz drastically reduces the size of files.
It is currently not yet available for Windows users.

Xdebug Cloud
------------

To help with funding my work on Xdebug, I have a paid-for-service, called
Xdebug Cloud.

Xdebug Cloud is the *Proxy As A Service* platform to allow for debugging in
more scenarios, where it is hard, or impossible, to have Xdebug make a
connection to the IDE. It is continuing to operate as Beta release.

I have been improving the admin side and I will soon release a new beta that
show cases that work.

Packages start at £49/month, and revenue will be used to further the
development of Xdebug.

If you want to be kept up to date with Xdebug Cloud, please sign up to the
`mailinglist <https://xdebug.cloud/newsletter>`_, which I will use to send out
an update not more than once a month.

Xdebug Videos
-------------

I have published two more videos on how to use Xdebug on my `YouTube channel
<https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_.

These are part of a series to explain how to use Xdebug:

- `Xdebug 3 Profiling: 3. Analysing Data
  <https://www.youtube.com/watch?v=iH-hDOuQfcY>`_, where I show how to
  use KCacheGrind to read and analyse profiling files to find bottlenecks in
  code.
- `Xdebug 3: Activation and Triggers
  <https://www.youtube.com/watch?v=9Fx1beTvR2w>`_, where I explain how to
  activate Xdebug's myriad of features with different methods, including
  cookies, GET/POST parameters, environment variables, and with a browser
  extension.

I will create more videos in the upcoming month, and if you would like to
suggest a topic for a 5 to 10 minute long video, feel free to request them
through this `Google Form <https://forms.gle/ugjGbxs6ZhiTyvCSA>`_.

Business Supporter Scheme and Funding
-------------------------------------

In August, no new supporters signed up.

If you, or your company, would also like to support Xdebug, head over to the
`support <https://xdebug.org/support>`_ page!

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page and a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_.
