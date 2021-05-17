PHP Internals News: Episode 85: Add IntlDatePatternGenerator
============================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-05-20 09:13 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin085
   :Enclosure: media/pin-085.mp3

In this episode of "PHP Internals News" I discuss the Add
IntlDatePatternGenerator RFC with Mel Dafert
(`GitHub <https://github.com/deltragon>`_).

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-085.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:14  
	Hi I'm Derick, welcome to PHP internals news, the podcast, dedicated to explain the latest developments in the PHP language. This is episode 85. Today I'm talking with Mel Dafert about the "Add Intl Date Pattern Generator RFC" that she's proposing for inclusion into PHP 8.1. Mel would you please introduce yourself?

Mel Dafert  0:35  
	Hello, I am Mel. I've been working professionally with PHP for about three years. Recently I started reading the internals mailing list in my free time, but this is my first time contributing.

Derick Rethans  0:46  
	What made you think starting to read the PHP internals mailing list?

Mel Dafert  0:50  
	I generally like reading mailing lists and issue trackers. And since I work with PHP, it was interesting to read what's, what's happening.

Derick Rethans  1:02  
	That's what I'm trying to read this podcast as well of course; explaining what happens in the PHP development. But let's get to your RFC. What is the problem that you're trying to solve for this?

Mel Dafert  1:14  
	Currently, PHP exposes the ability for locale dependent date formatting with the Intl Date Formatter class. It is basically only three options for the format: long, medium and short. These options are not flexible enough in some cases, however. For example, the most common German format is day dot numerical month, dot long version of the year. However, neither the medium nor the short version provide this, and they use either the long version of the month, or a short version of the year, neither of which were acceptable in my situation.

Derick Rethans  1:47  
	I realize that you basically ran into a problem that PHP wasn't doing something you wanted to do it. But what made you actually wanting to contribute this?

Mel Dafert  1:57  
	I ran into this exact problem at work where I wanted to format dates in this specific way. After some research, I found out that ICU, the library that powers Intl Date Formatter, exposes exactly this functionality already. It would be relatively easy to wire this up into PHP and expose it there as well. I also found in a bug report that other people had this problem as well, so I decided to try my best at hacking at the PHP source and make it available to everyone, using PHP.

Derick Rethans  2:25  
	Had you ever seen a PHP source code before?

Mel Dafert  2:28  
	I don't think so. No. 

Derick Rethans  2:29  
	But you are familiar with C a little bit?

Mel Dafert  2:32  
	On a very basic level, yes. 

Derick Rethans  2:34  
	As part of this RFC What are you trying to suggest to add to PHP?

Mel Dafert  2:39  
	ICU exposes a class called date time pattern generator, which you can pass a locale and so called skeleton and it generates the correct formatting pattern for you. Skeleton just includes which part are supposed to include it, to be included in the pattern, for example the numerical date, numerical month, and the long year, and this will generate exactly the pattern I wanted earlier. It is also a lot more flexible, for example the skeleton can also just consist of the month and the year, which was also not possible so far. I am proposing to add a Intl Date Pattern Generator class to PHP, which can be constructed for locale, and exposes the get best pattern method that generates a pattern from a skeleton for that locale.

Derick Rethans  3:22  
	The skeletons, what do you specify in these skeletons?

Mel Dafert  3:27  
	It's a similar format to the pattern itself. For example, it's lowercase y lowercase y uppercase M uppercase M, would give you only the year and only the month, if I'm correct, that's exactly what the skeleton looks like.

Derick Rethans  3:43  
	But it puts it in the right order?

Mel Dafert  3:45  
	It puts it in in the right order, and in some cases also adds extra characters, or even changes the format slightly, depending on the locale.

Derick Rethans  3:55  
	So it is a bit of a flexible way to tell the Intl extension to format them in a slightly more, well how do you say this, a slightly more intelligent way than what the standard, long, short and medium constants do for you.

Mel Dafert  4:11  
	Exactly. 

Derick Rethans  4:12  
	Why is it so important that you get these formats, right, or rather I should say, how do these locales influence formats and why is this important?

Mel Dafert  4:21  
	There are conventions of how to format dates and times vary rather strongly between languages and country. In Austria, for example, nobody would expect to understand the US format of month slash day last year. I assume people in England may have the same issue.

Derick Rethans  4:38  
	I think everybody has that issue except for people in the US. 

Mel Dafert  4:42  
	But that only shows the importance of using a format that people are used to and understand. Other languages like mainland Chinese even have the words for day and month included in the format, as far as I understand. I don't speak Chinese. 

Derick Rethans  4:59  
	Neither do I, but a long time ago when I, when I added the date time support, not Intl, but PHP standard date time support, I also looked at locales that operating systems have. And even these locales, which is not something that Intl uses now, also encode these extra characters at least for Japanese, so that was interesting to see there as well.

Mel Dafert  5:22  
	There is a lot of sometimes somewhat unexpected formats.

