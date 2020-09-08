Xdebug Update: August 2020
==========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-09-08 09:44 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-20aug

Another monthly update where I explain what happened with Xdebug development
in this past month. These will be published on the first Tuesday after the 5th
of each month. `Patreon <https://www.patreon.com/derickr>`_ and `GitHub
<https://github.com/sponsors/derickr/>`_ supporters will
get it earlier, on the first of each month. You can `become a patron
<https://www.patreon.com/bePatron?u=7864328>`_ to support my work on Xdebug.
If you are leading a team or company, then it is also possible to support
Xdebug through `a subscription <https://xdebug.org/support>`_.

In August, I worked on Xdebug for about 80 hours, with funding being around 70
hours. I worked mostly on the following things:

Xdebug 3
--------

This month I focussed on making it easier to spot what Xdebug is doing, and
how and why it is trying to connect to a debugging client, creating files,
etc. For this I reworked Xdebug's logging mechanism. Instead of it just being
used for the step debugger, the logging mechanism is now used for everything,
and as part of that I've renamed ``xdebug.remote_log`` to ``xdebug.log``. The log
will contain all messages with levels that match the ``xdebug.log_level``
setting, but Xdebug will now also send errors through PHP's standard error
mechanism. Errors include not being able to open a log file, to create a
profile or trace file, or issues with making debugging connections.

The biggest new addition is the ``xdebug_info()`` function , which acts as a
Xdebug specific ``phpinfo()`` and contains configuration information about
Xdebug, but more importantly lists warnings and errors. Where possible, a link
to the documentation is included as well, where I set up a new `page
<https://3.xdebug.org/docs/errors>`_ that includes all the warning and error
messages that Xdebug can generate. I have tried to add information about how
a specific message is created, and what you can do to resolve the issue if
possible.

Modes and Docker
~~~~~~~~~~~~~~~~

Xdebug 3 introduces modes that can be configured with the ``xdebug.mode``
setting. This setting can only be changed in ``php.ini``, and not in an
``.htaccess`` file or equivalent, nor in the script with ``ini_set``. Because
Docker and containers are a more common development set-up now, a method was
needed to be able to change Xdebug's mode without have to rebuild the
containers. Because of that, I added support for configuring the mode through
a new environment variable, ``XDEBUG_MODE``, too. This can be used when starting
your Docker container, and/or in `Compose
<https://docs.docker.com/compose/environment-variables/>`_.

PHP 8
-----

Now PHP 8 is getting closer I have started to pay a little bit more attention
to adding support for features, beyond making it compile. PHP 8 introduces
named parameters, and also has a project for adding (and updating) names for
internal functions. Xdebug's tracing and logging already showed variable names
for user-defined functions, and I've now adding support of doing that for
built-in functions and methods as well.

Showing argument names in Xdebug 2 is controlled through setting
``xdebug.collect_params`` to ``4``. In Xdebug 3 I have removed this setting
altogether, and argument names are now always visible if possible.

My focus in September is to add support for the rest of PHP 8's features and
capabilities as well.

Xdebug Cloud
------------

The only time I spend in August on Xdebug Cloud is some further conversations
with JetBrains. I am going to work on Xdebug Cloud when these conversations
move along a little further.

If you want to be kept up to date with Xdebug Cloud, please sign up to the
`mailinglist <http://cloud.xdebug.com>`_ I'll let you know as soon as
something can be tried-out. 

Business Supporter Scheme and Funding
-------------------------------------

In August, no new supporters signed up.

If you, or your company, would also like to support Xdebug, head over to the
`support <https://xdebug.org/support>`_ page!

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page and a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_.
