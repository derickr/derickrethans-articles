Private Properties Exposed
==========================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20051015 1847 CEST
   :Tags: cms, php, work

For `our`_  `Components project`_ we are ofcourse writing unit tests (with `PHPUnit`_ ). Sometimes you
would want to test whether a private property contains the correct data,
and of course with the normal visibility rules you can't access those
from your unit test. There is an interesting *trick* for this,
which I'll share here:

::

	<?php
	class foo {
	    private $bar = 42;
	}
	$obj = new foo;
	$propname="\0foo\0bar";
	$a = (array) $obj;
	echo $a[$propname];
	?>


.. _`our`: http://ez.no
.. _`Components project`: http://ez.no/community/news/ez_publish_enterprise_components
.. _`PHPUnit`: http://www.phpunit.de/en/index.php

