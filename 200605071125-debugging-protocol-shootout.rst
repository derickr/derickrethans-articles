Debugging Protocol Shoot-out
============================

.. articleMetaData::
   :Where: Dortmund, Germany
   :Date: 20060507 1125 CEST
   :Tags: php, xdebug

(Note: I might be biased as I'm one of the authors of the `DBGp`_ protocol.)

(Note: As the PHP IDE project seem to have swapped the meanings of
client and server in their specification I will use the terms
"debugging engine" and "IDE" in this overview)

Since a few days Zend has `published`_ their
debugging protocol that they're going to implement for their Eclipse
PHP IDE. This protocol is apparently a derivate of the protocol that
they now use for their Zend Studio. Lukas already `pointed out`_ in his
opinion open source should work in such a way that the best technical
solution should be used, but the Eclipse project `seems`_ to work a bit
different. In order to have a small view on what the "better"
debugging protocol is I made a small comparison between the protocol
that Zend published and `DBGp`_ that is used in `Komodo`_ and a number of other
commercial, free and open source IDEs.

*Protocol Communication*

The PHP IDE protocol is binary while DBGp is an ASCII/XML based format.

A binary protocol uses less data over the wire and will therefore be
faster to transport data with. However a binary protocol is also less
flexible as all data types need to be defined exactly and the protocol
specifications are required to be very rigid. An ASCII/XML based
protocol uses more bandwidth, but it is more flexible as the format
doesn't have to be so rigid. It is also much easier to debug by just
looking at what goes over the wire.

*Session Initialization*

The PHP IDE protocol allows you select a host and port to which the
debug engine should connect to the IDE through an HTTP parameter in the
request to the webserver. DBGp does not have such mechanism.

On first sight it seems like a good idea to have the visitor of the web
site to select which host and port to connect to as this makes things
more flexible. However you should realize that not only the person who
is debugging the application and set the host and port, but also
everybody else who is visiting the site. Unless the debugging engine
has configuration options to only allow a certain set of hosts this
makes it easy for an attacker to manipulate an application. DBGp
instead has the facility for `Proxy Servers`_ that allow you to have multiple people debugging
applications on the same development server.

*Transporting Variables*

The PHP IDE protocol uses serialized data to transfer variables from the
debugging engine to the IDE. (It does not define what kind of
serialization is used, so I have to assume it's the same as the output
of PHP's serialize() call). DBGp uses an XML format.

Of course serialized PHP data uses less data and an XML format to
describe a variable takes more space. But PHP's serialized data is a
format that is only specific for PHP and will not match correctly to
other languages. DBGp's XML format is language agnostic and allows the
definition of a typemap to see which language's native type maps to a
DBGp data type.

The PHP IDE protocol allows you to select the maximum depth of variable
contents to return when you request a variable, DBGp has something
similar by means of setting options for a debugging connection. Besides
a maximum depth DBGp also provides functionality for limiting the amount
of elements that is returned in one array, and how much of the content
of a variable is returned. This was added so that one array with 10.000
elements containing a 500 byte string can not clog down the connection
between debugging engine and IDE.

*Breakpoint Support*

The PHP IDE protocol only supports either conditional breakpoints with a
condition, or static breakpoints on a file/line number combination. The
DBGp protocol supports breakpoints on exceptions, function entry,
function return and watches as well. The DBGp protocol also facilitates
that certain breakpoints should only be used when they're reached after
or before a certain hit-count.

*Output Capturing*

The PHP IDE debugger protocol only supports "PHP Script
Output", DBGp supports both stdout and stderr capturing in two
types of streams.

*Conclusion*

The proposed protocol for the PHP IDE seems to be less powerful than the
DBGp protocol. I wonder then also whether people on the Zend team
actually had a look at the DBGp protocol. There were plenty of
opportunies where we suggest that there is some discussion about the
protocol but there was never any answer to this. In case you're
interested in the DBGp protocol and want to discuss things with it, or
if you want some help by adapting DBGp to use in an IDE that you are
developing feel free to contact us at the `Xdebug general mailinglist`_ .


.. _`DBGp`: http://xdebug.org/docs-dbgp.php
.. _`published`: http://www.eclipse.org/php/docs.php
.. _`pointed out`: http://www.pooteeweet.org/blog/357
.. _`seems`: http://www.pooteeweet.org/blog/359
.. _`Komodo`: http://activestate.com/komodo
.. _`Proxy Servers`: http://xdebug/docs-dbgp.php#just-in-time-debugging-and-debugger-proxies
.. _`Xdebug general mailinglist`: http://xdebug.org/support.php

