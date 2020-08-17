PHP Internals News: Episode 67: Match Expression
================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-08-20 09:30 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin067
   :Enclosure: media/pin-067.mp3

In this episode of "PHP Internals News" I chat with Derick Rethans (`Twitter
<https://twitter.com/derickr>`_, `GitHub <https://github.com/derickr/>`_,
`Website <https://derickrethans.nl>`_) about the new Match Expression in
PHP 8.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-067.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:15
	Hi, I'm Derick, and this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 67. Today we're going to talk about a match expression. I have asked the author of the match expression RFC, lija Tovilo, whether it wanted to come and speak to me about the match expression, but he declined. As I think it's important that we talk in some depth about all the new features in PHP eight, I decided to interview myself. This is probably going to sound absolutely stupid, but I thought I'd give it a go regardless. So here we go.

Derick Rethans  0:53
	Hi Derick, would you please introduce yourself?

Derick Rethans  0:56
	Hello, I'm Derick and I'm the author of Xdebug I'm also PHP seven four's release manager. I'm also the host of this podcast. I'm also you.

Derick Rethans  1:07
	What a coincidence!

Derick Rethans  1:10
	So what is the problem that is RFC is trying to solve?

Derick Rethans  1:13
	Well, before we talk about the match expression, we really need to talk about switch. Switch is a language construct in PHP that you probably know, allows you to jump to different cases depending on the value. So you have to switch statement: switch parentheses open, variable name, parenthesis close. And then for each of the things that you want to match against your use: case condition, and that condition can be either static value or an expression. But switch has a bunch of different issues that are not always great. So the first thing is that it matches with the equals operator or the equals, equals signs. And this operator as you probably know, will ignore types, causing interesting issues sometimes when you're doing matching with variables that contain strings with cases that contains numbers, or a combination of numbers and strings. So, if you do switch on the string foo. And one of the cases has case zero, and it will still be matched because we put type equal to zero, and that is of course not particularly useful. At the end of every case statement you need to use break, otherwise it falls down to the case that follows. Now sometimes that is something that you want to do, but in many other cases that is something that you don't want to do and you need to always use break. If you forget, then some weird things will happen sometimes. It's not a common thing to use it switch is that we switch on the variable. And then, what you really want to do the result of, depending on which case is being matched, assign a value to a variable and the current way how you need to do that now is case, say case zero result equals string one, break, and you have case two where you don't set return value equals string two and so on and so on, which isn't always a very nice way of doing it because you keep repeating the assignment, all the time. And another but minor issue with switch is that it is okay not to cover every value with a condition. So it's totally okay to have case statements, and then not have a condition for a specific type and switch doesn't require you to add default at the end either, so you can actually have having a condition that would never match any case, and you have no idea that that would happen.

Derick Rethans  3:34
	How's the match expression going to solve all of this stuff?

Derick Rethans  3:37
	The match expression is a new language keyword, but also allows you to switch depending on a condition matching a variable. You have matching this variable against a set of expressions just like you would switch, whereas a few major differences with switch here. So, unlike switch, match returns a value, meaning that you can do: return value equals match, then your variable that you're matching, and the value that gets assigned to this variable is the result of the expression on the right hand side of each condition. So the way how this works is that you do result equals match, parenthesis, opening your variable name parenthesis close curly braces and then you have your expression which can just be zero, or one, or a string, or floating point number, or it can be an expression. For example, a larger than 42. And then this condition is followed by an arrow, so equals, greater than sign, and then another statement. What a statement evaluates to get assigned to the return value of the match keyword. Which means that you don't have to do in every case, you don't have to return value equals value, or return value equals expression. Now the expression itself, the evaluator what evaluates to gets returned to match. So that is one thing but as a whole bunch of other changes. The matching with a match keyword is done based on strict type. So instead of using the equal operator, the two equal signs, match uses the identical operator which is the three equal signs so that is strict comparison. Normal rules for type coercion are suspended, no matter whether you have strict types defined in your script, meaning that the match keyword always takes care of the type. It will not be any type juggling in there that can create confusing results. I've already mentioned that it can also be an expression on both the left and right hand sides. Switch also allows you to have an expression on the left hand side, although that isn't used very much, I guess. Another difference between switch and match is that, in case a switch on your right hand side, you can have multiple statements so you can have case seven, colon, and then a bunch of statements and statements are ended by the break, pretty much. With match you can only have one expression. It's possible that is going to change in the future, but it is at the moment analogous with what you can do with the short arrow functions, the arrow function with the fn keyword. That also only allows one expression on the right hand side. Switch also doesn't do automatic fall through to the next condition. If you have a single statement that doesn't make a lot of sense anyway, but this is one of the other changes here. So just like switch you can add multiple conditions separated by comma, on the left hand side. So you can do case zero comma one arrow, and then your expression again. As I mentioned with switch, it is possible to not cover all the cases like it is possible to say four possible values for a specific variable. Take for example you have addition, subtraction, multiplication and division. If none of the conditions that you have set up with a match matches. Then you'll get a unhandled match error exception, it is still possible to have a default condition just like you have it switched out. But if you don't cover any of the conditions or default, then you will get an exception. Anything that is actually still the same between switch and a new match. Well yes, any implementation each condition is still, even evaluated in order, meaning that if the first condition doesn't match it start evaluating the second condition or the third condition and so on, and so on. Also, just like switch again, if all the conditions are either numeric values or strings, PHP's engine will construct a jump table. That was introduced somewhere in PHP seven three I think, that automatically jumps to the right case. Switch, all the conditions have to be either integer numbers like 01234 or all strings, like foo, like bar, like baz, or so on and so on. The match statement, actually, it's a bit more flexible here because it can construct one jump table, if all the conditions are numbers or strings. It doesn't matter that are all numbers or all strings. If there are all numbers or integer numbers or strings, then it can construct this jump table. That doesn't work with switch because switch's type coercion and match doesn't and the internal implementation already support like this hashmap which is pretty much an array that supports integer array keys as well as associative array keys. But because for match the, the matching happens independent on the type is actually ends up working, so does actually works a little bit better, which is great.

