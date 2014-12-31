London in Fives: The Making Of
==============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2014-11-25 08:55 Europe/London
   :Tags: blog, php, photography
   :Short: ldni5s

A few days ago I published a video called "London in Fives" on Vimeo. In this
article I explain how I made this, including the hardware and software that I
used.

But first, let's have a look at the video itself:

.. vimeo::
   :ID: 112413380
   :Title: London in Fives
   :Width: 640
   :Height: 360

There are 25 scenes in this video, each taking 5 seconds. Because the
frame rate is 25 fps, that means there are 125 frames per scene. All those
numbers are powers of 5, hence the title of the film: **London in Fives**.

The first and last scenes are merely the title and end credits, and are just
a series of images. The more interesting bits are the 23 scenes in between.

Hardware
--------

All these scenes are made from single frame shots from my Nikon D300 DSLR
camera. It has a feature that allows a picture to be taken every 5 seconds
automatically. For all scenes, except for the night time shot of Covent
Garden and the Ice Skating, that created the raw images.

For each scene, I usually took a few more shots than the 125 required,
usually up to a 150, to have a bit of a choice of where to start and end the
scene. In one case (the Regent's Park sunrise in the fog scene), I was happy
that I did! Due to a hard drive failure I fortunately managed to only lose
a few images, so that I still had just 125 left!

Of course it is important to keep the camera steady between all of the shots.
In most of the scenes I used a `GorillaPod`_, a three legged flexible
tripod where each leg can wrap around objects. In the later scenes, I used a
normal stand-up tripod, a `Manfrotto befree`_.

.. image:: /images/content/astro.png
   :align: right

The camera movements are all done in post production, except for the night
time shot of Covent Garden and the Ice Skating scenes. Instead of
using my camera's "take a photo every x seconds" feature, I relied on hardware
to take both a photo every 5 seconds, but also rotate the camera slightly on
top of its tripod. The time lapsing device that I used to rotate and instruct
the camera to take a photo every 5 seconds is an Astro_. This is a disk like
device that can rotate around one axis and instruct the camera through a cable
to take a photo at specific intervals over a certain period of time. I think
that for future time lapses I will not rotate more than 30° for a 125 scene
shoot as otherwise it goes a bit too fast. 

To make sure I had my camera perfectly horizontal on my camera, I used a
`spirit level`_ that sits on top of my flash socket.

Post Processing
---------------

After taking the photos, some post-processing was necessary. There are three
types of post-processing that I had to do, depending on how the photos were
shot.

For the two scenes created with the Astro, I really only had to rescale the
photos from the camera's native resolution to 1280x720.

For one other scene (Regent's Park sunrise in the fog), the GorillaPod was
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
area that all images shared.

.. image:: /images/content/londonin5s-align.png

After the image alignment for this particular scene, I could process it the
same as the other 20 scenes that were not taken with help from the Astro.

For the sequences not taken with the Astro, the resulting video still shows
camera movement. This is absolutely artificial, and is basically done by
cropping the right section out of each image. I varied the size and location
of the cut out sections for each image to emulate a moving, and zooming
camera. For one scene, the Oxford Circus crossing one, I also had to adjust
rotation as the horizon wasn't flush. The rotating and cropping was done
through fairly simple PHP scripts using the GD library.

.. image:: /images/content/londonin5s-camera-pan.png

A simple script that I used for this is looks like::

	<?php
	mkdir('tmp');
	mkdir('small');

	$fstart =  9800;
	$fend   =  9924;

	$xs = 1464;
	$ys = 1008;
	$ws = 1660;
	$hs =  934;

	$xe = 1208;
	$ye = 1008;
	$we = 1808;
	$he = 1017;

	for ($i = $fstart; $i <= $fend; $i++)
	{   
		$x = $xs + (($i-$fstart)/($fend-$fstart)) * ($xe - $xs);
		$y = $ys + (($i-$fstart)/($fend-$fstart)) * ($ye - $ys);
		$w = $ws + (($i-$fstart)/($fend-$fstart)) * ($we - $ws);
		$h = $hs + (($i-$fstart)/($fend-$fstart)) * ($he - $hs);
		
		$fn = sprintf( "tl3_%04d.jpg", $i );
		$cmd = sprintf( "convert -rotate 1.7 tl3_%04d.jpg tmp/tl3_%04d.jpg", $i, $i );
		`$cmd`;

		$img = imagecreatefromjpeg( "tmp/{$fn}" );
		$n = imagecreatetruecolor( 1280, 720 );
		imagecopyresampled( $n, $img, 0, 0, $x, $y, 1280, 720, $w, $h );
		imagejpeg( $n, "small/{$fn}", 100 );

		echo "Done with $i\n";
	}

This script loops over all the images in the sequence (``9800`` to ``9824``).
Given the ``x``, ``y``, ``width`` and ``height`` values for the beginning and
end images of the sequence, it calculates the intermediate coordinates of
where to crop from. Before cropping, the script uses convert_ to rotate each
image by ``1.7`` degrees to put the horizon horizontal. After the rotation is
performed, the call to imagecopyresampled_ cuts out the right part of the
original image and scales it down to the target frame size of 1280x720.

Creating Video from Images
--------------------------

After I post-processed all the image sequences, I used ffmpeg_ with a "magic
incantation" to create video scenes. I wanted to render to webm_ as that
seemed to be the best encoder. For a sample scene, the ffmpeg_ arguments
looked like::

	ffmpeg -y -framerate 25 \
		-start_number 5711 -i 12-south-bank/small/tl3_%04d.jpg \
		-vframes 125 -r 25 \
		-vcodec vp8 -b:v 50000k -vf scale=1280:720 -crf 8 \
		12.webm

Which basically means: use ``125`` frames starting from image number ``5711``
in directory ``12-south-bank/small/`` with file format ``tl3_%04d.jpg`` at
``25`` frames a second. The codec is ``vp8`` with a video bit rate of
``50mbit`` and a rescaled result of ``1280:720`` pixels. The ``-crf 8``
selects ultra-high quality for ``vp8``. The output file is ``12.webm``.

I wanted ultra high quality for each scene, as later on I would be
re-encoding all the scenes into the final video file, keeping as much image
definition as I could.

Music
-----

I have been a fan of `Kevin MacLeod's`_ Creative Commons licenced music for a
while, having used it for some of the newer `OpenStreetMap edit videos`_. In
this case, I did not want to release the video under a Creative Commons
license so I paid for one of his tracks: "Touching Moments Four - Melody". 

With audacity_ I added the 5 beeps at the start, and I also slightly stretched
the sound to cover the full two minutes that I needed it to last. I think the
speed was reduced by about 8%. This does make the sound a bit lower in pitch,
but I do not think it hurt its original composition.

Assembling
----------

With the video scenes and audio prepared, all I had to do is to stitch it
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
.. _convert: http://www.imagemagick.org/script/convert.php
.. _imagecopyresampled: http://docs.php.net/manual/en/function.imagecopyresampled.php
