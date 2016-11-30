Not Finding the Symbols
=======================

.. articleMetaData:: :Where: London, UK
   :Date: 2016-11-30 14:55 Europe/London
   :Tags: blog, php, mongodb
   :Short: undefsym

Yesterday we released the new version of the `MongoDB Driver for PHP`_, to
coincide with the release of `MongoDB 3.4`_. Not long after that, we received
`an issue`_ through GitHub titled "**Undefined Symbol php_json_serializable_ce
in Unknown on Line 0**".

*TL;DR*: Load the JSON extension before the MongoDB extension.

.. _`MongoDB Driver for PHP`: https://pecl.php.net/package/mongodb
.. _`MongoDB 3.4`: https://docs.mongodb.com/master/release-notes/3.4/
.. _`an issue`: https://github.com/mongodb/mongo-php-driver/issues/475

The newly released version of the driver has support for PHP's
`json_encode()`_ through the `JsonSerializable`_ interface, to convert some of
our internal BSON types (think `MongoDB\\BSON\\Binary`_ and
`MongoDB\\BSON\\UTCDateTime`_) directly to JSON. For this it uses functionality
in PHP's `JSON extension`_, and with that the ``php_json_serializable_ce``
symbol that this extension defines.

.. _`json_encode()`: https://php.net/json_encode
.. _`JsonSerializable`: https://php.net/jsonserializable
.. _`MongoDB\\BSON\\Binary`: https://php.net/mongodb_bson_binary
.. _`MongoDB\\BSON\\UTCDateTime`: https://php.net/mongodb_bson_utcdatetime
.. _`JSON extension`: https://php.net/json

We run our test suite on many different distributions, but (nearly) always
with our own compiled PHP binaries as we need to support so many versions of
PHP (5.4-5.6, 7.0, and now 7.1), in various configurations (ZTS_, or not;
32-bit or 64-bit). It came hence quite as a surprise that a self-compiled
extension would not load for one of our users.

.. _ZTS: http://php.net/manual/en/internals2.buildsys.environment.php

When compiling PHP from its source, by default the JSON extension becomes part
of the binary. This means that the JSON extension, and the symbols it
implements are always available. Linux distributions often split out each
extension into their own package or shared object. Debian_ has `php5-json`_
(on which `php5-cli`_ depends), while Fedora has `php-json`_. In order to make
use of the JSON extension, you therefore need to install a separate package
that provides the shared object (``json.so``) and a configuration file.
Fedora installs the  ``20-json.ini`` file in ``/etc/php.d/``. Debian installs
the ``20-json.ini`` file in ``/etc/php5/mods-available`` with a symlink to
``/etc/php5/cli/conf.d/20-json.ini``. In both cases, they include the required
``extension=json.so`` line that instruct PHP to load the shared object and
make its symbols (and PHP functions) available.

.. _Debian: https://www.debian.org/
.. _`php5-json`: https://packages.debian.org/jessie/php5-json
.. _`php5-cli`: https://packages.debian.org/jessie/php5-cli
.. _`php-json`: https://apps.fedoraproject.org/packages/php-json/overview/

A normal PHP binary uses the dlopen_ system call to load a shared object, with
the ``RTLD_LAZY`` flag. This flag means that symbols (such as
``php_json_serializable_ce``) are only resolved lazily, when they are first
**used**. This is important, because PHP extensions and the shared objects
they live in, can depend on each other. The MongoDB extension `depends on`_
``date``, ``spl`` and ``json``. After PHP has loaded all the shared
extensions, it registers the classes and functions contained in them, in `an
order to satisfy this dependency graph`_. PHP makes sure that the classes and
functions in the JSON extension are registered before the MongoDB extension,
so that when the latter uses the ``php_json_serializable_ce`` symbol to
declare that the ``MongoDB\\BSON\\UTCDateTime`` class implements the
``JsonSerializable`` interface the symbol is already available. 

