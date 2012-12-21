Friday afternoon toying: eZ Components as phar
==============================================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20080627 1615 CEST
   :Tags: blog, cms, php, work, xdebug

PHP 5.3 will have a new cool feature: `phar`_ . A phar is to PHP what a jar is to
Java. I spent a little time to see how easy it would be to make our
latest `eZ Components`_ release
into a workable phar.

First of all, a phar can be build from a directory structure with a few
functions only:

::

	<?php 
	$phar = new Phar(
	    'ezcomponents-2008.1.phar', 0,
	    'ezcomponents-2008.1.phar' );
	$phar->buildFromDirectory(
	    dirname(__FILE__) . '/ezcomponents-2008.1',
	    '/\.php$/');
	$phar->compressFiles( Phar::GZ );
	$phar->stopBuffering();
	?>

This build script will create a phar from the directory contents in
"ezcomponents-2008.1", but only include the PHP files (See
http://php.net/phar.buildfromdirectory). We compress all the files and
with `stopBuffering`_ we
write the file to disk. With the following code, we can now use the phar
in our application:

::

	require 'ezcomponents-2008.1.phar';

It is also possible to run a bit of code when including the phar. You do
this, by adding a stub to the phar. To do so, we include the following
code just before the compressFiles() call:

::

	$stub = <<<ENDSTUB
	<?php
	Phar::mapPhar( 'ezcomponents-2008.1.phar' );
	require 'phar://ezcomponents-2008.1.phar/Base/src/base.php';
	spl_autoload_register( array( 'ezcBase', 'autoload' ) );
	__HALT_COMPILER();
	ENDSTUB;
	$phar->setStub( $stub );

After we re-create the phar with "php -dphar.readonly=0
build.php". The new phar once required will now setup the autoload
mechanism of the eZ Components. The following script demonstrates that
it actually works:

::

	<?php
	require 'ezcomponents-2008.1.phar';
	$f = ezcFeed::parse( 'http://derickrethans.nl/rss.xml' );
	foreach ( $f->item as $item )
	{
	    echo $item->title, "\n";
	}
	?>

Conclusion: phar is cool!


.. _`phar`: http://php.net/phar
.. _`eZ Components`: http://ezcomponents.org
.. _`stopBuffering`: http://php.net/phar.stopbuffering

