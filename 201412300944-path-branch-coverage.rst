Code Coverage: Finding Paths
============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2015-11-04 08:55 Europe/London
   :Tags: blog, php
   :Short: pathcoverage

Picking up where we left `last time`_, in this second article we will look at
some upcoming functionality in Xdebug_. Sebastian has been pressuring me for
years to add branch and path coverage to Xdebug, with issue `#1034`_. In the
post I will show you what "branch and path coverage" is, and how it helps.

In the previous post, I showed you a trivial script as an example for code
coverage. Let's have a look at a different script this time
(``article2-test.php``)::

	<?php
	function ifthenelse( $a, $b )
	{
		if ($a) {
			echo "A HIT\n";
		}
		if ($b) {
			echo "B HIT\n";
		}
	}

We now run this through `PHP CodeCoverage`_ (``article2.php``)::

	<?php
	require 'vendor/autoload.php';
	include 'article2-test.php';

	$coverage = new PHP_CodeCoverage;

	$coverage->start('coverage1');
	ifthenelse( true, false ); 
	$coverage->stop();

	$coverage->start('coverage2');
	ifthenelse( false, true );
	$coverage->stop();

	$writer = new PHP_CodeCoverage_Report_HTML;
	$writer->process($coverage, '/tmp/code-coverage-article');
	?>

Which outputs:

.. image:: /images/content/code-coverage-new.png

And this nicely tells you that all your lines have been executed. However, you
have not *tested* everything. With two ``if`` statements, there are four
possible paths through your code:

===== =====
$a    $b
===== =====
false false
false true
true  false
true  true
===== =====

But we have only tested for the second and the third cases. Because we are
doing *line* coverage, this sort of coverage issues don't show up. In order to 
address this, we need to do branch and path coverage.

I wrote in the previous article that Xdebug follows all branches to find dead
code during code coverage, but it never did really more with that. So, again,
I started experimenting through VLD_ to see how I could get it to list all
possible branches (easy) and paths through a function or method.

So now VLD_ would say the following (besides dumping the opcodes) about the
code::

	filename:       /home/httpd/html/test/xdebug/code-coverage/article2-test.php
	function name:  ifthenelse

	…

	branch: #  0; line:     2-    4; sop:     0; eop:     4; out1:   5; out2:   8
	branch: #  5; line:     5-    6; sop:     5; eop:     7; out1:   8
	branch: #  8; line:     7-    7; sop:     8; eop:     9; out1:  10; out2:  13
	branch: # 10; line:     8-    9; sop:    10; eop:    12; out1:  13
	branch: # 13; line:    10-   10; sop:    13; eop:    14; out1:  -2
	path #1: 0, 5, 8, 10, 13, 
	path #2: 0, 5, 8, 13, 
	path #3: 0, 8, 10, 13, 
	path #4: 0, 8, 13, 
	End of function ifthenelse

It's a bit difficult to see what goes on here, but in general there are five
branches and four paths. The first two branches (``0`` and ``5``) are for the
first ``if`` statement, the second two branches (``8`` and ``10``) are for the
second ``if`` statement and the last branch (``13``) is the end of function,
with an implicit return to the calling function. Branches are named after the
opcode entry that forms the start of a branch.

Seeing the different branches is probably easier to see in a graph, which VLD
can also produce (using the ``vld.save_paths=1`` php.ini setting). This
produces a ``paths.dot`` file in ``/tmp`` which we can convert to an image
with `GraphViz'`_ ``dot`` tool::

	dot -Tpng /tmp/paths.dot > /tmp/paths.png

And then this graph looks like:

.. image:: /images/content/paths.png


.. _`last time`: /code-coverage.html
.. _`#1034`: http://bugs.xdebug.org/view.php?id=1034
.. _Xdebug: http://xdebug.org
.. _VLD: http://derickrethans.nl/projects.html#vld
.. _`PHP CodeCoverage`: https://packagist.org/packages/phpunit/php-code-coverage
.. _`GraphViz'`: http://www.graphviz.org/
