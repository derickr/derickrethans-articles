Xdebug Update: January 2023
===========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2023-02-07 09:52 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-23jan

In this monthly update I explain what happened with Xdebug development
in this past month. These are normally published on the first Tuesday on
or after the 5th of each month.

`Patreon <https://www.patreon.com/derickr>`_ and `GitHub
<https://github.com/sponsors/derickr/>`_ supporters will get it earlier,
around the first of each month.

You can `become a patron <https://www.patreon.com/bePatron?u=7864328>`_
or support me through `GitHub Sponsors
<https://github.com/sponsors/derickr>`_. I am currently 41% (4% less
than last month) towards my $2,500 per month goal. If you are leading a
team or company, then it is also possible to support Xdebug through `a
subscription <https://xdebug.org/support>`_.

In the last month, I spend 18 hours on Xdebug, with 26 hours funded.
Sponsorships, especially through Patreon, are continuing to decline,
which makes it harder for me to dedicate time for maintenance and
development.

Xdebug 3.2
----------

I have continued to triage new bug reports in Xdebug 3.2, and most
notably trying to find the bug that people reported with regards to the
``xdebug.mode`` setting not sticking, or being wrong. So far I have not
managed to reproduce this in a reliable environment. If you run into
this bug, please get in contact so that I can figure out what the cause
is, by being able to reproduce this.

During a support call with one of my supporters, we discussed an issue
that prevented correct breakpoints from being set through the `PHP Debug
Adapter for Visual Studio Code
<https://marketplace.visualstudio.com/items?itemName=xdebug.php-debug>`_
for virtual file systems, such as with the `SSH FS
<https://marketplace.visualstudio.com/items?itemName=Kelvin.vscode-sshfs>`_
plug-in. The debug adaptor did not know how to use a path mapping with
this specific schema to work. There is now a new release (1.13.0) of the
plug-in
(https://marketplace.visualstudio.com/items/xdebug.php-debug/changelog) to
make the following work::

	"pathMappings": {
	    "/home/derick/dev": "ssh://singlemalt/home/derick/dev"
	},

Which maps the server path ``/home/derick/dev/`` to the virtual file
path ``ssh:://singlemalt/home/derick/dev``, which I have added to my
workspace.


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

I have published no new videos this month.

I have continued writing scripts for videos about Xdebug 3.2's features,
and am also intending to make a video about "Running Xdebug in
Production", the updated "xdebug.client_discovery_header" feature (from
Xdebug 3.1), and the new SSH FS path mapping functionality in the PHP
Debug Adaptor for VS Code.

You can find all previous videos on my `YouTube channel
<https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_.

Business Supporter Scheme and Funding
-------------------------------------

In January, no new business supporters signed up.

If you, or your company, would also like to support Xdebug, head over to
the `support <https://xdebug.org/support>`_ page!

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page, a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_, as well as an `OpenCollective
<https://opencollective.com/xdebug>`_ organisation.
