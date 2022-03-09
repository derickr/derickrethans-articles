PHP Internals News: Episode 99: Allow Null and False as Standalone Types
========================================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2022-03-10 09:04 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin099
   :Enclosure: media/pin-099.mp3

In this episode of "PHP Internals News" I talk with George Peter Banyard
(`Website
<https://gpb.moe>`_, `Twitter
<https://twitter.com/Girgias>`_, `GitHub <https://github.com/Girgias>`_,
`GitLab <https://gitlab.com/Girgias>`_)
about the "Allow Null and False as Standalone Types" RFC that he has proposed.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-099.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:15
	Hi, I'm Derick. Welcome to PHP internals news, a podcast dedicated to explain the latest developments in the PHP language. This is episode 99. Today I'm talking with George Peter Banyard, about the Allow null and false at standalone types RFC that he's proposing. Hello, George Peter, would you please introduce yourself?

George Peter Banyard  0:36
	Hello, my name is George Peter Banyard. I work on the PHP language, and I'm an Imperial student in maths in my free time.

Derick Rethans  0:44
	Are you're trying to say you're a student in your free time or contribute to PHP in your free time?

George Peter Banyard  0:49
	I feel like at this time, it's like, both are true at the same time.

Derick Rethans  0:53
	Let's hop into this RFC. It is titled allow null and false as standalone types. What is the problem that it is trying to solve?

George Peter Banyard  1:02
	This is the second iteration of this RFC. So the first one was to just allow null initially, and null is the unit type In type theory parlance of PHP, ie the type which only has one value. So null is a type and a value. And the main issue is that when for leads more with like inhabitants, and like the Liskov substitution principle. If you have like a method, like the parent method, which can be told like either null or an object, and your implementation in a child class always returns null, for various reasons, maybe because it doesn't support this feature, or whatever is out, or... If your child method only returns null, currently, you can't document, that you can't type this properly, you can document it in a doc comment or something like that. But due to how PHP type handling works, you need to specify at least like another type with null in the union. Basically resort to always saying like mimicking the parent signature, when you could be more specific. This was the main use case I initially went into.

Derick Rethans  2:08
	If I understand correctly, you can't just have an inherited method that has hinted as to just return null?

George Peter Banyard  2:14
	Exactly. If you always return null, maybe because you always work or something like that, then you must still declare the return type as like null or exception, which is not a concrete because you say what, like why never fail. And like static analysers, if they can figure it out that you're using a child class, they can't maybe like do some assumptions or work further down that like what you're doing is redundant or things like that. So that's one of the main reasons I initially went with it. And I didn't add false initially, because it was like, well, false, it's not really a type properly. It's, it's what's called a value type. False is one value from the Boolean type. And I was like, Well, okay, we're just going to limit it to like, being the type theory purist, limited to proper types, where null is a proper type, although it's a bit sometimes misunderstood, I feel in the PHP community at large. And then people were like, well, if we add null, then by the only type-ish thing, which you can use in a type declaration, or whatever, which can't be used in a return type on its own, is false. And it's just weird. So why not add it in full. So that was the second thing as to why I added it. Some of PHP internal's functions being terribly designed because they were designed back in the early noughties, return null on success and false on failure, which you can't probably type at the moment. Currently, we need to type them as like Boolean or null, but true can never be returned in this case. And there are some other some other people have reached out to me it's like, well, yeah, but I always return false in this case. Or I also return always true in this case, although true, we have this weird asymmetry that we have false as a value type and not true.

Derick Rethans  3:49
	What was the reason for having false but not true?

George Peter Banyard  3:53
	When the union type RFC got discussed and passed for PHP 8.0, false was added, because a lot of traditional behaviour of PHP internal functions, was to return false on failure, instead of the technically more correct thing would be to return null. Because loads of functions return a false on failure, and saying that like in returns, these types, or a Boolean would be basically lying because you could never have true, false was included in it. With the restrictions that you can only use false as the complement with other types. So you need to do for example, array, or false, you couldn't just use false.

Derick Rethans  4:37
	Would it also mean that you can define a return type of a method that inherited a method that returns a bool, as false?

George Peter Banyard  4:48
	Yes, that would be now possible with the amended proposal. Yeah, which goes back to this weird a symmetry, we're probably. Adding true to make a complete would be a future RFC to do.

Derick Rethans  5:00
	Now, we've talked about return types. But I guess the reverse applies to arguments?

George Peter Banyard  5:06
	Arguments and property types also would, would be allowed to, like declare themselves as like null or false. The usefulness here is way more limited. Because if you declare an argument to be of type null, then basically you can only ever pass a null to it. And then therefore, the type doesn't do anything.

Derick Rethans  5:26
	But in an inherited method, you could then widen it.

George Peter Banyard  5:31
	Yes, exactly. You could always say: Well, this argument exists, it's always null. If you extend like your class or message, then you can add other types. But in theory, you can already do that by adding like an argument at the end of the message, because that's LSP compliant. The case for, and properties of those, because they are typing, they're in like their beads. Kind of debatable why you would do that. But it's just that like, well, if you accept types at one point, just restricting them like somewhere else gets very weird. At this point is more like look at the human review, or like use static analysis for the analyser to tell you like this argument is redundant and just remove it or this property doesn't make any sense. Because if it can only ever be null, why does it even exist in the first place?

