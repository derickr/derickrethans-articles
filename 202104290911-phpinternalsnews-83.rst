PHP Internals News: Episode 83: Deprecate implicit non-integer-compatible float to int conversions
==================================================================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-04-29 09:11 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin083
   :Enclosure: media/pin-083.mp3

In this episode of "PHP Internals News" I talk with George Peter Banyard
(`Website
<https://gpb.moe>`_, `Twitter
<https://twitter.com/Girgias>`_, `GitHub <https://github.com/Girgias>`_,
`GitLab <https://gitlab.com/Girgias>`_)
about the "Deprecate implicit non-integer-compatible float to int conversions"
RFC that he has proposed.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-083.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:14  
	Hi, I'm Derick. Welcome to PHP internals news, a podcast dedicated to explaining the latest developments in the PHP language. This is episode 83. Today I'm talking with George Peter Banyard, about another tidying up RFC, George, would you please introduce yourself?

George Peter Banyard  0:31  
	Hello, my name is George and I work on PHP in my free time.

Derick Rethans  0:35  
	Excellent. I was just talking to Larry Garfield, and he was wondering whether you or himself, are the second often guests on this podcast, but I haven't run a stats. But it's good to have you on again. Following on for from other numeric RFCs, so to speak. This one is titled deprecate implicit non integer compatible floats to int conversions. That is a lovely small title you have come up with. 

George Peter Banyard  1:01  
	Yeah, not the best title.

Derick Rethans  1:03  
	What is the problem that this RFC is trying to solve, or rather, what's the change that is in this RFC is trying to solve?

George Peter Banyard  1:11  
	Currently in PHP, which is a dynamic language, types are not known at the statically at compile time, so it's so everything's mostly runtime. And most type conversions are relatively sane now in PHP 8, because like numeric strings have been kind of fixed. But one last standing issue is that floats will pass an integer check, without any notices or warnings. Although floats, don't usually fit in integer will have like extra data which can't be represented as an integer. For example, they can have a fractional part, or they can be infinity, or not a number if you divide, like infinity by infinity, or 0 over 0 or other things like that.

Derick Rethans  1:55  
	These are specific features of floating point numbers on computers?

George Peter Banyard  1:59  
	Yes.

Derick Rethans  2:00  
	Is there any prior work that is RFC is building on top of

George Peter Banyard  2:03  
	It builds up on top on the saner numeric string RFC, because it tries to like make the whole numericness of PHP, as a concept better and like less error prone, but in essence it's mostly self contained. If you use a floating point number, were you should be using an integer. If the floating point number, is considered an integer because it only has like decimal zeros, and it fits in the integer range, then you'll have like no error. So if you use 15.0 as an array key, it gets converted to 15, you'll get don't get any error because it's like well it's just 15 like it doesn't mind. But if you do 15.5, then you'll get like a, like a deprecation notice which will tell you it's like, well, here's the key gets implicitly converted, you should be aware of this because if you use 15 somewhere else, you'd be overriding the value.

Derick Rethans  2:54  
	And that currently doesn't happen yet. And you say, fits in the integer range, what ranges are we talking about here?

George Peter Banyard  3:01  
	On 32 bit, which I would imagine most people don't use any more, is a very just like minus 2 billion to 2 billion, because PHP integers are signed, and on 64 bits, it's like nine quintillion?

Derick Rethans  3:15  
	It is a 64 bit range.

George Peter Banyard  3:17  
	You should be fine by not hitting them, but like if you do some maths computation with it, you might hit like the boundaries, or you do like very edge cases where you try to like mess up with PHP, like that so it's something which you can do it.

Derick Rethans  3:30  
	From what I understand floating point numbers they store, integer things as well as fractional things, and from what I remember is that the range that floating point numbers can store things in without losing any precision is something like 53 bits I think. So if it's larger than 53 bits, then it would have to store something in its floating point part of it, and hence starts losing numbers.

George Peter Banyard  3:56  
	Yeah, I'm not the expert on floating point numbers.

Derick Rethans  4:00  
	They are tricky. 

George Peter Banyard  4:01  
	They are tricky and the standard is very confusing at times, especially with like NaNs and like signalling NaNs and like, but the basic concept is like exponential numbers like exponential like scientific notation. You have like your base number, and then you have like a power, and then just gives you like a larger range, but with exponential scientific notation, you also lose like precision, because you don't really care about like the minute numbers.

