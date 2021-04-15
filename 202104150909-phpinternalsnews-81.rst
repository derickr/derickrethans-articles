PHP Internals News: Episode 81: noreturn type
================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-04-15 09:09 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin081
   :Enclosure: media/pin-081.mp3

In this episode of "PHP Internals News" I chat with Matthew Brown (`Twitter
<https://twitter.com/mattbrowndev>`_) and Ondřej Mirtes (`Twitter
<OndrejMirtes>`_) about the "noreturn type" RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-081.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:15
	Hi I'm Derick. Welcome to PHP internals news, a podcast dedicated to explaining the latest developments in the PHP language. This is episode 81. Today I'm talking with Matt Brown, the author of Psalm and Ondřej Mirtes, the author of PHPStan, about an RFC that I propose to alter the noreturn type. Matt, would you please introduce yourself?

Matthew Brown  0:37
	Hi, I'm Matthew Brown, Matt, I live in New York, I'm from the UK. I work at a company called Vimeo, and I've been working with for the past six years on a static analysis tool called Psalm, which is my primary entry into the PHP world, and I, along with Ondřej authored this noreturn RFC.

Derick Rethans  1:01
	Alright Ondřej, would you please introduce yourself too?

Ondřej Mirtes  1:04
	Okay, I'm Ondřej Mirtes, and I'm from the Czech Republic, and I currently live in Prague or on the suburbs of Prague, and I've been developing software in PHP for about 15 years now. I've also been speaking at international conferences for the past five years before the world was still alright. In 2016, I released PHPStan, open source static analyser focused on finding bugs in PHP code basis. And somehow, I found a way to make a living doing that so now I'm full time open source developer, and also father to two little boys.

Derick Rethans  1:35
	Glad to have you both here. We're talking about something that clearly is going to play together with static analysers. Hence, I found this quite interesting to see to have two competitive projects, or are the competitive, or are the cooperative.

Matthew Brown  1:56
	I think half and half.

Derick Rethans  1:57
	Half and half. Okay.

Ondřej Mirtes  1:59
	Competition is a weird concept in open source where everything is released for free here that

Derick Rethans  2:04
	That's certainly true, but you said you're making your living out of it now so maybe there was something going on that I'm not aware of. In any case, we should probably chat about the RFC itself. What's the reason why you're wanting to add to the noreturn type?

Ondřej Mirtes  2:18
	I'm going to start with a little bit of a detour, because in recent PHP development, it has been a trend to add the abilities to express various types natively, in in the language syntax. These types, always originally appeared in PHP docs for documentation reasons, IDE auto completion, and later were also used, and were being related with static analysis tools. This trend of moving PHP doc types tonight this type started probably with PHP seven that added scalar type hint. PHP 7.1 added void, and nullable type hints, 7.2 added object type, 7.4 added typed properties. And finally, PHP, 8.0 added union types. Right now to PHP community, most likely waits for someone to implement the generics and intersection types, which are also widely adopted in PHP docs, but there's also a noreturn, a little bit more subtle concept, that would also benefit from being in the language. It marks functions and methods that always throw an exception, or always exit, or enter an infinite loop. Calling such function or method guarantees that nothing will be executed after it. This is useful for static analysis, because we can use it for type inference. I have an example, when you're accepting nullable object as a function parameter, you probably want to eliminate the null value before you can safely call a method on it. So, you will write if $object, three equal signs null, somehow handle this situation, and at the end of the if statement, you will return, or throw an exception. But instead of return, or throw, you might choose to call framework specific or a library specific function, that also always throws or exits the process. This will also tell the user, the IDE, and the static analyser, that below the if statement, the variable can no longer be null. For example, if you ever called mark test skipped in PHP unit, or if you call the abort function in Laravel, you've already used the function that would benefit from being marked with noreturn keyword.

Derick Rethans  4:24
	You mentioned that currently people use the docblock no it @noreturn for that. Why would it be better to have it in the language?

