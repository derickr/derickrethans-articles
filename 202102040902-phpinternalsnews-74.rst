PHP Internals News: Episode 74: Fibers
======================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-02-04 09:02 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin074
   :Enclosure: media/pin-074.mp3

In this episode of "PHP Internals News" I talk with Aaron Piotrowski (`Twitter
<https://twitter.com/_trowski>`_, `Website <http://https://trowski.com/>`_,
`GitHub <https://github.com/trowski>`_) about an RFC that he is proposing
to add Fibers to PHP.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-074.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:14
	Hi I'm Derick. Welcome to PHP internals news, the podcast dedicated to explaining the latest developments in the PHP language. This is Episode 74. Today I'm talking with Aaron Piotrowski about a Fiber RFC, that he's working on together with Nicolas Keller. Aaron, would you please introduce yourself.

Aaron Piotrowski  0:33
	Hi everyone I'm Aaron Piotrowski, I started programming with PHP back in 1998 with PHP three, so I've just dated myself there but, but I've worked with a lot of different languages over the last few decades but PHP is always continually remaining, one of my favourite and I'm always drawn back to it. I've gotten a lot more involved with the PHP projects since PHP seven. The Fiber RFC is my first major contribution I have attempted though. In the past I did the RFC for the throwable exception hierarchy. And the Iterable pseudo type in PHP 7.1.

Derick Rethans  1:12
	Yeah, these things are both before I started doing the podcast so hence we haven't spoken yet, at least on here. We've actually met at some point in the past. I've had a read through the Fiber RFC this morning, but I'm still fairly baffled. Could you perhaps explain in short what Fibers are where the idea comes from. And what's your specific interest is in adding them to PHP?

Aaron Piotrowski  1:35
	A few other languages already have Fibers like Ruby, and they're sort of similar to threads in that they contain a separate call stack, and a separate memory stack, but they differ from threads in that they exist only within a single process, and that they have to be switched to cooperatively by that process rather than actively by the OS like threads. So sometimes they're called Green threads, and the generators that are in PHP already are actually somewhat similar to Fibers; but generators differ in that they're stack less. And so what that means is that generator function can only be interrupted at one layer. Whereas a Fiber can be interrupted anywhere in the call stack. So like it'd be imagine if you had a generator where yield could be very deep in a function call. Rather than at the top level. Like, how generators can be used to make interruptible functions, Fibers can also be used to create similarly interruptible functions, but with again without having to know exactly when it's going to be interrupted not at the top level but at any point in the call stack. And so the main motivation behind wanting to add this feature is to make asynchronous programming in PHP much easier and eliminate the distinction that usually exists between async code that has used promises and synchronous code that we're all used to.

Derick Rethans  3:09
	So what specifically are you proposing to ask to PHP here then?

Aaron Piotrowski  3:12
	Specifically I'm looking at adding a low level Fiber API, that's really aimed specifically at async framework authors to create their own more opinionated API's on top of that low level API. So adds just a couple of classes: Fiber, and a FiberScheduler on within a couple of exception classes and reflection classes for inspecting Fibers. When a Fiber is suspended to the execution switch is to FiberScheduler, which is then a special Fiber, that's able to start and resume, regular user Fibers. So a Fiber scheduler is generally going to be something like an event loop that then, when a Fiber is suspended that our scheduler event loop will resume certain Fibers, like in response to events like data becoming available on a socket or like a timer expiring.

Derick Rethans  4:17
	How would the event loop, decide which Fiber to resume, depending on on input, for example?

Aaron Piotrowski  4:24
	It's largely up to, how like that framework choose to write that event loop, but in general, like when a Fiber is going to suspend it'll set up some sort of callback, or add it to like an array of Fibers that's waiting on events, and when execution switches to the event loop.

Derick Rethans  4:44
	A Fiber before it suspends itself set up add another Fiber to the scheduler.

Aaron Piotrowski  4:48
	That's exactly right. Before a Fiber suspends itself it adds itself to some sort of event in the event loop that when it triggers, it will resume that Fiber. So, if you're familiar with how some of the other async frameworks that work now they'll add something like a callback or promise to the event loop that's resolved. This is sort of working the same way except that it's just resuming a Fiber that of like invoke a callback, although you know that Fiber might be resumed and by invoking a callback.

Derick Rethans  5:23
	Is the Fiber scheduler or the event loop as however if you want to call it, is that something you would, or can also make use of in a normal PHP applications, I mean by normal I mean, not like an async PHP framework? Is that the intention is all?

Aaron Piotrowski  5:39
	Using Fibers than kind of helps you eliminate that boundary that that exists, trying to put asynchronous code into something using synchronous code because you end up with a promise that you have to await in that doesn't really work very well, where you're trying to mix it with sync code, Fibers eliminate the need for a promise. Since, the asynchronous function can still return types, you can mix async code into a traditional like sync application, even like something running in Nginx or Apache, it doesn't have to be a fully asynchronous app to make use now of some async I/O.

