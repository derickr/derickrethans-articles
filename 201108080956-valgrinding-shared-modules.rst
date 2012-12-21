Valgrinding shared modules
==========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2011-08-08 09:56 Europe/London
   :Tags: blog, php, extensions, opensource
   :Short: vlgrnd-shared

Over the past year I've been writing various commercial_ (more about that later) 
and open source PHP extensions (such as QuickHash_, more about that later too).
Most of the time they are shared extensions that are not part of PHP.
While testing whether I correctly free all memory with Valgrind_, I 
ran into the issue where I couldn't see the stack frames of where the memory
leaks occurred in the extensions, and once I even ran into a `Valgrind bug`_.
The reason why Valgrind could not show the function names belonging to the
stack frames is because PHP had already unloaded the shared extensions from
memory.

.. _commercial: /available-for-php-extension-writing.html
.. _QuickHash: https://github.com/derickr/quickhash
.. _Valgrind: http://valgrind.org/
.. _'Valgrind bug`: https://bugs.kde.org/show_bug.cgi?id=277045

I often found myself compiling the extensions into PHP statically so that
there was nothing to unload for PHP, but that was becoming annoying. So
instead I added a patch_ to PHP that prevents the shutdown sequence from
actually unloading the modules. You can trigger this behaviour by setting
the ``ZEND_DONT_UNLOAD_MODULES`` environment variable before running your
script::

	# export ZEND_DONT_UNLOAD_MODULES=1
	# valgrind --leak-check=full --show-reachable=yes php -r 'echo "Hello World\n";'

.. _patch: http://news.php.net/php.cvs/65492

Without ``ZEND_DONT_UNLOAD_MODULES`` exported, my Valgrind output shows 
a block like::

	# unset ZEND_DONT_UNLOAD_MODULES
	# valgrind --leak-check=full --show-reachable=yes php -r 'echo "Hello World\n";'
	...
	==25829== 8 bytes in 1 blocks are indirectly lost in loss record 2 of 21
	==25829==    at 0x4C25E84: calloc (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
	==25829==    by 0xCE440DC: ???
	==25829==    by 0xCE44316: ???
	==25829==    by 0xCE44368: ???
	==25829==    by 0xCBEE55F: ???
	==25829==    by 0xCBD3F87: ???
	==25829==    by 0x949A85: zend_activate_modules (zend_API.c:2285)
	==25829==    by 0x8B5EBC: php_request_startup (main.c:1491)
	==25829==    by 0xA83D60: do_cli (php_cli.c:954)
	==25829==    by 0xA84F7B: main (php_cli.c:1356)
	...

And with ``ZEND_DONT_UNLOAD_MODULES`` exported, it shows instead::

	# export ZEND_DONT_UNLOAD_MODULES=1
	# valgrind --leak-check=full --show-reachable=yes php -r 'echo "Hello World\n";'
	...
	==25824== 8 bytes in 1 blocks are still reachable in loss record 2 of 30
	==25824==    at 0x4C25E84: calloc (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
	==25824==    by 0xCE440DC: event_base_priority_init (in /usr/lib/libevent-1.4.so.2.1.3)
	==25824==    by 0xCE44316: event_base_new (in /usr/lib/libevent-1.4.so.2.1.3)
	==25824==    by 0xCE44368: event_init (in /usr/lib/libevent-1.4.so.2.1.3)
	==25824==    by 0xCBEE55F: zm_activate_http_request_pool (http_request_pool_api.c:58)
	==25824==    by 0xCBD3F87: zm_activate_http (http.c:373)
	==25824==    by 0x949A85: zend_activate_modules (zend_API.c:2285)
	==25824==    by 0x8B5EBC: php_request_startup (main.c:1491)
	==25824==    by 0xA83D60: do_cli (php_cli.c:954)
	==25824==    by 0xA84F7B: main (php_cli.c:1356)
	...

As you can see all the symbols are now nicely resolved. This patch is part of
the upcoming PHP 5.4, but applies to `PHP 5.2 and PHP 5.3`__ as well.

__ /files/php-zend-unload-modules-20110808.diff.txt
