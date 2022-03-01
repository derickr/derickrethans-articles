PHP Internals News: Episode 98: Deprecating utf8_encode and utf8_decode
=======================================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2022-03-03 09:02 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin098
   :Enclosure: media/pin-098.mp3

In this episode of "PHP Internals News" I chat with Rowan Tommins
(`GitHub <https://github.com/IMSoP>`_, `Website <https://rwec.co.uk>`_,
`Twitter <https://twitter.com/Stavron>`_) about the "Deprecate and Remove
utf8_encode and utf8_decode" RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-098.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:14
	Hi, I'm Derick. Welcome to PHP Internals News, a podcast dedicated to explaining the latest developments in the PHP language. This is episode 98. Today I'm talking with Rowan Tommins about the "Deprecate and remove UTF8_encode and UTF8_decode" RFC that he's proposing. Hi, Rowan, would you please introduce yourself?

Rowan Tommins  0:38
	Hi, I'm Rowan Tommins. I'm a PHP software architect by day and try and contribute back to the community and have been hanging around in the internals mailing list for about 10 years and contributed to make the language better, where I can.

Derick Rethans  0:57
	Excellent. Yeah, that's how I started out as well, many, many more years before that, to be honest. This RFC, what problem is this trying to solve?

Rowan Tommins  1:08
	PHP has these two functions, utf8_encode and utf8_decode, which, in themselves, they're not broken. They do what they are designed to do. But they are very frequently misunderstood. Mostly because of their name. And because Character Encodings in general, are not very well understood. People use them wrong, and end up getting in all sorts of pickles that are worse than if the functions weren't there in first place.

Derick Rethans  1:37
	What are you proposing with the RFC then?

Rowan Tommins  1:39
	Fundamentally, I'm proposing to remove the functions. As of PHP 8.2, there will be a deprecation notice whenever you use them, and then in 9.0, they would be gone forever, and you wouldn't be able to use them by mistake, because they just wouldn't be there.

Derick Rethans  1:56
	I reckon there's going to be a way to actually do what people originally intended to do with it at some point, right?

Rowan Tommins  2:02
	So yeah, there are alternatives to these functions, which are much clearer in what you're doing, and much more flexible in what you can do with them so that they cover the cases that these functions sound like they're going to do, but don't actually do when you understand what they're really doing.

Derick Rethans  2:20
	I think we'll get back to that a little bit later on. You're wanting to deprecate these functions. But what do these functions actually do?

Rowan Tommins  2:27
	What they actually do is convert between a character encoding called Latin-1, ISO 8859-1, and UTF-8. So utf8_encode converts from Latin-1 into UTF-8, utf8_decode does the opposite. And that's all they do. Their names make it sound like they're some kind of fix all the UTF 8 things in my text. But they are actually just these one very specific conversion, which is occasionally useful, but not clear from their names.

Derick Rethans  3:01
	It's certainly how I have seen it used in the past, where people just throw everything and the kitchen sink at it, and expecting it to be valid UTF 8, and then at the end, decode. I mean, the decoding was not even part much of this, right? It's just throw everything at it, and then magically it will all be UTF 8. But I reckon that's not really quite the case. When and how does that go wrong?

Rowan Tommins  3:26
	So what actually ends up happening is, because text doesn't know what encoding it's in. Something that people misunderstand about character encoding is they think it's like, the text is a certain colour, and the computer knows what colour it is. And if you tell the computer to make it a different colour, then it will work. But it's not like that. In the computer, there's just the sequence of binary. And the encoding is how to read that binary as text. And if you tell the computer to read it as Latin 1, it will read it as Latin 1. If you take to convert from Latin 1 to UTF 8, it will assume the input is Latin 1, it will convert to UTF 8 on that basis. If your text actually wasn't Latin 1 in the first place, you're just going to end up with garbage. And some of the worst cases of that is when you already have UTF 8, and then you run utf8_encode on it, because the language doesn't know that you've already got UTF 8, so it tries to read its Latin 1, write it out ass UTF 8 and you get this weird Mojibake. I don't know pronouncing that right.

Derick Rethans  4:27
	I think it's pronounced Mojibake.

Rowan Tommins  4:30
	Mojibake.

Derick Rethans  4:31
	It's a Japanese term, because clearly these things, these issues happened with Japanese text quite a lot because they have a lot more different and difficult characters and encodings as well. With which things often go wrong though?

Rowan Tommins  4:44
	Using an unco on text that's already UTF 8 is obviously a big one. Usually obvious, but occasionally people just getting a muddle with that. The other thing that often happens is confusing with similar encoding. Latin 1 is often mistaken for a different coding windows 1252. To the extent that web pages labelled as Latin 1, web browsers will assume that they're actually in Windows 1252. These PHP functions don't make that assumption. If your text is actually in Windows 1252, and it's been mislabelled Latin 1, you might still think you're doing the right thing. So I've got Latin 1 text, but you haven't. And then the characters that are different, are going to get mangled again. And there's a few other related encodings that often look the same. There are a few other encodings that look the same at a glance that again, will go wrong on any character that's different between the different encodings.

