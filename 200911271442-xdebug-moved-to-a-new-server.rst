Xdebug moved to a new server
============================

.. articleMetaData::
   :Where: London, UK
   :Date: 20091127 1442 GMT
   :Tags: blog, cms, php, xdebug

`Xdebug`_ 's CVS server was previously hosted at `eZ Systems`_' office, but as
I am leaving soon I needed to move this to a different server. At the same
time, I decided to also convert from CVS to SVN. So from now on, there is no
more cvs.xdebug.org, but just a `svn.xdebug.org`_. This SVN server doesn't
only host Xdebug, but also some of my other projects, such as `VLD`_ (opcode
dumper to aid PHP engine development), the `DBGp`_ specs (debugger protocol as
used by Xdebug and `ActiveState's debuggers`_) and my OpenMoko `perversions
and projects`_.

Xdebug can now be checked out from SVN with:

::

	svn co svn://svn.xdebug.org/svn/xdebug/xdebug/trunk xdebug

All other Xdebug services (`mailinglists`_, web,
e-mail and the `bug system`_)
moved to the same server as well. Just be aware that DNS servers might
still need updating when you try to access the new services.


.. _`Xdebug`: http://xdebug.org
.. _`eZ Systems`: http://ez.no
.. _`svn.xdebug.org`: http://svn.xdebug.org
.. _`VLD`: http://svn.xdebug.org/cgi-bin/viewvc.cgi/vld/?root=php
.. _`DBGp`: http://svn.xdebug.org/cgi-bin/viewvc.cgi/?root=xdebug
.. _`ActiveState's debuggers`: http://aspn.activestate.com/ASPN/Downloads/Komodo/RemoteDebugging
.. _`perversions and projects`: http://svn.xdebug.org/cgi-bin/viewvc.cgi/?root=openmoko
.. _`mailinglists`: http://xdebug.org/support.php#list
.. _`bug system`: http://bugs.xdebug.org