.. _dlopen: http://man7.org/linux/man-pages/man3/dlopen.3.html
.. _`depends on`: https://github.com/mongodb/mongo-php-driver/blob/1.2.0/php_phongo.c#L2189
.. _`an order to satisfy this dependency graph`: https://github.com/php/php-src/blob/php-7.1.0beta3/Zend/zend_API.c#L1862

Distributions often want to harden their provided packages with additional
security features. For that, they compile binaries with additional features
and flags.

Debian patches_ PHP to replace the ``RTLD_LAZY`` flag with ``RTLD_NOW``.
Instead of resolving symbols when they are first **used**, this signals to the
dlopen_ system call to resolve the symbols **when the shared object is
loaded**. This means, that if the MongoDB extension is loaded before the JSON
extension, the symbols are not available yet, and the linker throws the
"**Undefined Symbol php_json_serializable_ce in Unknown on Line 0**" error
from our bug report. This is not a problem that only related to PHP; TCL has
`similar issues`_ for example.

With Fedora, the same issue is present, but shows through slightly different
means. Instead of patching PHP to replace ``RTLD_LAZY`` with ``RTLD_NOW``, it
uses linker flags (``"-Wl,-z,relro,-z,now"``) to force binaries to resolve
symbols as soon as they are loaded process wide. This `Built with BIND_NOW`_
security feature goes hand in hand with `Built with RELRO`_. The explanation
on **why** these features are enabled on Fedora is well described on their
wiki_. Previously, this did expose `an issue`_ with an internal PHP API
regarding creating a DateTime object.

.. _patches: https://anonscm.debian.org/git/pkg-php/php.git/tree/debian/patches/0046-php-5.4.0-dlopen.patch?h=master-5.6
.. _`similar issues`: https://groups.google.com/forum/#!topic/comp.lang.tcl/RRumv23ZIJc
.. _`Built with BIND_NOW`: https://fedoraproject.org/wiki/Security_Features_Matrix#Built_with_BIND_NOW
.. _`Built with RELRO`: https://fedoraproject.org/wiki/Security_Features_Matrix#Built_with_RELRO
.. _wiki: https://fedoraproject.org/wiki/Security_Features_Matrix#Built_with_RELROa
.. _`an issue`: https://jira.mongodb.org/browse/PHP-1270

But where does this leave us? The solution is fairly simple: You need to make
sure that the JSON extension's shared object is loaded before the MongoDB
extension's shared object. PECL's ``pecl install`` suggests to add the
``extension=mongodb.so`` line to the end of ``php.ini``. Instead, on Debian,
it would be much better to put the ``extension=mongodb.so`` line in a separate
``99-mongodb.ini`` file under ``/etc/php5/mods-available``, with a symlink to
``/etc/php5/cli/conf.d/99-mongodb.ini`` and
``/etc/php5/apache2/conf.d/99-mongodb.ini``::

	cat << EOF > /etc/php5/mods-available/mongodb.ini
	; priority=99
	extension=mongodb.so
	EOF
	php5enmod mongodb

On Fedora, you should add the ``extension=mongodb.so`` line to the new file
``/etc/php.d/50-mongodb.ini``::

	echo "extension=mongodb.so" > /etc/php.d/50-mongodb.ini

Alternatively, you can install the distribution's package for the MongoDB
extension. Fedora currently has the `updated 1.2.0 release for Rawhide`_
(Fedora 26). Debian however, does not yet provide a package for the latest
release yet, although an `older version (1.1.7) is available` in Debian
unstable. At the time of this writing, Ubuntu only provides `older versions`
for Xenial and Yakkety.

.. _`updated 1.2.0 release for Rawhide`: https://apps.fedoraproject.org/packages/php-pecl-mongodb
.. _`older version (1.1.7) is available`: https://packages.debian.org/sid/php-mongodb
.. _`older versions`: http://packages.ubuntu.com/search?keywords=php-mongodb&searchon=names
