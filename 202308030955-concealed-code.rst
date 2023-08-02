Concealed Code
==============

.. articleMetaData::
   :Where: London, UK
   :Date: 2023-08-03 09:55 Europe/London
   :Tags: blog, php, xdebug
   :Short: concealed-code

Last week, the author of the `PHP Debug Adapter for Visual Studio Code
<https://github.com/xdebug/vscode-php-debug>`_ asked me to look at an `issue
<https://github.com/xdebug/vscode-php-debug/issues/919>`_. A user noticed that
configured breakpoints in the editor would be greyed out for any request
besides the first one for each process when using PHP's built-in web server.

Xdebug "resolves" breakpoints when it sees code compiled by PHP and then
notifies IDEs that the configured breakpoints are valid. Sometimes it also
means it moves them to a line with executable code on it, as in some cases,
PHP is "confused" about where lines of code live.

I spent some time delving into this, and initially I could not reproduce this.
On my side (Linux, PHP 8.1/8.2) with ``php -S`` the behaviour was always
correct, with the breakpoints resolved for each request through the dev
server.

When I had another good look at the ``phpinfo()`` output from the user, I
noticed::

	Zend Engine v4.2.8, Copyright (c) Zend Technologies
		with Xdebug v3.2.2, Copyright (c) 2002-2023, by Derick Rethans
		with Zend OPcache v8.2.8, Copyright (c), by Zend Technologies

The above shows that Xdebug is loaded first and OPcache second, which the
`documentation <https://xdebug.org/docs/compat#compat>`_ says you shouldn't
do:

	Zend Opcache

	Can be loaded together with Xdebug, but it is not 100% compatible.

	Load Xdebug after Opcache in ``php.ini`` for better compatibility. When
	running ``php -v`` or when looking at ``phpinfo()`` output, Xdebug should
	be listed below Opcache.

After I switched the loading order of the two Zend extensions, loaded on the
command line after ignoring (through ``-n``) the normal ``php.ini`` file
from::

	XDEBUG_MODE=debug XDEBUG_TRIGGER=yes \
		php -n -d zend_extension=opcache -d zend_extension=xdebug \
		-S localhost:9112 -t /tmp

To::

	XDEBUG_MODE=debug XDEBUG_TRIGGER=yes \
		php -n -d zend_extension=xdebug -d zend_extension=opcache \
		-S localhost:9112 -t /tmp

I **could** reproduce this issue.

The explanation for this is that both Xdebug and OPcache override PHP's
compile file handler.

Xdebug uses this to analyse newly loaded files for lines of code that can have
breakpoints to resolve them. Before doing its magic, it calls the already
present handler, nominally, the built-in PHP one that converts a PHP script
into byte code that the PHP engine can run.

OPcache uses the handler to see whether it sees a file being converted
(parsed) for a second time. If it is in its cache, it doesn't call PHP's
original compile handler again but instead returns the byte code from its
cache.

If OPcache is loaded first and then Xdebug, the following sequence occurs:

- OPcache replaces the compile file handler with ``opcache_compile_file``, and
  remembers the previous one, ``php_compile_file``.
- Xdebug replaces the compile file handler with ``xdebug_compile_file``, and
  remembers the previous one, now ``opcache_compile_file``.

In this situation, when PHP runs the compile file handler, it first calls
``xdebug_compile_file``, which then calls ``opcache_compile_file`` and all is
well.

The process reverses if OPcache is loaded last:

- Xdebug replaces the compile file handler with ``xdebug_compile_file``, and
  remembers the previous one, ``php_compile_file``.
- OPcache replaces the compile file handler with ``opcache_compile_file``, and
  remembers the previous one, now ``xdebug_compile_file``.

When PHP runs the compile file handler, it calls ``opcache_compile`` first.
OPcache checks whether it has seen the file already and, if not, calls the
previous handler (``xdebug_compile_file``), but if it **has** seen the file
already (the second request through a `php -S` server), it does **not** call
the previous compile file handler.

Typically, that is what you want, as compiling files is expensive. However,
because it does not call the previous compile file handler, that means that
``xdebug_compile_file`` does not get run, which means Xdebug doesn't know
anything about which lines of code can have breakpoints on them. It can not
resolve them henceforth.

I can not work around this in Xdebug.

Luckily, there are workarounds:

- Make sure to load Xdebug **after** OPcache — which is what the documentation
  says you should do.
- Disable OPcache by setting ``opcache.enable=0``.
- Don't load OPcache by commenting out the ``zend_extension=opcached`` line in
  ``ext-opcache.ini`` (or similar filename).

Loading Xdebug after OPcache is what you should strive for.

I usually name the `xdebug.ini` file `99-xdebug.ini` and the INI file for
OPcache ``10-opcache.ini``, to enforce the loading order by respecting the
sorting order of INI files as stored on the file system.

Alternatively, if you only have one INI file, make sure that Xdebug is listed
**after** OPcache in this file, such as in::

	zend_extension=opcache
	zend_extension=xdebug

From Xdebug 3.3, Xdebug will include a warning in the Diagnostic Log section
of ``xdebug_info()`` output to warn users that you should load Xdebug after
OPcache.

As with all warnings in the HTML version of Xdebug's Diagnostic Log, there is
also a `link to documentation <https://xdebug.org/docs/errors#DBG-W-OPCACHE>`_
which explains what the problem is and what possible solutions are.

I have also made a video about the Xdebug 3 Diagnostic Log, which you can find
on `YouTube <https://www.youtube.com/watch?v=IN6ihpJSFDw>`_.
