Typehints and = null
====================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20051115 1702 CET
   :Tags: php, work

I just committed a patch to PHP CVS which allows you to do the
following:

::

	<?php
	class foo {
	    }
	    function bla( foo $blah = null )
	    {
	        var_dump( $blah );
	    }
	    $f = new foo;
	    bla( $f );
	    bla();
	    bla( null );
	?>