Matthew Brown  4:31
	Jumping off, Ondřej's point. PHP has this has this thing, right, you know things where the doc block, but PHP also, it's a language where developers are used to the language telling them if they did something wrong. So whereas other languages, you might need, like for example, JavaScript, they can be a bit more permissive. Developers when they write PHP code, they're used to getting errors instantly. They call a function with an object instead of a string, and expects a string, and it's marked in the signature as expecting a string, when they run that they get an error. And so that's just a kind of way that most PHP developers write cod. With a noreturn type, we sort of thought that there's a benefit to developer, having written a noreturn type, instantly getting an error if they actually do something that returns. So it follows that pattern that PHP has adopted, of, if I do something that violates a type that I've annotate, that I've explicitly added to the function, PHP should error. There's also a useful sort of side effect here, which is that when you add noreturn to a function, it's guaranteed that it will never return the context. If you call it, it will never not return because it will either whenever not throw an exception or exit, because if the noreturn is invalid, if it does actually do something where it's returning somehow, PHP will then throw a Type error. Cause it's supported by the language. If it wasn't supported by the language, you'd be able to use a function that called noreturn, and it wouldn't actually return. I mean obviously Ondřej  and I are big fans of static analysis. The language itself isn't just to pat ourselves on the back and think, you know, we had the right idea when we were doing static analysis, it's because it can help PHP developers write code.

Derick Rethans  6:16
	The void return type can only being used in specific locations, I mean you can't type a property for examples void. Are the similar restrictions on where noreturn can be used?

Ondřej Mirtes  6:27
	Yeah, right now it can be used just as a return type. There might be some other possible usages, but they are not part of this RFC. For example the noreturn bottom type could be used as a parameter type to denote a function that shouldn't be called. So, this might be some relevant use case, but I've already had a feature request for PHP Stan, to actually support this type as a parameter type for callbacks that should never be called, but I don't remember why that person wanted this. Once we have generics, or at least the possibility to type what's in an array, we could also use the no return type for that. For example, array that contains noreturn, or never, would mean that the array is empty. And also during static analysis, the type inference engine also produces this type internally, basically to mark dead code. So for example if you ask better variable that can only ever contain an integer, if that variable can be a string, you're creating a condition that cannot be executed, that will be always false, and the resulting type of the variable inside that condition is the same type as noreturn or never.

Derick Rethans  7:41
	You mentioned never there we haven't spoken about yet, but we'll get back to that in a moment I'm sure. Is there any prior art for this?

Matthew Brown  7:47
	Yes, a number of languages have a noreturn type. Hack has specifically a noreturn type, Hack, if anyone listening doesn't know, hack is a language created as a sort of offshoot of PHP. Engineers at Facebook, when they were running into issues with PHP from about the moment they started using it in 2007/2008 as the site started growing, and performance really became an issue. And so eventually they created their own version, basically. And one of the benefits of working at Facebook is, you have lots and lots of smart engineers, and they added a lot of different typing functionality to this new language. And so one of the things I added was a noreturn type, as well as adding generics and many, many other things. Another language with prior art is type script. TypeScript has a never type, which is essentially the same. It's a bottom type as Ondřej was talking about. And a bottom type is the subtype of all subtypes. You have a class structure, you have exception, and then you have a child class of logic exception, and noreturn, is the subclass of subclasses of the child class, the thing right at the bottom of the type hierarchy, and so it can always be returned when you would expect some other thing. But basically, this is the understanding of what a bottom type is. I talked about interpreted languages to interpreted languages, but also many compiled languages, most recently Rust, that have the notion of a bottom type. It's a type, where you're guaranteed that program execution ends, in some way shape or form.

Derick Rethans  9:23
	You mentioned that noreturn is the bottom type, how does that play with variance that PHP implements?

