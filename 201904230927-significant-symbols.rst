Significant Symbols
===================

.. articleMetaData::
   :Where: London, UK
   :Date: 2019-04-23 09:27 Europe/London
   :Tags: blog, php
   :Short: sigsym

Last week a person on the ``#php`` IRC channel on freenode_, mentioned that he
had problems loading some extensions with his self-compiled PHP binary. For
example, trying to activate the timezonedb_ PECL extension failed with::

	sapi/cli/php:
		symbol lookup error: /usr/local/php/extensions/debug-non-zts-20180731/timezonedb.so:
		undefined symbol: php_date_set_tzdb

.. _freenode: https://freenode.net/
.. _timezonedb: https://pecl.php.net/package/timezonedb

Which is odd, as the ``php_date_set_tzdb`` is a symbol that PHP has made
available since Date/Time support was added. I asked the user to check whether
his PHP binary exported the symbol by using ``nm``, and the answer was::

	$ nm sapi/cli/php | grep php_date_set_tzdb
	000000000018bc75 t php_date_set_tzdb

The small letter ``t`` refers to a *local only* text (code) section: the
symbol was not made available to shared libraries to use. In other words,
extensions that make use of the symbol, such as the timezonedb extension can
not find it, and hence fail to load.

In PHP, the ``php_date_set_tzdb`` function is defined with the ``PHPAPI``
prefix, which explicitly should mark the symbol as a *global symbol*, so that
shared libraries can find it::

	PHPAPI void php_date_set_tzdb(timelib_tzdb *tzdb)

The ``PHPAPI`` macro is used, because on Windows it is required to explicitly
make symbols available::

	#ifdef PHP_WIN32
	#       define PHPAPI __declspec(dllexport)

On Linux (with GCC) symbols are made available *unless* marked differently
(through for example the ``static`` keyword).

When looking into this, we discovered that his PHP binary had **no** exported
symbols at all.

After doing a bit more research, I found that more recent GCC versions support
a specific compiler flag that changes the default behaviour of symbol
visibility: ``-fvisibility=hidden``. In recent versions of PHP, we enable this
flag if it is supported by the installed GCC version through a check in the
configure system::

	dnl Mark symbols hidden by default if the compiler (for example, gcc >= 4)
	dnl supports it. This can help reduce the binary size and startup time.
	AX_CHECK_COMPILE_FLAG([-fvisibility=hidden],
	                      [CFLAGS="$CFLAGS -fvisibility=hidden"])

As this makes **all** symbols hidden by default, the same commit also made sure
that when the ``PHPAPI`` moniker is used, we set the visibility of these
specific symbols back to ``visible``::

	#   if defined(__GNUC__) && __GNUC__ >= 4
	#       define PHPAPI __attribute__ ((visibility("default")))
	#   else
	#       define PHPAPI
	#   endif

When the original reporter saw this, he mentioned that was using an older GCC
version: 3.4, and that he *could* see the ``-fvisibility=hidden`` flag when
running ``make``, just like here::

	… cc -Iext/date/lib … -fvisibility=hidden … -c ext/date/php_date.c -o ext/date/php_date.lo

Because his GCC supported the ``-fvisibility=hidden`` flag, the check in the
configure script enabled this feature, but because his GCC version was older
than version 4, the counter-acting ``((visbility("default")))`` attribute was
not set for symbols that are explicitly marked with the ``PHPAPI`` specifier.
Which means that no symbols were be made available for shared PHP extensions
to use.

The user created a `bug report`_ for this issue, but as GCC 3.4 is a really
old version, it seems unlikely that this issue will get fixed, unless somebody
contributes a patch. In the end, it was quite a fun detective story and to get
to the bottom of this!

.. _`bug report`: https://bugs.php.net/bug.php?id=77883
