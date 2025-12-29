Multiple PHP versions set-up
============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2011-11-07 09:11 Europe/London
   :Tags: blog, php, xdebug, extensions
   :Short: multi-php

For many of my projects (both hobby_ and commercial_) I need to support many
different PHP configurations. Not only just different PHP versions, but also
debug builds, ZTS builds and 32-bit builds. In order to be able to test
and build extensions against all those different PHP configurations I have
adopted a simple method that I'm sharing with you here.

.. _hobby: http://derickrethans.nl/projects.html
.. _commercial: https://derickrethans.nl/who.html#derickrethansltd

The major part of it is that I use a different installation prefix for
every configuration of PHP that I make. Right now, ``ls /usr/local/php``
shows::

	5.1dev  5.3.2  5.3.8dev  5.3dev-32bit    5.3dev-zts  5.4dev      trunk
	5.2dev  5.3.7  5.3dev    5.3dev-nodebug  5.4.0beta1  5.4dev-zts

There are two types of PHP installs that I make: "dev" versions from SVN
branches, and "release" versions build from SVN tags. From the list above you
see I have the SVN versions of 5.1, 5.2, 5.3.8, 5.3 (in various forms) and 5.4
(both normal, and ZTS).

I have a script_ to compile PHP which whatever combination I want: version,
debug (default) or non-debug, normal (default) or ZTS; and 64-bit (default) or
32-bit. The configure options are nicely hardcoded :-)

The scripts accept arguments in a specific order::

	version ["debug"|"nodebug" [, "no-zts"|"zts" [, ""|"32bit" ] ] ]

For a simple 5.3dev build I run::

	./build 5.3dev

This compiles PHP 5.3 from SVN, in debug mode, without ZTS and for the 64-bit
architecture. Something more exotic variants could be::

	./build 5.3.8 debug zts
	./build 5.4dev nodebug no-zts 32bit

Each invocation of the script will configure and build PHP, and then install
into ``/usr/local/php/{{variant}}``.

The script also has a hard coded location where the PHP sources are. In 
my case, that's a `sparse checkout`_ into ``/home/derick/dev/php/php-src``.

.. _`sparse checkout`: https://wiki.php.net/vcs/svnfaq#sparse_directory_checkout_instructions

With the help of a small ``.bashrc`` snippet::

	function pe () {
		version=$1
		shift

		if [ "$#" == "0" ]; then
			export PATH=/usr/local/php/${version}/bin:/usr/local/bin:/usr/bin:/bin
		else
			PATH=/usr/local/php/${version}/bin:$PATH $@
		fi
	}

I can now easily switch between PHP versions by typing on the shell::

	pe 5.3dev

or::

	pe 5.4dev-zts

And each version will have a totally separated environment for me to install
PEAR packages and PECL extensions in, and do my own extension development. 
Of course, each separated environment also comes with its own ``php.ini`` file
(in ``/usr/local/php/{{variant}}/lib/php.ini``).

.. _script: https://github.com/derickr/php-build

This set-up makes my life a whole lot easier, and I hope it is useful for you
as well. Thanks for listening!

