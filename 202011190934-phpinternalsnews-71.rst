PHP Internals News: Episode 71: What didn’t make it into PHP 8.0?
=================================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-11-19 09:34 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin071
   :Enclosure: media/pin-071.mp3

In this episode of "PHP Internals News" we're looking back at all the RFCs
that we discussed on this podcast for PHP 7.4, but did not end up making the
cut. In their own words, the RFC authors explain what these features are, with
your host interjecting his own comments on the state of affairs.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-071.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:15
	Hi, I'm Derick, and this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 71. At the end of last year, I collected snippets from episodes about all the features that did not make it into PHP seven dot four, and I'm doing the same this time around. So welcome to this year's 'Which things were proposed to be included into PHP 8.0, but didn't make it. In Episode 41, I spoke with Stephen Wade about his two array RFC, a feature you wanted to add to PHP to scratch an itch. In his own words:

Steven Wade  0:52
	This is a feature that I've, I've kind of wish I would have been in the language for years, and talking with a few people who encouraged. It's kind of like the rule of starting a user group right, if there's not one and you have the desire, then you're the person to do it. A few people encouraged to say well why don't you go out and write it? So I've spent the last two years kind of trying to work up the courage or research it enough or make sure I write the RFC the proper way. And then also actually have the time to commit to writing it, and following up with any of the discussions as well.

Steven Wade  1:20
	I want to introduce a new magic method the as he said the name of the RFC is the double underscore to array. And so the idea is that you can cast an object, if your class implements this method, just like it would toString; if you cast it manually, to array then that method will be called if it's implemented, or as, as I said in the RFC, array functions will can can automatically cast that if you're not using strict types.

Derick Rethans  1:44
	I questioned him on potential negative feedback about the RFC, because it suggested to add a new metric method. He answered:

Steven Wade  1:53
	Beauty of PHP is in its simplicity. And so, adding more and more interfaces, kind of expands class declarations enforcement's, and in my opinion can lead to a lot of clutter. So I think PHP is already very magical, and the precedent has been set to add more magic to it with seven four with the introduction of serialize and unserialize magic methods. And so for me it's just kind of a, it's a tool. I don't think that it's necessarily a bad thing or a good thing it's just another option for the developer to use

Derick Rethans  2:21
	The RFC was not voted on and a feature henceforth did not make it into PHP eight zero.

Derick Rethans  2:27
	Operator overloading is a topic that has come up several times over the last 20 years that PHP has been around as even an extension that implements is in the PECL repository. Jan Bøhmer proposed to include user space based operator overloading for PHP eight dot zero. I asked him about a specific use cases:

Jan Böhmer  2:46
	Higher mathematical objects like complex numbers vectors, something like tensors, maybe something like the string component of Symfony, you can simply concatenate this string object with a normal string using the concat operator and doesn't have to use a function to cause this. Most basically this should behave, similar to a basic string variable or not, like, something completely different.

Derick Rethans  3:16
	For some issues raised during the RFC process and Jan explains to the most notable criticisms.

Jan Böhmer  3:21
	First of all, there are some principles of operator overloading in general. So there's also criticism that it could be used for doing some very weird things with operator overloading.  There was mentioned C++ where the shift left shift operator is used for outputting a string to the console. Or you could do whatever you want inside this handler so if somebody would want to save files, or modify a file in inside an operator overloading wouldn't be possible. It's, in most cases, function will be more clear what it does.

Derick Rethans  4:01
	He also explained his main use case:

Jan Böhmer  4:04
	Operator overloading should, in my opinion, only be used for things that are related to math, or creating custom types that behave similar to build types.

Derick Rethans  4:15
	In the end, the operator overloading RFC was voted on. But ultimately declined, although there was a slim majority for it.

Derick Rethans  4:24
	In Episode 44, I spoke with Máté Kocsis about the right round properties RFC and asked him what the concept behind them was. He explained:

