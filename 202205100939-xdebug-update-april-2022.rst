Xdebug Update: April 2022
=========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2022-05-10 09:39 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-22apr

In this monthly update I explain what happened with Xdebug development in this
past month. These are normally published on the first Tuesday on or after the
5th of each month.

`Patreon <https://www.patreon.com/derickr>`_ and `GitHub
<https://github.com/sponsors/derickr/>`_ supporters will get it earlier,
around the first of each month.

You can `become a patron <https://www.patreon.com/bePatron?u=7864328>`_ or
support me through `GitHub Sponsors <https://github.com/sponsors/derickr>`_.
I am currently 46% towards my $2,500 per month goal.
If you are leading a team or company, then it is also possible to support
Xdebug through `a subscription <https://xdebug.org/support>`_.

In April, I spend 29 hours on Xdebug, with 27 hours funded.

Development
-----------

I continued my exploration of different set-ups that developers user, and have
created a prototype branch that adds support for the "pseudo-host"
``xdebug://gateway`` which can be used with the ``xdebug.client_host`` setting
instead of, and in addition to the Docker specific ``host.docker.internal``.
This pseudo-host automatically evaluates to the network gateway address in the
container, which will then allow Xdebug to connect to an IDE on the host
machine.

This value will work for:

- PHP/Xdebug running in Docker on Linux, Windows, and macOS; with the
  IDE on the host.
- PHP/Xdebug running in WSL, with the IDE on the (Windows) host.
- PHP/Xdebug running in Docker in Ubuntu on WSL, with the (Linux) IDE running in WSL.

It will not work where network isolation is used, for example with the
following set-ups:

- PHP/Xdebug running in root-less Docker on Linux, IDE on (Linux) host
- PHP/Xdebug running in Docker for Windows in WSL, with the Linux version of
  the IDE inside WSL.

I have not figured out how to do it will all different set-ups, so if you have
extra information, or if I am still missing set-ups, feel free to comment on
the `Google Doc
<https://docs.google.com/document/d/1W-NzNtExf5C4eOu3rRQm1WlWnbW44u3ANDDA49d3FD4/edit?usp=sharing>`_.

In the more complicated set-ups, it would likely be easier to use `Xdebug
Cloud <https://xdebug.cloud>`_ as it has none of these networking
complications.

Course
------

I have spend a fair amount of time developing the Xdebug course, with the
first lesson now written and ready to be recorded. I have decided to include a
"Tech Corner" with each lesson, to explain how Xdebug interacts with PHP to do
the things that are explained in each lesson. Hopefully you'll find this
interesting as well. It will also serve to reduce the bus factor.

For further lessons I have started to draft outlines, and they are in
different states of completion.

If you want to make sure that the course covers specific tasks that you find
hard to do, or what you would like explained, please drop me an email, or
leave a comment.

Xdebug Recorder
---------------

I have been making little progress as I have been focused on developing the
course. There is a persistent bug while creating the recording, which is hard
to track down, as it only happens occasionally and does not seem to be easily
reproducible.

You can follow the development in the `recorder
<https://github.com/derickr/xdebug/tree/recorder>`_ branch.

Xdebug Cloud
------------

Xdebug Cloud is the *Proxy As A Service* platform to allow for debugging in
more scenarios, where it is hard, or impossible, to have Xdebug make a
connection to the IDE. It is continuing to operate as Beta release.
Packages start at £49/month.

If you want to be kept up to date with Xdebug Cloud, please sign up to the
`mailinglist <https://xdebug.cloud/newsletter>`_, which I will use to send out
an update not more than once a month.

Xdebug Videos
-------------

I have published one new video this month:

- `Xdebug 3: Laravel Sail with PhpStorm <https://www.youtube.com/watch?v=Xgn0EtB4chc>`_

You can find all previous videoes on my `YouTube channel
<https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_.

Business Supporter Scheme and Funding
-------------------------------------

In April, no new business supporters signed up.

If you, or your company, would also like to support Xdebug, head over to the
`support <https://xdebug.org/support>`_ page!

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page and a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_.
