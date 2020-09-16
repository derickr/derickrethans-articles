PHP Internals News: Episode 68: Observer API
============================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-09-17 09:31 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin068
   :Enclosure: media/pin-068.mp3

In this episode of "PHP Internals News" I chat with Levi Morrison (`Twitter
<https://twitter.com/morrisonlevi>`_, `GitHub
<https://github.com/morrisonlevi>`_) and Sammy Kaye Powers (`Twitter
<https://twitter.com/SammyK>`_, `GitHub <https://github.com/SammyK>`_,
`Website <https://www.sammyk.me/>`_) about the new Observer API.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-068.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:15  
	Hi, I'm Derick, and this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 68. Today I'm talking with Levi Morrison, and Sammy Powers, about something called the observer API, which is something that is new in PHP eight zero. Now, we've already passed feature freeze, of course, but this snuck in at the last possible moment. What this is observer API going to solve?

Levi Morrison  0:44  
	the observer API is primarily aimed at recording function calls in some way so it can also handle include, and require, and eval, and potentially in the future other things, but this is important because it allows you to write tools that automatically observe, hence the name, when a function begins or ends, or both. 

Derick Rethans  1:12  
	What would you use that for? 

Levi Morrison  1:13  
	So as an example, Xdebug can use this to know when functions are entered or or left, and other tools such as application performance monitoring, or APM tools like data dog, New Relic, tideways, instana so on, they can use these hooks too.

Derick Rethans  1:38  
	From what I understand that is the point you're coming in from, because we haven't actually done a proper introduction, which I forgot about. I've been out of this for doing this for a while. So both you and Sammy you work for data dog and work on their APM tool, which made you start doing this, I suppose.

Sammy Kaye Powers  1:54  
	Yeah, absolutely. One of the pain points of tying into the engine to to monitor things is that the hooks are insufficient in a number of different ways. The primary way that you would do function call interception is with a little hook called zend_execute_ex and this will hook all userland function calls. The problem is, it has an inherent stack bomb in it where if, depending on your stack size settings you, you're going to blow up your stack. At some point if you have a very very deeply deep call stack in PHP, PHP, technically has a virtually unlimited call stack. But when you use zend_execute_ex, it actually does limit your stack size to whatever your settings are your ulimit set stack size. One of the issues that this solves is that stack overflow issue that you can run into when intercepting userland calls but the other thing that it solves is the potential JIT issues that are coming with PHP eight, where the optimizations that it does could potentially optimize out a call to zend_execute_ex where a profiling or APM tracing kind of extension would not be able to enter set that call, because of the JIT. The Observer API enables to solve multiple issues with this. Not only that, there's more. there's more features to this thing, because zend_execute_ex by default will intercept all userland function calls, and you have no choice but to intercept every single call, whereas, this API is designed to also allow you to choose which function calls specifically you want to intercept, so there is on the very first call of a function call. And it'll basically send in the zend function. This is a little bit of a point we've been kind of going back and forth on what we actually send in on the initialisation. But at the moment it is a zend function so you can kind of look at that and say okay do I want to monitor this function or observe it. If your extension returns a handler and says this is my begin handler This is my end handler. Those handlers will fire at the beginning and end of every function call. So it gives you a little bit of fine grain sort of resolution on what you want to observe. The other really kind of baked in design, part of the design is, we wanted it to play well with neighbours, because some of the hooks, at the moment, well, pretty much all of the hooks. Aside from typical extension hooks. Whenever you tie into the engine it's very easy to be a noisy neighbor. It's very easy not to forward a hook along properly it's very easy to break something for another extension. This has like kind of neighbour considerations baked right in. So when you actually request a begin and end hook. It will manage on its side, how to actually call those, and so you don't have to forward along the hook to other other extensions to make it a little bit more stable in that sense.

Derick Rethans  4:52  
	From working on Xdebug, this is definitely problem forwarding both zend_execute_ex and also zend_execute_internal, which is something I also override are of course. And I think there are similar issues with, with the error display as well and PHP eight will also have a different, or a new API for that as well. Also coming out of a different performance monitoring tool, which is interesting to see that all these things works. You mentioned the Zend function thing and I'm not sure how well versed the audiences and all this internal things what is this zend function thing?