Máté Kocsis  4:33
	Write once properties can only be initialized, but not modified afterwards. So you can either define a default value for them, or assign them a value, but you can't modify them later, so any other attempts to modify, unset, increment, or decrement them would cause an exception to be thrown. Basically this RFC would bring Java's final properties, or C#'s read only properties to PHP. However, contrary to how these languages work, this RFC would allow lazy initialization, it means that these properties don't necessarily have to be initialized until the object construction ends, so you can do that later in the object's lifecycle.

Derick Rethans  5:22
	Write once properties was not the only concept that he had explored before writing this RFC. We discussed these in the same episode:

Máté Kocsis  5:31
	The first one was to follow Java and C# and require all right, once properties to be initialized until the object construction ends, and this is what we talked about before. The counter arguments were that it's not easy to implement in PHP, the approach is unnecessarily strict. The other possibility is to let unlimited writes to these properties, until object construction ends and then do not allow any writes, but positive effect of this solution is that it plays well with bigger class hierarchies, where possibly multiple constructors are involved, but it still has the same problems as the previous approach. And finally the property accessors could be an alternative to write once properties. Although, in my opinion, these two features are not really related to each other, but some say that property accessors could alone, prevent some unintended changes from the outside, and they say that maybe it might be enough. I don't share this sentiment. So, in my opinion, unintended changes can come from the inside, so from the private or protected scope, and it's really easy to circumvent visibility rules in PHP. There are quite some possibilities. That's why it's a good way to protect our invariance.

Derick Rethans  7:02
	In the end this RFC was the client, as it did not wait to two thirds majority required with an even split between the proponents and the opponents.

Derick Rethans  7:11
	Following on from Máté's proposal to add functionality to our object orientation syntax. I spoken Episode 49 with Jakob Givoni on a suggested addition COPA, or in full: contact object property assignments Jakob explains why he was suggesting to add this.

Jakob Givoni  7:28
	As always possible for a long time why PHP didn't have object literals, and I looked into it, and I saw that it was not for lack of trying. Eventually I decided to give it a go with a different approach. The basic problem is simply to be able to construct, populate, and send an object in one single expression in a block, also called inline. It can be like an alternative to an associative array: you give the data, a well defined structure, the signature of the data is all documented in the class.

Derick Rethans  8:01
	Of course people abuse associative arrays for these things at the moment, right. Why are you particularly interested in addressing this deficiency as you see it?

Jakob Givoni  8:11
	Well I think it's a common task. It's something I've been missing as I said inline objects, obviously literals for a long time and I think it's a lot of people have been looking for something like this. And also it seemed like it was an opportunity that seemed to be an fairly simple grasp.

Derick Rethans  8:28
	I also asked them what the main use case for this was.

Jakob Givoni  8:32
	Briefly, as I mentioned, they're data transfer objects, value objects, those simple associative arrays that are sometimes used as argument backs to constructors when you create objects. Some people have given some examples where they would like to use this to dispatch events or commands to some different handlers. And whenever you want to create, populate, and and use the object in one go, COPA should help you.

Derick Rethans  9:04
	COPA did also not make it into PHP eight with the RFC being the client nearly unanimously. The proposals by both Máté and Jakob where meant to improve PHP object syntax by helping out with common tasks. The implementation ideas of what they were trying to accomplish were not particularly lined up. This spurred on Larry Garfield to write a blog post titled: object ergonomics, which are discussed with him in Episode 51. I first asked him why he wrote this article:

Larry Garfield  9:33
	As you said, there's been a lot of discussion around improving PHP's general user experience of working with objects in PHP, where there's definitely room for improvement, no question. And I found a lot of these to be useful in their own right, but also very narrow, and narrow in ways that solve the immediate problem, but could get in the way of solving larger problems later on down the line. I went into this with an attitude of: Okay, we can kind of piecemeal attack certain parts of the problem space, or we can take a step back and look at the big picture and say: All right, here's all of the pain points we have, what can we do that would solve, not just this one pain point, but let us solve multiple pain points with a single change, or these two changes together solve this other pain point as well, or, you know, how can we do this in a way that is not going to interfere with later development that we talked about. We know we want to do, but hasn't been done yet. Are we not paint ourselves into a corner by thinking too narrow.

