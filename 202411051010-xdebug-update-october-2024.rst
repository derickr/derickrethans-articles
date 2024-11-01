Xdebug Update: October 2024
===========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2024-11-05 10:10 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-24oct

In this monthly update I explain what happened with Xdebug development

`GitHub <https://github.com/sponsors/derickr/>`_ and `Pro/Business supporters
<https://xdebug.org/support>`_ will get it earlier, around the first of each
month.

On GitHub sponsors, I am currently 31% towards my $2,500 per month goal, which
is set to allow continued **maintenance** of Xdebug. This is again less than
last month.

If you are leading a team or company, then it is also possible to
support Xdebug through `a subscription <https://xdebug.org/support>`_.

In the last month, I spend around 10 hours on Xdebug, with 19 hours funded.
I also spent 16 hours on the Native Path Mapping project.

PHP 8.4
-------

I made the first beta release of Xdebug 3.4 that is compatible with PHP 8.4,
which is due to be released on November 21st. I hope to have a GA release of
Xdebug 3.4.0 out around that date too.

Beyond making that beta release, I have also looked into many bug reports, and
fixed a few bugs.

As side-project to the new PIE installer, which is expected to replace PECL in
the near future, a new Windows extension build action for GitHub actions has
been created. I have been evaluating that, and provided some feedback. This is
not quite ready for prime time, and I am not sure yet whether I will be able
to use that for the 3.4 releases.

Native Xdebug Path Mapping
--------------------------

I also continued with the `Native Xdebug Path Mapping
<https://xdebug.org/funding/001-native-path-mapping>`_ project.

This is a separately funded project, I don't classify the hours that I
worked on this in "Xdebug Hours". I also publish a separate report on the
`project page <https://xdebug.org/funding/001-native-path-mapping>`_.

In short, I have finished the parser, and also made some performance
improvements in the way I store data internally. I dive into the depths of
that in its `dedicated report
<https://xdebug.org/funding/001-native-path-mapping/updates/2024-11-01>`_.

I am now working on integrating it into Xdebug, and realised that there is one
thing that I had forgotten to add, and that is a reverse mapping. The
algorithms can only map remote a file/line to a local file/line, but in order
to be able for an IDE to set a breakpoint on a local file line (which is how
you would edit it), it also needs to have a reverse map. 

By the end of this month I hope to have a working version to try out, but it
will likely not be in the Xdebug 3.4 release, as that has to be released near
when PHP 8.4 is released.


Xdebug Videos
-------------

I have created no new videos since last month, but you can see all the
previous ones on my `channel
<https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_.

If you have any suggestions, feel free to reach out to
`me on Mastodon <https://phpc.social/@derickr>`_ or via `email
<http://derickrethans/who.html>`_.

Business Supporter Scheme and Funding
-------------------------------------

In the last month, no new business supporters signed up.

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page, a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_, as well as an `OpenCollective
<https://opencollective.com/xdebug>`_ organisation.

If you want to contribute to specific projects, you can find those on the
`Projects <https://xdebug.org/funding>`_ page.

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
