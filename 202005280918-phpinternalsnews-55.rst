PHP Internals News: Episode 55: Dealing with Bugs
=================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-05-28 09:18 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin055
   :Enclosure: media/pin-055.mp3

In this episode of "PHP Internals News" I chat with Ignace Nyamagana Butera
(`Twitter <https://twitter.com/nyamsprod>`_, `GitHub
<https://github.com/nyamsprod>`_, `Blog <https://nyamsprod.com>`_) about how
the PHP project handles bugs and bug reports.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-055.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:16
	Hi, I'm Derick. And this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 55. Today I'm talking with Ignace Nyamagana Butera after he'd asked me on Twitter, how PHP deals with bugs. A few episodes ago, I did a Q&A session about the RFC process. And this time again, we'll have Ignace Nyamagana Butera asking the questions. Would you please introduce yourself?

Ignace Nyamagana Butera  0:46
	Hello, everyone. Hello, Derick. My name is Ignace Nyamagana Butera, but you can call me Nyamsprod. I've been a PHP developer for around 15 years now. Currently, I'm working as a software developer, and technical lead in the internet content provider agency. When I have free time, I'm doing some open source, I have a couple of projects that you may have heard of, like, league CSV and league URI. I created them and I am currently maintaining them.

Derick Rethans  1:23
	Yeah, as I said, it is not me asking the questions as you this time. So I think we should jump straight in actually.

Ignace Nyamagana Butera  1:30
	So my first question will be somehow really simple, because we are talking about bugs. And I was wondering if we had some statistics about bugs in PHP.

Derick Rethans  1:44
	Though there are some statistics. I mean, it's not really easy to get that information out of our bug system. But just having had a look, it's about on average, maybe one bug a day gets reported at the moment or is nearly 80,000 bugs in the bug system of course, not all of these are closed, some of them are open, but the majority of them are closed.

Ignace Nyamagana Butera  2:07
	Do bugs from the EOL PHP still being taken into account or we just say: okay, these bugs for instance, are for PHP five, will no longer look at them.

Derick Rethans  2:18
	If it's a bug, unless it's a security bug fix, we won't look at them for unsupported PHP versions. So at the moment, PHP, seven three, and seven four are still supported. So those bugs will of course look at, if it's a security bug, we only will go back to PHP seven two. If it's reported to any older version and seven two for example, seven one or seven zero, or even PHP four or five, which does happen occasionally, we'll tell them to upgrade first because we won't spend time doing that.

Ignace Nyamagana Butera  2:47
	Because I manage and maintain open source project. I know that PHP as a language is used everywhere and you can have multiple reports. First thing first, what is a bug? Because there are multiple definition of it.

Derick Rethans  3:03
	And I'm sure if you asked 12 people, you get 13 definitions. I think it is unexpected behavior of something that is documented. So if something is documented do this, and it does something else, or it does something really wrong like crash your program, then that will be a bug.

Ignace Nyamagana Butera  3:21
	What is the source of truth? Is it the PHP documentation? Is it the PHP specification language, what is the source of truth? Nothing. Okay. This is expected behavior because it is documented, or how does it work?

Derick Rethans  3:38
	For most of the syntax, it's what the source does. And of course, you always find edge case. And I don't have a good example right now. For anything that the syntax, I mean, documentation and behavior should absolutely always work the same. If it doesn't, it's likely going to be a bug in the documentation. If you for example, look at other functionality like in an extension, there is almost as likely that the documentation is sometimes wrong than it is that the code's behavior is wrong. In that case, we need to have a good look at what what the expected behavior should have been. Now, with all the new features that have been put in, since we have the RFC process, pretty much anything that the RFC describes how it should work, is how the feature should work. And if it doesn't, that pretty much means there's a bug. Having said that, not everybody writes on all the expected behavior for all the functionality that an RFC has been put up for. And in those cases, you just need to see what makes the most sense whether it's about core feature.

Ignace Nyamagana Butera  4:40
	What is the best way to report a bug? Okay, you have to go to bugs.php.net, I suppose. Yes. But apart from that, what is the best way to report a bug?

Derick Rethans  4:51
	As you said, PHP is issue tracker is bugs.php.net. It tells you to fill in your problem, your expected behavior and what you actually get out, what is always really important for people to be able to fix an issue and to find out whether there is an issue to begin with, because that's not always the case either of course, is always to have a short reproducible script that reproduces your problem. And by short, that means it the short you can get it. 10 lines at most for most syntax features who probably do the job. In some cases, if it's a bug for a database related system, then of course, there's going to be some database setup necessary for it. But if it's just syntax, then a short script that reproduces the problem that shows what goes wrong, is really important. And of course, it's also important to say what it did, and what you expected it to do. Also, don't lie about your PHP version, because in some cases, people try to report a bug with a higher PHP version than they're actually using, which is kind of frustrating at times.

Ignace Nyamagana Butera  5:52
	I guess that yeah, if we report something that didn't work in PHP five, but it was fixed in PHP 7.2 or PHP 7.3 everybody loses a little bit of time.

