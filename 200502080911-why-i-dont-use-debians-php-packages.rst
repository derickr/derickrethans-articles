Why I Don't Use Debian's PHP Packages
=====================================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20050208 0911 CET
   :Tags: php, work

From their 4.3.10-3 changelog:

::

	Enable Zend Thread Safety for all SAPIs, meaning that our modules
	are now compiled for ZTS APIs as well.

I couldn't believe that they did this, so I checked it in the source... their rules indeed include the
--enable-experimental-zts switch. Tip: Compile your own PHP packages for Debian!



