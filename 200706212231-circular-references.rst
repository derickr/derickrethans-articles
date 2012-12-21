Circular References
===================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20070621 2231 CEST
   :Tags: blog, php, work

Circular-references has been a long outstanding issue with PHP. They are
caused by the fact that PHP uses a reference counted memory allocation
mechanism for its internal variables. This causes problems for longer
running scripts (such as an `Application Server`_ or
the `eZ Components`_ test-suite) as the memory is not freed until the end of the request. But
not everybody is aware on what exactly the problem is, so here is a
small introduction to circular references in PHP.

In PHP a refcount value is kept for every variable container (zval).
Those containers are pointed to from a function's symbol table that
contains the names of all the variables in the function's scope. Every
variable, array element or object property that points to a zval will
increase its refcount by one. The refcount of a zval container is
decreased by one whenever call unset() on a variable name that points to
it, or when a variable goes away because the function in which it was
used ends. For a more thorough explanation about references, please see
the `article on this`_ that I
wrote for `php|architect`_ some time
ago.

The problems with circular references all start by creating an array or
an object:

::

	<?php
	$tree = array( 'one' );
	?>

This creates the following structure in memory:

.. image:: /images/content/circ1.png
   :align: center

Now if we proceed to add a new element to the array, that points back to
the array with:

::

	<?php
	$tree[] = $tree;
	?>

We create a circular reference like this:

.. image:: /images/content/circ2.png
   :align: center

As you can see there are two variable names pointing to the array
itself. Once through the $tree variable, and once through the 2nd
element of the array. Because there are two variable names pointing to
the container, its refcount is 2.

Now, the next step that actually creates the problem if we unset() the
$tree variable. As I mentioned before an unset() on a variable name will
decrease the refcount of the variable container the variable points to,
in this case the array value. The structure in memory now looks like
this:

.. image:: /images/content/circ3.png
   :align: center

Note that there is no variable pointing to the array container anymore.
Because of this PHP has no way of freeing this data anymore, and the
memory leak is born. PHP however does remove all allocated variables at
the end of each request, so it's not a hard-memory leak, but it's still
annoying enough for long running scripts and daemons written in PHP.

Luckily there are a few solutions to this problem. The first one is to
use a new `garbage collection`_ algorithm, another is to augment the reference counting
system with some `cyclic tracing`_ . The latter solution is something that is currently `undertaken`_ by David Wang, one of the `Google Summer of Code`_ students. He mentioned he is making good progress
and I can hardly wait until I can play with it :)


.. _`Application Server`: http://blog.milkfarmsoft.com/?p=51
.. _`eZ Components`: http://ez.no/ezcomponents
.. _`article on this`: /php_references_article.php
.. _`php|architect`: http://phparch.com
.. _`garbage collection`: http://en.wikipedia.org/wiki/Garbage_collection_%28computer_science%29
.. _`cyclic tracing`: http://www.research.ibm.com/people/d/dfb/papers/Bacon01Concurrent.pdf
.. _`undertaken`: http://code.google.com/soc/php/appinfo.html?csaid=53B3A537E855F518
.. _`Google Summer of Code`: http://code.google.com/soc/