Derick Rethans  6:02
	And in some cases people find a bug report for, say, PHP 7.4.1. Right, and we're currently at 7.4.6. We will always ask them first to upgrade if they can, because upgrading PHP should take a lot less time than trying to reproduce and fix a problem that has already been fixed.

Ignace Nyamagana Butera  6:20
	And what is the strategy between the release of each version of PHP and the bug fix? Does PHP wait for all the bug fixes to be done and then a release is made. Or if for instance, I report a bug like today before a release is scheduled, then this bug will be skipped from the next release and will be tackled after

Derick Rethans  6:46
	Every minor version of PHP, be at seven two, seven three, or seven four a moment, has a release every four weeks. Two weeks and two days before a release gets made, we make our release candidates. Everything that has made it in the release candidate will make it into the release. If in between the release candidate gets created and the final release, if bugs get fixed, unless they are really critical, they will make it into that release. But we'll have to wait until the next cycle. So we don't necessarily wait for all the bugs to be fixed before we make a release. Now, there is an exception here, and that is for security bugs. If you find security bugs, they don't end up in a normal PHP seven four branch. They get committed to a security repository that very few people have access to. And these security bug fixes. They get merged into the release branches two days before the release comes out. They don't end up in a release candidate builds because we don't want people 16 days to be able to exploit security bugs if they are remote exploitable, for example.

Ignace Nyamagana Butera  7:53
	And can security bugs, or critical bugs push a release?

Derick Rethans  7:59
	Technically, yes. If somebody ends up finding, like a remote exploitable bug in PHP, then there will be an emergency release for them. But I can't remember the last time we had to do that.

Ignace Nyamagana Butera  8:10
	I remember, like one or two years ago, there was a bug that was going from the bugtrack to the internal mailing list and coming back again to the bugtrack, because there was some kind of indecision to know if it is a bug, or if it should be a feature. How is this possible?

Derick Rethans  8:32
	We don't really have a set method for doing this. But our bug tracker isn't the most advanced system in the world. And sometimes it just makes sense to trash out a discussion over email on our PHP internals mailing lists, or sometimes these discussions happen on other chat channels as well I'm sure, just to go through to see what's the case. And sometimes if it is hard to take a decision while there's a bug, then it is always a good idea that more PHP core developers have a look at it and see what's going on there. So sometimes it makes it easier if that's discussed on the mailing list, then in the bug tracker.

Ignace Nyamagana Butera  9:04
	Is it possible that for instance, someone submit an RFC. And then during the course of discussion of this RFC, it becomes clear that this is not an RFC, but more of a bug fix.

Derick Rethans  9:16
	I don't think I can think of an example here actually.

Ignace Nyamagana Butera  9:19
	I remember one example.

Derick Rethans  9:21
	Okay.

Ignace Nyamagana Butera  9:23
	Because I think it was yeah two years ago about the behavior of the CSV escape character. And I remember at some point, it was suggested to be an RFC. And because of the amount of background compatibility breaks, it was better to treat it like a bug. But I remember when between the bug tracker and the note sufficient there was a whole discussion to exactly being able to say: Okay, this is a bug. And this is an RFC and it was really not, it was a call at the end saying, okay, we will treat it like an RFC, and we will change the way the escape corrector works today. But it won't be as impacting as if it was an RFC that introduced a completely new behavior

Derick Rethans  10:12
	CSV is a very difficult format, because everybody slightly implements a standard in a different way. And the way how it originally got implemented in PHP for reading CSV files was done in a very different way than for example, what Microsoft products would create. I mean, it has to do with escaping, if I remember correctly. And I mean, what do you decide, right? I mean, since then Microsoft have made a specification for this. And of course, what we then want to do in PHP is to make sure that we support a specification, but by doing so, we will then break previous behavior, and that is always a really difficult decision to do, right. If it is very clear that it is a bug, then we don't mind changing PHP, even though that could technically break people's code. But if it's unsure or whether it's based on a subjective decision, then that makes it a lot harder to write because we can't definitively say that, yeah, we have a bug here. But if we look at other codebase out there, so many people rely on this. So is the old behavior bug, or is it a feature in PHP? I mean, these things, you have to take one by one, and it's very hard to decide on what is what is a feature, and what is the bug in this case.

Ignace Nyamagana Butera  11:22
	I think another subject that comes with bugs is people should be able to fix them. But I suppose that every one of us has a work and who can fix those bugs?

Derick Rethans  11:33
	Technically, everybody who has time and know C code could fix a bug. PHP is an open source projects. Our repositories are available on GitHub, or on git.php.net, which is our source of truth, although most people submitted bug fixes against the GitHub repository because it makes it easier to review them and comment on pull requests, for example. But it's open for everybody. It's the same thing about triaging bugs. Trying to find out if the bugs that are actually reported are actual bugs and the bugs.php.net website has in the top right hand corner, it has a random link. And if you click that you get a random bug that hasn't been resolved yet. If somebody, if any of the listeners, or maybe you, are interested in looking at these bugs or wanting to attempt to fix them, click random and see what happens. Maybe you get something interesting, maybe because something really complicated, but in any case, it's possible for everybody to fix a bug. They will get reviewed. For a good enough bug fix it will get merged.