Derick Rethans  6:13
	Right now, you can already use false in union types, but why not with null or false?

George Peter Banyard  6:19
	That goes back to the when a union type RFC got introduced. Null got added as a keyword. Before you could only use the question mark, before a type to make the type nullable. If you have a more complex union type, to not use the question mark in front of it. Therefore, the null keyword got added as a proper type. And because the logic was, Well, you shouldn't ever be able to return just null. Because then that function is kind of equivalent to void. Because of that, it was said that like, Well, okay, null and false basically have like kind of the same status is that like, if you just want to use null on its own, you're doing something kind of weird. And if you're returning more than false, like that signature is very strange. I think when that was discussed, nobody knew initially that an actual PHP function within one of the extensions, like in core had such a weird signature. Which mainly, we just started discovering that after this got, like accepted and we could like actually start properly typing the internal functions, and then you discover these weird edge cases where sounds like, that's a bit strange, can't properly document it. We just need to make like a note on the PHP documentation side. And like the type signature kind of lies to you. PHP's type hierarchy is a bit strange, void kind of lives on its own. So if the function is marked as void, it must always like any child inheritance, or whatever needs to be void. And when you type return in the function body, you need to always use return with like a semicolon afterwards, you can't even return null. Although, under the hood, PHP will always return a value when you call a function, even if the function is void, which will be null.

Derick Rethans  7:58
	The RFC also talks about question mark null, what is that supposed to be? Is that null or null?

George Peter Banyard  8:03
	PHP has this limited type redundancy checks at compile time. It will basically check if you're duplicating types. So if you write for example, int or int, even if it's capitalized differently, PHP was like, okay, just specifying twice the same type in this union. This is redundant. And then it will throw a compile error, we're basically saying, maybe you're just doing a mistake, maybe you meant something else. In the same vein, basically the question mark, gets like, translated into like, any seeing pipe null. And so if you write null with a question mark in front of it, it's just saying like, well, you're doing null or null, which is basically redundant. Therefore, you'll get like a compile time error telling you like.

Derick Rethans  8:41
	That seems sensible to me. What's been the feedback so far?

George Peter Banyard  8:45
	The most feedback, I think I've got it when I first proposed it in October. And at the time, it was like, Yeah, this is useful. And it's kind of needed because well, having always more type expressiveness is I think, good in general. But the main feedback at the time was like, Well, why not include false? The other feedback I got was basically, well, for consistency, what shouldn't you also add true? Yes, I do agree with this. I frankly, find it very strange that we landed in this situation where we only have one of these value types, either true and false, or none of them would make more sense to me. But that's expanding the scope. And it's kind of not this is not really concerned with this specific detail. Probably next, another RFC to do, for either myself or somebody else. It's just like propose true as a value type.

Derick Rethans  9:33
	Is the implementation of this RFC complicated?

George Peter Banyard  9:36
	It's very simple. It basically removes checks, because currently in the compile step, which checks for like type signatures, it needs to check that like, Well, are you declaring false or are you declaring null, and these checks get removed, so it makes the code a bit more streamlined. Oh, there's one change in reflection. For backwards compatibility reason, because of the fact of the question mark, any union type which is composed of a only two types, where one of them is null,will get converted in reflection to use the question mark notation, which is kind of a bit weird because it then gets converted into like a name type instead of a union type in reflection. But that's there, because of backwards compatibility reasons. I am breaking this into the more sensible reflection type. If you have a type of null and false, then you'll get a reflection union type instead of a named. From my understanding from reading the reflection code, the intention was always probably in PHP 9, to remove this distinction. So if you get a named type, it's only a single type instead of a possible nullable type. And any nullable types get converted into like a reflection union type when you have like null and the other type. Maybe this is a good test case to see if your code breaks.

Derick Rethans  10:50
	I would probably call that a BC break though.

George Peter Banyard  10:53
	This only happens if you do false union null. You can't use false currently on its own. And I think like, if you get false, as a named argument type, with like a question mark in front of it. Because it would be completely new, and you would never deal with it. It's like, well, this can also break because false can be in the Union type. If your library or the tool supports union types with the reflection thing, it will automatically know how to deal with false because it needs to know how to deal with it. And null. So that was kind of also the logic. It's like, well, okay, like if the tool supports that, which it needs to, then if you put this case into that bracket, it will work. It makes kind of the reflection code a bit more complicated at the moment. The whole fact that we need to juggle between like figuring out like, should we use the old like the backwards compatible thing reflection of like using a name type instead of the Union type, if there's a null and depending on the type, makes a reflection code unwindy and everything. And we have like a special function in C, which is basically just like, okay, which object do I need to create, depending on this type signature?

Derick Rethans  11:53
	When do you think you'll be putting this up for a vote?

George Peter Banyard  11:56
	I suppose I could put it up for vote immediately now. I am planning on maybe putting this on to vote at the end of the week or something like that.

Derick Rethans  12:04
	Well, thank you for taking the time today to talk about this RFC.

George Peter Banyard  12:09
	Thank you for having me on the podcast.

Derick Rethans  12:13
	Thank you for listening to this installment of PHP internals news, a podcast dedicated to demystifying development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next time.



Show Notes
----------

- RFC: `Allow Null and False as Standalone Types <https://wiki.php.net/rfc/null-false-standalone-types>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