Derick Rethans  6:23
	For example if he wants to do multiple database calls at the same time. Will you be able to use a uses for that?

Aaron Piotrowski  6:30
	Exactly.

Derick Rethans  6:31
	If a user would have a PHP application and want to use multiple database calls at the same time, how would, how would they set it up it with Fibers and Fibers scheduler?

Aaron Piotrowski  6:40
	This is a low level API is aimed primarily at like a framework author. Generally if you're writing application with it, you're probably going to use one of those frameworks so it would largely depends on how that framework would set that up. Although, in general, those frameworks are going to provide some sort of abstraction for running code concurrently, that they probably have their own sort of placeholder object like like a promise again. So that when you start running things concurrently, they return to something that you can then wait on for all those things to end up, or when those things, complete executing. So it doesn't totally eliminate the need for promises, but it does allow for both to do not always that async to not always have to return a promise rather a promise is only required when you want concurrency, and that, you know, a framework will provide tools to await that can still be mixed in with synchronous code.

Derick Rethans  7:48
	Do I understand this correctly that you won't need to promise unless you mix it with synchronous code?

Aaron Piotrowski  7:54
	You won't need a promise unless you explicitly need concurrency.

Derick Rethans  7:57
	Okay, that makes more sense I suppose.

Aaron Piotrowski  7:59
	It's difficult to explain it's so much easier with examples.

Derick Rethans  8:03
	Yeah but examples are very difficult to do in audio only.

Aaron Piotrowski  8:07
	Yes, exactly. You have like a database query that returns a result. If you want to run multiple queries at the same time, the async library that uses Fibers underneath would be able to provide an abstraction that would allow you to run multiple queries at once. But that those two run concurrently would return a promise. But you would be able to collect those promises together, and use like a await function, provided by that async framework to then get the results of all of the queries at once.

Derick Rethans  8:47
	You mentioned that Fibers are not threads, they just are more, they're sort of logical threads, but not physical threads in the same process. PHP isn't multi threaded, how would this work internally? What would have Fiber do or store, so that the scheduler can resume them for example? What is the internal mechanism, how does this interact with PHP itself.

Aaron Piotrowski  9:11
	Each Fiber is allocated a C stack and a VM stack on the heap. So switching between them is similar to generators, when switching between Fibers the current VMs stack is swapped and the C stack is swapped, but it doesn't touch any of the other memory in the process, so things like globals are still accessible to each Fiber, since only one Fiber can be executing at the same time, you don't have some of the same race conditions that you have with threads of memory being accessed or written to by two threads at the same time. It can't happen with Fibers that you can have two Fibers that might be dependent on the same memory, and you may have to do some of the same sort of synchronization, that you have to do with threads to that memory if you don't want interleaving of Fibers to be potentially overriding that memory. That's the sort of thing that's being left again to like the async frameworks that would use this to provide that sort of mechanism over a low level Fiber API.

Derick Rethans  10:15
	Of course when a Fiber is running, there's no need for locking anything because nothing runs at the same time anyway. And of course, when a Fiber suspense itself it then sort of knows that, well, I'm unlocking what I'm wanting to use of don't have this synchronization issue there.

Aaron Piotrowski  10:32
	You don't have the synchronization issue where you have to worry that while while this Fiber is running, another Fiber might overwrite the same memory. But there is a potential that if a Fiber suspends that while it's suspended another Fiber could have overwritten some global memory, so if you're if you're sharing memory between Fibers it's best to use some sort of abstraction, like channels in Go to share data between Fibers rather than like a global. It could just be a global, it could even be like a class property or something, anything that you might share between two Fibers you could give the same object to two different Fibers, and those Fibers could modify that object. Well, I wouldn't recommend doing that, I would share that object over like a channel instead.

Derick Rethans  11:23
	Your RFC doesn't talk about channels. So, I reckon that'd be something else that has to be implemented, probably with Fibers in the async framework.

Aaron Piotrowski  11:31
	Exactly, yes.

Derick Rethans  11:32
	What is your reason to want to others to PHP core instead of having it sitting in a PECL extension because I could argue that this isn't something that many PHP developers would ever use.

Aaron Piotrowski  11:43
	I definitely see that point. I think that availability for being able to use that in any sort of application would be important for some reason there still seems to be a hesitation on certain platforms to install extensions. But more beyond that, there are reasons that you'd want to have it in core all the time, extensions that would want to profile code will need to be aware of Fibers. And if, if Fibers are an extension well then actually making use of it in a real application might be difficult because your code profilers don't work very well because they don't understand the Fiber switching. So that is one area that if this were merged into core, code profilers would probably have to be updated to account for that. There was also a bit of an issue in the extension right now that due to destructor order, how the shutdown logic goes. And what hooks are available in PHP, that if a registered shutdown function or a destructor suspends a Fiber, it might have to restart the scheduler unnecessarily. But if it were in core, I could avoid that. And then there's there's also issues with how to handle some of the global stacks that PHP provides when switching Fibers should those be reset, should they remain, but those are issues that can only be addressed if Fibers were part of the core rather than extension. Otherwise I have no choice but to just leave them as stacks that aren't switched.

