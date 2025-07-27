Debugging with multiple users
=============================

.. articleMetaData::
   :Where: Amsterdam, The Netherlands
   :Date: 20090611 1047 CEST
   :Tags: blog, cms, php, work

As author of `Xdebug`_, people ask me
often the question how to handle the case in teams when there is one
development server and multiple developers working on the same project
on that server. Xdebug only allows you to specify *one* IP address
to connect to (through `xdebug.remote_host`_ )
while doing `remote debugging`_. It does not automatically connect back to the IP
address that runs the browser the request the PHP scripts because of
security reasons. You don't want everybody on the Internet to be able to
run a debugging session against your code for example. There is no
problem if all developers are working on a different project, because
the xdebug.remote_host setting can be made for each directory (through
Apache's .htaccess functionality). However, for the case where multiple
developers work on the same code, the .htaccess trick won't work as the
directory in which the code lives is the same.

.. image:: /images/content/xdebug_logo.png
   :align: left

Now,
in order to solve the above mentioned issue, you will need to run a DBGp
proxy. DBGp is the protocol, designed by `ActiveState`_ and myself to
facilitate communication between an IDE (such as `Komodo`_, or any of the other
`listed clients`_ )
and PHP+Xdebug. A DBGp proxy is a bit of software that acts as a
redirector for DBGp streams. In order to make things work for multiple
developers and one source base, you set Xdebug's xdebug.remote_host
setting to the machine on which the DBGp proxy runs. This is most likely
going to be on the same machine that acts as development server, so that
the xdebug.remote_host setting should be set to "127.0.0.1"
(i.e. localhost). The proxy server the listens for IDE connections. An IDE
needs to register itself with the DBGp proxy by using the `proxyinit command`_.
This command requires an "idekey" that is a
unique identifier for each client (IDE). Every developer should have its
own unique idekey (I usually just pick my name), and this idekey should
be configurable in the IDE. For Komodo, it's at
Edit->Preferences->Debugger->Connection->"I am running
a debugger proxy and Komodo should use it"->"Proxy
Key". In Komodo you also need to select "a system-provided
free port" in the same configuration panel. When initiating the
debugging session from the browser with either
XDEBUG_SESSION_START=session_name as GET/POST/COOKIE parameter, or
export XDEBUG_CONFIG="idekey=session_name" from the command
line, make sure to change "session_name" to the idekey as
configured in your IDE. (See `the documentation`_ on
how to set this up). The Xdebug `Firefox extension`_ also has a setting for
this. You have to configure Xdebug's `xdebug.remote_host`_ setting to the IP
address of the machine that the proxy runs at. Xdebug
itself does not see a difference between either the proxy and a normal
IDE. But the proxy itself now knows because of the configured idekey on
how to forward the requests and responses to the correct client.

You can find the DBGp proxy code as part of the **python** remote
debugging package that ActiveState `provides`_.
By default it listens for IDE registrations at port 9001, and for Xdebug
connections at port 9000. To run the proxy, do:

::

	tar -xvzf Komodo-PythonRemoteDebugging-5.1.3-28369-linux-x86_64.tar.gz
	cd Komodo-PythonRemoteDebugging-5.1.3-28369-linux-x86_64
	cd bin
	./pydbgpproxy

This outputs:

::

	INFO: dbgp.proxy: starting proxy listeners.  appid: 30430
	INFO: dbgp.proxy:     dbgp listener on 127.0.0.1:9000
	INFO: dbgp.proxy:     IDE listener on  127.0.0.1:9001

Running a DBGp proxy also allows you to avoid NAT issues where (as seen
from PHP+Xdebug on the server) all connections seem to come from the
same IP (because your internal network is NATted). In this case, you can
simple run the dbgp proxy on your NAT machine, configure
xdebug.remote_host setting to the IP address of your NAT machine, and
configure the IDEs to connect to the proxy running at
<NAT-machine>:9001.

As a last note; there is a patch to allow the
connect-back-to-requesting-IP-address functionality that is not
available directly in Xdebug. This patch, written by Brian Shire and
Lucas Nealan of `Facebook`_ made its
way into Xdebug 2.1. However, great care should be taken by using this
functionality. It does not make the NAT situation as outlined above work
however.


.. _`Xdebug`: http://xdebug.org
.. _`xdebug.remote_host`: http://xdebug.org/docs/remote#remote_host
.. _`remote debugging`: http://xdebug/docs/remote
.. _`ActiveState`: http://activestate.com
.. _`Komodo`: http://activestate.com/komodo
.. _`listed clients`: http://xdebug.org/docs/remote#clients
.. _`proxyinit command`: http://xdebug.org/docs-dbgp.php#just-in-time-debugging-and-debugger-proxies
.. _`the documentation`: http://xdebug.org/docs/remote#starting
.. _`Firefox extension`: https://addons.mozilla.org/en-US/firefox/addon/3960
.. _`provides`: http://aspn.activestate.com/ASPN/Downloads/Komodo/RemoteDebugging
.. _`Facebook`: http://facebook.com

