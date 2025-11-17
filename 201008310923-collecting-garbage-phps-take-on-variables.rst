Collecting Garbage: PHP's take on variables
===========================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2010-08-31 09:23 Europe/London
   :Tags: blog, php

This is the first part of three-parts column that was originally published
in the `April 2009`_ issues of `php|architect`_.

.. _`April 2009`: http://www.phparch.com/magazine/2009/april/
.. _`php|architect`: http://www.phparch.com/magazine

-----

In this three part column I will explain the merits of the new Garbage
Collection (also known as GC) mechanism that is part of PHP 5.3. Before we start with the
intricate details of PHP's new GC engine I will explain why it is actually
needed. This, combined with an introduction how PHP deals with variables in
general is explained in this first part of the column. The second part will
cover the solution and some notes on the GC mechanism itself, and the third
part covers some implications of the GC mechanism, as well as some
benchmarks. But now first on to the introduction.

PHP stores variables in containers called a "zval". A zval container contains
besides the variable's type and value, also two additional bits of
information. The first one is called "is_ref" and contains a boolean value
whether this variable is part of a "reference set". With this bit PHP's engine
knows how to differentiate between normal variables, and references. However,
PHP has user-land references—as created by the & operator, but also an
internal reference counting mechanism to optimize memory usage. The second
piece of additional information, called "refcount", contains how many
variables names—also called symbols—point to this one zval container. All
symbols are stored in a symbol table, of which there is one per scope. There
is a scope for the main script (ie, the one requested through the browser), as
well as for every function or method.

A zval container is created when a new variable is created with a constant
value, such as::

	$a = "new string";

In this case the new symbol name "a" is created in the current scope, and a
new variable container is created with type "string", value "new string". The
"is_ref" bit is by default set to "false" because no user-land reference has
been created. The "refcount" is set to "1" as there is only one symbol that
makes use if this variable container. Also, if the "refcount" is "1", "is_ref"
is always "false". If you have Xdebug_ installed you can display this
information by calling::

	xdebug_debug_zval('a');

which displays::

	a: (refcount=1, is_ref=0)='new string'

.. _Xdebug: http://xdebug.org

Assigning this variable to another variable name, increases the refcount::

	$a = "new string";
	$b = $a;
	xdebug_debug_zval( 'a' );

which displays::

	a: (refcount=2, is_ref=0)='new string'

The refcount is "2" here, because the same variable container is linked with
both "a" and "b". PHP is smart enough not to copy the actual variable
container when it is not necessary. Variable containers get destroyed when the
"refcount" reaches zero. The "refcount" gets decreased by one for each symbol
linked to the variable container leaves the scope (f.e. if the function ends)
or when unset() is called on a symbol. The following example shows that::

	$a = "new string";
	$c = $b = $a;
	xdebug_debug_zval( 'a' );
	unset( $b, $c );
	xdebug_debug_zval( 'a' );

which displays::

	a: (refcount=3, is_ref=0)='new string'
	a: (refcount=1, is_ref=0)='new string'

If we now call "unset( $a );" the variable container, including the type and
value will be removed from memory.

Things get a tad more complex with compound types such as arrays and objects.
Instead of a scalar value, arrays and objects store their properties in a
symbol table of their own. This means that the following example creates
**three** zval containers::

	$a = array( 'meaning' => 'life', 'number' => 42 );
	xdebug_debug_zval( 'a' );

which displays (after formatting)::

	a: (refcount=1, is_ref=0)=array (
		'meaning' => (refcount=1, is_ref=0)='life', 
		'number' => (refcount=1, is_ref=0)=42
	)

Graphically, it looks like:

.. image:: images/gc-part1-figure1.png

You can see the three zval containers here: "a", "meaning" and "number".
Similar rules apply for increasing and decreasing "refcounts". Below we add
another element to the array, and set it's value to the contains of an already
existing element::

	$a = array( 'meaning' => 'life', 'number' => 42 );
	$a['life'] = $a['meaning'];
	xdebug_debug_zval( 'a' );

