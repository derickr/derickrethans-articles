Microsoft Web Developer Summit wrap-up
======================================

.. articleMetaData::
   :Where: Redmond, US
   :Date: 20081217 1556 CET
   :Tags: conference, php, travel, work

About a month ago I visited Microsoft's Web Developers Summit to come
and talk with many other people from the PHP Community on how to get
closer together and make the PHP experience better on Windows and with
other Microsoft products. Although I was quite skeptical about the whole
event and was slightly worried about it having only marketing and
brainwashing sessions I set off just after php|works to Redmond (or
actually Bellevue).

The first day started with Sam Ramji introducing the event, and
Microsoft's strategy and "dedication" towards Open Source and
open standards. Of course as part of this, OOXML was mentioned which
caused some small discussion about the standardization process of it.
Also mentioned was the grant to the Apache foundation as well as that
Microsoft (or at least parts of it) now see that being open about things
could mean growth in revenue. Some of the feedback that he got included
making sure that it's actually easier for Unixy people to remotely
maintain a Windows server — for example by providing the set of Unix
tools from the /bin directory as well as providing built-in SSH
support.

The OOXML thing got a follow up later in the day, where a member of the
Office team tried to debunk the myths that the OOXML standard process
was all messed up. Instead however, we were more interested in seeing
how we could get a C-library to deal with office documents.
Unfortunately we didn't really go into that specific topic.

Lauren Cooney introduced the `Microsoft Web Platform`_ and installer that can install a whole web stack. At
the moment PHP isn't part of this *yet* but it will come soon. One
of the things we discussed here was which extensions should be bundled.
To which the obvious answer should be: "at least everything that
PHP has enabled by default". Besides the installer, they're also
working on a `Web application installer`_ with which also PHP applications that can be
installed. This could also be something that we would like to see `eZ Publish`_ in for example.

IIS now also features a `"mod_rewrite"`_ extension that can also import Apache mod_rewrite rules from .htaccess
files. It would also be nice if php_value settings could be imported and
applied as well through this extension, just like the mod_rewrite
rules.

The SQL Server team was demoing the new SQL-server driver for PHP.
However, they got sort of grilled about not listening to what the users
want. At the moment, there are the following "irritating"
issues: There is no PDO driver, UTF-8 is not supported and it only works
on Windows. The first two are definitely deal-breakers for `us`_ if we'd want to support MS SQL Server
properly in our software. We discussed those issues later and it seems
mostly a resource issue. Let's hope they can prioritize those points
soon.

Juan Rivera introduced a Visual Studio plugin for editing PHP: `VS.Php`_ . I had spoken to
Juan before regarding `Xdebug`_ that he
integrated into VS.Php.

There were many other talks, but they didn't qualify for writing notes
about. What I found most interesting were the interactions with the
developers, and the least interesting where the attendee show cases. I
think it was very worthwhile to see what kind of direction Microsoft is
going, especially the parts where they want to be part of the community.
There's still quite a road to go, especially because of legal issues,
but they're at least on the road. Contact has been made :)


.. _`Microsoft Web Platform`: http://www.microsoft.com/web/channel/products/WebPlatformInstaller.aspx
.. _`Web application installer`: http://www.microsoft.com/web/channel/products/WebApplicationInstaller.aspx
.. _`eZ Publish`: http://ez.no/ezpublish
.. _`"mod_rewrite"`: http://www.iis.net/extensions/URLRewrite
.. _`us`: http://ez.no
.. _`VS.Php`: http://www.jcxsoftware.com/vs.php
.. _`Xdebug`: http://xdebug.org

