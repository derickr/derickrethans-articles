PHP Internals News: Episode 78: Moving the PHP Documentation to GIT
===================================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-03-11 09:06 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin078
   :Enclosure: media/pin-078.mp3

In this episode of "PHP Internals News" I chat with Andreas Heigl (`Twitter
<https://twitter.com/heiglandreas>`_, `GitHub
<https://github.com/heiglandreas>`_, `Mastodon
<https://phpc.social/@heiglandreas>`_, `Website <https://andreas.heigl.org>`_)
to follow up with his project to move the PHP Documentation project from SVN to
GIT, which has now completed.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-078.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:15
	Hi, I'm Derick. Welcome to PHP internals news, the podcast dedicated to explaining the latest developments in the PHP language. This is Episode 78. In this episode, I'm talking with Andreas Heigl about moving the PHP documentation to GIT. Andreas, would you please introduce yourself?

Andreas Heigl  0:35
	Hi yeah I'm Andreas, I'm working in Germany at a company doing PHP software development. I'm doing a lot of stuff in between, as well. And one of the things that I got annoyed, was always having to go through hilarious ways of contributing to the PHP documentation, every time I found an issue with that. So at one point in time, I thought why not move that to Git and, well, here we are.

Derick Rethans  1:07
	Here we are five years later, right? Because we already spoke about moving the documentation to GIT back in 2019 and Episode 28. But now it has finally happened, so I thought it'd be nice to catch up and see what actually has changed and how we ended up getting here. Where would you want to start. What was the hardest thing to sort out in the end?

Andreas Heigl  1:27
	Well the hardest thing in the end to sort out was people, as probably always in software development. The technical oddities and the technical bits and pieces were rather fast to solve. What really was taking a long time was, well for one thing, actually, consistently working on that. And on the other hand, chasing down people to actually get stuff done. Because one of the major things here was not the technical side but getting the bits and pieces of information together to get access to the different servers, to the infrastructure of the PHP ecosystem, and chasing down the people that want to help you is one thing, and then chasing down the people that actually can help you is a completely different one. That was for me the most challenging bit, getting actually, to know who can do what and getting, yeah in the end, getting access to the actual machines, the whole ecosystem is running on that was really heavy.

Derick Rethans  2:34
	The System Administration of PHP.net systems is very fragmented. There's some people having access to some machines and some other people having access to other machines and yea it sometimes takes some time to track down where are all these bits actually run.

Andreas Heigl  2:51
	One thing is getting tracking down, where the bits run, the other one is, there is an excellent documentation in the wiki, the PHP wiki, which in some parts is kind of outdated. The other thing is, if you don't actually address the different people themselves personally, it is like screaming into the void so you can can send an email to 15 different people that have access to a box, like someone needs to do this and this and this. And everyone kind of seems to think, yeah, someone else can do that. I just don't have the time at this point in time Things get delayed, so you're waiting for an answer for a week; you do some other stuff, so two weeks go into the lab four weeks go into the land, and suddenly two months have passed. You didn't receive an answer and oh wait a minute, there was this project with moving the documentation to GIT so perhaps I should have a look at that again.

Derick Rethans  3:44
	So what has changed for people that want to contribute to the PHP documentation. Can you explain a little bit the difference between before and after?

