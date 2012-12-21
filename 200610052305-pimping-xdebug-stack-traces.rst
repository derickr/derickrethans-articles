Pimping Xdebug stack traces
===========================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20061005 2305 CEST
   :Tags: cms, php, work, xdebug

I've always been annoyed by the way how Xdebug's stack traces looked
liked. So I spend some time on making them look better. I will show the
differences according to the following script:

::

	<?php
	    ini_set('xdebug.show_local_vars', 'on');
	    ini_set('xdebug.dump_globals', 'on');
	    ini_set('xdebug.dump.GET', '*');
	    ini_set('xdebug.collect_params', 'on');
	    ini_set('xdebug.var_display_max_depth', 2);
	    function foo( $a )
	    {
	        for ($i = 1; $i < $a['foo']; $i++)
	        {
	        }
	    }
	    set_time_limit(1);
	    $c = new stdClass;
	    $c->bar = 100;
	    $c->foo = "12";
	    $a = array(
	        42 => false,
	        'foo' => 912125235,
	        0 => null,
	        3.141592654,
	        "testing",
	        array(),
	        $c, new stdClass,
	        fopen( '/etc/passwd', 'r' )
	    );
	    foo( $a, 42, null, false, "testing",
	        $c, fopen( '/etc/passwd', 'r' ) );
	?>

Old:

.. image:: /images/content/old.scaled.png
   :align: center
   :target: /images/content/old.png

New:

.. image:: /images/content/new.scaled.png
   :align: center
   :target: /images/content/new.png

The code for this will make it into CVS soonish.



