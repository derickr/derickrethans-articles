PHP Internals News: Episode 84: Introducing the PHP 8.1 Release Managers
========================================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-05-13 09:12 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin084
   :Enclosure: media/pin-084.mp3

In this episode of "PHP Internals News" I converse with Ben Ramsey
(`Website
<https://dev.to/ramsey>`_, `Twitter
<https://twitter.com/ramsey>`_, `GitHub <https://github.com/ramsey>`_)
and Patrick Allaert (`GitHub <https://github.com/patrickallaert>`_, `Twitter
<https://twitter.com/AllaertPatrick>`_, `StackOverflow
<https://stackoverflow.com/users/201453/patrick-allaert>`_, `LinkedIn
<https://www.linkedin.com/in/patrickallaert/>`_) about their new role as PHP
8.1 Release Managers, together with Joe Watkins.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-084.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:14  
	Hi, I'm Derick, welcome to PHP internals news, a podcast, dedicated to explaining the latest developments in the PHP language. This is episode 84. Today I'm talking with the recently elected PHP 8.1 RMs, Ben Ramsey and Patrick Allaert. Ben, would you please introduce yourself.

Ben Ramsey  0:34  
	Thanks Derick for having me on the show. Hi everyone, as Derick said I'm Ben Ramsey, you might know me from the Ramsey UUID composer package. I've been programming in PHP for about 20 years, and active in the PHP community for almost as long. I started out blogging, then writing for magazines and books, then speaking at conferences, and then contributing to open source projects. I've also organized a couple of PHP user groups over the years, and I've contributed to PHP source and Docs and a few small ways over the years, but my first contributions to the project were actually to the PHP GTK project.

Derick Rethans  1:14  
	Oh, that's a blast from the past. You know what, I actually still run daily a PHP GTK application. 

Ben Ramsey  1:21  
	Oh, that's interesting. What does it do?

Derick Rethans  1:23  
	It's Twitter client.

Ben Ramsey  1:24  
	Did you write it.

Derick Rethans  1:26  
	I did write it. Basically I use it to have a local copy of all my tweets and everything that I've received as well, which can be really handy sometimes to figuring out, because I can easily search over it with SQL it's kind of handy to do. 

Ben Ramsey  1:41  
	It's really cool. 

Derick Rethans  1:42  
	Yep, it's, it's still runs PHP 5.2 maybe, I don't know, five three because it's haven't really been updated since then.

Ben Ramsey  1:49  
	Every now and then there will be some effort to try to revive it and get it updated for PHP seven and eight, but I don't know where that goes.

Derick Rethans  1:59  
	I don't know where that's gone either. In this case, for PHP eight home there are three RM, there's Joe Watkins who has done it before, Ben, you've just introduced yourself, but we also have Patrick Allaert, Patrick, could you also please introduce yourself.

Patrick Allaert  2:13  
	Hi Derick, thank you for the invitation for the podcast, my name is Patrick Allaert. I am a Belgian freelancer, living in Brussels, and I spent half of my professional time as a IT architect and/or a PHP developer, and the other half, I am maintaining the PHP extension of Blackfire, a performance monitoring solution, initiated by Fabien Potencier.

Derick Rethans  2:39  
	I didn't actually know you were working on that.

Patrick Allaert  2:40  
	I'm not talking much about it but more and more. So I succeeded to Julian Pauli, who by the way was also released manager before so now I'm working with Blackfire people. It's really great, and this gives me the opportunity to spend about the same amount of time developing in C and in PHP. This is really great because at least I don't. It's not just only doing C. I, at least I connect with what you can do with PHP. I see the evolution from both sides. And this is really great. It's great, it's also thanks to you Derick, you granted me access to PHP source codes. That was to contribute to testfest something like 12/13 years ago, it was, CVS, at that time.

Derick Rethans  3:28  
	CVS, so now I remember that. Basically, what you both of you're doing is making me feel really old and I'm not sure what I like that or not. I think we all have gotten less head on our heads and greyer in our beards. In any case, what made you volunteer for being the PHP 8.1 RM?