Andreas Heigl  3:52
	Before the documentation was moved everything was an SVN and there was generally there were two kinds of people. There were regular contributors that had an SVN account. They had the documentation on their machine. They could actually modify the documentation and just do an SVN commit, and everything was working smoothly, so that documentation was then actually built. We ran into some issues with that as well but that's a different story. Now, it is as everything is has moved to GIT, the sources are now on git.php.net. Before I come to that, what was it for people that did not have an SVN account. There was an awesome piece of technology on a web server called edit.php.net, which was a graphical user interface to the PHP documentation and to the translation so everyone could more or less log in there, with an anonymous account for example, modify the documentation, and create yeah well a kind of a pull request Create a patch that was then reviewed and could then be merged by people with SVN access. That awesome piece of technology was an awesome piece of technology when it was created some 12 years ago or something like that. It has not changed much in between. So it was still kind of working, not always, it was offline for some time. And the people that had access to the SVN were not really that responsive at all times so it could take some while for your patch to actually be merged in. So how is it now, now all the sources are on git.php.net, and there is a mirror for all translations, on https://github.com/php/, for example, doc-en for the English documentation. You have the repository, and you can create a pull request there. So you just move there, edit the file you want to move, create a pull request. And then the pull request will be merged into the main sources, again, by people with merge access. As this is a pull request that usually happens, a bit faster than before. We are still not at that point where we can create a GitHub action for that so that perhaps that can be completely automated after some technical things are resolved, like yes that does build, and we don't have any issues, technical issues with that. We could just build that automatically and merge that automatically into to the GIT sources. Those are possibilities that we now have, and from that point of view, now the contribution is much easier as we are using the technology that every one of us knows already.

Derick Rethans  6:34
	The original translation to work with sub modules in SVN, now how is  with the GIT approach?

Andreas Heigl  6:40
	It actually didn't work with sub modules in SVN, it worked with, actually, one folder for the documentation, and then sub folders for the different translations. So there was a base folder PHP Doc, and within that base folder PHP Doc, there was a folder EN for the English base, kind of, and then DE for the German translation or PT_BR for the Brazilian Portuguese translation, or IT for the Italian translation. Or some other rather outdated translations that we actually didn't move over, so we only moved everything that was touched within the last two years, which brought some interesting bugs. Of course we actually moved something that was touched within the last two years, but that is not considered an active translation, and that caused some havoc.

Derick Rethans  7:30
	Which translation was that?

Andreas Heigl  7:31
	That was the Italian translation. So actually, there is no official Italian translation of the PHP documentation. But there is an Italian translation that is actually worked on, and hopefully at one point in time, we can actually promote that to a valid active translation, so that Italian people can actually see some Italian documentation for them. In GIT on the other hand, we have different repos, different repositories for the different languages. It is now, not possible to just check out phpdoc, and have every translation. Now you actually have to say, I want to check out the English documentation, and I want to check out the Italian documentation, and I want to check out the Japanese documentation, because I want to work on each of them. That has some disadvantages, especially for people that are working on multiple documentations or multiple translations. On the other hand, that also has advantages because you don't need to actually download all the translations that you're not at all interested in.

Derick Rethans  8:35
	But that shouldn't be something new because I'm pretty sure that I've never checked out all the translations, even with SVN.

Andreas Heigl  8:41
	SVN you could decide to only check out certain translations but if you check out the PHP doc base folder you would get all the documentation. Yes, there were some sub modules that actually did exactly that, like, if you check out the sub module for Italian for example you would get the English base and the Italian translation. That was all.

Derick Rethans  9:04
	I remember we employed some SVN magic to do that kind of things, but I forgot the most about that because it's so long ago,

Andreas Heigl  9:11
	Not really to worry about.

Derick Rethans  9:14
	No.

Andreas Heigl  9:14
	We're thinking about doing this similar thing for GIT now for the by using GIT sub modules, but we have not yet implemented that, because there were other pressing issues like getting the ref check documentation up and running, where you can actually see which files are outdated which files need to be translated and stuff like that. So that was more more pressing some other people have done, also work on that need to check what the current status is, to be honest, because I didn't check that. That was going on very strongly in January, after we moved between Christmas and New Year's Eve. After we equalized some glitches that happened during the whole process, because of Yeah, sometimes also processes that were nowhere really documented, and I got just got plain wrong. So then I had to invest some time and fix all that, but luckily that was during my holiday time so I had a lot of time for that so that was not an issue.

Derick Rethans  10:15
	So working on the documentation during your holiday's huh?

Andreas Heigl  10:18
	Yes, definitely.

Derick Rethans  10:20
	That makes it different from travelling to visit family, because that is of course not something we could do to here.