Derick Rethans  5:27  
	And I think German sometimes once the add the in front, and sometimes behind and things like that. I know there's lots of little intricacies, yes. I see that he RFC makes an argument about which name to pick for the new class. Can you elaborate on the two different options that are?

Mel Dafert  5:44  
	Yes, this is certainly for us and what I would call bike shedding. ICU has something of an inconsistency in its naming. The formatting class is called date formatter. And the pattern generator class is called Date Time pattern generator.

Derick Rethans  6:00  
	So it has the extra word time in it?

Mel Dafert  6:03  
	Between some inconsistency with Intl Date Formatter, which already exists in PHP, and the Intl Date Time pattern generator, or if we make sure PHP is internally consistent and omit the time in all cases. So far consensus seems to lean towards the second option. This is also what the Hack people decided to use.

Derick Rethans  6:24  
	And I believe that's the one you are wanting to go with in this RFCs as well, right?

Mel Dafert  6:28  
	Exactly. So far, everybody voted slide, or like express themselves to slightly favour the version without time. So that's the one I'm going with.

Derick Rethans  6:40  
	Of course, as you mentioned, this is a fairly small change to it, but the RFC talks a bit about things to add in the future, because I believe you weren't suggesting to add all of these Intl functionality straightaway. What is this future scope?

Mel Dafert  6:55  
	ICU would also expose more methods around the skeletons, for example, turning a pattern back into its skeleton, or building a list of skeleton and then mapping to the patterns from scratch. That's what you would do in theory if you added your own special locale to this.

Derick Rethans  7:17  
	I'm not sure how to do that with PHP actually, but I think ICU allows you to build your own basically files with settings right?

Mel Dafert  7:25  
	Exactly. This is omitted all of this, for simplicity, and because they couldn't think of a use case for it, personally, at least. If someone does need them, they could easily be added. It would just be a bunch of extra methods on the, on the class.

Derick Rethans  7:43  
	I know that ICU has so much functionality that hasn't been exposed to PHP, because there's just so much of it right?

Mel Dafert  7:50  
	Extremely, yes. I did see that Hack decided to expose all of them, like all the methods that the class has, but I really don't see the use of having to document and test all of these methods when really only one is going to be used. So I've decided to just go for the one that I can actually see people using.

Derick Rethans  8:14  
	And it is always easy to get smaller parts added to PHP than big things, to begin with. 

Mel Dafert  8:21  
	Exactly.

Derick Rethans  8:22  
	How has the reception been so far?

Mel Dafert  8:24  
	I haven't gotten feedback from too many people, but it seems positive so far. A few people that did give some feedback were constructive and seem to seem to like the idea of adding this.

Derick Rethans  8:36  
	I reckon outside of English speaking countries this is quite an important thing to actually support, especially as we just discussed, people are picky about how these things are formatted.

Mel Dafert  8:46  
	Very picky.

Derick Rethans  8:48  
	So the name that you're going for would be Intl Date Pattern Generator, would it also support patterns for the time itself?

Mel Dafert  8:55  
	Of course, just like Intl Date Format also support formatting time.

Derick Rethans  9:02  
	It would be strange if it didn't, to be honest.

Mel Dafert  9:04  
	Yeah.

Derick Rethans  9:05  
	When do you think you're going to put us up for a vote for inclusion to PHP 8.1?

Mel Dafert  9:10  
	I think I sent out the first email about two weeks ago for opening the discussion. So I was planning to send out the heads up, either today or tomorrow, and opening the vote after that.

Derick Rethans  9:23  
	Okay. To be fair, I think there is very little controversy in this one, so it would surprise me if it didn't pass.

Mel Dafert  9:30  
	That's reassuring. I am somewhat anxious about them.

Derick Rethans  9:33  
	It's not controversial, it is an, it is perhaps a niche thing but it is something that is useful, so I can't see people really be opposing to this. To be fair, I think it looks like just an omission from when the Intl extension was written in the first place.

Mel Dafert  9:48  
	That's true. It might have not been supported in ICU at that point.

Derick Rethans  9:54  
	That is a good point as well because I think the Intl extension came with PHP five three, or five four, which I think is now eight years ago or something like that.

Mel Dafert  10:04  
	I think, I think ICU might have not had it at the end. It's an old word, like it's an all supported versions of PHP.

Derick Rethans  10:13  
	That is good to know. Would you have anything else to add?

Mel Dafert  10:16  
	No, I think that's it.

Derick Rethans  10:17  
	Thank you for taking the time today to talk to me about your proposal to add the Intl date pattern generator to PHP 8.1

Mel Dafert  10:25  
	Of course. Thank you for having me.

Derick Rethans  10:29  
	Thank you for listening to this installment of PHP internals news, a podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast as well as the Xdebug debugging tool, you can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening and I'll see you next time.


Show Notes
----------

- RFC: `Add IntlDatePatternGenerator <https://wiki.php.net/rfc/intldatetimepatterngenerator>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
