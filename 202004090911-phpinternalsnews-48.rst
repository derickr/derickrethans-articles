PHP Internals News: Episode 48: PHP 8, JIT, and complexity
==========================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-04-09 09:11 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin048
   :Enclosure: media/pin-048.mp3

In this episode of "PHP Internals News" I discuss PHP 8's JIT engine with Sara Golemon
(`GitHub <https://github.com/sgolemon>`_).

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-048.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:16
	Hi, I'm Derick. And this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 48. Today I'm talking with Sara Golemon about PHP 8 and JIT. Sara, would you please introduce yourself?

Sara Golemon  0:33
	Hi there. Hi there, everybody listening to PHP internals podcast. I'm Sara. I've been on this podcast before. But in case you're just getting here to for the first time, welcome to the podcast. You have a nice backlog to go through. I am a lapsed web developer, come database security engineer by day, and an opinionated open source dev slash PHP 7.2 release manager by night and also day. I've been involved with the project for about 20 years now off and on. Somehow I just keep coming back for more punishment.

Derick Rethans  1:03
	We're leading up to PHP 8, with lots of new features being added. But one of the biggest thing in PHP 8 that I've spoken about on the podcast on before all the way back last year in Episode 7, is that PHP eight is going to get a JIT engine. Would you care to explain what a JIT engine does again?

Sara Golemon  1:20
	Well, I'm going to give you the short, you can look this up on Wikipedia in two seconds definition of JIT, means just in time compilation. That doesn't really tell you much, unless you listen to it on the sort of other half of that of AOT, or ahead of time compilation. AOT is what you expect from applications like GCC, you know, you just make an application that you've got C or C++ kind of source code to that's ahead of time. JIT is saying, well, let's take the source for application. And let's just run with it. Let's just start executing it as fast as I can. And eventually we're going to get down to some compiled code. That's going to run a little bit quicker than the initial stuff did. PHP already has this nice little virtual machine built into it. We call it the Zend engine. That takes your script and immediately just says: All right, well, what does this say in computer terms? Well, a computer readable term is a series of these op codes, they're also called byte codes in other languages that give you instructions for: run this type of instruction at this time and get something done. The PHP runtime interpreter interprets that one instruction at a time basically pretending to be a CPU. This works quite well, it runs quite efficiently. But there's still this sort of bottleneck in the middle there of a program pretending to be a CPU running on top of a CPU in order to run other code. The idea of JIT is that this thing sitting in the middle is going to gradually figure out what your program really is trying to do and how it's intended to run, and It's going to take those PHP instructions and it's going to turn them all the way down into CPU instructions, so that it can get out of the way and let the CPU run your code natively as if it had been written in a compiled AOT kind of language. What that actually means for execution of PHP code in PHP 8 is still sort of a, you know, a question that's, that's left to be answered here. I listened to your interview with Zeev. Episode 7, is a good episode of getting some good information on that. We do definitely agree on what the status of the JIT within PHP is, right now we can. It's subjective facts like this is how much work has been done largely by Dmitri, where we can kind of expect to see the best gains come from. I personally think I might be a little bit more pessimistic than him in terms of the actual performance impact we get out of it. I think we both recognise we're not going to see the two to one kind of improvements we saw from five to seven. Nobody's realistically expecting that, but if you look at the demo that Zeev ran a few months ago, where he shows the Mandelbrot set being generated in two different PHP requests, and then WebSocket out to a nice pretty display, it's a very visceral reaction because you can see one Mandelbrot set being calculated much, much faster than the other. And he acknowledges though this is not realistic PHP code, nobody's writing the Mandelbrot calculation in PHP. We can see that under certain workloads, it's definitely getting faster. But for PHP core mission, which is web serving, I mean, we both know that it's not going to be massively fast. I think it's going to be almost imperceptibly fast.

Derick Rethans  4:41
	One question for my site, the Mandelbrot set, the implementation of that is all in a specific function, right? And it's all CPU heavy code, not IO.

Sara Golemon  4:51
	Yes.

Derick Rethans  4:52
	And it's all that in the same function.

Sara Golemon  4:54
	Yes.

Derick Rethans  4:55
	Now, what I was thinking of the other day is that how does this interact with calling standard library functions, because the JIT engine is going to have to go out of basically running things on the CPU and calling things that are then implemented in C to begin with.

