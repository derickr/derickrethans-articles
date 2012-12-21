Xdebug and tracing memory usage
===============================

.. articleMetaData::
   :Where: London, UK
   :Date: 20091113 1159 GMT
   :Tags: blog, cms, php, work, xdebug

Recently people started to ask me how to use `Xdebug`_ to figure out which parts of
applications use a lot of memory. Traditionally this was part of
Xdebug's `profiling`_ functionality. Unfortunately the cachegrind format didn't fit this so
well, and because it returned incorrect data I removed this
functionality from the profiler. However, there is other functionality
in Xdebug that does provide the correct data: the `function traces`_.

Function traces log every include, function call and method call to a
file. If the `xdebug.trace_format`_ setting is set to "1" then the trace file is an easy-to-parse
tab separated format. The information that is logged includes the
time-index when the function started and ended, and it also contains the
amount of memory that was in use when entering the function, as well as
when leaving it. With the last two numbers it's rather trivial to write
a script to figure out which functions/methods increase the memory usage
a lot. Of course, nobody had written a script yet to do anything with
this information

As part of my preparations for my Xdebug talk at `IPC`_ next week, I now have written
such a script. The script parses the tab-separated function trace files
and aggregates all the information by function name. You can sort the
output on a few different keys: time-own, memory-own, time-inclusive,
memory-inclusive and calls. You can also configure how many elements it
will show. As an example here is some output from a trace of one of the
presentation system's PHP scripts:

::

	$ php tracefile-analyser.php trace.2043925204.xt memory-own 20

::

	parsing...
	Done.
	Showing the 20 most costly calls sorted by 'memory-own'.
	                                               Inclusive        Own
	function                               #calls  time     memory  time     memory
	-------------------------------------------------------------------------------
	require_once                                9  0.0541  4595160  0.0277  2548104
	{main}                                      1  0.0600  2906032  0.0034   249744
	fread                                       4  0.0001    33296  0.0001    33296
	session_start                               1  0.0002    31824  0.0002    31824
	XML_Presentation->startHandler             38  0.0073    36360  0.0035    18424
	_pres_slide->_pres_slide                   27  0.0009    10152  0.0009    10152
	_presentation->_presentation                1  0.0001     7912  0.0001     7912
	strtolower                                 67  0.0017     6456  0.0017     6456
	compact                                     1  0.0000     4832  0.0000     4832
	each                                        5  0.0001     4320  0.0001     4320
	XML_Presentation->endHandler               38  0.0014     3800  0.0014     3960
	_slide->_slide                              1  0.0001     3896  0.0001     3896
	XML_Slide->startHandler                     4  0.0009    10800  0.0004     3736
	_image->_image                              1  0.0000     3040  0.0000     3040
	fopen                                       2  0.0001     2816  0.0001     2816
	getimagesize                                1  0.0001     2296  0.0001     2296
	display->display                            1  0.0001     2120  0.0001     2120
	explode                                     2  0.0001     2120  0.0001     2120
	xml_parser_create                           2  0.0001     1680  0.0001     1680
	XML_Parser->_initHandlers                   2  0.0011     1600  0.0005     1360

The script is available in Xdebug's download_ package or at
https://github.com/derickr/xdebug/raw/master/contrib/tracefile-analyser.php

The script to run is then "tracefile-analyser.php" from inside
the "xdebug/contrib" directory.

**Update 2009-12-28**: Changed the CVS instructions to the new SVN
instructions.

**Update 2012-03-13**: Changed the SVN instructions to a link to github.

.. _`Xdebug`: http://xdebug.org
.. _`profiling`: http://xdebug.org/docs/profiler
.. _`function traces`: http://xdebug.org/docs/execution_trace
.. _`xdebug.trace_format`: http://xdebug.org/docs/execution_trace#trace_format
.. _`IPC`: http://phpconference.com
.. _download: http://xdebug.org/download.php#source
