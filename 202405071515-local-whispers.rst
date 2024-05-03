Local Whispers
==============

..
articleMetaData::
   :Where: London, UK
   :Date: 2024-05-07 15:15 Europe/London
   :Tags: blog, php, ai
   :Short: local-whispers

For most of the `videos
<https://www.youtube.com/@DerickRethansXdebug/videos>`_ that I make, I also
like to have subtitles, because sometimes it's easier to just read along.

I used to make these subtitles with an online service called Otter.io, but
they stopped allowing uploading of video files.

And then I found `Whisper <https://openai.com/index/whisper>`_, which allows
me to upload audio files to create subtitles. Whisper is an API from
`OpenAI <https://openai.com/>`_, mainly known for `ChatGPT
<https://openai.com/chatgpt>`_.

I didn't like having to upload everything to them either, as that means that
they could train their model with my original video audio.

Whisper never really worked that well, because it broke up the sentences in
weird places, and I had to make lots of edits. It look a long time to make
subtitles.

I recently found out that it's actually possible to run Whisper locally, with
an `open source project on GitHub <https://github.com/openai/whisper>`_.
I started looking into this to see whether I could use
this to create subtitles instead.

The first thing that their `documentation
<https://github.com/openai/whisper?tab=readme-ov-file#setup>`_ tells you to do
is to run: ``pip install openai-whisper``.

But I am on a Debian machine, and here Python is installed through
distribution packages, and I don't really want to mess that up. ``apt-get``
actually suggests to create a virtual environment for Python.

In a virtual environment, you can install packages without affecting your
system setup. Once you've made this virtual environment, there's actually
Python binaries symlinked in there, that you can then use for installing
things.

You create the virtual environment with::

	python3 -m venv `pwd`/whisper-local
	cd whisper-local

In the ``bin`` directory you then have ``python`` and ``pip``.
That's the one you then use for installing packages.

Now let me run pip again, with the same options as before to install Whisper::

	bin/pip install -U openai-whisper

It takes quite some time to download. Once it is done, there is a new
``whisper`` binary in our ``bin`` directory.

You also need to install ``fmpeg``::

	sudo apt-get install ffmpeg

Now we can run Whisper on a video I had made earlier::

	./bin/whisper ~/media/movie/xdebug33-from-exception.webm

The first time I ran this, I had some errors.

My video card does not have enough memory (2GB only). I don't actually have a
very good video card at all, and was better off disabling it, by instructing
"Torch" that I do not have one::

	export CUDA_VISIBLE_DEVICES=""

And then run Whisper again::

	./bin/whisper ~/media/movie/xdebug33-from-exception.webm

It first detects the language, which you can pre-empt by using ``--language
English``.

While it runs, it starts showing information in the console. I quickly noticed
it was misspelling lots of things, such as my name Derick as Derek, and Xdebug
as XDbook.

I also noticed that it starts breaking up sentences in a odd way after a
while. Just like what the online version was doing.

I did not get a good result this first time.

It did create a JSON file, ``xdebug33-from-exception.json``, but it is all in
one line.

I reformatted it by installing the ``yajl-tools`` package with ``apt-get``,
and flowing the data through ``json_reformat``::

	sudo apt-get install yajl-tools
	cat xdebug33-from-exception.json | json_reformat >xdebug33-from-exception_reformat.json

The reformatted file still has our full text in a line, but then a
``segments`` section follows, which looks like::

    "segments": [
        {
            "id": 0,
            "seek": 0,
            "start": 3.6400000000000006,
            "end": 11.8,
            "text": " Hi, I'm Derick. For most of the videos that I make, I also like to have subtitles, because",
            "tokens": [
				50363, 15902, 11, 314, 1101, 9626, 624, 13, 1114, 749, 286,
				262, 5861, 326, 314, 787, 11, 314, 635, 588, 284, 423, 44344,
				11, 780, 50960
            ],
            "temperature": 0.0,
            "avg_logprob": -0.20771383965152435,
            "compression_ratio": 1.5128205128205128,
            "no_speech_prob": 0.31353551149368286,
                },

Each segment has an ``id``, a ``start`` and ``end`` time (in seconds), the
``text`` for that segment, and a bunch of auxiliary information.

But it is still sort of a sentence at a time, which isn't really what we want.
Additionally, I really don't want it to say XDbook and misspell my name.

For both of those there are actually options that we can use.

To make things easier for us later, we're turning on word timestamps.
That allows us to by word see where the time index actually was. You do that
with the ``--word_timestamps True`` option.

To provide a hint on what the video is about, and to get better words, we can
use the initial prompt. The option that I used for this video was:
``--initial_prompt="This video by Derick introduces a new Xdebug feature,
regarding exceptions"``.

To make things more accurate or less accurate, there's also an option that you
can specify which is which model to use. Normally the standard one medium is
fine, but in order to speed up generation, you can use ``--model tiny``. This
will give less accurate results.

If the model has not been downloaded the before, Whisper will automatically do
this.

It is also possible to specify the language, which makes things go faster if
it's English only as well: ``--language English``. It supports a bunch of
languages.

The full command that I used is::

	CUDA_VISIBLE_DEVICES="" \
		./bin/whisper ~/media/movie/xdebug33-from-exception.webm \
		--word_timestamps True \
		--initial_prompt="This video by Derick introduces a new Xdebug feature, regarding exceptions" \
		--language English

In the output file, we now see another elements in each sentence section::

            "words": [
                {
                    "word": " Hi,",
                    "start": 3.6400000000000006,
                    "end": 4.16,
                    "probability": 0.6476245522499084
                },
                {
                    "word": " I'm",
                    "start": 4.28,
                    "end": 4.44,
                    "probability": 0.9475358724594116
                },
                {
                    "word": " Derick.",
                    "start": 4.44,
                    "end": 4.8,
                    "probability": 0.12672505341470242

For each word, it has the ``start`` and ``end``, as well as the probability of
it being correct. You see that for ``Derick`` it was only 13% certain.

With this information you can do some analysis to create an actual subtitle
script out of this. For that I have written a PHP `script
<https://gist.github.com/derickr/f30bc04c394e89573071171459f6a9d4>`_.


It loops over all the segments, and for each of the segments over all the
words. If the difference in time between the end of a word and the start of a
new word is more than a quarter of a second, it emits a new section of
subtitles.

Similarly if a sentence is longer than 60 characters it also breaks it up into
an extra line. This keeps all the subtitles reasonably well sorted.

The ``emit`` function formats it like how the SRT files are supposed to be.

With this script, I now create a new SRT file, overwriting the one that
Whisper had created::

	php whisper-to-srt.php xdebug33-from-exception.json > xdebug33-from-exception.rst

The output looks like::

	0
	00:00:03,640 --> 00:00:04,799
	Hi, I'm Derick.

	1
	00:00:05,919 --> 00:00:10,980
	For most of the videos that I make, I also like to have subtitles,

	2
	00:00:11,640 --> 00:00:15,779
	because sometimes it's easier to just read along for various
	different reasons.

These subtitles I can now add when uploading a video to YouTube. I might also
create a similar script to generate output that can be used as the base for an
textual article. Not everybody likes learning from watching videos.

I would also prefer not to use a model that has been trained with questionable
sources. I will be investigating if I can use `Mozilla's Common Voice
<https://commonvoice.mozilla.org>`_ project's data instead.
