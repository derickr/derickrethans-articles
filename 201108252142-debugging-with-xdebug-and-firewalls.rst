Remote Debugging PHP with a Firewall in the Way
===============================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2011-08-25 21:42 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-fw

*Updated on March 25th, 2022* to reflect the new setting names for Xdebug 3.
See also the `upgrade guide <https://xdebug.org/docs/upgrade_guide>`_ for
various other changes.

The PHP debugging extension Xdebug_ has "remote" debugging_ capabilities for
single-step debugging PHP applications. This works by setting your favourite
IDE into listening mode and instructing Xdebug (with one of the handy browser
extensions_ for example) to initiate debugging. When Xdebug is instructed it
tries to make a connection to the IP address and port number that are
configured_ in php.ini. On this IP address_ and port_ Xdebug expects to find a
listening IDE. This IP address can just be ``127.0.0.1`` in case your IDE and
web server/CLI script run on the same machine. But as Xdebug supports remote
debugging, PHP/Xdebug could also be running on a totally different machine than
your IDE. In that case, you need to pick the IP address of the machine on which
the IDE is running as the configuration value for ``xdebug.client_host``
(Xdebug 2 used ``xdebug.remote_host``).
Alternatively you can set ``xdebug.discover_client_host`` (Xdebug 2 used
``xdebug.remote_connect_back``) to true, so that
Xdebug may simply pick up your client machine's IP address from the HTTP
headers.

.. _Xdebug: https://xdebug.org
.. _debugging: https://xdebug.org/docs/remote
.. _extensions: https://xdebug.org/docs/remote#browser-extensions
.. _configured: https://xdebug.org/docs/remote#client_host
.. _address: https://xdebug.org/docs/remote#client_host
.. _port: https://xdebug.org/docs/remote#client_port

There could however be a firewall in the way that prevents Xdebug connecting
directly to your IDE's IP address. That can be because the network you are on
employs NAT_ in which case you could try (to convince) to get your system
administrator to install a DBGp_ proxy_. In some cases, the system administrator might not be
willing to do so, or perhaps, there is simply a black box that doesn't allow
either a proxy to be run on it, or port forwarding to be set-up on. In this
case, there is no way Xdebug can connect to your IDE's IP address and port.
Or is there?

.. _NAT: http://en.wikipedia.org/wiki/Network_address_translation
.. _DBGp: https://github.com/derickr/dbgp
.. _proxy: /debugging-with-multiple-users.html


**Breaking the Firewall Down**

There is another solution, at least, if you have SSH_ access to the
development machine where Xdebug is running on. In this case, you can simply
ssh in and set-up a tunnel::

	ssh -R 9003:localhost:9003 username@dev.example.com

This command opens up port ``9003``, configured with the first ``9003`` in
the command, for listening on the machine where you ssh-ed into. Every
connection that is made to this port is then forwarded to ``localhost:9003``,
which in this case is port ``9003`` on the machine from where you ran the
``ssh`` command from. When you set Xdebug's ``xdebug.client_host`` setting to
``localhost`` and ``xdebug.client_port`` to ``9003`` you know have instructed
Xdebug to connect to your SSH-tunneled connection which is forwarded to your
local port ``9003``. And that is exactly where your IDE is listening for
incoming debugging connections. Result: Firewall circumvented.

.. _SSH: http://en.wikipedia.org/wiki/Secure_Shell
.. _putty: http://www.chiark.greenend.org.uk/~sgtatham/putty/

If you are using Windows on the client side then you
don't have the luxury of being able to use the simple SSH client. However,
you can set-up tunnels with putty_ as well. After configuring your normal
session, go to the ``Connection``, ``SSH``, ``Tunnels`` section of the
configuration and fill in ``Source port`` and ``Destination``, and select
``Remote``, just like in this screen shot:

.. image:: /images/content/putty-tunnel-settings.png
   :align: center

Then click the ``Add`` button and you will see the following on your screen:

.. image:: /images/content/putty-tunnel-result.png
   :align: center

Don't forget to go back to the ``Session`` tab and press save. You're now
ready to login and when you do so an SSH tunnel will be created just like in
the Unix case above.

You can check whether it worked by running: ``netstat -a -n | grep 9003`` on
the SSH prompt after logging in. You should then see::

	derick@xdebug:~$ netstat -a -n | grep 9003
	tcp        0      0 0.0.0.0:9003            0.0.0.0:*               LISTEN
	tcp6       0      0 :::9003                 :::*                    LISTEN

The ``tcp6`` portion might not be there, but that's all right. There, even
with Windows: Firewall circumvented.