Sara Golemon  5:10
	So you're asking that question, because you already know some of the pitfalls of JIT, and you're leading me into it. And that's fine. When a JIT emitter is taking the language that it's emitting, so PHP. As long as it remains within the scope of PHP, it can sort of keep track of where it's at. It's like, Okay, I know this variable's init, your because I saw it get set. I know that this is going on here. I know that's going on there. And it can carry those assumptions around as it's admitting code. And emit very efficient code that doesn't need a whole bunch of double check guards of like: Wait, is this still an integer? Wait, is that still a string? All of these sort of like escape hatches for when things go wrong. Anytime you cross over into, I will say C-land, or internals land, or ahead of time compiled land. It's basically calling into what it sees as a black box. And it just says: Okay, here's some data, I know the types going in, have fun with it. And something air quotes happening in the air happens with that code and the black box spits out an answer. Well, by the time the black box has spit out the answer, the JIT that has taken that PHP code, no longer knows if any of its assumptions are true or not. It just has to say: Well, time to start from scratch, time to keep track of where we are from here, build up a new set of assumptions. So we get this speed bump in the road of executing code. And it turns out most PHP applications are using a whole lot of those internal API's because they're quite useful. There is a kitchen sink in PHP, and it does stuff. So you have these repeated hits of this road bump happening, and that's not great. If we want to compare this to other JIT languages that are out there. I might suggest we compare this to HHVM because of course, HHVM, at least in the beginning implemented a fairly close kin cousin to the dialect of PHP. It has since diverge much more and become hacklang. But it was doing the same thing, taking PHP code, running it native on the CPU and occasionally having to make that cross to this its own version of internals, or it was running C++ code. One of the ways to reduce those numbers of jumps is that they took a lot of those internal functions, the ones that actually didn't need to do anything, particularly internals ish, and just rewrote them in PHP code. And if you look at the HHVM source code right now, there is a big directory called systemlib and that's a whole bunch of hacklang code, read it as PHP code, that is implementing a lot of these very common quote unquote internal functions. We just had an RFC for function called str_contains(), that is a function that could have been hundred percent been written just as PHP code. Something could have thrown that into packagist. For the record, I voted against it because of exactly that. I think you should write that in packagist and just put it in your composer.json is okay. It's gonna pass anyway, it got a lot of votes. That aside over, that is a sort of function that if we were putting it into sort of an 8.X version of PHP, where we did have our own type of systemlib, we would have probably just said, let's write that as PHP code. So that the JIT, when it enters that function, can keep all those assumptions intact, and potentially even inline some of those instructions and avoid the function call entirely. That's basically taking all of the instructions that are part of the in this case, str_contains() function, and implementing them within the scope of the function that was calling it. So you skip that entire function call overhead, which a lot of people know is still one of PHPs sort of weaker points in terms of where that fat to trim is, as Zeev said in Episode Seven, we still have some parts of PHP that are a bit slow, irrespective of a JIT.

Derick Rethans  8:50
	There are actually a few functions that have been inlined now into op codes. strlen() is an example of this where instead of it now being a function call, it's actually directly an opcode. Because it is a function that is used so much and actually gain a bit of performances there.

Sara Golemon  9:05
	Yeah, I think all of these functions as well are just a single opcode for type check. Yeah.

Derick Rethans  9:10
	There's a whole bunch of them for sure. I saw that earlier this morning, Dmitri produced, or proposed another branch in which he implemented tracing JITs, instead of the JIT that we already have, and I have no idea what the difference is between a normal JIT engine and the tracing JIT engine,

Sara Golemon  9:25
	Ultimately, the distinction is not that important to end users, it's going to function the same, but it is a sort of an internal implementation detail. HVVM's by the way, is a tracing JIT. It basically looks at any given unit of work that it needs to translate, let's say a function, and it says, what are the pieces that have these sort of non branching parts attached to them? Let me look at each of the non branching pieces. And let me create a version of that translation based on the types that I expect to be going in there. If the types fail, I'm gonna have to create a new version of that piece. But then that piece can plug into this sort of chain of tracelets to create a full function. Most of the time, especially if you've written code that is well type hinted, you've got, you know, strict types turned on, you've got all of your types on the on the function parameters set. And it's very easy for the JIT to infer the types out of what you've put into your function. You're only ever going to need to create a single tracelet of any given section, and your full trace is going to be a single, unbroken chain of: do this, do this, maybe do a jump to another spot, just keep doing this, doing this, doing this. If you have, let's say, slightly messier code, maybe you're not using any kind of type hinting it becomes very difficult to infer any of the types, because there's lots of different call sites, that are doing lots of different things. We may end up having some functions that have multiple tracelets per body section that get built into the giant bush of interconnected edges, that's less ideal in terms of maximising performance, but it still at least functions.