which displays (after formatting)::

	a: (refcount=1, is_ref=0)=array (
		'meaning' => (refcount=2, is_ref=0)='life',
		'number' => (refcount=1, is_ref=0)=42, 
		'life' => (refcount=2, is_ref=0)='life'
	)

Graphically, it looks like:

.. image:: images/gc-part1-figure2.png

From the above Xdebug output,we see that both the old and new array elements
now point to a zval container whose "refcount" is "2".  Although Xdebug's
output shows two zval containers with value "life", they are the same one.
The function `xdebug_debug_zval()`_ function does not show this, but you could
see it by also displaying the memory pointer.

Removing an element from the array is like removing a symbol from a scope. By
doing so, the "refcount" of a container that an array element points to is
decreased.  Again when the "refcount" reaches zero, the variable container is
removed from memory. Again an example to show this::

	$a = array( 'meaning' => 'life', 'number' => 42 );
	$a['life'] = $a['meaning'];
	unset( $a['meaning'], $a['number'] );
	xdebug_debug_zval( 'a' );

which displays (after formatting)::

	a: (refcount=1, is_ref=0)=array (
		'life' => (refcount=1, is_ref=0)='life'
	)

.. _`xdebug_debug_zval()`: http://xdebug.org/docs/all_functions#xdebug_debug_zval

Now, things get interesting if we add the array itself as an element of the
array, which we do in the next example—in which I also sneaked in an reference
operator as otherwise PHP would create a copy here::

	$a = array( 'one' );
	$a[] =& $a;
	xdebug_debug_zval( 'a' );

which displays (after formatting)::

	a: (refcount=2, is_ref=1)=array (
		0 => (refcount=1, is_ref=0)='one', 
		1 => (refcount=2, is_ref=1)=...
	)

Graphically, it looks like:

.. image:: images/gc-part1-figure3.png

You can see that the array variable ("a") as well as the second element ("1")
now point to a variable container that has a "refcount" of "2". The "..." in
the display above shows that there is recursion involved, which of course in
this case it means that the "..." points back to the original array.

Just like before, unsetting a variable removes the symbol, and the reference
count of the variable container it points to is decreased by one. So if we
unset variable $a after running the above code, the reference count of the
variable container that $a and element "1" point to gets decreased by one,
from "2" to "1". This can be represented like::

	(refcount=1, is_ref=1)=array (
		0 => (refcount=1, is_ref=0)='one', 
		1 => (refcount=1, is_ref=1)=...
	)

Graphically, it looks like:

.. image:: images/gc-part1-figure4.png

Although there is no symbol in any scope pointing to this structure anymore,
it can not be cleaned up either because the array element "1" still points to
this same array. Because there is no external symbol pointing to it, there is
no way for a user to clean up this structure anymore, and thus you get a
memory leak. Fortunately, PHP will clean up this data structure at the end of
the request, but before then this is taking up valuable space in memory. The
mentioned situation happens often if you're implementing parsing algorithms or
other things where you have a child point back at a "parent" element. The same
situation can also happen with objects of course, where it actually happens
easier as objects are always implicitly used by reference.

This might not be a problem if this only happens once or twice, but if there
is thousands, or even millions of these memory losses, this obviously starts
being a problem. Especially in long running scripts, such as daemons where the
request basically never ends, or in large sets of unit tests. The latter
caused problems for us while running the unit tests for the Template component
of the eZ Components library. In some cases it would require over 2 GiB of
memory, which our test server didn't quite have. 

With that we conclude this introduction, for more information on how PHP deals
with variables I can point you at June 2005 issue of php|architect. That
article is also available on-line as PDF
(http://derickrethans.nl/files/phparch-php-variables-article.pdf).
In the next installment_, we're going to discuss the solution to the memory leak
problem with circular references.

.. _installment: /collecting-garbage-cleaning-up.html
