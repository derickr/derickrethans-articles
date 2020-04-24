PHP Internals News: Episode 52: Object Ergonomics
=================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-05-07 09:15 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin052
   :Enclosure: media/pin-052.mp3

In this episode of "PHP Internals News" I talk with George Banyard
(`Website
<https://gpb.moe>`_, `Twitter
<https://twitter.com/Girgias>`_, `GitHub <https://github.com/Girgias>`_,
`GitLab <https://gitlab.com/Girgias>`_)
about an RFC that he has proposed together with Máté Kocsis (`Twitter
<https://twitter.com/kocsismate90>`_, `GitHub <https://github.com/kocsismate>`_,
`LinkedIn <https://www.linkedin.com/in/kocsismate90/>`_)
to make PHP's float to string logic no longer use locales.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-052.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:16
	Hi, I'm Derick. And this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 52. Today I'm talking with George Banyard about an RFC that he's made together with Mate Kocsis. This RFC is titled locale independent floats to string. Hello, George, would you please introduce yourself?

George Banyard  0:39
	Hello, I'm George Peter Banyard. I'm a student at Imperial College and I work on PHP in my free time.

Derick Rethans  0:47
	All right, so we're talking about local independent floats. What is the problem here?

George Banyard  0:52
	Currently when you do a float to string conversion, so all casting or displaying a float, the conversion will depend on like the current local. So instead of always using like the decimal dot separator. For example, if you have like a German or the French locale enabled, it will use like a comma to separate like the decimals.

Derick Rethans  1:14
	Okay, I can understand that that could be a bit confusing. What are these locales exactly?

George Banyard  1:20
	So locales, which are more or less C locales, which PHP exposes to user land is a way how to change a bunch of rules on how string and like stuff gets displayed on the C level. One of the issues with it is that like it's global. For example, if you use like a thread safe API, if you use the thread safe PHP version, then set_locale() is not thread safe, so we'll just like impact other threads where you're using it.

Derick Rethans  1:50
	So a locale is a set of rules to format specific things with floating point numbers being one of them in which situations does the locale influence the display a floating point numbers in every situation in PHP or only in some?

George Banyard  2:06
	Yes, it only impacts like certain aspects, which is quite surprising. So a string cast will affect it the strval() function, vardump(), and debug_zval_dump() will all affect the decimal locator and also printf() with the percentage lowercase F, but that's expected because it's locale aware compared to the capital F modifier.

Derick Rethans  2:32
	But it doesn't, for example, have the same problem in the serialised function or say var_export().

George Banyard  2:37
	Yeah, and json_encode() also doesn't do that. PDO has special code which handles also this so that like all the PDO drivers get like a constant treat like float string, because that could like impact on the databases.

Derick Rethans  2:53
	How is it a problem that with some locales enabled and then uses a comma instead of the decimal point. How can this cause bugs and PHP applications?

George Banyard  3:02
	One trivial example is if you do, you take a float, you convert it, you cast it to string, and then you cast it back to float. If you're on a locale, which is the dot decimal separator, you will get back the original float. However, if you have like locale which com... which changes the decimal separator, like the German one, you'll get a string; you'll get like three dash, three comma 14, and then when you convert it back to float, you will only get three because PHP doesn't recognise the comma as a decimal separator in its string to float conversion and so it will loses the decimal information.

Derick Rethans  3:39
	That doesn't seem particularly very useful as a feature. So my question here is we talked about floating point numbers and, and I think floating point numbers have other issues as well. Not sure whether we want to go into the details of how floating point numbers and computers work, but we can if you want to.

George Banyard  3:56
	The easy way to explain floating points is to use like exponential notation, or to use the scientific exponential notation, which most people will know from engineering or physics, where you usually have like, one significant like the number, like a comma, a couple of numbers, and then you have like an exponent which raises it to usually, so to your power 10 to the something, which then gives you an order of magnitude. Floating points, basically that but in base two.

Derick Rethans  4:26
	Positions have magnitudes attached to them. They're all powers of two.

George Banyard  4:30
	Yeah.

Derick Rethans  4:31
	And of course, when we use numbers an decimal, like pi being a bad example.

George Banyard  4:36
	Once said.

Derick Rethans  4:37
	I was going to say if you divide 10 by three, you get 3.33333 that never ends, right. And I reckon if you have a specific number in decimal like three point 14, then you can't necessarily always exactly represent it in binary.

