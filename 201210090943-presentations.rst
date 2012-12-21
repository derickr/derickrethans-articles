Presentations
=============

.. articleMetaData::
   :Where: London, UK
   :Date: 2012-10-09 09:43 Europe/London
   :Tags: blog, php
   :Short: pres2

In the past 10 years I have given plenty of presentations_. Since I started,
I've always used the same presentation format. Because I am getting more and
more questions about this, here is some information on what I use.

The presentation tool that I use is called "pres2". Or rather, it is simply
the name derived from its original directory name. It is an evolution of
Rasmus' original "pres" tool that he used to give PHP related presentations
with. It has been used in the past by many other people as well, but as
far as I know, it is now only used by Rasmus_ and me.

The "pres2" software is also what powers http://talks.php.net. It uses XML
files stored in PHP's GIT at http://git.php.net/?p=presentations.git;a=tree.
The software itself is at http://git.php.net/?p=web/pres2.git;a=tree. I have
however written my own renderer that uses Zeta Components' Template_ component
in order to make it easier for me to do something a little bit more exciting
with the content (such as some effects). Sadly, this makes the slides on
http://talks.php.net not render properly anymore. My version of the renderer is
available through the "dericks-renderer" branch of the pres2 GIT repository:
http://git.php.net/?p=web/pres2.git;a=tree;h=refs/heads/dericks-renderer;hb=refs/heads/dericks-renderer

You can obtain the software and presentations by running::

	git clone https://git.php.net/repository/web/pres2
	cd pres2
	git checkout dericks-renderer
	git clone https://git.php.net/repository/presentations

I have used this tool for all my 259 presentations, so I have invested a large
amount of time into creating all the XML files. I mostly like it because:

 - It is simple
 - I don't need to click around to make presentations
 - I can version the presentations and slides through GIT/SVN
 - I can run PHP examples directly in the slides

And because of this, there is very little reason for me to switch to anything
else.

.. _Rasmus: http://toys.lerdorf.com/
.. _presentations: /talks.html
.. _Template: https://github.com/zetacomponents/Template
