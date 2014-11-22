London in Fives: The Making Of
==============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2014-11-25 08:55 Europe/London
   :Tags: blog, php, photography
   :Short: ldni5s

A few days ago I published a video called "London in Fives" on Vimeo. In this
article I am explaining how I made this, including the hardware and software
that I used.

But first, let's have a look at the video itself:

.. vimeo::
   :ID: 112413380
   :Title: London in Fives
   :Width: 1280
   :Height: 720

There are 25 sections in this video, each taking 5 seconds. Because the
frame rate is 25 fps, that means there are 125 frames per segment. All those
numbers are powers of 5, hence the title of the film: **London in Fives**.

The first and last segments are merely the title and end credits, and are just
a series of images. The more interesting bits are the 23 segments in between.

Hardware
--------

All these segments are made from single frame shots from my Nikon D300 DSLR
camera. It has a feature that allows a picture to be taken every 5 seconds
automatically. For all segments, except for the night time shot of Covent
Garden and the Ice Skating, that created the raw images.

For each segment, I usually took a few more shots than the 125 required,
usually up to a 150, to have a bit of a choice of where to start and end the
segment. In one case (the Regent's Park sunrise in the fog segment), I was happy
that I did! Due to a hard drive failure I fortunately managed to only lose
a few images, so that I still had just 125 left!

Of course it is important to keep the camera steady between all of the shots.
In most of the segments I used a `GorillaPod`_, a three legged flexible
tripod where each leg can wrap around objects. In the later scenes, I used a
normal stand-up tripod, a `Manfrotto befree`_.

The camera movements are all done in post production, except for the night
time shot of Covent Garden and the Ice Skating segments. Instead of
using my camera's "take a photo every x seconds" feature, I relied on hardware
to take both a photo every 5 seconds, but also rotate the camera slightly on
top of its tripod. The time lapsing device that I used to rotate and instruct
the camera to take a photo every 5 seconds is an Astro_. This is a disk like
device that can rotate around one axis and instruct the camera through a cable
to take a photo at specific intervals over a certain period of time. I think
that for future time lapses I will not rotate more than 30° for a 125 segment
shoot as otherwise it goes a bit too fast. 

To make sure I had my camera perfectly horizontal on my camera, I used a
`spirit level`_ that sits on top of my flash socket.

Post Processing
---------------

After taking the photos, some post-processing was necessary. There are three
types of post-processing that I had to do, depending on how the photos were
shot.

For the two segments created with the Astro, I really only had to rescale the
photos from the camera's native resolution to 1280x720.

For one other segment (Regent's Park sunrise in the fog), the GorillaPod was
sitting on a bench that didn't turn out to be stable enough and lots of
instability was introduced among the different images. I used `Hugin's`_
`align_image_stack`_ tool to align them in such a way they formed a stable
sequence of images. This tool is usually used to "align overlapping images for
HDR creation", but it also suited my use case very well. Basically, I ran the
following command::

	align_image_stack -a aligned/a -v -i -C --threads 3 *jpg

I first also tried enabling GPU support for remapping, but that just ended up
crashing the tool. The tool here is called with an output prefix of
``aligned/a`` and the ``-C`` auto-cropped the image sequence so that it covered an
area that all images shared. After the image alignment for this particular
segment, I could process it the same as the other 20 segments that were not
taken with help from the Astro.

For each of the two non-Astro taken sequences, the resulting video still shows
camera movement. This is absolutely artificial, and is basically done by
cropping the right section out of each image. I varied the size and location
of the cut out sections for each image to emulate a moving, and zooming
camera. For one segment, the Oxford Circus crossing one, I also had to adjust
rotation as the horizon wasn't flush. The rotating and cropping was done
through fairly simple PHP scripts using the GD library.

Creating Video from Images
--------------------------

After I post-processed all the image sequences, I used ffmpeg_ with a magic
incantation to create video segments. I wanted to render to webm_ as that
seemed to be the best encoder. For a sample segment, the ffmpeg_ incantation
looked like::

	ffmpeg -y -framerate 25 \
		-start_number 5711 -i 12-south-bank/small/tl3_%04d.jpg \
		-vframes 125 -r 25 \
		-vcodec vp8 -b:v 50000k -vf scale=1280:720 -crf 8 \
		12.webm

Which basically means, use ``125`` frames starting from image number ``5711``
in directory ``12-south-bank/small/`` with file format ``tl3_%04d.jpg`` at
``25`` frames a second. The codec to use is ``vp8`` with a video bit rate of
``50mbit`` and a rescaled result of ``1280:720`` pixels. The ``-crf 8``
selects ultra-high quality for ``vp8``. The output file is ``12.webm``.

I wanted ultra high quality for each segments, as later on I would be
re-encoding all the segments into the final video file, keeping as much image
definition as I could.

Music
-----

I have been a fan of `Kevin MacLeod's`_ Creative Commons licenced music for a
while, having used it for some of the newer `OpenStreetMap edit videos`_. In
this case, I did not want to release the video under a Creative Commons
license so I paid for one of his tracks, "License: Touching Moments Four -
Melody". 

With audacity_ I added the 5 beeps at the start, and I also slightly stretched
the sound to cover the full two minutes that I needed it to last. I think the
speed was reduced by about 8%. This does make the sound a bit lower in pitch,
but I do not think it hurt its original composition.

Assembling
----------

With the video segments and audio prepared, all I had to do is to stitch it
all together. Again, I used ffmpeg_ with another magic incantation to do the
dirty work. First I stitched all the videos together::

	ffmpeg -y \
		-i 00.webm \
		-i 45.webm -i 01.webm -i 17.webm -i 18.webm -i 02.webm \
		-i 20.webm -i 03.webm -i 11.webm -i 12.webm -i 27.webm \
		-i 31.webm -i 48.webm -i 10.webm -i 30.webm -i 06.webm \
		-i 34.webm -i 39.webm -i 28.webm -i 23.webm -i 43.webm \
		-i 24.webm -i 47.webm -i 08.webm \
		-i 99.webm \
		-filter_complex concat=n=25:v=1:a=0 -b:v 100M -crf 8 temp.webm

And then I finally I added the music track::

	ffmpeg -y \
		-i temp.webm \
		-i '/home/derick/media/mp3/cc-by/TouchingMomentsFour-Melody-updated.ogg' \
		-map 0 -map 1 -shortest -vcodec copy -acodec copy -strict experimental \
		final.webm

With this done, the full video was ready. I uploaded it to Vimeo, where you
can see it in all its HD glory at http://vimeo.com/derickr/london-in-fives

I hope you enjoy the video as much as I did doing all the work for this!

.. _GorillaPod: http://joby.com/gorillapod/slrzoom
.. _`Manfrotto befree`: http://www.manfrotto.co.uk/compact-lightweight-tripod-for-travel-photography
.. _`spirit level`: http://www.amazon.co.uk/gp/product/B005FRI4G0
.. _Astro: http://orderastro.com/welcome
.. _`Hugin's`: http://hugin.sourceforge.net/
.. _align_image_stack: http://wiki.panotools.org/Align_image_stack
.. _ffmpeg: https://www.ffmpeg.org/
.. _webm: http://www.webmproject.org/
.. _`Kevin MacLeod's`: http://incompetech.com/music/royalty-free/collections.php
.. _`OpenStreetMap edit videos`: https://vimeo.com/channels/osm
.. _audacity: http://audacityteam.org/