Matthew Brown  9:32
	The concept of variance for return types is essentially, if a parent method returns something like a, an exception class, the child classes can either return an exception class, or they can return children for that same method of the exception. So let's say I have a method getException, that is described as returning an exception, the child methods in our child class, so child::getException can either return an exception, or they can also say they return a child class, so they can say, I actually return a logical exception, and this is valid according to Liskov substitution principle, which is to say: you're allowed to return a child type of whatever the overridden method was. So where this comes into play with noreturn is, noreturn is defined as is the bottom type is at the very bottom of all those class structures, you can always return a bottom type, basically And this makes sense if we just think about it, you're not breaking a contract, if your function always returns or exits; the variance rule to kind of follow that.

Derick Rethans  10:43
	How would that compare with void? Because void has some interesting variance rules as well right?

Ondřej Mirtes  10:49
	Actually, no or little similarities between void, and noreturn. Because when you are calling a void function or method, you expect it to do something, and then actually continue in the execution flow. Not expect to read the return value, but with noreturn, you call a method, and you don't expect it, the execution flow to continue. These are completely different, and I actually don't know how people can mistake one for the other.

Derick Rethans  11:22
	Yes, seems very, very different to me as well. The RFC talks about alternative ways of introducing noreturn. And one of the things that had mentioned, is using the attribute. Attributes, being introduced in PHP 8.0. Why did you decide not to implement it as an attribute or suggested as an attribute instead?

Matthew Brown  11:43
	Attributes I think are really cool. I think attributes have a place in the language, obviously they have a place as the RFC described, in place a docblocks, they can be reflected very quickly at runtime. And I also I'm interested in ideas like a deprecated attributes. And also I've just been kind of toying around in my head, the idea of a pure attribute, which could guarantee at runtime that a function with that attribute, was pure. It would never, for example, use a property, or it would never use like a static variable. We could guarantee purity of functions which would interest the pure functional programming people

Derick Rethans  12:26
	Could you explain what a pure function is?

Matthew Brown  12:28
	A pure function is a function that doesn't use any other data but the data you provide it. If I have a multiply function that takes two parameters, x and y, and it returns the multiplication of those things, you would call that function pure. There are many ways the function can become impure. One of the ways is it can have output, you can have IO output for example so if the body of the function you then echo the value of x, before returning, that function becomes impure because it's changed the environment that it operated in slightly. Additionally, if you metal memorize the value of x. So let's say you have x and y as inputs, and then in the first line, you take the value of a property elsewhere, and you add x to that value, and then you multiply that result, then that function is also impure, you're using data from outside the function to return this value. So the idea of a pure function is one which essentially can be modelled mathematically, and that's why some kind of purists, like the this idea because it allows things to be modelled mathematically, but more importantly, then it allows those functions to be tested very effectively. Some implements purity, so that you can add a docblock annotation to function and it will tell you that, whether or not the function is pure. This has extra benefits when writing really complex code. So the most complex code that Psalm has, which performs some boring computation, I've added these pure annotations everywhere. And what it does, it forces me to write the code in a way that avoid side effects. The hope is from my end that writing this very complicated code in a pure fashion, makes it easier to debug at some later date.

Derick Rethans  14:20
	Thanks for that.

Matthew Brown  14:21
	I think attributes are great and have these uses. I don't believe that attributes are useful to encode types, because PHP has a place where you can already represent types, you know, we've introduced into the language itself, the notion of typing, you know obviously many years ago. I think there is a benefit to where possible, keeping the types as types. There was a suggestion that noreturn could be an attribute instead, because it in some way it's it's really about behaviour. But it's still a type, and in the wider programming community, there is prior art for it to be considered a type. So there's basically no benefit to my mind so making it an attribute. And as well, the implementation as a type is very small, you know, it's less than, well under 100 lines of actual written PHP to implement this feature because it uses the existing checks that we already use. We also use for other return types, and to make an attribute we kind of take it out and very much expand the implementation. There are two good reasons there to not want to use an attribute.