Patrick Allaert  3:46  
	In my case, I think there were two two reasons is that PHP really brings a lot to me in my career, everything is built around my expertise in PHP and its ecosystem. By volunteering as a release manager. I think I can give something back to PHP, because the last time I contributed to source code of PHP, it was really years ago. If I remember it was array to string conversion that was very silent and not emitting any notice; now it's warning. In the meantime, so I think that was PHP 5.0,

Derick Rethans  4:22  
	Ages ago.

Patrick Allaert  4:23  
	Ages ago. Indeed. I was quite passive I was mostly reading on PHP internals, and most of the time now that is quite big so if, if I had to say something I could always see some someone who already just said the same thing so I was not saying: plus one. This is one of the reason and the second one I think is that I think it's kind of a unique opportunity, and I can learn a couple of things. I think, on day one when the Rasmus gave me the access, saying that I can do to OAuth authentication on SSH and that was: okay, day one I already learned something, so that was really cool.

Derick Rethans  4:58  
	And you Ben, I think you tried to be the PHP eight zero release manager as well at some point. That didn't happen at the time, but you've tried again.

Ben Ramsey  5:06  
	I almost didn't try again. I don't know why but when Sara announced it this year, I thought about it, and I don't know, I tossed it around a little bit, but I've been wanting to do it for a long time and I've noticed as Joe Watkins recently put it on a blog post that we need to help the internals avoid buses. So since this is a programming language that I've spent a lot of time with just as Patrick mentioned, both in and out of my day jobs. I want it to stick around to thrive. Since I'm not a C guru, but I do have a lot of experience managing open source software. I wanted to volunteer as a release manager, and I hope that I can use this as an opportunity to inspire others who might want to get involved, but don't know how.

Derick Rethans  5:55  
	And of course you just mentioned Joe, Joe Watkins, who is the third PHP release manager for 8.1, and that is a bit of a new thing because in the past, when the past many releases I can remember you've only had two most of the time.

Ben Ramsey  6:09  
	I think, on the mailing list that came up early on in the thread, and there was a general consensus, I think, consensus may be the wrong word, but there were a couple of people who spoke up and said that they wouldn't mind seeing multiple rookies or mentees or whatever you want to call us, and Joe when he volunteered to be the veteran, and he was the only one who volunteered as the veteran. He said that he would take on two. And so that's that's why Patrick and I are both here and I think that's a good idea, because it will continue to help, you know, us to avoid buses.

Derick Rethans  6:46  
	Yep. And if you're three, you only have once every 12 weeks. Whereas of course, in my case doing it for PHP 7.4 it's every four weeks, because it's me on my own, isn't it. Which is unfortunate that these things happen because people get busy in life sometimes. Getting started being a PHP release manager can be a bit tricky sometimes because just before we started recording, I had to add you to a few mailing lists. Do you think you've now have access to everything, or what do you need access to to begin with?

Patrick Allaert  7:18  
	There is the documentation about release managers, what are you supposed to do, and, and there is an effort of documentation, what you have to ask, in terms of access, and that's great. We are probably going to contribute with our findings to, to improve the documentation. Once you did a bit of the setup, mainly needs to access the servers. You should also know what is the workflow and what are the usual tasks. This is mentioned in the documentation, but I think it would be better to have a live discussion with someone that already did it. The fact that we are doing it with Joe Watkins, who is not only a release manager of 8.1, but also previous release manager, that should be really smooth, to, to see what the the orders and what is the routine to do. To do so, why do you think Ben?

Ben Ramsey  8:16  
	I agree. I think that, I mean we've only just gotten started. It's only this I believe is what was it two weeks ago that we, that this was announced. So this is the first time that Patrick and I have actually spoken face to face. Hi, Patrick! We've communicated by email and slack. I'm sorry not Slack, StackOverflow chat. Joe has given us a lot of good pointers. I feel like some of the advice he's given his been really good, but it's like Patrick said, we haven't really had like a live, like one on one chat, or face to face chat, where we could kind of get caught up on things and understand what the flow looks like. So last week I started going through a lot of the pull requests on GitHub. And I've been tagging them as bug fixes or are enhancements, and there's also an 8.1 milestone that I've been adding to a lot of the tickets, are the pull requests, and I've merged a few of them, but I think that I've merged them a little prematurely. So there were some funny things that came up out of that. I do plan to blog on this, but one of Nikita's comments in the Stack Overflow chat was, you've just made it your personal responsibility to add tests for uncovered parts of the Ristretto255 API.

