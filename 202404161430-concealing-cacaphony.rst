Concealing Cacophony
====================

.. articleMetaData::
   :Where: London, UK
   :Date: 2024-04-16 14:30 Europe/London
   :Tags: blog, php, ai
   :Short: noise-reduction

Over the last few weeks I have been publishing a `series of videos
<https://www.youtube.com/watch?v=WjbKYHzoKM0&list=PLg9Kjjye-m1hW4z0J-546qaFpysjlo27x>`_
on writing PHP extensions.

I record these videos through `OBS <https://obsproject.com/>`_, and then slice
and dice them with `Kdenlive <https://kdenlive.org/en/>`_. This editing is
necessary to make up for my mistakes, shorten the time we wait for things to
compile, and to remove the noise of me hammering away on my keyboard.

Editing takes a lot of time, and I still wasn't always pleased with the result
as there was still a fair amount of noise while I am talking.

For the `PHP Internals News podcast <https://phpinternals.news>`_, I used a
set of noise cancellation filters, which worked wonders. But it turns out that
Kdenlive does not come with one built in.

I had a look around on the Internet, and learned that there is a `LADSPA
<https://en.wikipedia.org/wiki/LADSPA>`_ Noise Suppressor for Voice plugin.
LADSPA is an open API for audio filters and audio signal processing effects.
LADSPA plugins can be used with Kdenlive.

Some Linux distributions have a package for this LADSPA Noise Suppressor for
Voice, but my Debian distribution bookworm does not.

I found `instructions <https://kentwest.neocities.org/westk/librnnoise>`_ that
explain how to build the plugin from source. These instructions worked after
some tweaks. I ended up creating the following script::

	#!/bin/bash

	sudo apt install cmake ninja-build pkg-config libfreetype-dev libx11-dev libxrandr-dev libxcursor-dev
	git clone https://github.com/werman/noise-suppression-for-voice /tmp/noise
	cd /tmp/noise
	cmake -Bbuild-x64 -H. -GNinja -DCMAKE_BUILD_TYPE=Release
	sudo ninja -C build-x64 install

After running this script, and restarting Kdenlive, I found the installed
plugin when I searched for it.

With the plugin loaded, I now have much clearer sound, and I also don't have
to edit the sections where I am typing, as the plugin automatically handles
this.

I will still have to edit out my mistakes.

I then also had a look at how it worked. It turns out that `this plugin
<https://github.com/werman/noise-suppression-for-voice>`_ uses neural networks
to cancel the noise.

In the background, it uses the `RNNoise <https://github.com/xiph/rnnoise>`_
library which implements an algorithm by Jean-Marc Valin, as outlined in this
`paper <https://arxiv.org/pdf/1709.08243.pdf>`_. There is an easier to read
version of how the `algorithm works <https://jmvalin.ca/demo/rnnoise/>`_ on
his website.

The `data to train the model <https://media.xiph.org/rnnoise/README>`_ is also
freely available, and uses resources from the `OpenSLR
<http://www.openslr.org/>`_ project. Noise data is also available there. From
what I can tell, all this data was contributed under reasonable conditions,
and not scraped from the internet without consent. That is important to me.

Hopefully, from the third video in the series, you will find the sound quality
much better.
