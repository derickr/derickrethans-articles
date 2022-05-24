Luminous Logitech Litra on Linux
================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2022-05-24 16:17 Europe/London
   :Tags: blog, linux, opensource
   :Short: linuxlitra

I have been working on developing a course for Xdebug, PHP's Debugger. I am
now close to start making the first recordings, so I thought it would be good
to invest in some lighting and a green screen.

The light that I bought is a Logitech Litra Glow, but once it arrived I
quickly found out that if you want to control its brightness and light
temperature, you need a Windows or macOS app. I have neither, bummer.
It was not a total disaster as there are buttons on the light to do the same.

When I have had "Windows/macOS" only tech in the past, there was usually
already somebody who has reverse engineered it. For my old TomTom smart watch,
there was `ryanbinns/ttwatch <https://github.com/ryanbinns/ttwatch>`_, which I
ended up `contributing <https://github.com/ryanbinns/ttwatch/commits?author=derickr>`_
to.

It turned out that the Logitech Litra was no exception. I found a Python
implementation of a `command line and UI tool
<https://github.com/kharyam/litra-driver>`_, which does the job after messing
around with some UDEV rules to allow non-root users make use of it::

	sudo su -
	echo 'SUBSYSTEM=="usb", ATTRS{idVendor}=="046d", ATTRS{idProduct}=="c900", MODE:="0666", GROUP="plugdev"' > /etc/udev/rules.d/82-litra-glow.rules
	udevadm control --reload-rules && udevadm trigger

After I plugged in the light again, I could now control it through the Python
UI and command line tools.

However, a separate tool is not really what I was after. I rather would like
to control it from my Elgato Stream Deck through a `Go application
<https://github.com/derickr/streamdeck-goui>`_ that I have written for it.
Although I could configure it to use a command line tool invocation, I thought
it would be nicer if I could control it directly from that Go applications.
Which meant that I had to write a Go driver for it. 

.. image:: /images/content/streamdeck-litra.gif
   :title: Stream Deck controlling Litra
   :align: right

Building on top of the work on the Python driver, I extracted the specific
bytes to send to the USB device, and wrapped that with the
`derickr/go-litra-driver <https://github.com/derickr/go-litra-driver>`_. After
some research into how to talk directly with a USB device, that turned out to
be not too hard. I could now control the Litra from Go!

The next step was to `integrate the driver
<https://github.com/derickr/streamdeck-goui/commit/3fb5addaba144c93251bef41a4f05bfe1100b0bc>`_
into my Stream Deck control app. I `added a configuration
<https://github.com/derickr/streamdeck-goui/commit/27661e96f474c19925f3928e683bf7e1d3ec97d5>`_
with 2 different colour temperatures, and each with three light levels.
Combined with a state to turn the light off, that makes seven configurations
that the button can cycle through.

Success! I can now control the Logitech Litra Glow from my Stream Deck.