George Banyard  4:55
	Yeah, one common example would say it's like one 10th which has like a perfect representation in decimal. But like in binary is a never ending repeating sequence. When you try to like display naught point one, like how it's saved in floating point, it's really weird and everything to get like these rounding errors which can propagate.

Derick Rethans  5:15
	And hence you often hear people recommend to never use float for things like monetary values, but then as you said that you sentence that right?

George Banyard  5:23
	Yeah, put everything in integers and work with integers and just like format it afterwards.

Derick Rethans  5:29
	So let's get back to what you and Mate are actually suggesting to change. What are the changes that you want to make through this RFC?

George Banyard  5:36
	The change's more or less to always make the conversion from float to string the same, so locale independent, so it always uses the dot decimal separator, with the exception of printf() was like the F modifier, because that one is, as previously said, locale aware, and it's explicitly said so.

Derick Rethans  5:56
	Doesn't printf also have other floating related format specifiers? I believe there's an E and a G as well. And uppercase F. What is the difference between these?

George Banyard  6:06
	Lowercase F is just floating point printing with locale awareness. Capital F is the same as lowercase, but it's not locale aware. So it always uses the dot decimal separator. Lowercase E is, what I've learned recently also locale aware, and uses the exponential notation, like with a lowercase e. Uppercase E is the same as lowercase E, but instead of having a small like a lowercase e in the printing format, it's a uppercase E, and lowercase G has some complicated rules onto when it decides which format to choose between lowercase F and lowercase E, depending on like how big like the number of significant digits are after the comma, or like the dot. And uppercase G is the same but using uppercase F and uppercase E instead of lowercase E and lowercase F.

Derick Rethans  6:58
	And all of them can be locale dependent then except for uppercase F.

George Banyard  7:02
	Yeah.

Derick Rethans  7:02
	Do you think this is going to impact people's applications, if you change the default of normal casts to be locale independent?

George Banyard  7:10
	I would have expected it to not be that significant. And only that would affect displaying floating point. So if you're like in Germany, instead of like seeing a comma, you would now see a dot, which can be annoying, but I wouldn't imagine is the most, the biggest problem for you like end users. But apparently, people made tooling to work around the locale awareness of it. And so they could maybe break with passing stuff, which I suppose that happens because it's been, PHP's 25 years old. And that behaviour has been there for like ever. So people worked around it or work with it.

Derick Rethans  7:49
	Is this going to be purely a displaying change or something else as well?

George Banyard  7:54
	For example, if you would send like a float to like an API via HTTP, you would usually already need to have like code around to like work around like the locale awareness, or like all by resetting set locale or by using number_format or like sprintf or something like that. Because most other APIs or like you would like contact would expect like the float to use like a decimal point. PHP. If you do the string to float conversion again, which was not a point, then you get only an integer basically.

Derick Rethans  8:27
	Because PHP's parser, strips it out once it stops recognising digits, which is in this case, the comma.

George Banyard  8:33
	Yeah, that would make the code nicer. The main reason why me and Mate like decided to propose this RFC is because like most APIs, and also databases and everything, expect strings to be formatted in like a standard way. Currently, like if you for whatever reason, use a locale, then it's not, but yeah, like apparently people worked around that when they were maybe stripping stuff from like HTML whatever displayed and try to work around it because that got raised in the list quite recently.

Derick Rethans  9:06
	This change does not necessarily remove the ability of using locales for formatting numbers, because PHP still has the lowercase F as format specifier for printf. And sprintf and friends. Does PHP have other ways of rendering numbers according to locales?

George Banyard  9:24
	According to locales? I don't think so. You can format it something like manually, or the number format a class from the Intl extension.

Derick Rethans  9:35
	Yeah, from what I understand, number_format, you have to do it all by yourself. And the intl extension doesn't support the posix or C locales from the operating system, right. It uses its own locale rule set from the Unicode project. The RFC lists some alternative approaches. Would you mind touching a little bit on these as well?

