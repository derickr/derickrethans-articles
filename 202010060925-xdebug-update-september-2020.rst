Xdebug Update: September 2020
=============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-10-06 09:25 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-20sep

Another monthly update where I explain what happened with Xdebug development
in this past month. These will be published on the first Tuesday after the 5th
of each month.

`Patreon <https://www.patreon.com/derickr>`_ and `GitHub
<https://github.com/sponsors/derickr/>`_ supporters will
get it earlier, on the first of each month.

**I am currently looking for more funding.**

You can `become a patron <https://www.patreon.com/bePatron?u=7864328>`_ or
support me through `GitHub Sponsors <https://github.com/sponsors/derickr>`_
I am currently 59% towards my $1,000 per month goal.

If you are leading a team or company, then it is also possible to support
Xdebug through `a subscription <https://xdebug.org/support>`_.

In September, I worked on Xdebug for about 60 hours, with funding being around 70
hours. I worked mostly on the following things:

Xdebug 3
--------

This month I mostly focussed on getting Xdebug 3 in shape for a first beta
release, with all the new configuration names in place. There are now only a
few `tasks <https://bugs.xdebug.org/roadmap_page.php?version_id=83>`_ before I
can release Xdebug 3.0.0beta1. I plan to release this around PHP 8.0RC2.

The main changes that I made was to rename the following four configuration
settings, mostly to get "rid" of the *remote* naming:

- ``xdebug.remote_host`` → ``xdebug.client_host``
- ``xdebug.remote_port`` → ``xdebug.client_port``
- ``xdebug.remote_connect_back`` → ``xdebug.discover_client_host``
- ``xdebug.remote_addr_header`` → ``xdebug.client_discovery_header``

I hope that these new names are easier to explain, and of course the `upgrade
guide <https://3.xdebug.org/docs/upgrade_guide>`_ explains the changes too.


PHP 8
-----

Although RC1 of PHP 8 is released today, there are still numerous items still
in flux. In September there were various changes due to PHP 8's new named
parameters. This mostly has an effect on Xdebug's tests, where these names are
exposed as part of tracing or debugging tests. Because there are still
changes, I am delaying Xdebug 3.0.0beta1 until PHP 8.0RC2.

Other API changes also meant that I had to make changes in Xdebug. Most
notably related to code coverage, where PHP 8 will emit fewer Op codes and now
(finally!) creates ASSIGN operations on the correct line (which makes Xdebug's
breakpoint resolving feature less necessary).

Releases
--------

There were two Xdebug releases in September. `2.9.7
<https://xdebug.org/announcements/2020-09-16>`_ changes the step debugger to
set up TCP Keepalive probes. This results in better time-out management in
case network connections between Xdebug and an IDE drops.

Unfortunately this patch caused compilation issues on FreeBSD where some OS
specific flags are different (but the same as OSX, which Xdebug did handle
correctly). A fix for this, as well as a fix for path/branch coverage with
foreach loops resulted in the `2.9.8
<https://xdebug.org/announcements/2020-09-28>`_ release. I expect to create
one more release related to the TCP Keepalive addition as the current release
still does not compile for AIX.

Beyond this, I do not expect any more release of the Xdebug 2.9 series unless
security or crash bugs are present.


Xdebug Cloud
------------

Beyond fixing an off-by-one error in host name generation, I did not write any
code for Xdebug Cloud. However, work is ongoing on a web site, and JetBrains
is also working on supporting Xdebug Cloud in PhpStorm. Reports state that a
prototype is now working. With one IDE soon to support Xdebug Cloud I am also
sharing the protocol changes with other IDE/debug client authors before I make
it part of the `DBGp specification
<https://github.com/derickr/dbgp/blob/master/debugger_protocol.rst>`_.

If you want to be kept up to date with Xdebug Cloud, please sign up to the
`mailinglist <http://cloud.xdebug.com>`_ I'll let you know as soon as
something can be tried-out. 

Business Supporter Scheme and Funding
-------------------------------------

In September, no new supporters signed up, but four existing supports renewed.

**Thank you `Tideways <https://tideways.com>`_, `TYPO3 <https://typo3.com>`_,
Stephen Reay, and `James Titcumb <https://twitter.com/asgrim>`_!**

If you, or your company, would also like to support Xdebug, head over to the
`support <https://xdebug.org/support>`_ page!

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page and a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_.