Derick Rethans  9:40  
	Right, exactly say because I'm doing release management for PHP seven four. I don't do any merging at all. The only thing I'm doing is making the packages, and then coordinating around them. I'm not even sure whether it is a responsibility of a release manager to do.

Ben Ramsey  9:55  
	It may not be a responsibility. I felt like it was helpful maybe to go ahead and take a look and see where things were trying to follow up with people, to get them to respond if something had been sitting there for two weeks or so without any kind of movement. I would, you know, leave a message saying what's the status of this.

Derick Rethans  10:19  
	I know from the documentation that we have on our Release Management process. And many of these steps actually been replaced by a Docker container that actually builds the binaries, so I'm not sure whether Joe I've mentioned that to you yet, because I'm not sure whether that was around when he did release management, the previous time.

Ben Ramsey  10:36  
	Right, it wasn't around either when he did release management, but he's also mentioned that he would like for us to learn how to do it without the Docker container, even if we do plan to use the Docker container.

Derick Rethans  10:48  
	That's fair enough, I suppose. I have never had to do that, but that there you go. Now, what is the timeline like?

Patrick Allaert  10:56  
	In terms of timeline I think the very first thing is being all three release managers having live discussion to define what, what we should do, when we should do, and how. This way we clearly knows our responsibility and the sequence, and also how we are going to organize. Do we do every three releases? We share the task? How are we going to do the work together. In terms of timeline I think the very first release is going to happen in June, if I remember correctly. I set up an agenda sheet with ICAL so that we all can put that in our calendar, nothing really clear on my side.

Derick Rethans  11:41  
	From what I can see from the to do list that the first alpha release is June, 10, which is exactly a month away from when we are recording this.

Patrick Allaert  11:51  
	Right, yeah, it's one month come down before the very first one. I think it might be great that the very first release being made by by Joe, so that we can really see every single step he's doing, so that we can do the same. However, I guess it's kind of a shared responsibility to do triage of bugs and pull requests.

Ben Ramsey  12:14  
	Right. I think there is some desire among the community to see these releases in real time at least a few of them. So I'm going to try to encourage us to stream some of them maybe live, or at least record it and put it up somewhere for people to kind of just see the process to demystify it, so to speak.

Derick Rethans  12:35  
	I actually tried it a few months ago to record it, but there were so many breaks and pauses and me messing things up, and me swearing at it, that I had to throw away the recording. I mean the release went out just fine but like absolute as again... I can imagine the first few times, you're trying this there might be some swearing involved, even though you might not vocalize that swearing.

Ben Ramsey  12:56  
	Oh I'll vocalize it.

Derick Rethans  12:58  
	Fair enough. This is something that is that you're going to have to do for the next three and a half years. Do you think you'll be able to have the time for it in another three years?

Ben Ramsey  13:08  
	I mean for myself I I'm committed to it, I definitely believe that I'll have the time over the next three and a half years, and I'll make the time for it.

Derick Rethans  13:18  
	What about you, Patrick?

Patrick Allaert  13:20  
	Exactly the same. I think it's the least that I can do to PHP, in terms of contributing back, there will be some changes because I it's not like it's, it's not like the infrastructure is something that doesn't change, like for example recently, GitHub, being more having more focus rather than our Git infrastructure. So the changes that will happen, we will have to adapt, I have the impression that release manager has to, every time it's adapting to change this, and that will be very interesting.

Derick Rethans  13:53  
	Luckily we haven't had too many. The only thing I had to change with a change from git dot php.net to get up, was my local remote URLs. So there wasn't actually a lot to do, except for running git remote set-url. I was pleasantly surprised by this because if anything messing around with Git isn't my favourite thing to do. 