Derick Rethans  4:24  
	This is why there is a conversion issue both ways right, a floating point number, without fraction or NaN or INF, will always fit in the 64 bit integer. 

George Peter Banyard  4:32  
	Yeah.

Derick Rethans  4:33  
	But although you can represent a 64 bit integer in a floating point number. You can't do that without losing data if the integer takes up more than 53 bits, so the round trip conversion also has issues.

George Peter Banyard  4:44  
	That's why, like, that's the check. I'm using, because initially I was just doing an F mod check, because I was like oh just check for like fractional parts, and then Nikita was like: Well, you probably should do round trip checks. Because you also catch infinity, not a number, which also has like some interesting implications, that like if your floating string is considered infinity and you cast it to an integer, you get max int. If it's a floating point number, you'll get zero, which is an handy thing that needs to be like also dealt with, because I just discovered that while working on that. Trying to already get rid of like conversions, is I think a good first step on making most things sane. And we already do that with string offsets. So it's also just like making it more of a global aspect of the language.

Derick Rethans  5:35  
	So this RFC only talks about converting floats to int, but not int to float?

George Peter Banyard  5:40  
	Yeah, because mostly integer to float is a, is a safe conversion, because you can, it fits usually in a floating point number, except apparently 64 bits.

Derick Rethans  5:52  
	I think it is something that we actually should also look at and this is not something I'd realized because I originally thought that before reading the RFC, this that is what you were trying to get, but it's they other way, the other the way around here; so I can see another upcoming RFC to do the other side of the conversion as well.

George Peter Banyard  6:11  
	I imagine so. I put that on my to do list, which is already growing larger and larger with every small idea. I encountered in the language which I'm like, why on earth is PHP doing that?

Derick Rethans  6:22  
	But to get back to this RFC in which kind of situations can this trip up developers?

George Peter Banyard  6:27  
	I would expect most of the time it shouldn't, because this every time you use integers, floating points is mostly maths code, or, if you're doing something very weird, like storing money as a floating point number, which you shouldn't do, but people do it anyway.

Derick Rethans  6:45  
	Does PHP have an arbitrary precision type?

George Peter Banyard  6:48  
	No we don't. But you can use GMP.

Derick Rethans  6:52  
	I don't know what it stands for either, what GMP stands for. They also used to be BCMath, is that's still around as well?

George Peter Banyard  6:58  
	Yeah, BCMath is still around. Most of the time you don't need arbitrary precision, at least for traditional PHP code which is a web based and possibly like E commerce so you're not hitting like insane numbers, but it is mostly full of direct cases or also with like string conversions to like integers, that's I think like, my main point is to try make also string conversions to then numeric type like to make them safer. I think was the previous RFC was the saner numeric string, there's maybe an expectation that you can finally bypass the strict type mode, because everything is strings in HTTP land. So if you get a value, and you just wanted to let like the engine, take care of like making it, it's a valid number and it doesn't lose precision, and you get an integer. That makes it helpful that you get these warnings and notices that hopefully in PHP nine which is who knows how many years away, we finally can like lock down on these edge case behaviours.

Derick Rethans  7:57  
	The RFC is not making this stop working, but rather it will throw a deprecation notice?

George Peter Banyard  8:04  
	Yes. Currently, yes.

Derick Rethans  8:06  
	Why do you say currently?

George Peter Banyard  8:07  
	The plan is in PHP nine, to make this a type error, which initially, I wanted to make it a warning instead of a deprecation notice, but then people on list were, well, like a warning is too strong, and it doesn't imply anything. And if you want to change this to like a type error you should make it a deprecation notice because it means the behaviour will stop working in the, in the later version. So that's why I changed it to the deprecation notice in the second iteration of the RFC. 

Derick Rethans  8:34  
	Because, I mean, she just said that could potentially impact already existing code. What kind of BC issues are ever this, by introducing this deprecation warning?

George Peter Banyard  8:43  
	There are various operators, that will implicitly convert floating or float strings to integers. So those are like bitwise operators, shift operators, the modulo operator, the assignment operators of the above. If you try to assign a float to an integer type property. If you try to pass a float to a parameter integer type, or as a return type. Those will show deprecation notices. And then only for floats, not float strings, is the bitwise NOT operator, because that one works with strings as well. And if you use a float string it will use the normal string semantics. And then, as an array key because floating strings already so I noticed was as an array key.

Derick Rethans  9:28  
	Do you think it is better to have a deprecation notice than in stead PHP silently truncating data?

