PHP Internals News: Episode 87: Deprecating Ticks
=================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-06-03 09:15 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin087
   :Enclosure: media/pin-087.mp3

In this episode of "PHP Internals News" I chat with Nikita Popov (`Twitter
<https://twitter.com/nikita_ppv>`_, `GitHub <https://github.com/nikic/>`_,
`Website <https://nikic.github.io/>`_) about the "Deprecating Ticks" RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-087.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:14  
	Hi I'm Derick, welcome to PHP internals news, a podcast dedicated to explaining the latest developments in the PHP language. This is episode 87. Today I'm talking with Nikita Popov about a much smaller RFC this time: Deprecating Ticks. Nikita, would you please introduce yourself.

Nikita Popov  0:34  
	Hi Derick, I'm Nikita, and I'm working on PHP core development on behalf of JetBrains.

Derick Rethans  0:40  
	Let's jump straight into what this RFC is about, and that's the word ticks. What are ticks?

Nikita Popov  0:46  
	Ticks are a declare directive,. You write declare ticks equals one at the top of your file, and then PHP we'll call a tick function after every statement execution. Or if you write ticks equals two, then as we'll call it the function after every two statement executions.

Derick Rethans  1:05  
	Do you have to specify which function that calls?

Nikita Popov  1:08  
	Of course, so there is also a register tick function and unregister tick function and that's how you specify the function that should be called rather the functions.

Derick Rethans  1:17  
	How does this work, historically, because the RFC talks about the change being made in PHP seven?

Nikita Popov  1:22  
	Technically ticks work by introducing an opcode after every statement that calls the tick function depending on current count. The difference that was introduced in PHP seven is to what the tick declaration applies. The way PHP language semantics are supposed to work, is that declare directives are always local. The same way that strict types, only applies to a single file, ticks should also only apply to a single file. Prior to PHP seven, it didn't work out way. So if you had declare ticks, somewhere in your file, it would just enable ticks from that point forward. If you included the different file or even if the autoloader was triggered and included a different file that one would also make use of ticks. That was fixed in PHP seven, so now it is actually file local, but that also means that the ticks functionality at that point behaviour became, like, not very useful. Because usually if you want to use tics you actually want them to apply it to your whole codebase. There are ways around that. I'm afraid to say that people have approached me after this RFC and told me that they actually do that. The way around that is to register a stream wrapper. It's possible in PHP to unregister the file stream wrapper and register your own one, and then it's possible to intercept all the file includes and rewrite the file contents to include the declare ticks at the top of the file. I do use that general mechanism for real things in other places, but apparently people actually use that to like instrument, a whole application with ticks, and essentially restore the behaviour we had in PHP 5.

Derick Rethans  3:03  
	What was the intended use case for ticks to begin with?

Nikita Popov  3:07  
	Well I'm not sure what was the intended use case, but at least it was the main use case, and that's signal handling. In the PCNTL extension allows you to register a signal handler, and when the signal arrives, we can't just directly call that signal handler, because signals are only allowed to call functions without that our async signal safe. Which excludes things like memory allocation, and a lot of other things that PHP uses. What we do instead is we only set the flag that okay signal has arrived and then we have to actually run the signal handler at some later point in time. In PHP five, that worked using ticks. You declare ticks, and the PCNTL extension registered the tick handler, and then after this flag was set, it would execute your callback on the next tick. In PHP seven, an attentive mechanism was introduced, that is based on virtual machine interrupts. Those were originally introduced for time-out handling, because there we have a similar problem, that when timeout arrives, we might be in some kind of inconsistent state, like the middle of the allocator right now, and if we just bail out at that point, we are likely to see crashes down the road. So that was a significant problem in PHP five. PHP seven changed that. We now set an interrupt flag on timeout, and then the virtual machine checks this flag at certain points. The interrupt flag is not checked after every instruction, but only, like, just often enough to make sure that it's checked, at some point. So that you can't like go in an infinite loop, that ends up never checking. These points are basically function calls, and jumps that go higher up in the function, PCNTL signals can now use the same mechanism. If you call PCNTL async signals true, then those will also set the interrupt flag, and execute the signal handler on the next opportunity. The next time the interrupt flag is checked. The nice thing about that is that it's essentially free. I mean we already, we already have to do these checks for the interrupt like anyway, adding the handling for PCNTL signals doesn't add any cost on top. Unlike ticks, which have to be like executed on every instruction or at least regularly, and that does add significant cost.

Derick Rethans  5:28  
	Execution time itself because it's an opcode that needs to be executed. 