George Banyard  9:58
	One of the alternatives approaches is to deprecate setlocale altogether. Because as a byproduct, this just fixes the issue because you can't define any locale anymore. So, there will always be locale independent. This has been discussed like in back in 2016, mostly because of the non thread safe behaviour. Because it affects global states and everything. But at the time, the conclusion was, because HHVM, like did a patch, making a thread safe, setlocale function was to mimic this patch and like implement it into PHP, which hasn't been done yet. Another one that we thought about was to deprecate kind of the behaviour and like raise a notice, like a deprecation notice, because that would happen like basically on every float to string conversion. The penalty, like the performance penalty, seemed pretty like strong. One other thing we considered was with Mate was to deprecate the current behaviour in some way. However, emitting a deprecation notice on basically every float to string conversion seemed not to be ideal. And just like flood, the log, the log output, and like also bring like a performance penalty because like outputting warnings isn't like most friendly thing to do performance wise.

Derick Rethans  11:21
	What has the feedback been so far?

George Banyard  11:24
	Feedback currently has been that like most people, well, one person because there hasn't been that much feedback.

Derick Rethans  11:30
	There hasn't been that much feedback because you've only just proposed?

George Banyard  11:33
	some of the feedback we got officiates the change However, they have concerns about like the modification of like, in every case for locales without having any upgrade paths. In some sense. It's just, oh, you have the change, and then you need to execute it and see what breaks. We may be currently considering like ways to figure that out, maybe by adding a temporary ini setting which would kind of be like a debug mode, where when you use that it would like emit notices when like this conversion would happen before and they would notice: Oh, this is not happening anymore. You need to like be aware of this change in behaviour

Derick Rethans  12:17
	Did we not used to have E_STRICT for this at some point or E_DEPRECATED?

George Banyard  12:24
	E_DEPRECATED is still a thing. E_STRICT got mostly removed with PHP seven. There've been like a couple of remaining notices which I got rid off or put back to normal E_WARNINGS or E_NOTICES in PHP seven point four. There were like two or three remaining. But yeah, like so that's one way to maybe approach it of like implementing a debug ini setting which would only be used for like dev because then where if you get like warnings and everything, you don't really care about the performance impact. And then in production, you would like disable that and the warnings wouldn't pop up.

Derick Rethans  12:56
	How would that setting be any different from just putting it behind an E_DEPRECATED warning?

George Banyard  13:00
	So with an E_DEPRECATED warning, we would need to show this behaviour, and we would need, and we could only change the behaviour in like PHP nine. Currently if we do that with like debug setting, we could change it with PHP 8.

Derick Rethans  13:13
	That's a bit cheating isn't that?

George Banyard  13:15
	Could say so.

Derick Rethans  13:16
	I'm interested to see how this ends up going. Do you have any timeframe of when you want to put it for a vote?

George Banyard  13:23
	Currently, we've only started this discussion. And I think until we figure it out, if we get like an upgrade pass, or multiple upgrade passes that we could then put into a secondary vote. I wouldn't expect it to go to voting that soon. Maybe end of April would be nice.

Derick Rethans  13:41
	So around the time when this podcast comes out?

George Banyard  13:44
	Ah! For once!

Derick Rethans  13:46
	For once I got my timing right.

George Banyard  13:49
	Yes. Don't you have like the string contain one which just got out.

Derick Rethans  13:53
	Yes.

George Banyard  13:54
	Then that vote close like last week.

Derick Rethans  13:57
	Yeah, it's really tricky because there's so many, so many small now that I can't keep up.

George Banyard  14:02
	Yeah, Mark also did like his debug.

Derick Rethans  14:04
	Yeah. And there's like two or three tiny ones more that I would quite like to talk about. But by the time there's an opening in the schedule, it's pretty much irrelevant. So I'm trying to see whether I can wrap a few of the smaller ones just in one episode because there's the throw expression, the is literal check, and typecasting in array destructuring expressions, and all showed up in the last three days.

George Banyard  14:26
	I suppose people have like, lots of time now. Now, it's a taint checker, basically, like I know, there's been like this paper by Facebook like six or eight years ago, which talks about how they kind of tried to implement in their static analyzer, but like, a static analyzer doesn't need to be something in the engine. That's what I don't really get.

Derick Rethans  14:45
	Thank you, George, for taking the time this afternoon to talk to me about a locale independent float to string RFC.

George Banyard  14:53
	Thanks for having me on the podcast again. Derick.

Derick Rethans  14:55
	You're most welcome. Thanks for listening to this installment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.



Show Notes
----------

- RFC: `Locale-independent float to string cast <https://wiki.php.net/rfc/locale_independent_float_to_string>`_
- `Floating Point Numbers <https://floating-point-gui.de/formats/fp/>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
