PHP Internals News: Episode 46: str_contains()
==============================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-03-26 09:09 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin046
   :Enclosure: media/pin-046.mp3

In this episode of "PHP Internals News" I chat with Philipp Tanlak
(`GitHub <https://github.com/philippta>`_,
`Xing <https://www.xing.com/profile/Philipp_Tanlak2>`_)
about his ``str_contains()`` RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-046.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:16
	Hi, I'm Derick. And this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 46. Today I'm talking with Phillipp Tanlak, about an RFC that he's made titled str_contains. Phillipp, would you please introduce yourself.

Philipp Tanlak  0:35
	Hey, Derick. My name is Philipp. I'm 25 years old and I live in Germany. I work for an IT service company, which does mainly development and maintenance of IT projects. We specialise in the maintenance of e-commerce website and create enterprise applications.

Derick Rethans  0:52
	How long have you been using PHP for?

Philipp Tanlak  0:54
	I've been using PHP for quite a long time now that might be six years I guess.

Derick Rethans  0:58
	What brought to you creating an RFC?

Philipp Tanlak  1:02
	The main reason I've created this RFC was out of necessity and interest, mainly to scratch my own itch.

Derick Rethans  1:08
	That is how most things make it into PHP in the end isn't it?

Philipp Tanlak  1:11
	Yeah, I guess.

Derick Rethans  1:12
	The RFC is titled str_contains, that tells me something that is about strings and containing things. How do we currently find a string in a string?

Philipp Tanlak  1:22
	The current approach to find the string in a string is to use the strpos() function or the strstr() function. But on Reddit, I found someone also use preg_match which I find kind of interesting.

Derick Rethans  1:35
	There are multiple amount of different methods in use, what are the general problems with these approaches that people have made?

Philipp Tanlak  1:41
	So the current approach which I find is not very intuitive, and mainly because of the return values of these functions. For example, the strpos() returns either the position where the string is found, or a false value if the string is not found, but there has to be a check with a !== operation, and the strstr() function just returns a string. So you have to convert that to a boolean to check if the string is found or not.

Derick Rethans  2:11
	Because with strpos(), if you wouldn't use the === or !== operator. Of course, if it would find it at the first position of the string, it'd be zero position, and it would return false, even though it's sfound it.

Philipp Tanlak  2:26
	Yeah.

Derick Rethans  2:27
	So there's a few different problems with these things. Also, I don't think it's particularly vary intuitive to do because you sort of need to come up with like a whole construct to see whether it's part of a string.

Philipp Tanlak  2:37
	Correct. I don't think it's intuitive for a beginner. So if someone is learning PHP for the first time, then he has to search through the documentation, what are the exact return values for these functions, and has to remember that so I thought, string or str_contains() might be a better fit for that to just return a true or false value.

Derick Rethans  2:58
	We've mentioned str_contains() a few times now, I guess the RFC is producing to add this function. How would this function differ from what PHP already has?

Philipp Tanlak  3:07
	So this function does not differ in a lot of ways. It's basically the same implementation of the strpos() function. But instead of returning the position of the found string, it just simply returns it as a boolean value. So either true or false.

Derick Rethans  3:23
	I can imagine some people will say, well, you can just do this in your own wrapper function, right? Because pretty much what it deos is converting the results from strpos() to a boolean. But you must have a good reason of why to want to add an extra function here.

Philipp Tanlak  3:38
	The reason for this function, and maybe someone might disagree is, mainly a user experience for the developer. So this is just out of necessity which I found, and I've been using this function quite a lot. So I thought this might be a valid add to the PHP language. So I tried to implement it and it got some great reviews. So I thought that wasn't a very bad idea I had.

Derick Rethans  4:04
	Is the RFC suggesting just out a single function: str_contains().