Derick Rethans  10:40
	The article mentions many different categories and possible solutions. I can't really sum these up in this episode because it would be too long. Although, Larry did not end up proposing RFC based on this article, it can be called responsible for constructor property promotions, which I discussed with Nikita Popov in Episode 53 and Named Arguments which are discussed with Nikita in Episode 59. Both of these made it into PHP 8.zero and cover some of the same functionality that Jakob's COPA RFC covered. I will touch on the new features that did make it into PHP 8.0 in next week's episode. There are two more episodes where discuss features that did not make it into PHP eight zero, but these are still under discussion and hence might make it into next year's PHP eight dot one. In Episode 57, I spoke with Ralph Schindler about his conditional code flow statements RFC. After the introduction, I asked what he specifically was wanting to introduce.

Ralph Schindler  11:36
	This is, you know, it's, it's very closely related to what in computer science is called a guard clause. And I used that phrase lightly when I originally brought it up on the mailing list but it's very close in line to that it's not necessarily exactly that, in terms of the syntax. In terms of like when you speak about it in the PHP code sense, it really is sort of a change in the statement. So putting the return before the if, that's really what it is. So a guard clause, it's important to know what that is is it's a way to interrupt the flow of control

Derick Rethans  12:08
	Syntax proposals are fairly controversial, and I asked Ralph about his opinions of the type of feedback that he received.

Ralph Schindler  12:15
	The smallest changes always get the most feedback, because there's such a wide audience for a change like this.

Derick Rethans  12:23
	The last feature that did not make it into PHP eight zero was property write/set visibility, which I discussed with André Rømcke in Episode 63. I asked him what his RFC was all about:

Derick Rethans  12:34
	What is the main problem that you're wanting to solve with what this RFC proposes?

André Rømcke  12:40
	The high level use case is in order to let people, somehow, define that their property should not be writable. This is many benefits in, when you go API's in order to say that yeah this property should be readable. But I don't want anyone else but myself to write it. And then you have different forms of this, you have either the immutable case where you, ideally would like to only specify that it's only written to in constructor, maybe unset in destructor, maybe dealt with in clone and so on, but besides that, it's not writable. I'm not going into that yet, but I'm kind of, I was at least trying to lay the foundation for it by allowing the visibility or the access rights to be asynchoronus, which I think is a building block from moving forward with immutability, read only, and potentially also accessors but even, but that's a special case.

Derick Rethans  13:39
	At the time of our discussion he already realized that it would be likely postponed to PHP eight dot one as it was close to feature freeze, and the RFC wasn't fully thought out yet. I suspect we'll hear more about it in 2021. With this I would like to conclude this whirlwind tour of things that were proposed but did not make it in. Next week I'll be back with all the stuff that was added to PHP for the PHP eight zero celebrations. Stay tuned.

Derick Rethans  14:09
	Thanks for listening to this installment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.


Show Notes
----------

- Episode `#41 <https://phpinternals.news/63>`_: `__toArray() <https://wiki.php.net/rfc/to-array>`_
- Episode `#42 <https://phpinternals.news/63>`_: `PECL overload <https://github.com/php/pecl-php-operator>`_ 
- Episode `#44 <https://phpinternals.news/63>`_: `Write Once Properties <https://wiki.php.net/rfc/write_once_properties>`_
- Episode `#49 <https://phpinternals.news/63>`_: `COPA <https://wiki.php.net/rfc/compact-object-property-assignment>`_
- Episode `#51 <https://phpinternals.news/63>`_: `Object Ergonomics <https://hive.blog/php/@crell/improving-php-s-object-ergonomics>`_
- Episode `#57 <https://phpinternals.news/63>`_: `Conditional Codeflow Statements <https://wiki.php.net/rfc/conditional_break_continue_return>`_
- Episode `#63 <https://phpinternals.news/63>`_: `Property Write/Set Visibility <https://wiki.php.net/rfc/property_write_visibility>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
