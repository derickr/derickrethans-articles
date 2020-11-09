PHP Internals News: Episode 70: Explicit Octal Literal
======================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-11-12 09:33 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin070
   :Enclosure: media/pin-070.mp3

In this episode of "PHP Internals News" I talk with George Peter Banyard
(`Website
<https://gpb.moe>`_, `Twitter
<https://twitter.com/Girgias>`_, `GitHub <https://github.com/Girgias>`_,
`GitLab <https://gitlab.com/Girgias>`_)
about an RFC that he has proposed to add an Explicit Octal Literal to PHP.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-070.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:15  
	Hi, I'm Derick, and this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. 

Derick Rethans  0:24  
	This is Episode 70. Today I'm talking with George Peter Banyard, about a new RFC that he's just proposed for PHP 8.1, which is titled explicit octal literal. Hello George, would you please introduce yourself?

George Peter Banyard  0:38  
	Hello Derick, I'm George Peter Banyard, I'm a student at Imperial College London, and I contribute to PHP in my free time.

Derick Rethans  0:46  
	Excellent, and the contribution that you're currently have up is titled: explicit octal literal. What is the problem that this is trying to solve?

George Peter Banyard  0:56  
	Currently in PHP, we have four types of integer literals. So decimal numbers, hexadecimal, binary, and octal. Decimal is just your normal decimal numbers; hexadecimal starts with 0x, and then hexadecimal characters so, null to nine and A to F, and then binary starts with 0b, and then it's only zeros and ones. However, octal notation is just a decimal, something which looks like a decimal number, which was a leading zero, which doesn't really look that much different than a decimal number, but it comes from the days from C and everything which just uses like a zero as a prefix. 

Derick Rethans  1:48  
	But I have seen is people using like array keys for the, for the month names right and they use 01, 02, 03, you get 07, and 08 and 09, and then they look at the arrays. They notice that they actually had the zeroth element in there but no, but no eight or nine. That's something that is that PHP no longer does I believe. No, it's mostly that the parser doesn't pick it up anymore. Instead of silently ignoring the eight, it'll just give you an error. You've mentioned that there's these four types of numbers with octal being the one started with zero. But what's the problem with is that a moment?

George Peter Banyard  2:31  
	Sometimes when you want to use, which looks like decimal number. So, for example, you're trying to order months, and use like the full two digits for the month number, instead of just one, you use 01, as an array key. When you get to array, it will parse error because it can't pass 08 as an octal number, which is very confusing, because it. Most people don't deal with octal numbers that often, and you would expect everything to be decimal. Because numeric strings are always decimal, but not integers literals. So, the proposal is to add an explicit octal notation, which would be 0o. So python does that, JavaScript has it, Rust also has it, to allow like a by more explicit to say oh I'm dealing with an octal number here. This is intended.

Derick Rethans  3:33  
	Beyond having the 0b for binary, and the 0x for hexadecimal, the addition of 0o for octal is the plan to add. And is that it?

George Peter Banyard  3:45  
	That's more or less the proposal. It's non-BC, because the parser before would just parse or if you had 0o, so there's no PC very possible numeric strings are not affected because since PHP 7.0 hexadecimal strings are not handled anymore as numeric strings. Numeric strings will always be decimal integers, literals will have your four different variants, and maybe a future proposal is to deprecate the implicit octal notation to always make a decimal, even if you have leading zeros.

Derick Rethans  4:21  
	At the moment, if I do as a string literal 014, and do an echo that I get 12.

George Peter Banyard  4:27  
	Because then it's interpreted as an octal. The most bizarre example is if you do var_dump string of 014 double equal to 014, you will get false, because one is interpreted as well 14, like the numeric string is interpreted as 14, whereas the octal number, which says 014 as an integer literal is interpreted as an octal number, which is 12, which is slightly confusing for most people, because that also if you because PHP, most, we all deal with like HTTP requests, and I GET and POST a data, which everything is in strings because it's a text protocol. And if you get user output, which is like I don't know, naught 14 and you're, are you intending to compare munz numbers which are or. 01201, and then you get to array, well then you just fail.

Derick Rethans  5:22  
	Of course, removing that support means a BC breaking change, which phones happen until PHP nine, of course, which might be a while away from now let's say that. 

George Peter Banyard  5:31  
	Probably five years, if we're going through the timelines from PHP seven to PHP 8, but to be able to deprecated and remove it. Well, you need to add support for something else. So that's more the long term plan.

Derick Rethans  5:46  
	And your proposal is basically to make it equivalent to binary and hexadecimal numbers, so that it is less confusing in general.

George Peter Banyard  5:55  
	Yes, that's why the RFC is very short.

Derick Rethans  5:58  
	What are octal numbers actually used for?

George Peter Banyard  6:02  
	The only practical use case that I've seen is for Linux permissions, so chmod. Execute read and write, are those who permissions which chmod will use an octal number.

Derick Rethans  6:15  
	In a different order though but

George Peter Banyard  6:17  
	Yes, I don't know chmod though on top of my heart.

Derick Rethans  6:20  
	Is it only Linux permissions that you can think of? Is there anything else? I can't either so I'm asking you.

