PHP's segmentation faults GDB-fu
================================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20081007 1947 CEST
   :Tags: php, work, xdebug

Sometimes PHP segfaults (crashes) in a production environment, where `Xdebug`_ is often not available (and
shouldn't be either of course). In those cases trying to figure out
where in your code PHP crashes can be hard to find out. In some cases
it's a real bug in PHP, where you would need some more intricate
knowledge of PHP's internals — in many cases it's rather a coding
error that provides you with infinite recursion.

Trying to figure out the functions that were called in a loop is not
trivial if you do not possess GDB- and PHP internals-fu. However,
because we as PHP developers are lazy, provide a few GDB tricks to make
this easier. First of all, it's only really going to work if you haven't
stripped the symbols from your PHP and Apache binaries. Secondly, you
still need to have the PHP source lying around somewhere — preferably
from where you've built PHP. After you're in GDB (either by opening an
already existing core dump, or when the process aborts after starting it
from GDB) you can "source" the macros that make your life
easier. Basically you have to run this on the GDB prompt:

::

	(gdb) source ~/dev/php/php-5.2dev/.gdbinit

If you then run the following on the GDB prompt, you get a nice stack
trace — but without variable information that you're used to from
seeing Xdebug traces.

::

	(gdb) zbacktrace

The start of the output looks like:

::

	[0xd03bb330] a() /tmp/recur.php:5 
	[0xd03bb530] d() /tmp/recur.php:4 
	[0xd03bb730] c() /tmp/recur.php:3 
	[0xd03bb930] b() /tmp/recur.php:2 
	...

In PHP 5.3 and higher, PHP will not segfault when you do infinite
recursion as the engine has been changed. Instead, PHP would simply run
out of memory and show an error not unlike:

Fatal error: Allowed memory size of 134217728 bytes exhausted at
/home/derick/dev/php/php-5.3dev/Zend/zend_execute.h:157 (tried to
allocate 523800 bytes) in /tmp/recur.php on line 2

*Update:* Instead of "dump_bt
executor_globals.current_execute_data" you can simply run
"zbacktrace".


.. _`Xdebug`: http://xdebug.org