Ignace Nyamagana Butera  12:31
	People are usually thinking when they think about open source nowadays they think about semver and people may think that if they look at the versioning of PHP, then they have an idea of it is a patch release, it is a bug release, it is a feature release. How is this related to bugs and how is it versioning of PHP working?

Derick Rethans  12:53
	PHP's versions number consists out of three numbers. At the moment, we are the latest version is 7.4.6. The six is your bug fix release. In bug fix releases, there will not be any new functionality. Unless there are very minor, small contained parts in extensions. We tend not to want to have these. And unless you can make a good case for it, it's unlikely to happen. But it isn't unheard of. An example I think I can remember is that open SSL, added a bunch of new API's in there, and other technically new function functions in PHP, they sort of had to be supported, because as part of making sure that you could run the latest version of open SSL or something like that, but that being an exception there. Now, the middle number, traditionally, in semver, is there for features, right, you've bump the middle number, the middle digit, if you have new features, and that is the same in PHP. What we don't really have is a major number that indicates that we are going to break things. The major number in PHP is mostly a marketing number. So at the moment, we have PHP seven four out there. We don't have PHP eight zero next. But that is pretty much a PHP seven five, but with additional functionality that we find important enough to bump the major version from seven to eight for. Having said that, we do have a rule that we don't remove functionality, unless we bump the major number. For example, from five to seven, or from seven to eight. So there will be in the course of time, we might deprecate functionality, we don't tend to remove that until we bump the major number. And you also see that if the major number gets increased, that there is potentially more effort in removing or deprecating more functionality that would otherwise do say for example, it changed from 7.3.0 to 7.4.0. But it doesn't mean that we don't bump major numbers so that we can break all the things for example. So I think the PHP protect tries to, we don't always succeed of course, try to never break people's code. Unless it's a bug fix

Ignace Nyamagana Butera  15:03
	That was it for my questions.

Derick Rethans  15:06
	Maybe I have some questions for you now. I think it is good to talk about these issues. What are you most surprised with in the way how the PHP process handles bugs and bug reports?

Ignace Nyamagana Butera  15:15
	The first thing is, like I say, I've been coding in PHP for more than 15 years, but I only started really to report bugs once I start doing some open source project. Because before I think, and I think it's the majority of people, it's like, yes, there is a bug, oh it's something for PHP, or for any kind of language. I'm not the maintainer. So it's a bug, someone else will report it not to me. Since I've changed because I'm doing myself some open sourcing. I'm like, hey, if I found a bug, I think the best way to resolve that bug is first, to report it and to report it correctly, to the project, to the language or to whatever has that bug. And once you've made this change of how you think about the language, then you start to ask yourself, okay, how can I do it the most efficient way so that the bug get reported? And then the bug can get tackled by the people who can.

Derick Rethans  16:19
	Yeah, and the start of that, as you say's, always send us a bug report or sent your favorite open source project a bug report.

Ignace Nyamagana Butera  16:26
	Exactly.

Derick Rethans  16:27
	I can sort of see where you're coming from. Because I can understand that if you're just in an agency, for example, and the only thing, the only thing you have to do is to make sure that your project is done on time. You can't necessarily wait for the bug to be fixed in PHP anyway, because the product needs to be done by tomorrow or yesterday. And you're going to have to find a workaround you issue in that case anyway. And then you spending time reporting the bug will just takes you time and you don't have time for that, for example. But of course, if you do that, then everybody else that runs into this bug will have to come up with a workaround, and that means you're all end up wasting lots of time.

Ignace Nyamagana Butera  17:04
	I remember I had a small story. In one of my previous jobs, someone came to me and we're talking about something and he said: Oh, but there is no constant on the DateTimeImmutable. That's very sad. And I said: no, there is because I remember I submitted the bug, and it was tackled. And now the constants are on the interface. So DateTimeImmutable has the constant and was like: Oh, yeah, but I didn't know. And I was; it was reported and someone use it. And if you don't report it, then maybe in two years, you will ask yourself the same question. Indeed, it takes time. Between the moment it is reported the moment it is tacked, because people need to have time to resolve the issue. But if you don't do the first step, which is reporting it correctly, then it will never be solved.

Derick Rethans  17:53
	And by correctly that also means doing in the PHP bug tracker and not complaining on Twitter.

Ignace Nyamagana Butera  17:58
	Exactly. Exactly.

Derick Rethans  18:02
	Of which I see quite a bit of for Xdebug for example. Thank you very much for taking the time to talk to me, or I should say thank you very much for taking the time to interview me to talk about bugs today. I hope you enjoyed this.

Ignace Nyamagana Butera
	Thank you for having me. And hopefully we'll meet again.

Derick Rethans
	I'm looking forward to that. Thanks very much.

Ignace Nyamagana Butera  18:21
	Thank you.

Derick Rethans  18:23
	Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.

Show Notes
----------

- `League CSV <https://csv.thephpleague.com>`_
- `PHP Bug Tracker <https://bugs.php.net>`_
- `Random Bug <https://bugs.php.net/random>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
