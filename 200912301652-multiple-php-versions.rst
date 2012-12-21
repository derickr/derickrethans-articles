PHP Development Environment 2.0
===============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2009-12-31 02:26 UTC
   :Tags: blog, php, xdebug

When moving_ the Xdebug_ website and my own website to a new server I had to
pick a web server to serve the pages through PHP. Up to then I was still using
Apache 1.3 with zero intentions to ever upgrade to any later version. I'd
heard a lot about lighttpd_ and decided to give that a try—yes, that meant
something that I didn't really know to well was going to run in a production
environment. Unlike Apache, with lighttpd PHP doesn't run as a module, but
instead you run it out of process with something called FastCGI.

On my development machine I was also running Apache 1.3 with PHP as static
module embedded. If I want to test something with a different PHP version, I
would start a different Apache daemon. When I had a quick look the other day,
I found that I had about 28 different binaries named ``httpd-{Apache
version}-{PHP version}`` ranging from PHP 4.1.2 to 6.0dev. Some of those I
hadn't used for years.

All my PHP installations were installed with the same prefix (``/usr/local``),
where the last built PHP version was available as ``php`` and additional
versions as ``php-{version}``. An annoyance with this is, is that I had
to run ``make install`` in one of the PHP source directories before I could
compile extensions against a different PHP version than the *latest installed
one*. This combined with the enormous amount of httpds and the inability to
run multiple versions of PHP at the same time prompted me to come up with
something better: *PHP Development Environment 2.0*.

**Multiple PHP Installations**

The first thing I wanted to do was to make sure that each minor PHP version
(5.1.x, 5.2.x, 5.3.x and 6.0.x) could be installed alongside each other, and
that I could compile extensions to all of them without having to re-run ``make
install``. I did this by specifying ``--prefix=/usr/local/php/{PHP
version}`` to the PHP configure call. At the same time, I also removed the
``--with-apache`` configure option and for PHP 5.1 and PHP 5.2 I added the
``--enable-fastcgi`` option which is default enabled for PHP 5.3 and PHP 6.0.
The initial section of the configure invocation now looks like for the PHP 5.2
build::

  ./configure --prefix=/usr/local/php/5.2dev --enable-fastcgi \
    --with-gd  ...more configure options here...

Although ``/usr/local/bin`` is usually in the path, but
``/usr/local/php/5.2dev/bin`` is definitely not. The initial goal was to be
able to easily switch between different PHP versions, so I added a snippet to
my ``.bashrc`` that sets the correct ``PATH`` environment variable depending
on the selected PHP version::

  function pe () {
    version=$1
    shift

    if [ "$#" == "0" ]; then
      export PATH=/usr/local/php/${version}/bin:/usr/local/bin:/usr/bin:/bin
    else
      PATH=/usr/local/php/${version}/bin:$PATH $@
    fi
  }

This bash function is then evoked like ``pe 5.3dev`` to set the path in such a
way that ``php`` runs the PHP 5.3 binary, as well as additional tools such as
``pear``, ``phpize`` (required to build extensions) and ``php-config``. If you
forget to call the ``pe`` function, then when running ``php`` you will
encounter the warning ``bash: php: command not found``. You could pick a
default version if you wanted, but I prefer that I always have to be explicit
about which PHP version I pick. Multiple install paths makes it easier to
maintain multiple PHP installs, but it also means that you have to install
tools such as `PHP Unit`_ for each of the installations.

**Setting up Lighttpd**

Now that I had multiple PHP installations, I could move on to setting up
lighttpd. As I have mentioned, I never used lighttpd before and decided to
take the easy way by installing it with ``apt-get install lighttpd``. PHP
works through the FastCGI interface with lighttpd which you can enable with
``lighttpd-enable-mod cgi fastcgi simple-vhost`` (I also enabled CGI and
simple VHOSTing).

To enable PHP as FastCGI server, and to be able to run multiple versions at
the same time, I modified the ``/etc/lighttpd/conf-enabled/10-fastcgi.conf``
configuration file. Find my annotated version below::

  ## Enable the FastCGI module.
  server.modules   += ( "mod_fastcgi" )

Next we start an FastCGI server for the default PHP version (5.3) on the
default port (80) by enabling it in the global scope::

  fastcgi.server    = ( ".php" =>
    ((
      ## The path to the PHP-CGI executable
      "bin-path" => "/usr/local/php/5.3dev/bin/php-cgi",

      ## The listening socket that allows lighttpd to talk 
      ## to the PHP FastCGI process. This needs to be
      ## different for each fastcgi.server definition.
      "socket" => "/tmp/php80.socket",

      ## Configure the number of parent processes that are
      ## spawned by lighttpd (max-procs) and configure how
      ## many many workers should be started (4) and the
      ## maximum number of requests that they can process
      ## before they get recycled (10000).
      "max-procs" => 1,
      "bin-environment" => (
        "PHP_FCGI_CHILDREN" => "4",
        "PHP_FCGI_MAX_REQUESTS" => "10000"
      ),

      ## configure additional bits
      "bin-copy-environment" => (
        "PATH", "SHELL", "USER"
      ),
      "broken-scriptfilename" => "enable"
      "idle-timeout" => 20,
    ))
  )

Now tell lighttpd to also listen on port 8502. And in case requests come in
over this port, use the defined FastCGI server for \*.php. In this case we
configure PHP 5.2. Similarily you can also define a block for PHP 5.1 (on a
suggested port 8501) and PHP 6.0::

  $SERVER["socket"] == ":8502" {
    fastcgi.server    = ( ".php" =>
      ((
        "bin-path" => "/usr/local/php/5.2dev/bin/php-cgi",
        "socket" => "/tmp/php8052.socket",
        "max-procs" => 1,
        "idle-timeout" => 20,
        "bin-environment" => (
          "PHP_FCGI_CHILDREN" => "4",
          "PHP_FCGI_MAX_REQUESTS" => "10000"
        ),
        "bin-copy-environment" => (
          "PATH", "SHELL", "USER"
        ),
        "broken-scriptfilename" => "enable"
      ))
    )
  }

Tell lighttpd to also listen on port 8502. Because we're not overriding the
fastcgi.server configuration from the global configuration (the bit were we
set-up PHP 5.3 in the first few lines of the file), it will reuse this
fastcgi.server configuration enabling requests for .php files coming in on
port 80 or 8503 to be processed by the globally configured FastCGI server::

  $SERVER["socket"] == ":8503" {
  # Just some dummy text because lighttpd doesn't allow
  # empty definitions.
  }

With the configuration made, you only have to restart lighttpd with
``/etc/init.d/lighttpd force-reload`` and PHP is ready to go on multiple ports
with different PHP versions.

.. _moving: http://derickrethans.nl/xdebug-moved-to-a-new-server.html
.. _Xdebug: http://xdebug.org
.. _lighttpd: http://www.lighttpd.net/
.. _`PHP Unit`: http://www.phpunit.de/