Derick Rethans  5:43
	How could a function tell which encoding a certain text was in?

Rowan Tommins  5:49
	It's tricky. There are libraries out there that try to do it. Some encodings that are sequences of bits that aren't a valid character. So if any of those appear, it's definitely not in that encoding. Unfortunately, a lot of encodings, every pattern of bits has a meaning. It's just not necessarily mean. So you can't look at the string and just tell at a  glance. The only way I've seen that does it effectively, is trying to guess based on what language text it might be in. If your text suddenly has a load of symbols in the middle of sentences, you're probably using the wrong encoding. If it's suddenly got a load of capital letters, in the middle of words, you're probably using the wrong encoding. So you can make guesses like that, that ultimately, there are only ever guesses.

Derick Rethans  6:38
	It's only always going to be a guess, right? You can't really tell for certain what it it is, which I've seen people assume that she can just tell. We have concluded that utf8_encode and decode don't actually do what they say they don't magically encode everything to UTF 8. What if things go wrong? How are errors handled?

Rowan Tommins  6:58
	If you're converting from Latin 1 into UTF 8, there Latin 1 covers all 256 possible eight bit binary strings. Those will correspond directly to a single mapping in Unicode and therefore in UTF 8. So there are no errors as such, when that happens, but it might not be what you want. One of the most notable ones that's different between these encodings is Latin 1 was standardized in 1985, the Euro didn't exist, then. The euro symbol doesn't have an encoding in Latin 1. If you've got a euro sign, you haven't got Latin 1 text, but you might think you've got Latin 1 text, and it will just encode it to what to a control character, which is where the windows 1252 code page puts the euro symbol, it replaces some control characters in Latin 1. One of the reasons why these character encodings are so easily confused is they've all nicely built to being compatible on top of each other. Latin 1 is deliberately an extension of ASCII. Windows 1252 is deliberately an extension of Latin 1, replacing some control characters. UTF 8 is also based on Latin 1, the first section of Unicode is actually the Latin 1, characters UTF 8 will encode and slightly differently so that it can carry on above 256. So in that direction, you can't actually get an error, you could just get a string, that doesn't make sense. Going back the other way. Unicode has, I think, potentially 11 million or something, and actually, at least a million assigned code points. Latin 1 only has 256. So you can't map all those back. And this function, the utf8_decode just replaces any that it can't match with the question mark. Similarly, if the input string isn't valid UTF 8. Again, if you've just misunderstood what strings doing and you haven't actually got a UTF 8 string in the first place, any sequence that doesn't look like valid UTF 8, again, just gets replaced with a question mark. Completely silently you get no warnings in your logs or anything. So you'll just get a few question marks. And problem is, a lot of people are writing text, mostly in English. So it's mostly ASCII. And all of these encodings agree on those first 127 things including all the letters and digits, most of your text will look fine. But if you're using utf8_encode, some of the accented letters will just look a bit funny. If using utf8_decode some of the characters will just turn into question marks. And you might just not notice that for a while until your applications been in production. And now all your strings a messed up.

Derick Rethans  9:48
	And I reckon that there's no way to fix that?

Rowan Tommins  9:52
	No. If you've saved saved the text, particularly with the decode direction. Run utf8_encode wrong, if you're careful and tracked carefully where what you've used, you can retrace your steps back to the original string. But if you've not understood what it was doing in the first place, you might have run it more than once, or put it into a system and then re interpreted it in a different way. And it can sometimes be quite hard to trace back what the original string was. You'll sometimes just have to edit it by hand. And guess that, oh, that's probably any acute because that was the word that was trying to be there. That was probably a curly quote mark that somebody was trying to type and those kinds of things.

Derick Rethans  10:35
	Talking about curly quote marks, I just found out that those are actually are code points in the windows 1252 encoding. Because I just had to edit a document that had these things in there. But the file was set as... this is UTF 8, which was a lie. It was a lie to begin with. We've established that these functions are pretty much destructive to text potentially, as well as not really doing what they say they do: encode every random stuff to UTF 8 or the other way around. I saw any RFC that you've done some research into their usage, didn't bring up anything interesting to talk about?

