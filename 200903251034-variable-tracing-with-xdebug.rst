Variable tracing with Xdebug
============================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20090325 1034 CET
   :Tags: blog, php, xdebug

Some time ago `Matthew`_ mentioned on IRC that he'd like to see variable modifications in `Xdebug's`_  `function traces`_ .
After I had a quick look at the feasibility of this feature I spend some
time on implementing it for Xdebug's HEAD branch that is going to become
Xdebug 2.1.

Variable modification tracing can be enabled by setting the php.ini
xdebug.collect_assignments setting to 1. Of course this can also be done
in either .htaccess or by using ini_set(). This setting requires general
execution tracing to be enabled as well and it's only available for
human readable trace files (the default format).

If variable modification tracing is turned on, all variable assignments,
as well as the update-in-place operators such as += and *=, are being
logged to the trace file. Take for example the following script:

::

	1  <?php
	2  function multiply( $n, $multiplier )
	3  {
	4      $val = $n;
	5      $val *= $multiplier;
	6      return $val;
	7  }
	8
	9  $a = 3;
	10 $a = multiply( $a, 14 );
	11 ?>

There are multiple statements that modify variables (on lines 4, 5, 9
and 10). Those statements generate lines in the execution trace file.
Combined with turning on `xdebug.collect_params`_ and `xdebug.collect_return`_ the trace file then turns out to be:

::

	TRACE START [2009-03-20 19:20:38]
	    0.0003     119632   -> {main}() /tmp/example1.php:0
	                         => $a = 3 /tmp/example1.php:9
	    0.0004     120104     -> multiply(3, 14) /tmp/example1.php:10
	                           => $val = 3 /tmp/example1.php:4
	                           => $val *= 14 /tmp/example1.php:5
	                           >=> 42
	                         => $a = 42 /tmp/example1.php:10
	                         >=> 1
	    0.0007      56360
	TRACE END   [2009-03-20 19:20:38]

Of course, this feature also works for updating object's properties and
array's elements as can be seen with the following script:

::

	1  <?php
	2  class Number
	3  {
	4      private $n;
	5
	6      public function __construct( $n )
	7      {
	8          $this->n = $n;
	9      }
	10
	11     public function multiply( $multiplier )
	12     {
	13         $this->n *= $multiplier;
	14     }
	15 }
	16 $a = new Number( 3 );
	17 $a->multiply( 14 );
	18
	19 $array['3x14'] = $a;
	20 ?>

And this creates the following trace file (without the time and memory
columns):

::

	-> {main}() /tmp/example2.php:0
	  -> Number->__construct(3) /tmp/example2.php:16
	   => $this->n = 3 /tmp/example2.php:8
	   >=> NULL
	 => $a = class Number { private $n = 3 } /tmp/example2.php:16
	  -> Number->multiply(14) /tmp/example2.php:17
	   => $this->n *= 14 /tmp/example2.php:13
	   >=> NULL
	 => $array->3x14 = class Number { private $n = 42 } /tmp/example2.php:19
	 >=> 1

At the moment, array keys and object properties both use the ->
syntax in the trace files, but this might change in the future. There
are also still issues with syntax like $this->foo['key'].


.. _`Matthew`: http://ishouldbecoding.com
.. _`Xdebug's`: http://xdebug.org
.. _`function traces`: http://xdebug.org/docs/execution_trace
.. _`xdebug.collect_params`: http://xdebug.org/docs/all_settings#collect_params
.. _`xdebug.collect_return`: http://xdebug.org/docs/all_settings#collect_return

