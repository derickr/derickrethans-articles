var_export and classes
======================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20051017 2300 CEST
   :Tags: php

Today I stumbled upon an `old problem`_ while using `var_export()`_ on an array with
objects. var_exports()'s description is " `Outputs or returns a parsable string representation of a variable`_ ", but it didn't work to well in
this case:

::

	array (
	  0 =>
	  class ezcTranslationData {
	    public $original = 'Node ID:';
	    public $translation = 'Knoop ID:';
	    public $comment = false;
	    public $status = 0;
	  },
	);

This snippet above can of course not be parsed as valid PHP code. After
thinking about it for a bit, I came up with a solution. Instead of the
code above, we now generate:

::

	array (
	  0 =>
	  ezcTranslationData::__set_state(array(
	    'original' => 'Node ID:',
	    'translation' => 'Knoop ID:',
	    'comment' => false,
	    'status' => 0,
	  )),
	);

The __set_state() method is then required to be implemented by the class
for this to work, but that is better than generating code which can
never be parsed. The name comes from the `Memento pattern`_ . As this fixes a `bug`_ this made it into PHP 5.1.0
and PHP 6.0.0.


.. _`old problem`: http://bugs.php.net/29361
.. _`var_export()`: http://php.net/var_export
.. _`Outputs or returns a parsable string representation of a variable`: http://php.net/var_export
.. _`Memento pattern`: http://en.wikipedia.org/wiki/Memento_pattern
.. _`bug`: http://bugs.php.net/29361