Ben Ramsey  14:14  
	Also, merging is a little bit more streamlined now you don't have to go to qa.php.net to do that.

Derick Rethans  14:21  
	I've never done anything without

Ben Ramsey  14:23  
	Really? Oh, I guess you would commit directly to git.php.net?

Derick Rethans  14:27  
	Yep.

Ben Ramsey  14:28  
	If there were PRs on GitHub, the only way to merge them well, probably wasn't the only way but one way to merge them was to go to qa.php.net, and if you were signed in with your PHP account, you were able to see all the pull requests, and choose to merge them.

Derick Rethans  14:46  
	Yep, also something I've never done as an RM. The only way how I have reacted with pull requests is commenting on the pull requests, and I wouldn't merge them myself.  With the only exception of security releases where you need to cherry pick from certain branches into your release branches. I'm not always quite sure about it as the responsibility for release managers actually do the merging into the main branches. From what I've understood is it's always the people that made the contributions, who just merge themselves, and you then sometimes need to make sure that they merge into the right branch instead of just master, which is what, as far as I know, the, the buttons on GitHub do.

Ben Ramsey  15:21  
	Well the individual contributors, in this case, if they're doing like a bug fix or something, most of them, or many of them aren't don't have permission to do the merging, so someone else has to merge it, like, often I see Nikita merging, a lot of the pull requests.

Derick Rethans  15:37  
	Maybe I've just been relying on Nikita to do that then. I'm not sure how, bug fixes are merchants debug fig branches. I think it's usually been done by people that have access already anyway, because it's often either Nikita or Cristoph Becker, or Stas, and the main developments, or the main other new things that people don't have access to are usually to master. So I guess there's a bit of a difference now. I'm not sure what if any other questions, actually, would have anything to add yourself?

Patrick Allaert  16:05  
	maybe something that would be quite challenging is the very recent discussion about the system that we, that we might change from. The system or the issue tracker with where we have all the bugs. I understand the current issues, I understand as well the drawbacks of what is possibly, for example GitHub issues. It might be great for some, would it be great for us? If we do it was going to be in the bring a lot of changes, and I think, 8.1 will be already slightly impacted by the change to GitHub in terms of pull request strategies, but potentially there will be another change, which is around the bugtracking system.

Derick Rethans  16:54  
	I have strong opinions about this, but we'll leave that for some other time. What about you, Ben?

Ben Ramsey  17:00  
	Right, I actually don't think that we're going to end up making a lot of changes in that regard, very, like, not in the near term, probably. But I did want to point out, or promote that I've started journaling some of these experiences, and capturing information mainly for my own purposes, but I'll be posting these publicly so that others can follow along. My blog is currently down right now.

Derick Rethans  17:28  
	That's because you're using Ruby isn't it?

Ben Ramsey  17:30  
	That's because I'm using Ruby. The short story of it is that there are some gems that were removed from the master gem repository at some point in the past, or the versions I'm using were removed, either for security reasons or what I have no idea why. And that's put, put it into a state where I just can't easily update. I just haven't, I just don't care, right now, so I plan on migrating to something else. In the short term, I'm not going to be doing that. So I've started writing at https://dev.to/ramsay and Dev.to is just a developer community website. If you're on Twitter. It's run by @thePracticalDev, I'll be, I'll be blogging there.

Derick Rethans  18:18  
	And I'll make sure to add a link to that in the show notes as well. Thank you for taking the time this afternoon, or morning, to talk to me about being a PHP 8.1 release managers.

Ben Ramsey  18:28  
	Thank you for having me on the show.

Patrick Allaert  18:30  
	Thank you, Derick for that podcast. I'm really glad you invited us.

Derick Rethans  18:39  
	Thank you for listening to this installment of PHP internals news, a podcast, dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening and I'll see you next time.


Show Notes
----------

- PHP 8.1 `Release Todo List <https://wiki.php.net/todo/php81>`_
- Ben's `journal <https://dev.to/ramsey>`_
- Joe Watkins' `Avoiding busses <https://blog.krakjoe.ninja/2021/05/avoiding-busses.html>`_ blog post

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
