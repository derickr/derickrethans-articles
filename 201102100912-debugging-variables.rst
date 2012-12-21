Debugging Variables
===================

.. articleMetaData::
   :Where: London, UK
   :Date: 2011-02-10 09:12 Europe/London
   :Tags: blog, php, opensource, xdebug
   :Short: debug-vars

Sometimes you want to inspect the contents of variables more closely, by
looking at its internal properties. For this, PHP has the  debug_zval_dump_
function.

The internal representation of a PHP variable container (called
zval), contains the type and value of a variable, but also whether it is a
reference and what its refcount is. Due to PHP's copy-on-write policy, one
specific zval container can be used by multiple variables at the same time as
we will see in a bit.

The ``refcount`` field contains how many symbols (variable names) point to a
zval, regardless of whether a reference set is used or not. For example, both
the following examples will have as end-result: two symbols and one zval::

	<?php
	$a = 42; // one symbol: 'a'; one zval: int(42)
	$b = $a; // two symbols: 'a', 'b'; one zval: int(42)
	?>

	<?php
	$a = 42; // one symbol: 'a'; one zval: int(42)
	$b =& $a; // two symbols: 'a', 'b'; one zval: int(42)
	?>

In order for PHP to know the difference, there is an extra flag on the zval:
``is_ref``, which is set if you assign a variable by reference with ``=&``.
It is important to know which of the situations actually occurred because PHP
needs to be able to do the correct thing when a third symbol comes into play.
In the first situation, a third symbol can be associated with the same
zval **only** if it is assigned by assignment. If the third symbol is assigned
by reference with something like ``$c =& $b``; then PHP needs to duplicate
(split) the zval into two zvals because only symbol ``b`` and ``c`` are
linked, whereas ``a`` is distinct. Without the separation PHP will not be able
to know which of the two symbols are referenced (``b`` and ``c``), and which
one is not part of the reference set (``a``).

The opposite happens in the second situation. If a third symbol is assigned by
reference with something like ``$c =& $b``; then PHP can just update the
``refcount`` on the zval. If the third symbol is assigned by value with ``$c =
$b`` then PHP needs to split the zval so that ``a`` and ``b`` still point to
the original zval, and ``c`` to the newly created one without having links to
either ``a`` or ``b``.

This brings us to the very simple rule for when PHP needs to split upon
assignment:

- If the type of assigning (by-value with ``=`` or by-ref with
  ``=&``) matches the value of ``is_ref``: just update ``refcount`` by adding 1.
- If the type of assigning does not match the value of ``is_ref``: split the
  zval.

Now let us get back to the original issue: the inspection of the internal
structure of variable contents with  debug_zval_dump_. When a variable is
passed to a function as an argument, an assignment to the function's local
scope is made; just like an assignment in a function. Which means that the
same splitting rules apply. The following are (mostly) equivalent regarding on
what happens to the ``is_ref`` and ``refcount`` values::

	<?php
	$a = 42;
	$b = $a; // assignment in local scope
	?>

and::

	<?php
	function foo($b)
	{
	}

	$a = 42;
	foo($a); // assignment to $b in the function's scope
	?>

PHP's  debug_zval_dump_ takes a variable as argument for analysis and is thus
also bound to the same splitting rules as outlined earlier. This means that's
not really well suited for dumping a zval's internal structure as it would
always modify the zval. Besides adding 1 to the ``refcount`` field, it could
also force a split resulting in unexpected output::

	<?php
	$a = 42;
	debug_zval_dump($a);
	?>

shows::

	long(42) refcount(2)

Xdebug_ has a similar function to display a zval's internal data:
xdebug_debug_zval_. This function does requires not a variable to be passed as
its argument, but instead requires a variable **name** to be passed in. With
this, the manipulation of the zval is avoided and the proper values are
shown::

	<?php
	$a = 42;
	xdebug_debug_zval('a');
	?>

which shows::

	a: (refcount=1, is_ref=0)=42

This is what is expected, and as bonus, it also shows the ``is_ref`` flag. 
PHP's `bug #53895`_ mentions a few issues with the  debug_zval_dump_ function.
Gustavo is quite right to say that passing the variable name by-ref is not
going to work, and any form of this function where the variable is passed in
is not going to work. Johannes however is also right that passing just the
variable name as a string might not be good enough; as you can't directly dump
the zval information for for example array keys. There is an Xdebug `feature
request`_ for this functionality to be implemented however. If implemented,
you should be able to debug the ``foo`` array key with something like::

	<?php
	$a = array('foo' => 3.1415);
	xdebug_debug_zval('a["foo"]');
	?>

Now if I only had some spare time...

.. _`bug #53895`: http://bugs.php.net/bug.php?id=53895
.. _Xdebug: http://xdebug.org
.. _debug_zval_dump: http://php.net/debug_zval_dump
.. _xdebug_debug_zval: http://xdebug.org/docs/all_functions#xdebug_debug_zval
.. _`feature request`: http://bugs.xdebug.org/view.php?id=310
