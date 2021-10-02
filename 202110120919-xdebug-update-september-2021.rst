Xdebug Update: September 2021
=============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-10-12 09:19 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-21sep

In this monthly update I explain what happened with Xdebug development
in this past month. These will be published on the first Tuesday after the 5th
of each month.

`Patreon <https://www.patreon.com/derickr>`_ and `GitHub
<https://github.com/sponsors/derickr/>`_ supporters will get it earlier,
around the first of each month.

You can `become a patron <https://www.patreon.com/bePatron?u=7864328>`_ or
support me through `GitHub Sponsors <https://github.com/sponsors/derickr>`_.
I am currently 57% towards my $2,000 per month goal.
If you are leading a team or company, then it is also possible to support
Xdebug through `a subscription <https://xdebug.org/support>`_.

In September, I worked on Xdebug for about 45 hours, with funding being
around 25 hours, which is just under half.

Xdebug 3.1
----------

I have continued to work on the task list for Xdebug 3.1, cumulating in the
release of Xdebug 3.1.0beta1 and beta2, with the latter being a repackaging
due to Windows binary file naming. I have changed from using AppVeyor to
GitHub Actions for the building of Windows binaries, which take much much
time. Unfortunately I swapped the Non-TS and TS binaries for that build.

Since 3.1.0beta2, Christoph Becker, with some help from me, has been adding
support for file compression on Windows as well, by integrating zlib. This was
the last outstanding task which means that Xdebug 3.1.0 is now ready to be
released. During the beta phase some bugs were brought to light which I have
also addressed.

Xdebug Cloud
------------

To help with funding my work on Xdebug, I have a paid-for-service, called
Xdebug Cloud.

Xdebug Cloud is the *Proxy As A Service* platform to allow for debugging in
more scenarios, where it is hard, or impossible, to have Xdebug make a
connection to the IDE. It is continuing to operate as Beta release.

I have now released the work that I have been doing on the admin site, to make
it easier to configure your session tokens and see subscription information.

Xdebug 3.1 also has new features to make it possible to allow for each team
member in a team that uses a (remote) shared development server to have their
own debugging session between the development server and their own IDE,
without interfering with other developers. The Xdebug Cloud and Multiple
Triggers video that I've linked below, explains this in greater detail.

Packages start at £49/month, and revenue will be used to further the
development of Xdebug.

If you want to be kept up to date with Xdebug Cloud, please sign up to the
`mailinglist <https://xdebug.cloud/newsletter>`_, which I will use to send out
an update not more than once a month.

Xdebug Videos
-------------

I have published three more videos on how to use Xdebug on my `YouTube channel
<https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_.

These are part of a series to explain how to use Xdebug and the new features
introduced in Xdebug 3.1:

- `Xdebug 3.1: File Compression <https://youtu.be/aZ4eH8J0uuA>`_:
  Explains how to use and configure the new file compression support for
  profiling and trace files.
- `Xdebug 3.1: xdebug_info() Improvements <https://youtu.be/S4Juvb3k3co>`_:
  Introduces what debugging information ``xdebug_info()`` can provide, and
  focusses the additions in Xdebug 3.1.
- `Xdebug 3.1: Xdebug Cloud and Multiple Triggers <https://youtu.be/Jny-RJDf2AM>`_:
  Explains how to use the new multiple shared secrets and "Cloud Token" as
  debugging session ID feature to support more use cases for Xdebug Cloud.

I will continue to create more videos, and also convert some of them to
tutorials.

If you would like to suggest a topic for a 5 to 15 minute long video, feel
free to request them through this `Google Form
<https://forms.gle/ugjGbxs6ZhiTyvCSA>`_.

Business Supporter Scheme and Funding
-------------------------------------

In September, no new business supporters signed up.

If you, or your company, would also like to support Xdebug, head over to the
`support <https://xdebug.org/support>`_ page!

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page and a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_.
