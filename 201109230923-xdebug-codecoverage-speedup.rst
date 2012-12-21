Xdebug's Code Coverage speedup
==============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2011-09-23 09:23 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-cc-speed

Besides a debugging and development aid, Xdebug_ also implements the back-end
code to provide code coverage information for use with PHPUnit_. Code coverage
tells you how much of your code base is actually being tested by your unit
tests. It's a very useful feature, but sadly, it slows down PHP's execution
quite a lot. One part of this slowdown is the overhead to record the information
internally, but another part is because I have to overload lots of opcodes.
(Opcodes are PHP's internal execution units, similar to assembler_ instructions)
They are always overloaded even if code coverage is not used, because it's
only safe to overload them for the whole request.

The upcoming Xdebug 2.2 has a few changes to improve code coverage
performance. First of all, the speed with which information about code
coverage is recorded has been improved by contributions_ by `Taavi Burns`_.
And secondly, Xdebug 2.2 features a new setting, ``xdebug.coverage_enable``
with which it is possible to disable the overloading of code coverage specific
opcodes. By default that setting will still be ``on``.

Just to show how those improvements have effect on the speed, I've run a
benchmark (running `Apache Zeta Components`_ Graph's tests) to compare
Xdebug 2.1.2 against Xdebug 2.2-dev. 

The results are as follows:

=======  =======  ==========  ======================  ==============
Version  With CC  Without CC  With coverage_enable=0  Without Xdebug  
=======  =======  ==========  ======================  ==============
2.1.2    06:44    00:49       *not available*         00:32
2.2-dev  05:37    00:48       00:46                   00:32
=======  =======  ==========  ======================  ==============

Another benchmark, run by `Ross McFarlane`_ gives:
`@derickr Based on 3 runs each, execution of 254 tests with 1070 assertions
has dropped from 19s to 12s. Nice work!`__.

In Xdebug 2.2 I would also like to introduce *modes*. Each mode will set
a default configuration for Xdebug's setting to be the most optimal for
that specific tasks. Examples of tasks could be: "profiling", "debugging"
or "tracing". Let me know (in the comments) whether you think that's a good
idea.


__ https://twitter.com/#!/rossmcf/status/116086201391398913
.. _contributions: https://github.com/taavi/xdebug/commits/coverage_line_array
.. _`Taavi Burns`: https://github.com/taavi
.. _`Apache Zeta Components`: http://incubator.apache.org/zetacomponents/
.. _`Ross McFarlane`: https://twitter.com/#!/rossmcf
.. _Xdebug: http://xdebug.org
.. _phpunit: http://www.phpunit.de
.. _assembler: http://en.wikipedia.org/wiki/Assembler_%28computer_programming%29#Assembly_language