Derick Rethans  13:22
	Okay yeah that makes sense, because the stack switching is something that is trickier to do from an extension.

Aaron Piotrowski  13:28
	Like the error handler, you know, how should that be handled. Should it be the error handler stack depends on which Fiber or should it remain just a constant global and I can't change that from an extension that would have to be part of core.

Derick Rethans  13:41
	Because Fibers allow you to basically switch between threads. Have you had a look at how how debuggers, for example deal with this?

Aaron Piotrowski  13:50
	In my testing with Xdebug, I didn't have any issue with inspecting execution stacks, or code coverage, that I will have to really defer to you. If you think that there's any anything that in Xdebug that would have to be updated or changed to accommodate. So far it's worked very well.

Derick Rethans  14:10
	I know you submitted a bug report with a crash, but that's been fixed already, of course. What was that issue actually, I don't quite remember what it was?

Aaron Piotrowski  14:18
	Something code coverage where I honestly don't really remember any more. It is invalid pointer for something.

Derick Rethans  14:26
	It's an interesting thing that's with all these fancy extensions, and Fiber and not being the only, sometimes you run into things that extensions do something very strange that, then make things crash in Xdebug. I can't always test for that of course up front. I actually have a slightly related question that pops into my head here is like, there's also something called Swoole PHP, which does something similar, but from what I understand actually allows things to run in threads. How would you compare these two frameworks, or approaches is probably the better word?

Aaron Piotrowski  15:00
	Swoole is, they try and be the Swiss Army knife, in a lot of ways where they provide tools to do just about everything and they provide a lot of opinionated API's for things that, in this case I'm trying to provide just the lowest level, just the only the very necessary tools that would be required in core to implement Fibers, I do believe Swoole implements Fibers as well. They use the term co-routine for their Fibers. I believe they actually use the same boost assembly language code that I used for swapping C stacks. I'm not sure if they provide actual threading as well. If they do, then that's great. Of course threading still requires a ZTS build of PHP. Fibers do not because it's still within one process.

Derick Rethans  15:55
	I know that Swoole definitely doesn't work with Xdebug because the way how they do things, but it sounds like Fibers will actually work just fine.

Aaron Piotrowski  16:02
	It seems so yes. I've used it already extensively with PhpStorm like setting breakpoints and things to debug. When I was upgrading some of the, the AMP libraries to figure out what was going wrong and it worked perfectly.

Derick Rethans  16:16
	Are you involved with AMP.

Aaron Piotrowski  16:18
	Yes, I am. One of the primary maintainers now along with Nicolas. I didn't start the library. The original author has moved on to other things, but it's it's pretty much just Nicolas and I doing most of it now. Bob still contributes occasionally as well.

Derick Rethans  16:38
	And I guess that's why are you interested in having Fibers in PHP come from then?

Aaron Piotrowski  16:42
	Yes, exactly.

Derick Rethans  16:44
	What has the feedback been so far?

Aaron Piotrowski  16:47
	Largely positive from the people that are more familiar with it. I haven't actually gotten a whole lot of feedback from the core contributors of PHP, so I'm not really sure where the proposal stands with them at the moment, but I guess maybe no feedback is good feedback if they had a problem with it somebody who's spoken up by now, I'm not sure.

Derick Rethans  17:09
	That is often the case right, if it's if there is something to be added that is quite complicated, you get a lot less feedback. Then where there's something very simple like picking a name for function right.

Aaron Piotrowski  17:19
	Yes, exactly.

Derick Rethans  17:21
	When do you think your will be putting this up for a vote?

Aaron Piotrowski  17:24
	I think I want to wait at least another month or so. I did make a recent change to how the Fiber scheduler API worked, and so I wanted to make sure that that people had time to review it. Maybe send another reminder email or two to internals, so that they, so that more people get a chance to look at it and play with it and provide feedback.

Derick Rethans  17:47
	Somewhere around mid February?

Aaron Piotrowski  17:49
	Something like that, yeah.

Derick Rethans  17:51
	Did we miss anything discussing Fibers. Do you have anything to add yourself?

Aaron Piotrowski  17:55
	No, I don't really think so. I think we covered the main points of it.

Derick Rethans  17:59
	I have to say I understand that quite a lot better now, which is always good, and hopefully the people listening to this episode will also find it interesting and understand it well. So I would say thanks for explaining Fibers to me today.

Aaron Piotrowski  18:13
	Yeah, thanks a lot for having me on.

Derick Rethans  18:18
	Thank you for listening to this instalment of PHP internals news, a podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next time.


Show Notes
----------

- RFC: `Fibers <https://wiki.php.net/rfc/fibers>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
