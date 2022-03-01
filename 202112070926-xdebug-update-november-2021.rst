Xdebug Update: November 2021
=============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-12-07 09:26 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-21nov

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

In November, I worked on Xdebug directly for only about 16 hours, with funding
being around 24 hours. Please become a supporter of Xdebug through Patreon or
GitHub.

Xdebug 3.1 and further
----------------------

Since my last report, there were no more new issues reported with either
functionality or crashes, and I have prepared for the Xdebug 3.1.2 release.
Although I didn't publish that in November, I did release it on December 1st.

It addresses a few crash bugs related to PHP 8.1 fibers, a crash bug when
Xdebug can't write a profiler file, and an issue with Xdebug's var_dump() not
using the magic __debugInfo method. 

The full list of changes can be found on the `updates
<https://xdebug.org/updates#x_3_1_2>`_ page on the Xdebug website. 

So what's next? Xdebug's profiler needs a rewrite. There are several design
and implementation issues, that can only be resolved by a rewrite. To create
less overhead it can also use PHP 8's new Observer API as well.

But this is going to take a large amount of work, and with the current low,
and decreasing, funding levels, I can not put a lot of effort behind this.

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

I have published more videos on how to use Xdebug on my `YouTube channel
<https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_.

- `Debugging: Short Closures and Conditional Breakpoints
  <https://youtu.be/_LTKyUBgPaE>`_ explains debugging with Short Closures and
  Conditional Breakpoints.
- `Xdebug 3: Setting up Apache, PHP, VS Code, and Xdebug in 10 minutes
  <https://youtu.be/MmyxWy8jl7U>`_ shows how to install Apache, PHP, VS Code,
  and Xdebug on Ubuntu 21.10, to get a PHP development set-up, all within 10
  minutes.

I will continue to create more videos, and also convert some of them to
tutorials.

If you would like to suggest a topic for a 5 to 15 minute long video, feel
free to request them through this `Google Form
<https://forms.gle/ugjGbxs6ZhiTyvCSA>`_.

Business Supporter Scheme and Funding
-------------------------------------

In November, no new business supporters signed up.

If you, or your company, would also like to support Xdebug, head over to the
`support <https://xdebug.org/support>`_ page!

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page and a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_.
