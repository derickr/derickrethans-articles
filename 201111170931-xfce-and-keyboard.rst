XFCE and moving windows around with the keyboard
================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2011-11-17 09:31 Europe/London
   :Tags: blog, linux
   :Short: xfcekeyb

During a recent ``apt-get update && apt-get dist-upgrade`` I suddenly found
myself with the horrors that are Gnome 3 and the Gnome shell, and of course, at
the least opportune moment when I was actually planning on heading into an
office (more about that later). I've always been a big Sawfish_ fan, and was
annoyed when Gnome replaced that with Metacity_, but I always managed to get
Sawfish back up again.  No more luck now, and without a `Debian package
maintainer`_ I decided to try out XFCE_.

I found XFCE_ to be as easy to use with the keyboard as Sawfish but with one
big missing issue: I couldn't change the size and position of windows at the
keyboard anymore. I hardly use a mouse as it hurts too much, and always relied
on those keyboard shortcuts to position my windows so I was quite a bit at a
loss. After some searching I ran into a post on nabble_ that contained a script
that almost did what I want. It uses a bunch of standard X tools (xprop_ and
xwininfo_) to find some information about the active window and its properties.
Then depending on some parameters it would use wmctrl_ to move and/or resize
the window. For example: ``./bin/window-geometry-control.sh -m left`` would
move the window 1 pixel to the left.

That's all good, but way too slow. I basically just want to bump my windows to
the top, right, bottom and left sides of my screen. So besides the simple move
and resize, I added what Sawfish originally also had: move left/right/top and
bottom; and well as resize-to left/right/top and bottom. Now with the command
``./bin/window-geometry-control -b left`` I can move the active window all the
way to the left, and with ``./bin/window-geometry-control -s bottom`` I can
resize my window from its current position all the way to the bottom of the
screen. To each of the four directions and two methods I assigned keyboard
shortcuts in ``xfce4-settings-manager``, Keyboard, Application Shortcuts.

.. image:: /images/content/xfce-keyboard.png

The screenshot above shows those assigned keyboard shortcuts. I've put the
modified script on github_. As you can see I hardcoded the width and height of
my own screen in it. Feel free to make this dynamic and send a pull request.

.. _Sawfish: http://xwinman.org/sawfish.php
.. _Metacity: http://xwinman.org/metacity.php
.. _XFCE: http://xwinman.org/xfce.php
.. _`Debian package maintainer`: http://bugs.debian.org/cgi-bin/pkgreport.cgi?pkg=sawfish-gnome;dist=unstable
.. _nabble: http://old.nabble.com/New-Behavior-about-moving-windows.-td22293281.html
.. _xprop: http://www.xfree86.org/current/xprop.1.html
.. _xwininfo: http://www.xfree86.org/4.2.0/xwininfo.1.html
.. _wmctrl: http://tomas.styblo.name/wmctrl/
.. _github: https://github.com/derickr/scripts/blob/master/window-geometry-control.sh
