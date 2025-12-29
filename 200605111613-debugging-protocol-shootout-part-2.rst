Debugging Protocol Shoot-out (Part 2)
=====================================

.. articleMetaData::
   :Where: Dieren, The Netherlands
   :Date: 20060511 1613 CEST
   :Tags: php, xdebug

`Guy Harpaz`_ posted a
reply to my `Debugger Protocol Shoot-out`_ on his newly created blog. In this `post`_ he tries to counter some of my arguments. I don't quite understand why
he couldn't just have posted his comments here though. (And in case why
you're wondering why I'm not adding a comment to his post: You need a
blogger.com account to comment, and I refuse to create one just to be
able to comment on a blog).

The first thing that strikes me as odd is the following quote:

::

	We chose performance (binary based protocol) over easy
	server side implementation (textual based protocol).

Easy implementation was never the reason for going with a textual based
protocol. The main reason was to allow extensions to the protocol be as
easy as possible. Cramming as much data in as little bytes as possible
is also not going to increase performance if your protocol has no ways
to paginate through arrays with many elements or when you have no means
of limiting the amount of data in one variable to be send over the wire.
This is very useful incase you have a few MB files in variables for
example.

The second argument why you should "choose Zend Studio's
protocol" is:

::

	DBGp supports several scripting languages and from this
	derives some of its disadvantages. We believe that a debug protocol
	should be as 'close' as possible to the PHP language.

This doesn't make much sense to me, as far as I know Eclipse is a multi
language IDE. Secondly, which disadvantages there are is not
mentioned.

Then Guy tries to compare some security points:

::

	The Proxy mechanism has few bugs in it

If that is the case, would you mind telling us where?

He further mentions:

::

	The PHP IDE debug protocol does not specify requirements for
	a security system but defines that the Debug server should receive the
	client IP before initializing the session. By using the IP,
	implementing a security mechanism is very simple.

Xdebug doesn't specify any requirements either. Xdebug could implement
an extra security that checks whether the IP is allowed to be used as
debugging IDE client. However, when you are on a NAT-ted local network,
and your dev server is somewhere outside of this network you have a
problem without a proxy. As there is no way for the debugging engine on
the developer machine to connect to individual IDEs anymore (as the IPs
are simply not accessible by the devel server because of NAT). The
proxy-server based approach of DBGp allows your devel server to connect
to the proxy on one IP and one port (which also means you have to poke
only one hole in your firewall). The IDE then simply have to make
itself known to the proxy server with its "idekey" to allow
debugging through a NAT-ted network.

The last point that Guy raises is:

::

	DBGp supports PHP execution stdout and stderr as a general
	solution for all supported languages. PHP IDE as a specific protocol
	for PHP can distingue between header output and standard output and
	between all types of PHP errors (warnings, errors and
	notices).

It really has nothing to do whether a protocol is specifically made for
PHP or not to allow the sending of HTTP headers. DBGp currently has no
options for sending headers, but that would not be hard to add. It
definitely would not be the selling point on why somebody should
"choose Zend Studio's protocol".


.. _`Guy Harpaz`: http://guyharpaz.blogspot.com/
.. _`Debugger Protocol Shoot-out`: /debugging-protocol-shootout.html
.. _`post`: http://guyharpaz.blogspot.com/2006/05/php-ide-debug-protocol.html