Ondřej Mirtes  15:31
	There are not very useful combinations. If noreturn was an attribute, then what would you write as a return type. There are not many useful combinations of what it could be.

Derick Rethans  15:44
	And it also can't be used with any kind of variance rules any more.

Matthew Brown  15:48
	Or at least if it were to be used for variance rules then we would have to write that logic. You'd be like why are we writing this logic in this particular way, it wouldn't make sense.

Derick Rethans  15:57
	Because noreturn is a type, and not a behavioural thing. Makes perfect sense.

Matthew Brown  16:03
	But it's both a type and a behaviour. In the same way that when you actually say, this function returns a thing, PHP then does a behavioural or check to make sure that that function always returns. You could argue that every type is essentially a behaviour, because you're saying the behaviour of this function has to return a value.

Derick Rethans  16:21
	Earlier, one of you mentioned instead of noreturn, the never keyword. Is that the only alternative that that was discussed are the further ones?

Ondřej Mirtes  16:32
	Well there's noreturn and never and the RFC is now going through the voting process, so the secondary vote is about the name, and some languages also use nothing. It feels more natural to say that that function never returns, or using the noreturn keyword, then, saying that it returns nothing which blends closer to the void, void keyword

Derick Rethans  16:58
	Earlier you were mentioning that for future scope you wanted to use this new keyword that you're suggested to introduce also in all the locations where perhaps noreturn does not make sense.

Ondřej Mirtes  17:08
	Yes. Also. What no return has going for it is that it's unlikely to be used as a class name somewhere so making it, whereas if key word isn't an issue, but just as you said, it looks like a key word for a single purpose being written in a return type thing that it's quite obvious which one of us two like which keyword, because I like never more. And one reason is that it's a single word and it reads more naturally in the source code, and it's also looks more like a full fledged type and TypeScript, uses the same keyword.

Derick Rethans  17:42
	Why did you put noreturn in the RFC?

Ondřej Mirtes  17:44
	Because Matt likes it more.

Matthew Brown  17:47
	Yeah, I wrote the first draft of the RFC, I got first dibs, but this is a big point of contention with Ondřej and I, and we're almost at the point of not speaking to each other, because I'm on one side and he's on the other. And it looks at the moment like never will succeed. I think the TypeScript thing is a good point. When I wrote the RFC originally, I wasn't thinking that so many PHP developers write TypeScript. I hadn't really factored into my head. And I think, given that it does make more sense that never is used.

Derick Rethans  18:21
	Looking at how recurrent voting is going, never has 32 votes going for it, and no return has 14 votes going for it.

Ondřej Mirtes  18:29
	Just kidding. I can't wait to have a beer with him again, once the world is, is fine again.

Derick Rethans  18:35
	Me as well.

Matthew Brown  18:36
	He can't start inventing new words; like yeah ironically naming is hard right.

Derick Rethans  18:41
	Definitely the case. At the moment it's very clearly looks like that, the new keyword is going to be never, with 40 votes for introducing a keyword to begin with and 10 against, so that looks like a done deal. Would either of you have anything else to add?

Ondřej Mirtes  18:57
	Yeah, Derick, last time I refresh the wiki, I noticed that you haven't voted yet so what is going to be your vote?

Derick Rethans  19:04
	I intend not to vote until I've spoken to the people on the podcast.

Matthew Brown  19:09
	Great, great.

Derick Rethans  19:10
	I will make sure to vote. Having said that, thank you very much for taking the time today to talk to me about this RFC.

Matthew Brown  19:17
	Thank you. It was a pleasure.

Ondřej Mirtes  19:19
	Yeah, I've been following this podcast closer since the beginning, so I'm happy I was able to finally join, and have something to talk about here. Thank you.

Derick Rethans  19:26
	Thank you for listening to this instalment of PHP internals news, a podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening and I'll see you next time.


Show Notes
----------

- RFC: `noreturn keyword <https://wiki.php.net/rfc/noreturn_type>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