Levi Morrison  5:24  
	as any function in the engine is what represents a function so not the scope that it's called in but the scope that it's defined in. It represents both method calls and function calls. It's triggered whenever a user land function is in play. So it has the function name, the name of the class that it's associated with, it tells you how many parameters you have and things like this. It does not tell you the final object that it's called with, and this is partly why we are debating what exactly should get passed in here, because some people may care. Oh, I only want to observe this with particular inheritors or, or other things of that nature so there's a little bit of fine tuning in the design perhaps still but the basic idea is you'll know the name of the function. What class it's in, and it's bound late enough in the engine that you would also have access to whatever parents that class has, etc.

Derick Rethans  6:33  
	Does it contain the arguments as well, that are being sent, or just a definition of the arguments?

Levi Morrison  6:38  
	The Zend function only contains the definition of the arguments. The hook is split into three sections kind of so there's like initialisation and then begin and end. Initialisation only gives you the Zendo function but to begin and gives you access to what's called the Zend execute data which has more information, including the actual arguments being passed.

Derick Rethans  7:03  
	Okay, so it's the idea of the initialisation, just to find out whether you actually want to intercept the function. And if you want remember that and if not it wouldn't ever bother are the trying to intercept that specific zend function either.

Sammy Kaye Powers  7:17  
	Actually what we actually pass into that initialization function is has been sort of up for debate. The original implementations, that is plural. We've had many different implementations of this thing over the, over the year. Derick you did mention that this got squeezed in last minute it has been a work in progress for a very long time and it actually is fulfilling JIT work so there's a specific mention in the JIT RFC that that mentions an API that is going to be required to intercept some function calls that are optimized out so that's why we were able to sneak in a little bit past feature freeze on the actual merge I think. But what we actually sent into this initialization function is spin up two for debate based on how we've actually implemented it. One of the original early implementations actually called this initialization function during the runtime cache initialization, just basically kind of a cache that gets initialized before the execute data actually is created. We didn't have the option of sending in the execute data at that time, we did have the zend function. So we were sending that in. Later on this implementation get refactored to a number different ways. We have the option now to send an execute data if we wanted to, but it might be sufficient to send in the op array instead of the Zend function. The op array should be the full sort of set of opcodes that basically is a function definition from the perspective of of internals, but it also includes like includes and evals. Having additional information at initialisation might actually be handy. I think we're still kind of maybe thinking about that potentially changing I don't know what do you think Levi.

Levi Morrison  8:59  
	Yeah, you can get the oparray from the function so it's a little pedantic on which one you pass in I guess, but yeah. The idea is that we don't want to intentionally restrict it. It's just that the implementations have changed over the year so we're not sure exactly what to pass in at the moment. I think a zend function's pretty safe, passing in a zend oparray is perhaps a better signal to what it's actually for, because it can measure function calls, but also include, require, eval. And the oparray technically does contain more information. Again, if you have zend function, you can check to see if it is an oparray and get the operate from the Zend function. So a little pedantic but maybe a little better in conveying the purpose and what exactly it targets.

Derick Rethans  9:56  
	And you can also get the oparray from zend_execute_data. 

Levi Morrison  10:00  
	Yeah. 

Derick Rethans  10:01  
	If I want to make use of this observe API I will do that? I guess, you said only from extensions and not from userland.

Sammy Kaye Powers  10:08  
	Exactly. At the moment you would as an extension during MINIT or startup, basically in the very early process with the actual PHP processes starting up, would basically register your initialization handler. And at that point, under the hood, the whole course of history is changed for PHP at that point, because there is a specific observer path that happens when an observer extension registers at MINIT or startup. At that point the initialization function will get called for every function call. The very first function call that that function called is called, I know that sounds confusing but if you think you have one function and it's called 100 times that initialization will run one time. That point you can return either a begin and or at an end handler. If you return null handlers it'll never, it'll never bother you again for that particular function, but it will continue to go on that is don't mentioned earlier for every new function that encounters every new function call and encounters, I should say.

Derick Rethans  11:12  
	There is not much overhead, because the whole idea is that you want to do as little overhead as possible I suppose.

Levi Morrison  11:19  
	Exactly, we have in our current design in pre PHP 8.0. You could hook into all function calls using that zend_execute_ex, but it has performance overhead just for doing the call. So let's imagine we're in a scenario where we have two extensions, say Xdebug and one APM product. Both of them aren't actually going to do anything on this particular call it will still call those hooks, which has overhead to it. So if nobody is interested in a function, the engine can very quickly determine this and avoid the overhead of doing nothing. This way we only pay significant costs, if there's something to be done.

