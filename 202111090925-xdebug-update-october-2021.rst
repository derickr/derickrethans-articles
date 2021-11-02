Xdebug Update: October 2021
=============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-11-09 09:25 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-21oct

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

In October, I worked on Xdebug for about 30 hours, with funding being
around 25 hours. Please become a supporter of Xdebug through Patreon or
GitHub.

Xdebug 3.1
----------

Xdebug 3.1.0 and 3.1.1 have now been released!

Xdebug 3.1 adds support for PHP 8.1, as well as a set of new features. These
new features include: file compression for trace and profiling files, the new
xdebug_notify() and xdebug_connect_to_client() functions, an API through
xdebug_info() to request which modes are enabled, and the possibility to set
the Xdebug Cloud ID through Xdebug's triggers, including browser extensions.

The full list of changes can be found on the `updates
<https://xdebug.org/updates#x_3_1_0>`_ page on the Xdebug website. 

Unfortunately Xdebug 3.1.0 introduced some bugs in the thread-safe version,
which is most notably used on Windows. After some back and forth with
Christoph Becker, the PHP on Windows maintainer, we managed to track down the
problem and fix it. This resulted with some other fixes in the Xdebug 3.1.1
release.

Since then, users found a few more minor issues which will result in 3.1.2, to
be released early in November.

Xdebug Cloud
------------

To help with funding my work on Xdebug, I have a paid-for-service, called
Xdebug Cloud.

Xdebug Cloud is the *Proxy As A Service* platform to allow for debugging in
more scenarios, where it is hard, or impossible, to have Xdebug make a
connection to the IDE. It is continuing to operate as Beta release.

One of the new features in Xdebug 3.1 makes it possible to allow for each team
member in a team that uses a (remote) shared development server to have their
own debugging session between the development server and their own IDE,
without interfering with other developers. 

Packages start at £49/month, and revenue will be used to further the
development of Xdebug.

If you want to be kept up to date with Xdebug Cloud, please sign up to the
`mailinglist <https://xdebug.cloud/newsletter>`_, which I will use to send out
an update not more than once a month.

Xdebug Videos
-------------

I have published one new video on how to use Xdebug on my `YouTube channel
<https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_.

This video is of a series to explain how to use Xdebug and the new features
introduced in Xdebug 3.1:

- `Xdebug 3.1: Improvements to Step Debugging <https://youtu.be/aLF6j5qdnvA>`_:
  Explains how to use and configure the new file compression support for
  profiling and trace files.

I will continue to create more videos, and also convert some of them to
tutorials.

If you would like to suggest a topic for a 5 to 15 minute long video, feel
free to request them through this `Google Form
<https://forms.gle/ugjGbxs6ZhiTyvCSA>`_.

Business Supporter Scheme and Funding
-------------------------------------

In October, no new business supporters signed up.

If you, or your company, would also like to support Xdebug, head over to the
`support <https://xdebug.org/support>`_ page!

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page and a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_.
