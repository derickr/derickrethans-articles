What is PHP doing?
==================

.. articleMetaData::
   :Where: London, UK
   :Date: 2012-07-13 09:42 Europe/London
   :Tags: blog, php
   :Short: phpwhat

Sometimes when you have a long running PHP script, you might wonder what the
hell it is doing at the moment. There are a few tools that can help you to
find out, without having to stop the script. Some of these work only on
Linux.

**strace**

The first tool that you can use is ``strace``. *strace* is a tool that
traces system calls. System calls are made by PHP for reading/writing
from/to the network and/or files; this also includes reading and writing
to databases over the network or Unix domain sockets. *strace* will also show
other system calls such as time. You can use *strace* by running something
like::

	strace -p <processid>

The ``processid`` you can figure out by running ``ps aux | grep php`` and then
guessing the correct one from the list - probably the one that uses 100% CPU
time. It is *also* possible to run *strace* directly with your program's
arguments::

	strace php yourscript.php

You can even restrict the output to only show certain system calls. With the
following command it will only show the ``open`` and ``close`` system calls::

	strace -e open,close php yourscript.php

The output of that last call, with the ``yourscript.php`` file not existing, 
has the following bits in it::

	...
	open("/lib/x86_64-linux-gnu/libnss_files.so.2", O_RDONLY) = 3
	close(3)                                = 0
	open("/etc/services", O_RDONLY|O_CLOEXEC) = 3
	close(3)                                = 0
	open("/sys/devices/system/cpu", O_RDONLY|O_NONBLOCK|O_DIRECTORY|O_CLOEXEC) = 3
	close(3)                                = 0
	open("yourscript.php", O_RDONLY)        = -1 ENOENT (No such file or directory)
	Could not open input file: yourscript.php

It is also very useful to find out which ``php.ini`` files PHP is trying to load::

	strace -e open php yourscript.php 2>&1 | grep php.ini

Grepping needs to be done for the ``stderr`` file descriptor, hence the
slightly cryptic ``2>&1 | grep php.ini``.

In my case, running this shows::

	derick@whisky:/tmp$ strace -e open php yourscript.php 2>&1 | grep php.ini
	open("/usr/local/php/5.3dev/bin/php.ini", O_RDONLY) = -1 ENOENT (No such file or directory)
	open("/usr/local/php/5.3dev/lib/php.ini", O_RDONLY) = 3


**ltrace**

The ``ltrace`` tool can be run and used in the same way as *strace*, and will
tell you which intra-library functions are being called. It produces a lot
more information than *strace* but it should give a good hint of all the 
C-library functions that are being called. Not as useful as *strace* though.

**gdb**

If neither *strace* or *ltrace* produce any output, it's likely that
your script or extension is doing lots of computing. To then figure out in
which PHP function your script currently is, you can use the ``gdb`` tool.

gdb—the GNU debugger—is a general purpose debugger for C/C++ applications and
provides the same functions as Xdebug_ provides for PHP script debugging.
Again, you can either run *gdb* with new arguments, and then type on the
``(gdb)`` prompt ``run``::

	gdb --args php yourscript.php
	(gdb) run

Or with an already running process and then running ``cont`` on the prompt::

	gdb -p <processid>
	(gdb) cont

Once the script gets to a state in which you are interested to see what it is
doing, you can press *Ctrl-C*. At that moment you come back at the ``(gdb)``
prompt where you can run ``bt`` to get a list of all the C-level functions that
PHP has been calling. For example::

	^C
	Program received signal SIGINT, Interrupt.
	xdebug_execute (op_array=0x1f03b00)
		at /home/derick/dev/php/xdebug/xdebug.c:1409
	1409			zval_ptr_dtor(EG(return_value_ptr_ptr));
	(gdb) bt
	#0  xdebug_execute (op_array=0x1f03b00)
		at /home/derick/dev/php/xdebug/xdebug.c:1409
	#1  0x000000000099d9a2 in zend_do_fcall_common_helper_SPEC (
		execute_data=0x7f3dec906090)
		at /home/derick/dev/php/php-src-git/branches/PHP_5_3/Zend/zend_vm_execute.h:344
	#2  0x00000000009a1cb7 in ZEND_DO_FCALL_SPEC_CONST_HANDLER (
		execute_data=0x7f3dec906090)
		at /home/derick/dev/php/php-src-git/branches/PHP_5_3/Zend/zend_vm_execute.h:1640
	#3  0x000000000099cce7 in execute (op_array=0x1ee2d50)
		at /home/derick/dev/php/php-src-git/branches/PHP_5_3/Zend/zend_vm_execute.h:107
	#4  0x00007f3de53debfe in xdebug_execute (op_array=0x1ee2d50)
		at /home/derick/dev/php/xdebug/xdebug.c:1391
	#5  0x00000000009696a8 in zend_execute_scripts (type=8, retval=0x0, 
		file_count=3)
		at /home/derick/dev/php/php-src-git/branches/PHP_5_3/Zend/zend.c:1236
	#6  0x00000000008f296d in php_execute_script (primary_file=0x7fff9ad6d8b0)
		at /home/derick/dev/php/php-src-git/branches/PHP_5_3/main/main.c:2308
	#7  0x0000000000a4b8b3 in main (argc=2, argv=0x7fff9ad6db48)
		at /home/derick/dev/php/php-src-git/branches/PHP_5_3/sapi/cli/php_cli.c:1189

In order to see in which PHP function you are, ``bt`` is not very useful. For
every internal function or method, you will see a function starting with
``zif_`` or ``zim_``, but user defined functions will only have an ``execute``
function listed. You can however find out which user defined functions are
being run as well, by means of the ``.gdbinit`` script.

This script is part of the PHP source distribution and lives in the top level
directory of the source tree. Because I always want this file to load, I have
copied this file to my home directory to live as ``/home/derick/.gdbinit``.
*gdb* will always load this file when it starts, but if you only want to do
this for specific cases, you can do that by running on the ``(gdb)`` prompt::

	(gdb) source /home/derick/dev/php/PHP_5_3/.gdbinit

In either case, with the script loaded you can then run on the ``(gdb)`` prompt::

	(gdb) zbacktrace

With this simple script::

	<?php
	function addOne( &$i )
	{
		$i++;
	}

	$i = 0;
	while (true) {
		addOne( $i );
	}
	?>

This command should (almost) always show::

	(gdb) zbacktrace
	[0xec906090] addOne() /tmp/yourscript.php:9

Hope this helps!

.. _Xdebug: http://xdebug.org