Derick Rethans  12:09  
	You're talking about not providing much overhead at all. Just having the observer API in place, was there any performance hits with that?

Sammy Kaye Powers  12:17  
	That was actually one of the biggest sort of hurdles that we had to overcome specifically with Dmitri getting this thing, merged in because it does touch the VM and whenever you touch the VM like we're talking like any tiny little null check that you have in any of the handlers is probably going to have some sort of impact at least enough for Dmitri, who understandably cares about like very very very small overheads that are happening at the VM level, because these are happening for every function call. You know, this is, this is not something that's just happening, you know, one time during the request is happening a lot. In order to apeace Dmitri and get this thing merged in, it basically had to have zero overhead for the production version non observer, his production version but on the non observed version on the non observed path it had to basically reach zero on those benchmarks. That was quite a task to try to achieve. We went through four, about four or five different ways of tying into the engine, we got it down to about, like, two new checks for every function call. And that still was not sufficient, so we end up going with based on Dmitris excellent suggestion, went with the opcode specialization, to implement observers so that at compile time. We can look and see if there's an observer extension present and if there is, it will actually divert the function call related opcodes to specific handlers that are designed for observing and that way once, once you get past that point, the observer path is already determined at compile time and all the observer handlers fire. In a non observed environment, all of the regular handlers will fire without any observability checks in them.

Derick Rethans  14:03  
	At the end of getting within the loss of zero or not?

Levi Morrison  14:07  
	It is zero for certain things. Of course, there are other places besides the VM that you have to touch things here and there for, you know, keeping code tidy and code sane but it's effectively zero, for all intents and purposes. Goal achieved. I will say zero percent.

Derick Rethans  14:30  
	I think the last version of the patch that I saw didn't have the code specialization in it yet. So I'm going to have to have a look at myself again.

Levi Morrison  14:39  
	Yeah, the previous version had very low overhead, so low overhead that you couldn't really observe it through any time based things. But if you measured instructions retired or similar things from the CPU, then it was about point four to 1% reduction, and personally I would have said that's good enough because all of them would correctly branch predict, because you either have handlers in a request, or you don't. And so they would perfectly predict, every time. But still, those are extra instructions technically so that's why Dmitri pushed for specialization and those are no longer there.

Derick Rethans  15:27  
	Does that mean there are new opcodes specifically for this, or is it just the specialization of the opcodes that is different?

Sammy Kaye Powers  15:33  
	It's just this specialization. During the process of going, figuring out what exactly Dmitri needed to mergeable actually proposed an implementation that added basically an observer version of every kind of function call related opcode like do_fcall_observed, or observed_return or something like that. With, opcodes specialization, it reduces the amount of code that you have to write sort of at the VM level, it doesn't change the amount of code that's generated though because with opcode specialization, basically the definition file will get expanded by this, this php file that actually generates C code. When you add a specialization, to a handler that already has specializations on it, it will expand quite considerably. The PR at one point ended up being like 10,000 lines or something like that, so we had to do some serious reduction on the number of handlers that were automatically generated. Long story short, is there are no new opcodes but there are new opcode handlers to handle this specific path.

Derick Rethans  16:40  
	Not sure what, if anything more to ask about the Observer API, do you have anything to ask yourself?

Levi Morrison  16:45  
	I think it's worth repeating the merits of the observer API and where we're coming from. The key benefits in my opinion are that it allows you to target per function interception for observing. It allows you to do it in a way that's that plays nice with other people in the ecosystem and increasingly that's becoming more important. We've always had debuggers and some people occasionally need to turn debuggers on in production and other things like this. But increasingly, there are other products in this space; the number of APM products is growing over time. There are new security products that are also using these kinds of hooks. And I expect over time we will see more and more and more of these kinds of of tools, and so being able to play nicely is a very large benefit. At data dog where Sammy and I both work we've hit product incompatibilities a lot of times, and some people are better to work with than others. I know that Xdebug has done some work to be compatible with our product specifically but you know competitors aren't so interested in that. We care a lot about the community right, we want the whole community to have good tools, and I don't think we actually mentioned yet that we did collaborate with some other people and competitors in this space. That hopefully proves that that's not just words of mine that, you know, we actually met with the competitors who were willing to and discussed API design, and use cases, and making sure that we could all work together and compete on giving PHP good products rather than, you know, hoarding technical expertise and running over each other and causing incompatibilities with each other. So I think those are really important things. And then lastly, it does not have that stack overflow potential that the previous hooks you could use did.

