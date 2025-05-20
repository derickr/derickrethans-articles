Missing Characters
==================

.. articleMetaData::
   :Where: London, UK
   :Date: 2025-03-11 14:50 Europe/London
   :Tags: blog, php, xdebug
   :Short: missing-chars

I release a new version of `Xdebug <https://xdebug.org>`_ on Sunday,
which fixes a few bugs. One of them is titled `emoji character become diamond
question marks <https://bugs.xdebug.org/2319>`_. This bug turned out to
be the same as `var_dump does not output some Cyrillic characters
<https://bugs.xdebug.org/2323>`_, which was originally reported a few days
earlier but hadn't come with a decent enough reproducible case.

At first I dismissed this, as it's not unlikely that people get their
character sets wrong, or mixed up.

But when I tested it, the following script really did not show the right
result::

	<?php
	$str = 'hello 👍';
	var_dump($str);
	?>

Instead of the expected::

	<pre class='xdebug-var-dump' dir='ltr'>
	<small>Standard input code:3:</small><small>string</small>
	<font color='#cc0000'>'hello 👍'</font> <i>(length=10)</i>
	</pre>

It showed::

	<pre class='xdebug-var-dump' dir='ltr'>
	<small>Standard input code:3:</small><small>string</small>
	<font color='#cc0000'>'hello ���'</font> <i>(length=10)</i>
	</pre>

The four bytes that should have made up the 👍 had turned into three.

Xdebug uses a function, ``xdebug_xmlize``, to escape XML and XHTML-special
characters such as ``"``, ``&``, and ``<`` when it outputs strings of data.

Its algorithm first calculates how much memory the resulting string would use
by looping over the source characters, and adding the lengths of the escaped
characters together. It uses a `256-entry table
<https://github.com/xdebug/xdebug/blob/3.4.2/src/lib/var.c#L1028>`_ for this.

The first row shows that byte 0's escaped length will be ``4`` (for ``&#0;``) and
the LF character's escaped length will be ``5`` (for ``&#10;``).

The replacement strings are recorded in the `table that follows
<https://github.com/xdebug/xdebug/blob/3.4.2/src/lib/var.c#L1047>`_. It only
has place for ``64`` elements, as none of the bytes above byte-64 need to be
escaped. You can see that because the ``xml_encode_count`` table only has
entries containing ``1`` after the fourth 16-element row.

Then in a second iteration it loops over all the source characters again to
construct the resulting output.

In this iteration, it checks if the `destination length is 1
<https://github.com/xdebug/xdebug/blob/3.4.2/src/lib/var.c#L1083>`_, in which
case it just copies the character over. If the destination length is not
``1``, then it adds the number of characters that correspond to the
`destination character's length
<https://github.com/xdebug/xdebug/blob/3.4.2/src/lib/var.c#L1086>`_. 

The bug here was that the table for ``xml_encode_count``, although it was
defined as having 256 entries, only had 240 entries. I had missed to add the
16th line, so instead there were only 15 lines of 16 elements.

And in C, that means that these missing elements were all set to ``0``. This meant
that if there was a character in the source string where the byte value was
larger or equal to hexadecimal ``0xF0`` (decimal: 240), the algorithm thought
the replacement length of these characters would be ``0``. This then
resulted in these characters to just be ignored, and not copied over into the
destination string.

For the 👍 character (hex: ``0xF0 0x9F 0x91 0x8D``) that meant that its first
byte (``0xF0``) was not copied into the destination string. And that meant a
broken UTF-8 character. Oops! 💩

In Xdebug 3.4.2 this is now fixed, as I have `added the 16th line to the table
<https://github.com/xdebug/xdebug/commit/12ecc846a4df079abfbe2497a3acf4bfabebf8b8>`_,
with 16 more elements containing ``1``.

What I did find curious that it took nearly five years for something to report
this issue, and with that, two in the same week!
