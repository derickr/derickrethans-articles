First release of the D-Bus extension
====================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2010-04-06 10:33 Europe/London
   :Tags: blog, php, cms, dbus, extensions

A few days ago I made the first beta release of the `D-Bus extension`_ that I have
been working on for a while. D-Bus_ is a message bus system, a simple way for
applications to talk to one another. I started working on this because my
cellphone, an OpenMoko FreeRunner_. This cellphone (or rather, a mobile Linux
computing device) implements `freesmartphone.org`_ APIs to talk to all the
hardware that is available on the device. This includes GSM and SIM card
interfaces, but also GPS, Audio and PIM services.

However, many other applications on the Linux desktop speak D-Bus. This
includes system services such as the notification daemon, the screen saver and
hardware plug-in detection as well as desktop applications such as Pidgin_ and
Empathy_.

I've given a presentation_ on the extension at the `PHP London conference`_
earlier this year, and will also be speaking on this subject as part of the
"PHP Inside" and "PHP on the D-Bus" talks at `Tek-X`_ (Chicago, May 18-21),
`International PHP Conference Spring`_ (Berlin, May 30-June 2) and the 
`Dutch PHP Conference`_ (June 10-12).

.. _`D-Bus extension`: http://pecl.php.net/package/dbus
.. _`D-Bus`: http://www.freedesktop.org/wiki/Software/dbus
.. _FreeRunner: http://www.openmoko.com/freerunner.html
.. _`freesmartphone.org`: http://www.freesmartphone.org/
.. _Pidgin: http://www.pidgin.im/
.. _Empathy: http://live.gnome.org/Empathya
.. _`Tek-X`: http://tek.phparch.com/schedule/
.. _`Dutch PHP Conference`: http://phpconference.nl/
.. _`International PHP Conference Spring`: http://it-republik.de/php/phpconference2010se/
.. _`PHP London Conference`: http://www.phpconference.co.uk/
.. _presentation: http://derickrethans.nl/talks/dbus-london2010.pdf