Andreas Heigl  10:25
	Exactly, though. Luckily, having a family at home was quite okay it was a nice change to actually be able to get away sometime, from seeing the same people over and over again.

Derick Rethans  10:38
	Now the documentation has moved from SVN to GIT, and everybody can now finally forget all their SVN commands. But the documentation itself is still written in Docbook. XML as far as I understand. Are there any discussions going on about changing that to something, perhaps a bit more modern?

Andreas Heigl  10:58
	Yes, there were a lot of discussions going on during the whole phase. I deliberately try to calm that down to not to too many things at once. The thing that I wanted to get going was moving the documentation from SVN to GIT. Just change the underlying source code repository, and not change anything else in the process, because that was already hard enough. Now that we have moved, it is easily possible to actually modify or move the documentation to some other toolage, whether that is markdown or ASCII doc or whatever. I don't care to be honest because that's someone else's job to do. In my opinion, Docbook is actually a pretty good format for that. Yes it is rather verbose for sure, but it allows you to create a lot of different documentations, because the HTML is not the only documentation that is created, there is also the possibility to create a PDF documentation or a CHM for Windows documentation, and stuff like that. I'm not 100% sure how that would work with a rather, with something like like markdown or ASCII doc or something like that.

Derick Rethans  12:12
	There's different strengths in different formats. Markdown for example doesn't really allow you to link in between documents, so that's probably not very handy but there's like Pandoc, which is stuff that the Python project uses. It's all pretty much designed around restructured text and linking in between them and stuff like that, so I guess that could be a way forward. It is certainly a lot easier to use than Docbook XML, but of course Docbook XML was created for this kind of rich marker without laying things out kind of situations right.

Andreas Heigl  12:44
	Yeah. The nice thing is actually that in with Docbook. What perhaps that is possible with other tools as well but in Docbook for example, you have one single file where all the links are located in, and every translation just links to this one file, so if a link changes for whatever reason, you can just modify that in one place and don't have to go through all the documentation. And, of course, leave half of the links unchanged, and broken, whatever. So there is a lot of stuff actually that is pretty cool. But as I said that's now up for discussion. If there is someone that actually wants to tackle that and move the documentation format to something else that is a different story. Go ahead, propose that to the internals mailing list, or to the documentation team, we'll see how it goes from there. The documentation itself, the source code itself, is hosted on a platform that we all understand, at least by now a bit better than SVN.

Derick Rethans  13:42
	Even though it started out on CVS, just like PHP that's.

Andreas Heigl  13:46
	Yes.

Derick Rethans  13:47
	A long long time ago.

Andreas Heigl  13:49
	I found the remaining stuff from the transfer from S from CVS to SVN, yes,

Derick Rethans  13:55
	I am sure there's still some commits lurking somewhere for that.

Andreas Heigl  13:59
	Oh yes, especially in the in the ref check, there is a lot of commented out code with yeah CV, in CVS we did it this way now we have to modify that for SVN. Yes, I'm pretty sure now we have some commits in there that modify the SVN stuff for making it usable with GIT.

Derick Rethans  14:16
	Andreas, do you have anything else to add?

Andreas Heigl  14:19
	In all it was an awesome experience for me. I got to know a lot of people, a lot of awesome people. That was really really insightful, and I'm really happy that I had the chance to do that, and the trust of the community was really amazing. If anyone wants to get into that stuff, kind of stuff, pick yourself something that the community needs, and go for it, and don't let yourself be derailed by unresponsive mailing lists, or just things not happening. It's not because people think you are stupid, or the task is stupid, it's just because everything is done by volunteers that just pays its price.

Derick Rethans  15:00
	It certainly does. Thank you, Andreas for taking the time today to talk to me about moving to PHP documentation to GIT.

Andreas Heigl  15:07
	Thank you very much. It was a pleasure to be here.

Derick Rethans  15:13
	Thank you for listening to this instalment of PHP internals news, a podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next time.


Show Notes
----------

- Episode #28: `Moving PHP Documentation to GIT <https://phpinternals.news/28>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