Nikita Popov  5:32  
	Exactly. 

Derick Rethans  5:33  
	So what are you proposing to do but the ticks in PHP eight one then?

Nikita Popov  5:36  
	I want to deprecate that. So both the declared directive itself, and the register tick function, unregister tick function.

Derick Rethans  5:44  
	How could users emulate the same behaviour as ticks allows them to do so now?

Nikita Popov  5:49  
	That's a good question. As I mentioned, if the use case is, use case of ticks was signal handling, then by using async symbols. If it was something else, then you have a problem. My assumption when writing this RFC was basically that signal handling was really the main remaining use case of ticks, because other use cases require this kind of you know stream wrapper instrumentation, and I didn't expect that people will be crazy enough to use something like that in production. 

Derick Rethans  6:21  
	Hopefully they catch these rewritten files?

Nikita Popov  6:23  
	Probably yeah. I think it's possible to make this integrate with opcache. If you use it for other purposes, then, I don't think there is a really good replacement. So I think what they use it for is some kind of well instrumentation, so profiling, memory profiling, for example, and the alternative there of course is to use a tool that is appropriate for that job, for example, Xdebug contains a profiler, but of course it is not a production profiler, but I think there are also production profilers.

Derick Rethans  6:54  
	As far as I know all the production or APM solutions. They do this on their own without having to use sticks. They don't need any user land modifications.

Nikita Popov  7:03  
	Yeah, definitely. All the APM solutions support this, they use internal handlers.

Derick Rethans  7:08  
	Because it's actually removing functionalities that some people use, what's the reaction been to removing this functionality?

Nikita Popov  7:14  
	Well on the mailing list at least positive, but as I mentioned at least some people have like pointed out on the pull request that they are using the functionality.

Derick Rethans  7:23  
	Enough in such a way to sway for not deprecating them? What is the benefits of getting rid of ticks, if you don't use them?

Nikita Popov  7:31  
	That's, I think the thing, that there is not really a big benefit to getting rid of them. Like they don't add a lot of technical complexity to the engine. They're pretty simple in that sense. I haven't seen those responses. I'm kind of rolling a bit unsure if we should really remove them, because you could argue that well they don't really hurt anyone. I do have to say that I think all the things that people use sticks for, all the cases I have heard about, and all of those cases ticks are not the right way to solve the problem. They are not the right way to solve the signal handler problem, they are not the right way to solve the profiling problem. And the other one I heard is also they're not the right way to solve the heartbeat problem, to make sure a service stays connected. While people do use them I think they use them for questionable purposes.

Derick Rethans  8:24  
	Developers, if they're using something to rewrite the PHP file to introduce ticks, they can also technically rewrite a file to introduce calls to their own functions, after every statement.

Nikita Popov  8:34  
	Yes, I actually have a very nice PHP fuzzing project that rewrites PHP files to introduce instrumentation functions at certain points. That needs a lot more control than ticks, because it's interested in branching statements in particular. That is definitely also possible, but it's kind of even more crazy than just adding ticks. If you're doing it like this, I think, if we want to keep ticks, then we should change ticks from a declare directive to a ini_set, because this kind of rewriting of files to introduce takes that's like not a great solution. On the other hand, that does mean that if you are, I don't know a library, implementing some code and expecting that, you know, it just runs normally, then someone can with by enabling an ini setting will suddenly run code in the middle of your library file that's like essentially any point. So enabling ticks us a major behaviour change, that's something we really don't like to have in ini settings which is I guess also, why does it declare in the first place, because that limits the scope. And you have to go out of your way if you want to not limit it using this rewriting hack. So I'm not really sure ultimately what to do here.

Derick Rethans  9:44  
	Are you thinking of bringing this up for vote before PHP eight dot one's feature freeze?

Nikita Popov  9:49  
	If I decide to go for it, then definitely before. I'm just not completely sure on this topic yet.

Derick Rethans  9:55  
	it'd be interesting to, to hear what other people think about removing this. I have no opinion about this. Other features I do but in this case, I'm happy with them being there, I'm happy with them not being there, because it's something I'm using myself. In any case, thank you for going through this RFC with me today, and we'll see what happens. 

Nikita Popov  10:14  
	Thanks for having me, Derick. 

Derick Rethans  10:18  
	Thank you for listening to this installment of PHP internals news, a podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug and debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening and I'll see you next time.

Show Notes
----------

- RFC: `Deprecating Ticks <https://wiki.php.net/rfc/deprecate_ticks>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