George Peter Banyard  9:35  
	Yeah, if you want that behaviour of like implicitly truncating, you can always use an int, cast, which will do the job for you. Which makes the code explicit and tells the intention of the developer, instead of just like, oh I got a float here, pass it to an integer.

Derick Rethans  9:49  
	What's the reaction to this been so far?

George Peter Banyard  9:51  
	Not many reaction on list, but voters currently one weekend, and it's been unanimously approved, so I'm pretty happy that most people are for it.

Derick Rethans  10:02  
	It's always good to hear unanimous agreement, maybe I should switch my vote to No. As you have said the reaction has been fairly good. And obviously this RFC passed, so the reaction was good enough for this to pass. Do you think there will be some follow up RFCs for ironing out more things like this?

George Peter Banyard  10:20  
	Possibly, I don't know if I'll get them into PHP 8.1. Because time, and I've got some other projects. But I think, maybe, to see you, I've just learned that like some integers lose precision as in floating point numbers, which I wasn't aware of. What's maybe a bit more controversial is to change the behaviour of casting floats, which don't fit into an integer range to now produce Max int, or minimum Int, instead of zero. You will need to put like deprecation notices or warnings when you use an explicit cast, which I don't know how people will feel about that. 

Derick Rethans  10:58  
	I see what you mean there. It will be an interesting discussion for when that happens I would say. 

George Peter Banyard  11:04  
	Yep. 

Derick Rethans  11:05  
	Would you have anything else out about is RFC itself? 

George Peter Banyard  11:08  
	Not really it's mostly straightforward. All the details are in the RFC, all the BC breaks are in the RFC. If you're an extension maintainer, there's only one BC break with like a function. When you take Zval and you convert it to an integer, you'll get a notice, which I expect most extension maintainers want their users to know that this is going to like throw at the later point. But you can also then do it manually if you want to support this behaviour implicitly in your extension. 

Derick Rethans  11:36  
	I think it is important that extension, that for extensions be if it doesn't suddenly change, but forcing an API change on them is often a better way than deciding to changing an existing API, I think.

George Peter Banyard  11:47  
	The problem is is the API I'm using is used all over the PHP source code, changing that everywhere, felt a bit like hassle, but I've added like a C function which is long compatible, so you can check in advance if it will also do stuff like that. And then there's also a version which, which doesn't serve any notices so you can do it anyway. 

Derick Rethans  12:08  
	And that is a new function I suppose?

George Peter Banyard  12:10  
	Yes.

Derick Rethans  12:11  
	I think it's something that extension authors should look at in any case, I mean, we have this lovely upgrading dot internals file, where this certainly should fit in as well in that case, I suppose.

George Peter Banyard  12:22  
	Yeah, it'll fit in. It's currently not that big as a file that usually gets big, a bit before feature freeze because all the changes, land then.

Derick Rethans  12:30  
	I know how this goes. This is also exactly the next debug starts breaking again because of API changes. So far I have been lucky there, so there's not been too many in PHP eight one. Do you know actually how much time there is until feature freeze?

George Peter Banyard  12:45  
	I would imagine it's end of July, as usual, that's the usual timeline. I don't know because RM selection hasn't happened yet, so I don't know how long that usually takes.

Derick Rethans  12:54  
	You're talking about RM for release manager selection here. Once this happens all hope to talk to the new release managers as well, and get them to introduce themselves here.

George Peter Banyard  13:02  
	Seems like a good idea.

Derick Rethans  13:03  
	To chats about any favourite things for PHP eight one. All right, George, thank you for taking the time this afternoon to talk to me about another tweak to PHP's handling of numbers in general, and I'm sure it won't be the last one.

George Peter Banyard  13:19  
	Thanks for having me, and I'll talk to you soon.

Derick Rethans  13:22  
	Hopefully in a pub with a pint.

George Peter Banyard  13:24  
	Yeah, that would be nice.

Derick Rethans  13:27  
	Thank you for listening to this instalment of PHP internals news, a podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast as well as the Xdebug debugging tool, you can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening and I'll see you next time.


Show Notes
----------

- RFC: `Deprecate implicit non-integer-compatible float to int conversions <https://wiki.php.net/rfc/implicit-float-int-deprecate>`_
- RFC: `Saner Numeric Strings <https://wiki.php.net/rfc/saner-numeric-strings>`_
- Episode #62: `Saner Numeric Strings <https://phpinternals.news/62>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