George Peter Banyard  6:25  
	No, I can't. That's why I find it very odd that like the leading zero just makes it octal instead of anything else. I mean it has precedence because many other languages do that like C, Java, I don't know, many any language I suppose was just picked it up from. I think C. But when I looked into the history, weirdly enough before C. They had a prefix for like binary, octal, and dec, and hexadecimal. But then the one for octal just got dropped, for some reason.

Derick Rethans  6:57  
	Maybe because the zero and the "o" next each other look very the same. We've already touched on whether there are BC breaks or not, BC standing for backwards compatibility. And, there shouldn't be any because it's something that a parser currently doesn't understand. But do other build-in extensions need to be modified for example?

George Peter Banyard  7:18  
	We have two extensions, which one which deals with numbers, so which is GMP, which is arbitrary precision arithmetic. And then there's the filter extension to filter octal, which filters data and tells you if it's valid or not and it gives you back a, like a correct integer or something like that, which is the filter extension, which has an octal filter. Both of these extensions have been modified to support like the prefix notation, and interpreted as a valid octal number. And then we have like the function which is oct2dec, which is basically octal to decimal, which which weirdly enough already supported like the octal prefix.

Derick Rethans  7:59  
	But that accepts strings, I suppose?

George Peter Banyard  8:01  
	Yes that that accepts strings.

Derick Rethans  8:04  
	And it already supported the 0o prefix?

George Peter Banyard  8:07  
	Yes, which is very on point for PHP I feel. Some things are just supported randomly in one side but not everywhere else.

Derick Rethans  8:15  
	It's a surprise for me that is what I can say. So, yeah, you mentioned as a short RFC, you think there will be any extensions to this in the future? You already mentioned having it maybe deprecating the current just zero prefix?

George Peter Banyard  8:31  
	So one other possible future scope is with the prefix to reintroduce octal, binary, and hexadecimal numbers. As with the prefixes as numeric strings. If you type, 0xAABBCC in, and you have that as a string, which could be useful if you get like colorus back from, from a webform, that would be automatically converted into an integer, or not automatically converted if you do like if you compare it to numbers, or if you cast it to an integer, because currently if you get 0x, something and you cast it to an integer, you will get zero. So that way you need to use like a function like hex2dec, or oct2dec, or bin2dec to convert from a string, or to another string and then cast that. Or it may be cast directly to an integer, I'm not exactly sure. But that's also debatable if it's something we want to add.

Derick Rethans  9:37  
	Is it actually possible to do, for example with hexadecimal numbers, do like if you have inside a string. Can you do \xAA, does that actually work? 

George Peter Banyard  9:48  
	I didn't think so.

Derick Rethans  9:49  
	That actually works. You can do var_dump("\x6A") and it gives you the letter J.

George Peter Banyard  9:55  
	The more, you know. 

Derick Rethans  9:56  
	But it doesn't work for binary, or octal. Only for hexadecimal with \x. So I guess that's something that could be added to string interpolation at some point.

George Peter Banyard  10:07  
	PHP is so weird sometimes.

Derick Rethans  10:10  
	Yes, I mean PHP does things in its own way, however, making this kind of small changes to it, just end up improving the language step by step and that is of course the way forward. Right. 

George Peter Banyard  10:23  
	Yeah. 

Derick Rethans  10:25  
	And I'm looking forward to more of these small incremental changes in the future as well.

George Peter Banyard  10:30  
	Seems like a good plan.

Derick Rethans  10:32  
	Are you planning any more?

George Peter Banyard  10:34  
	Well, so I went through some of the old RFCs, most notably the one about when the whole scalar type thing was going on. We had like strict types and then we had like the coercive types. One which was by Dmitri, Zeev, pretty sure Stas, and um forgot, forgetting somebody else. But some of them, some of the ideas they had, which was making some of the type juggling more strict, so float to integer conversions. Currently, even if the floating number has like decimal part, it will just truncate it to an integer, and it won't emit any warning and it will just like pass without any issue, I think that may be is kind of unexpected. I made the other warning to that to possibly make it a type error in the future.

Derick Rethans  11:24  
	You mean upon a cast?

George Peter Banyard  11:26  
	If you've type hint function as accepting only integers, so if you say foo(int $bar), and you pass it the float. And you would like in normal mode, it will truncate, and it will just pass an integer.

Derick Rethans  11:40  
	Because it's just typed.

George Peter Banyard  11:42  
	Yes, and we've had multiple reports of people being very confused about why it's just truncating the numbers, because it's not even rounding up. If you had like if you have like 0.9 it won't round up to one it will just truncate to zero, which a lot of people are confused by.

Derick Rethans  11:58  
	In strict mode doesn't do that?

George Peter Banyard  11:59  
	Yeah, because strict mode is very strict and will only allow you to pass explicitly what's been what you've requested, with the exception of the normal integer to float conversion which is lossless.

Derick Rethans  12:12  
	That's lossless up to a certain point yes.

George Peter Banyard  12:14  
	To a certain point like your integer doesn't fit, then it goes overflow to a float.

Derick Rethans  12:19  
	All right. George thank you very much for taking your time this afternoon to talk to me.

George Peter Banyard  12:23  
	Thank you for having me.

Derick Rethans  12:26  
	Thanks for listening to this installment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.


Show Notes
----------

- RFC: `Explicit octal integer literal notation <https://wiki.php.net/rfc/explicit_octal_notation>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
