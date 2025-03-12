And Then There Was PIE
======================

.. articleMetaData::
   :Where: London, UK
   :Date: 2025-03-18 14:50 Europe/London
   :Tags: blog, php
   :Short: pie1

In tandem with the `PECL <https://pecl.php.net/>`_ website, which hosts around
400 packages, PHP developers have been able to install third party extensions
for several decades now with the PECL installer.

PECL, which stands for the PHP Extension Community Library was a sister project to
`PEAR <https://pear.php.net>`_, the PHP Extension and Application Repository.
This repository has largely been replaced by `Packagist
<https://packagist.org/>`_, which hosts PHP user land code to be installed
through `Composer <https://getcomposer.org/>`_. Unlike PEAR, Packagist allows
anybody to register a library of PHP code, which can then be composed through
a file (``composer.json``) and made available to people's application code.

For extensions written in C, PECL was the only option. The PECL website has
not been properly maintained for a long while now. And there are issues where
the package description and release information XML files can not even contain
higher-byte characters due to a very old misconfiguration of MySQL—back when
it did not support Unicode and UTF-8 properly.

Updating the website, for example for adding new releases releases, is also
not the easiest thing to do. Unlike adding a Git tag as you can do with
Composer packages, it requires modifying an `unwieldy XML file
<https://github.com/xdebug/xdebug/blob/master/package.xml>`_, and uploading
the packages manually through a web form.

With this in mind, about a year and a half ago, I started drafting the
requirements for "New PECL". The initial goals I came up with were:

- A new simplified installer, that does not rely on PEAR
- Ways to configure which (already installed) PHP version should get which
  extension version(s)
- Cryptographic verification of released extension tags
- Removing the reliance on a PECL website
- No longer relying on ``package.xml``

The `PHP Foundation <https://thephp.foundation/>`_ bid for funding by the
`Sovereign Tech Agency <https://www.sovereign.tech/tech/php>`_ — an agency of
the German government — to develop this new tool. 

From my initial requirements, `James Titcumb <https://www.jamestitcumb.com/>`_
and others worked on a more detailed `design
<https://github.com/ThePHPF/pie-design>`_ as part of their work for the PHP
Foundation. James then set to `work on the installer
<https://github.com/php/pie>`_ resulting in the `first pre-release
<https://thephp.foundation/blog/2024/11/19/pie-pre-release/>`_ last November.

Since then, the tool has matured with more features, and better integration
with Packagist, which now also features `PHP extensions
<https://packagist.org/extensions>`_ besides public PHP packages. Many
extensions, including `Xdebug
<https://packagist.org/packages/xdebug/xdebug>`_, can now be installed with
PIE.

`Documentation for PIE <https://github.com/php/pie/blob/main/docs/usage.md>`_
is available on GitHub.

Binary builds for Windows are also supported, and we have developed a `GitHub
action <https://github.com/php/php-windows-builder>`_ to help extension
authors to do this in the right way for PIE.

PIE's feature set is fairly compatible with PECL, and now is the time to start
using it in your projects. There are sometimes still a few kinks, such as a
recent issue with the `MongoDB extension's
<https://github.com/mongodb/mongo-php-driver>`_ use of `Git submodules
<https://github.com/mongodb/mongo-php-driver/blob/v1.x/.gitmodules>`_. This is
already fixed in the 0.7.0 of PIE, but more issues are likely to be
encountered.

At some point in the future, the PECL website will be discontinued leaving PIE
installing PHP extensions from Packagist as the only available option.

**If you are an extension author,** then now is the time to `add support
<https://github.com/php/pie/blob/main/docs/extension-maintainers.md>`_ for PIE
in your extension.

In most cases, this requires creating a ``composer.json`` file in the root of
your GIT repository, such I have done for `Xdebug
<https://github.com/xdebug/xdebug/blob/master/composer.json>`_, and others
have done for the `APCu
<https://github.com/krakjoe/apcu/blob/master/composer.json>`_, `CSV
<https://gitlab.com/Girgias/csv-php-extension/-/blob/master/composer.json?ref_type=heads>`_,
`Imagick <https://github.com/Imagick/imagick/blob/develop/composer.json>`_,
and many other extensions.

**If you are a user of extensions,** that you currently install through PECL,
then now would be a great time to start using PIE in your development
environments. Only through superior testing will the new PIE installer reach
maturity, and become ready to have a 1.0 release.

And then we can celebrate with an actual pie. 🥧