Derick Rethans  18:54  
	Yeah, which is still an issue for Xdebug but but I fixed that in a different way by setting an arbitrary limit on the amount of nested levels you can call, right.

Levi Morrison  19:02  
	Yeah, and in practice that tends to work pretty well because most people don't have intentionally deep code. But for some people they do. And we can't as an APM product for instance say: sorry your code is just not good code, we can't observe a crash your your your thing and so we can't make that decision. And then the biggest con at the moment is that it doesn't work with JIT, but I want to specifically mention that, that's not a technical thing, that's just a not enough time has been put in that space yet because this was crunched to the last second trying to get it in. And so, some things didn't get as much focus yet. Hopefully by the time 8.0 gets released it will be compatible with JIT, or at least it will be only per function, so maybe a function that gets observed, maybe that can't be JIT compiled that specific function call, but all the other function calls that aren't observed would be able to. We'll see obviously there's still work to do there but that's our hope.

Derick Rethans  20:10  
	What happens now, if, if you use the observer API and the JIT engine is active? Does it just disable the JIT engine.

Sammy Kaye Powers  20:16  
	Yep. It just won't enable the JIT at all. In fact, it just goes ahead and disables it, if an observer extension is enabled and there is a little kind of debug message that's emitted inside of the OP cache specific logs that will will say specifically why the JIT isn't enabled just in case you're sitting here trying to turn the JIT on you're like, why isn't enabled, and it'll say there's an observer extension present so we can enable the JIT. Hopefully they'll be able to work a little bit, and maybe just change an optimization level or something in the future. I'd like to give a shout out to Benjamin Eberlei, who has been with us since the very beginning on this whole thing has been vetting the API on his products, has gotten xhprof on not only the original implementation but also on the newest implementation, and has just been a huge help in actually getting this thing pushed in, and was said some of the magic words that actually, this thing merged in, when it was looking like it wasn't gonna land for eight dot O and got it landed for eight dot one so Benjamin gets a huge thumbs up. So, Nikita Popov, Bob Weinand, and Joe Watkins really early on. These are awesome people from internals who have spent some time to help us vet the API, but also to help us with specific implementation details. It's been just a huge team effort from a lot of people and it was just like, really great to work across the board with everybody.

Derick Rethans  21:35  
	Yeah, and the only thing right now of course is all the extensions that do observe things need to be compatible with this. 

Sammy Kaye Powers  21:43  
	Exactly. 

Derick Rethans  21:44  
	Which is also means there's work for me, or potentially. 

Sammy Kaye Powers  21:47  
	Absolutely.

Levi Morrison  21:49  
	I guess one one minor point there is that if an extension does move to the new API, it is a little bit insulated from those that haven't moved to the new API. So, to some degree, it still benefits the people who haven't moved yet because the people who have moved have one less competitor in the same same hook, so it's just highlighting the fact that it plays nicely with other people.

Derick Rethans  22:14  
	Is opcache itself actually going to use it or not?

Levi Morrison  22:17  
	So this is focused only on userland functions; past iterations that was not the case. Dmitri kind of pushed back on having this API for internals and so that got dropped. I don't think at this stage there's any there's any value in opcache using it specifically, but there are some other built in things like Dtrace. I don't know how many people actually use Dtrace; I actually have used it once or twice, but Dtrace could use this hook in the future instead of having a custom code path and things like that.

Derick Rethans  22:49  
	For Xdebug I  still need to support PHP seven two and up, so I'm not sure how much work it is doing it right now, but definitely something to look into and move to in the future, I suppose. Well thank you very much to have a chat with me this morning. I can see that for Sammy the sun has now come up and I can see his face. Thanks for talking to me this morning.

Sammy Kaye Powers  23:10  
	Thanks so much, Derick and I really appreciate all the hard work you put into this because I know firsthand experience how much work podcasts are so I really appreciate the determination to continue putting out episodes. It's a huge amount of work so thanks for being consistent.

Levi Morrison  23:26  
	Yeah, thank you so much for having us Derick.

Derick Rethans  23:30  
	Thanks for listening to this installment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patroen. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.


Show Notes
----------

- Pull Request: https://github.com/php/php-src/pull/5857

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