Derick Rethans  8:54
	Where there, any other additions that were considered to add to the new match keyword?

Derick Rethans  8:58
	Well, there were a few things. There was a bit of discussion about blocks, meaning multiple statements to run for each condition. In the end it didn't become part of the RFC, perhaps because it made it a lot more complicated, or perhaps because it was really important to think that functionality through and also at the same time, think about what that does for the short array functions which also, just like match, only support one specific statement at the moment. There were some thoughts about adding pattern matching to the condition just like Perl does a little bit, where yeah like with regular expressions for example, but is also really difficult subject and lots of considerations have to be taken into account so that was also dropped from this RFC. The last one was a quick syntax tweak, which allow you to omit the variable name for a match expression. If you end up matching only against like expressions, like a larger than 42, or b smaller than 12, then it doesn't necessarily matter what you have behind match; the variable name there doesn't matter. So, well the trick that people already use with switch is, is to use switch (true). And with match you can also use match (true), and the addition that was suggested to do here was to be able to not have the true there at all. So, the match would only work on the conditions and not try to match these against a variable, but that also didn't become part of this current RFC.

Derick Rethans  10:31
	Are there any backward compatibility breaks?

Derick Rethans  10:34
	Well beyond match being a new keyword, there are none. But because match is a new keyword that we're introducing, it means it can't be used as a full namespace name, a class name, a function name, or a global constant. It shouldn't really be much of a surprise that PHP just can introduce these keywords, and that ends up breaking some code. I haven't looked at any analysis about how much code is actually going to break, but it is possible that it does actually do some. But also PHP also top level namespace so if you had a class name called match, you should have put it in your own namespace. And in PHP eight with Nikita's namespace token names RFC, as long as match is on its own, then your namespace name, you'd still be able to use it now, as part of a namespace name, which isn't possible or wouldn't have been possible with PHP seven four.

Derick Rethans  11:33
	Okay. What was the reception of this RFC?

Derick Rethans  11:37
	Initially, there was quite a little bit of going back and forth about especially the pattern matching or a few other things. But in the end were to slightly reduce scope of the RFC, it actually ended up passing very well with 43 votes and two votes against, which means it's now part of PHP eight. In the last week or so we did find a few bugs. Some crashes. But, Ilija the author of both RFC and the implementation is working on these to get those fixed, so I'm pretty sure we'll all quite ready for PHP eight with the new match expression.

Derick Rethans  12:12
	Thanks Derick for explaining this new match expression. It was a bit weird to interview myself, but I hope it turned out to be fun enough and not too weird.

Derick Rethans  12:21
	Thanks for having me Derick.

Derick Rethans  12:23
	This is going to be the last episode for a while, as PHP 8's feature freeze is now in effect, and no new RFCs are currently being proposed. Although I'm pretty sure they are being worked on. There's one exception to the feature freeze period, which is the short attribute syntax change RFC, which which I'm collaborating on the Benjamin Eberlei, whether that will turn into yet another episode about attributes, we'll have to see. For the PHP eight celebrations, I'm hoping to make two compilation episodes again, as it did last season with Episode 36 and 37. For the PHP eight celebrations episodes, I am again looking for a few audio snippets from you, the audience. I'm looking for a short introduction, with no commercial messages, please. After your introduction, then state which new PHP eight feature you're looking most forwards to, or perhaps another short anecdote about a new PHP eight feature. Please keep it under five minutes. With your audio snippet, feel free to email me links to your Twitter, blog, etc. The email address is in the closing section of each of the episodes. Here's an example of what I'm looking for.

Derick Rethans  13:35
	Hi, I'm Derick, and I host PHP internals news. I am the author of Xdebug and I'm currently working on Xdebug cloud. My favourite new feature in PHP eight are the additions to the type system, but union types and a mixed type continue to strengthen PHP's typing system. We've grown up from simple type hints to real proper types, following the additional property types and contract covariance and PHP seven four. I'm looking forward to PHP's type system to be even stronger, perhaps, with generics in the future.

Derick Rethans  14:07
	Please record us as a lossless compressed file, preferably FLAC, FLAC, recorded at 44,100 hertz. And if you save them bit of 24 bits that. If you want to make a WAV file or a WAV file that's fine too. Please make them available for me to download on a website somewhere and email me if you have made one.

Derick Rethans  14:33
	Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.


Show Notes
----------

- RFC: `Match expression <https://wiki.php.net/rfc/match_expression_v2>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