Derick Rethans  11:06
	We have spoken a little bit about what a JIT engine is and sort of how it works. It sounds quite complex and complicated.

Sara Golemon  11:14
	It is definitely complicated. And I'm feeling like that's another lead. And so I'll just run with it.

Derick Rethans  11:19
	I've also got to say my next leading question... Maybe I should actually ask the question?

Sara Golemon  11:24
	Well, let's actually take a step back from the JIT for a second. And let's look at where the engine is right now. So the engine is basically two very large pieces. That's the sort of the extension library of all of the runtime functions. Everything you see exposed in user space, and the actual scripting engine. There are some other smaller pieces, but those are two, the two really big pieces. There are a whole lot of people pay a whole lot of attention to the extension piece, because that's the flashy bit. That's the part that gives you some bit of binding that you didn't have before, or some bit of functionality that can be delivered out of the box as part of that kitchen sink. And that definitely needs attention. I'm glad that that continues to evolve. But the scripting engine is that piece that defines syntax and how code is actually going to run.

Derick Rethans  12:09
	Reading extension's code as a whole lot easier than reading the engine code.

Sara Golemon  12:13
	And that's where I was going to go with that, yes, if you look at the code that's under ext, you can even come into that code without knowing any C at all. And you can actually make pretty good sense of a lot of it because a) PHP uses a whole lot of macros. So every function is literally defined with a macro that says: PHP_FUNCTION, like right here, PHP function, every class method, PHP_METHOD, here's the class name. Here's the method name. And what these things do are pretty clear sort of API's. They're very small bite sized pieces for the most part. The bits that involves sort of defining a class and how it does its memory management, those get a little bit more complicated, but I think on the whole extension code is far more accessible. If you go and look at the engine, particularly the runtime pieces of the engine, although the compiler is complex as well. You have to do a lot of digging before you even get to a point that you can see how the pieces maybe start to fit together. You and I have spent enough time in the engine code that we know where to look for a particular thing. Like let's say that opcode, you mentioned that implements strlen(). We know that, oh, zend_vm_def.h has got the definition for that. We also know that that file is not real code. It's a pre processed version of code that gets built later on. Somebody coming to that blind is not going to see a lot of those pieces. So there's already this big ramp up just to get into these engine as it exists now in 7.4. Let's add JIT on top of that. You've got code that is doing call forward graphs, and single static analysis, and finding these tracelets, and making sense of the code at a higher level than a single instruction at a time, and then distilling that down into instructions that the CPU is going to recognise. And CPU Instructions are these packed complex things that deal with immediates, and indirects, and indirects of indirects, and registers. And the x86 call ABI is ridiculous thing that nobody should ever have to look at. So you add all this complexity to it, that by the way, sits in ext/opcache. It's all isolated to this one extension that reaches into the engine, and fiddles around with things to make all this JIT magic happen. You're going to take your reduced set of developers who know how to work on Zend engine, and you're going to reduce that further. I think at the moment, it's still only about three or four people who actually understand how PHP's JIT is put together enough that they can do any effective work on it. That worries me for sure. I don't think that's an insurmountable hill to climb, especially if we can start getting some documentation written about it, at least from a high level point of view. Hey, you know, look over here to find this stuff. Look over here to find that stuff. Something to get started. So the people who have at least that basic understanding of how the VM part of the Zend engine works can sort of upgrade their knowledge to get into to the JIT. I only think that's worth it. If we actually get real performance boost out of JIT. If we actually turn the JIT on, and we see that for PHP's core workload, which is web serving, we're only seeing a one to 2% gain. For me, that's not enough. It may be enough for others. But for me, I would call that experiment, not a failure, but a non success at that point. Certainly there are people out there who are still going to want to use it, because they are you doing command line applications, and they're doing complex math. And I'm not saying we can't have it. I'm just saying it takes less than a forward stage that point.

Derick Rethans  15:43
	Somebody mentioned earlier in the chat room. It's also another set of potential bugs, right?

Sara Golemon  15:48
	It is definitely another potential bugs.

Derick Rethans  15:51
	It's pretty much another implementation of the PHP syntax bits of PHP.

