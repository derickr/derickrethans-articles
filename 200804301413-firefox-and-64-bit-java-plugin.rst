Firefox and 64 bit Java Plugin
==============================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20080430 1413 CEST
   :Tags: blog, php, work

Because the lazy bastards at `Sun`_ still didn't manage to make a 64 bit version of their Java plugin, you
have to go through all sorts of hoops to make it actually work. Normally
I wouldn't really care about this, but unfortunately my bank decided to
require Java working in the browser for authentication. Four hours of my
time later, I managed to get it working. To save others from some of the
pain, here is how I did that:

1. Download from `ftp://ftp.tux.org/pub/java/JDK-1.4.2/amd64/`_ the file `j2sdk-1.4.2-03-linux-amd64.bin`_ .

2. I downloaded it to ~/install, so go into that directory and run:

   ::

      chmod +x j2sdk-1.4.2-03-linux-amd64.bin

3. Run:

   ::

      ./j2sdk-1.4.2-03-linux-amd64.bin

   and wait until it's done installing (make sure it mentions
   "Uncompressing Blackdown Java 2 Standard Edition SDK
   v1.4.2-03" at some point).

4. Now, to make it work as a plugin, you have to *link* (not copy,
   as that makes the browser crash) the plugin to your mozilla directory.
   For me:
   
   ::

      cd /home/derick/.mozilla/plugins
      ln -s /home/derick/install/j2sdk1.4.2⇢
         /jre/plugin/amd64/mozilla/libjavaplugin_oji.so .

5. Restart the browser and check whether the blackdown java plugin shows
   up if you go to `about:plugins`_ .


.. _`Sun`: http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=4802695
.. _`ftp://ftp.tux.org/pub/java/JDK-1.4.2/amd64/`: ftp://ftp.tux.org/pub/java/JDK-1.4.2/amd64/
.. _`j2sdk-1.4.2-03-linux-amd64.bin`: ftp://ftp.tux.org/pub/java/JDK-1.4.2/amd64/03/j2sdk-1.4.2-03-linux-amd64.bin
.. _`about:plugins`: about:plugins