Philipp Tanlak  4:09
	Yes, the RFC is currently adding just a single function, which is the str_contains(). When I first submitted the discussion about this RFC, there were quite a few people asking why is there no case insensitivity or multibyte versions for these, and I did not think of those at first. But in the discussion, it became clear that the multibyte version did not seem to be very necessary because the comparison is going to be byte by byte. Unlike strpos(), the position of the found string is not relevant. So it doesn't matter if there is any difference in encoding.

Derick Rethans  4:47
	I remember in last year, there was another RFC related to strings functions they were the string_starts_with() and a string_ends_with(). Those are two functions and there were also variants for both case insensitivity, ss well as multibyte. Which made eight different functions to be added to pretty much do a single thing. That RFC failed, potentially because there are so many things being added.

Philipp Tanlak  5:11
	Yeah, that was also the main reason, I think the case insensitivity of this function, or the variant of it was not so relevant. So I did not include it into the RFC just because of this case you mentioned. So instead of polluting the global space with more functions, someone suggested to just advance PHP incrementally and add in case sensitivity for this function just if it is necessary.

Derick Rethans  5:37
	This is a common recurring subject. Most of the people I spoke with in the last few episodes are all adding things to PHP bit by bit instead of coming up with big RFCs which I think is a good way of going forwards. When reading the RFC, I had a quick look at which argument the function would accept. PHP of course this weakly typed strings in most of time. Is this str_contains() function handling distinct different from what strpos() does for function arguments.

Philipp Tanlak  6:10
	So the str_contains() function uses the same internal function, which is php_memnstr(), if I recall correctly. It tries to interpret it as a string. And if it's not a string, it either throws a warning or notice, but I've just run some checks and it seems like in the next PHP version, non string values which are passed into the string functions will be interpreted as a string, and if that is not the case, it will throw an error or usually return false.

Derick Rethans  6:43
	So it doesn't do any special magic, and just relies on the PHP tends to do for parsing arguments and weak and strict typing.

Philipp Tanlak  6:51
	Yes, that's correct.

Derick Rethans  6:53
	Most RFCs they come with a patch, as does yours. How did you find it getting started with writing things for PHP instead of using PHP.

Philipp Tanlak  7:02
	So basically, I've looked at the PHP source code in the past, just to see how things are implemented. And I had some basic background in C. So I thought that this was not very hard for me. Most of the functions or things I had to do to include this patch, were already there. So basically, I just copied the strpos() function and remove the, when the string is found, use the position to calculate a new string and just remove that code and return the boolean value from the found position.

Derick Rethans  7:35
	Because it is not a very different function from strpos(), it's just pretty much a different return type. It's a lot easier to do.

Philipp Tanlak  7:44
	Yeah.

Derick Rethans  7:45
	When looking at feedback, what were the main criticisms of this?

Philipp Tanlak  7:48
	The main criticism of this was basically just the variants of these functions. So mainly the multibyte variant or the in case sensitivity. Other than that, the response was very, very nice and, and also very rewarding for me. So I thought I did a good job on this. And many people wanted to have this function in PHP, but either did not have the time to implement it or it was too easy. I'm not sure how that went. But I think the response from the devs and the overall PHP community was very nice.

Derick Rethans  8:23
	The RFC is already in voting, so I'm I'm a bit late to talk about them. Usually I'm and things are still in discussion. And at the moment, it looks like it is passing because the votes are 43 to 6 with another weeks ago, then.

Philipp Tanlak  8:37
	Yeah.

Derick Rethans  8:37
	Do you think this will be your last RFC? Or do you have something else in mind?

Philipp Tanlak  8:41
	At the time of this recording I don't have anything else in mind, but maybe if I find something. Since I'm working with PHP on a daily basis, which I think is worth adding to PHP I might create a new RFC.

Derick Rethans  8:54
	That's how I started and see what happens now. Thank you for taking the time to talk to me today Phillipp, I hope you enjoyed this.

Philipp Tanlak  9:01
	Yeah, thanks for having me Derick.

Derick Rethans  9:05
	Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.

Show Notes
----------

- RFC: `str_contains() <https://wiki.php.net/rfc/str_contains>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
