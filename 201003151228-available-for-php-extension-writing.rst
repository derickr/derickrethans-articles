Available for PHP Extension Writing
===================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2010-03-15 12:28 Europe/London
   :Tags: blog, php, xdebug, cms, work, extensions

Slightly more than a month ago I left my job as team-lead of `eZ Components`_ at `eZ
Systems`_ behind, to focus on something new. During the past month I've been
contemplating about what to do next and realized that I do not want to take on a
new full-time position right away.  Instead I will be available to work on
(custom) PHP extensions and internals related issues. Extensions are a great
way around PHP's limitations and performance issues.

As first project I am working on a "QuickHash_" extension for StumbleUpon_.
This extension circumvents PHP's hefty memory (and performance) overhead by
providing more specific data structures. The extension currently implements 
integer sets and integer to integer hashes. I am now adding integer to string
hashes and string to integer hashes. The QuickHash extension will be released
under the PHP Licence and I will dedicate another post to it later.

If, like StumbleUpon_, you are also interested in having work done on PHP or a
specific extensions feel free to contact_ me. I'd be happy to discuss things
with you.


.. _`eZ Components`: http://ezcomponents.org
.. _`eZ Systems`: http://ez.no
.. _`QuickHash`: https://github.com/derickr/quickhash
.. _`StumbleUpon`: http://www.stumbleupon.com/
.. _contact: /who.html