Rowan Tommins  11:13
	Yes, so there's a few things. So what I downloaded, it was last year, actually, I kind of had to pause on this RFC for real life happened a bit to me. So last year, I downloaded the 1000, I think top packages on Packagist, I'm most popular downloads, and went through all the uses, I could say of these functions. There were a handful that were using them correctly, they were checking that their input was Latin 1, or the output they needed was Latin 1. And using these, there were a few of those that were questionable, where they might have mistaken Latin 1 for Windows 1252. And actually, they were going to mess up any Euro signs or any of those few extra things that Microsoft added over the top of those control characters. There were a few using strftime, which can do translated Date Time strings. Those it turns out that functions been deprecated itself now, that will become a non issue, some people will have to find a different solution to that anyway. One of the odder ones that I've seen, which technically works, but only accidentally is people use it for what I describe as armour, where they've got a system that wants UTF 8 text, often encoding as JSON or something like that, where it needs to be UTF 8, they've got some unknown encoding that's not UTF 8, they encode  to UTF 8, transmitted through the system. And then on the other end, run utf8_decode and they'll get back the string that they put in, because it never errors, there will always be a mapping of any string of bits that this function will give in UTF 8, it just won't be a meaningful string. You could put a JPEG image through utf8_encode, and you will get a string that is valid UTF 8, it's just not going to be very useful UTF 8. It's kind of a bit of a weird way of doing the thing you might do with base 64, or quoted printable encoding or something like that almost something for transport, it technically works. But this probably isn't the function you want to be doing it with. It's not a very useful encoding. And then there were a good number, which just tried throwing all the functions they could. And I kind of I don't want to call out the people with this. I think they were genuine mistakes, they were genuinely trying to solve a problem. But some of them just in hindsight looking at them or kind of hilarious. I think the one that makes me laugh most is the person who raised the StackOverflow question because their CSV file, some of the fields had grown to 32 kilobytes long, because they'd repeatedly run the same string through utf8_encode so many times, that each time it was encoding a single byte to multiple bytes, and then single bytes of that to multiple bytes. And only when it got to 32 kilobytes in one field, did they question whether they were doing the right thing? By which time their text was probably irrevocably lost in whatever other processing they've done on this file.

Derick Rethans  14:22
	Excellent encryption.

Rowan Tommins  14:24
	Yes.

Derick Rethans  14:25
	The RFC talks about a few other approaches to instead of deprecating utf8_encode and decode. What are the things that you look at? And why did you reject them in the end?

Rowan Tommins  14:36
	One of the most obvious things you could do? The biggest problem is the name of the functions. Could you just rename them? The problem with that is you'd have to spend a long time doing it because you want to introduce the new name in one version of PHP, then deprecate in a later later version of PHP, and then finally remove. And then at the end of it, you'd have these very specific functions. We could call them latin1_to_utf8 and utf8_to_latin1. If we were designing those functions, if you put an RFC to, to add those functions to the language, it wouldn't pass. There's they're very why, why would we have these specific functions, and we'd still have this problem of Windows 1252, and other related encodings, like Latin 9, which is the official successor to Latin 1, and also has a few differences amongst it. They still wouldn't solve a lot of people's problems. A lot of the people that actually want Latin 1 are going to need the euro symbol. So they don't probably don't actually use Latin 1 any more. Because I guess Canadian French, and Mexican Spanish, need to probably that in one's probably still a decent encoding for but the Western European languages it was originally designed for, probably everyone's going to want a euro symbol. Changing the name just leaves us with these awkward functions still. You could instead or as well add options to them, you could add a parameter to them that indicated what the source or destination encoding was. That defaulted initially to Latin 1, and then you were forced to add it later. And then at least you'd be spelling out what encoding it was. The problem with that is, the more encodings, you add, there's actually quite a lot of code that would need to then be added to the function, and it will be duplicating functions we've already got.

Derick Rethans  16:31
	Such as?

Rowan Tommins  16:32
	So we've actually in PHP got three functions that can convert between any pair of encodings, including the ones that these functions do. They're all unfortunately in extensions, which are technically optional. Which is something that the way PHP is modular, means that a lot of things that you'd think were kind of just part of the language are technically optional, for one reason or another. But we've got mb_convert_encoding from the mbstring extension. We've got iconv, which uses an external library of the same name.

Derick Rethans  17:09
	Are you sure it just doesn't use a GCC function or the glib functionality in PHP?

Rowan Tommins  17:14
	The iconv function uses whatever iconv is available on the system, and seems to vary quite a lot between systems. Oddly, one online code running tool I tried, doesn't actually recognize 8859-1 as an encoding in the iconv function. I don't know why. Just something about the libraries, that version of PHP was built, built against. The most powerful one we've got but also the least documented is the intl extension, which is built on the ICU library, made by the Unicode Consortium. That has a lot of options around how you handle errors and missing characters and supports a lot of different character sets. Some was completely undocumented, I've tried to write a manual page for it, which will hopefully get merged and put live soon. So at least, there will be some documentation there's a, there's an object that you can use with lots of options. But there's a static method, which just takes a from and to encoding. So that's one option. The mb_convert_encoding is probably the most widely available. And maybe we should be looking at making that MB string, less optional. I don't know what that looks like, because of the way, unless you force people to compile it in a lot of the Linux distros. Distribute every module they can separately, they make optional.