Sara Golemon  15:57
	So if you run an application and you get behaviour you don't expect, where is that behaviour actually coming from? You can spend a lot of time looking in Zend engine because you're thinking like: Oh, well, this is the thing that executes opcodes. And when I run it in a single command line, it's definitely going through this bit of code, but it works on a single command line run. But at the twentiest request on my web server, it's not working. Why is that happening? Well, it turns out, it's happening, because that's when the JIT has finally kicked in, because it has enough information. And it's running through this tracelet that was just a little bit wrong. And well, crap. You mentioned I think, at one point, when we were talking in Miami just a couple months ago, that you're just gonna have to turn the JIT off entirely when Xdebug is running,

Derick Rethans  16:41
	Just like I'll already turn OPCache optimizations off, because there's just too confusing for people.

Sara Golemon  16:46
	It's confusing and complex, but it's also it may not even be 100% possible because we are right there down at the bare metal of running CPU instructions. There's not a lot of opportunity to just say like, Oh, hold on Mr. CPU, let me just take a look at your registers right now. Okay, this is okay, let's go ahead and keep going now. The VM that we have now in in Zend lends itself 100% to those kinds of activities, CPU does not. What that means is that what we experience in the development mode with Xdebug running is not going to be the exactly the same thing that we experience in real runtime code. And I don't know if we have a solution for that.

Derick Rethans  17:23
	As far as I know, there's no solution for it at all.

Sara Golemon  17:26
	I was trying to cage it in the hope that maybe we could someday have solution for it.

Derick Rethans  17:30
	It'd be lovely, but I can't see that happening to be honest. I think it's going to be important to find out how much this actually benefits, real live code. How does it benefit your Laravel project or your Symfony project or anything like that? I think it's going to be hard to now make a case for not shipping PHP 8 with a JIT. I think that'd be a bit unfair. But on the other side, if it's, as you say, only really gives you one or 2%, whether this is worth have the additional complexity. The additional maintenance burden as well as another opportunity for having bugs that are a lot harder to reproduce, but it's actually worth having it at all?

Sara Golemon  18:11
	I definitely don't want to poopoo on the JIT effort.

Derick Rethans  18:14
	Oh, no, absolutely not.

Sara Golemon  18:15
	I think this is an important experiment to run. And I think if 8.0 as a whole winds up being a sort of public beta experiment of it, that will definitely give us a lot of good information. And I am super hopeful that we see better percentages, that we see 5-10 maybe even 15%

Derick Rethans  18:31
	Absolutely.

Sara Golemon  18:32
	I want to be guarded in what I how I talk about it on a podcast like this because I don't want anybody say: Oh, 8's gonna be great. Our code is gonna run 10 times as fast as it was running before No, that's not gonna happen two x is not gonna happen. We're talking much lower numbers than that. Be guarded, be hopeful, but 8.0 is going to be, as I said, it's going to be that sort of public beta experiment.

Derick Rethans  18:55
	I think that's great. I think running this experiment again because ta similar experiment was, of course run during the PHP 5.6 days when PHP 7 came out. Originally with PHP 7, was PHP with a JIT engine. And then Dmitri and others found out that it was so much other things that could be done to make PHP run pretty much twice as fast.

Sara Golemon  19:16
	Yeah, there was a lot of really low hanging fruit.

Derick Rethans  19:19
	Yep. And that was great to see. I am apprehensive about people thinking that the JIT engine in PHP eight is going to similar performance boost.

Sara Golemon  19:29
	We'll see. Nothing to say about it, but then: we'll see.

Derick Rethans  19:32
	But I would suggest is that if you're interested in seeing what this can do for your projects, you should go try it out. Download PHP's master branch, enable it and see how it goes.

Sara Golemon  19:41
	And of course, make sure you are running on x86 hardware. I doubt very much that he's bothered to put more than one back end on this.

Derick Rethans  19:48
	I don't actually know.

Sara Golemon  19:49
	I haven't looked. He might be using some helper library for it. So it's possible that we're hitting multiple backends. But this is probably going to be an x86 only thing and possibly a Linux thing. I should find out the answer to that question.

Derick Rethans  20:00
	I should do too. Okay, Sara, thanks for taking the time this morning to have a chat with me about PHP 8' JIT efforts.

Sara Golemon  20:08
	It's fun as always, I always love to speak with you Derick. You bring a bright Corona of sunlight to my day.

Derick Rethans  20:16
	Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.


Show Notes
----------

- `Episode 7: PHP and JIT <https://phpinternals.news/7>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