Derick Rethans  18:39
	But they also make it easy for you to install them then.

Rowan Tommins  18:42
	They make it very easy to install. So I don't know how many people actually run PHP with just its minimal set of modules. And how many just install a default set. The default set is a bit vaguely defined, unfortunately. So that's one of the my main hesitation with this removal, that although we've got these alternatives, we've got these three alternatives. They've all got slight problems, and they're all optional.

Derick Rethans  19:08
	But considering that utf8_encode and decode don't actually really do well, they say they do, everybody that had to do character set conversions correctly, would have already been using these functions.

Rowan Tommins  19:23
	Indeed, yes. So I've seen people misuse all of these. Again, people do just generally misunderstand character encoding. MB string does have a function to guess character encoding. As you're saying earlier, people just kind of assume that that will work. A lot of the time, it can't really tell the difference between different character encodings. It can tell you whether a string is valid UTF 8, it can't tell you whether it's Latin 1 or Windows 1252, or any of these others that are single byte encodings.

Derick Rethans  19:52
	I think ICU actually as functionality for guessing an encoding as well, but it will give you back an array of possibilities and perhaps even with a confidence. But it's a long, long time since I've looked at that. So I'll have to revisit it.

Rowan Tommins  20:08
	Yeah, that would at least be a more kind of transparent way of doing it that. And that's I guess what I'm trying to do with removing these, is that if you're forced to specify a pair of encodings, as you do for these other functions, at least hopefully, somewhere in your mind, you're going to be thinking about what encodings you might have, rather than just reaching for the first function you find.

Derick Rethans  20:31
	Yep, exactly. What is the feedback being so far?

Rowan Tommins  20:34
	Generally positive. There hasn't been a lot of a lot of comments. But those that have been have generally been supportive. I liked somebody said: All the times they've seen it used, including when they've used it themselves, it's been a misunderstanding. I'd like to hear more feedback of anyone. Anyone does have quite. The main feedback I have had has been around making sure there are alternatives to recommend to people. So anyone who is using these correctly, or nearly correctly, what we tell them to use instead, how do we make sure that's clear, and clearly documented, and we're recommending the right thing. I'm going to think a bit more about that, whether we should be being more definite in recommending one of these options. Particularly I think iconv does seem to have these odd platform issues. They used to be a fourth option. While I was looking at this, they used to be another library called recode. That one seems to have been discontinued. Some references in the PHP manual still refer to recode as an optional option for doing this. But that's been long since shelved. So MB string has the benefit that it doesn't rely on any third party libraries. It's technically a third party library, but it's shipped with PHP, and I don't think anything other than PHP uses it any more. And there have been a lot of there's been a lot of work on that library recently, particularly somebody called Alex Douward, apologies, if you're listening to this, and I pronounce your surname wrong, has done a lot of great work. I've seen recently improving that extension, making sure the detection algorithm is doing as sensible results as it can and improving the test test coverage of that extension and things like that. So that gives me a bit more confidence in that extension, which initially was one of those PHP reinventing the wheel, it felt a bit like, so probably update the RFC to more explicitly say, that's the number one recommended path.

Derick Rethans  22:27
	And of course, you can link that from the utf8_encode and utf8_decode manual pages as well. Please don't use this instead, do this, right?

Rowan Tommins  22:36
	Yeah. And that's again, where it can be a nice clear drop in replacement, so that people are using it right. Here's exactly what to what to use instead. But hopefully, while they're replacing it, they may be at least think about whether it was doing what they what they were hoping for in the first place.

Derick Rethans  22:55
	When do you think you'll be bringing this up for a vote?

Rowan Tommins  22:59
	Unless I get more feedback, further changes? I'll probably tweak that wording in terms of the recommendation that we'll put to users. Otherwise, probably in the next couple of weeks, unless I hear any more, to see if any last minute criticism comes out the woodwork when people are asked to vote on it.

Derick Rethans  23:18
	Yeah that always happens, right? No comments when there isn't a request for comments. But loads of comments if people are voting on it, and it makes it to Twitter. Okay, Rowan, thank you for taking the time today then to talk about this RFC.

Rowan Tommins  23:32
	Thank you very much for having me.

Derick Rethans  23:39
	Thank you for listening to this installment of PHP internals news, a podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next time.



Show Notes
----------

- RFC: `Deprecate and Remove utf8_encode and utf8_decode <https://wiki.php.net/rfc/remove_utf8_decode_and_utf8_encode>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
