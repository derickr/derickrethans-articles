PHP Internals News: Episode 95: PHP 8.1 Celebrations
====================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-11-25 09:23 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin095
   :Enclosure: media/pin-095.mp3

In this episode of "PHP Internals News" we're looking back at all the RFCs
that we discussed on this podcast for PHP 8.1. In their own words, the RFC
authors explain what these features are, with your host interjecting his own
comments on the state of affairs.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-095.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:14  
	Hi, I'm Derick, and this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language.

Derick Rethans  0:23  
	This is episode 95. I've been absent on the podcast for the last few months due to other commitments. It takes approximately four hours to make each episode. And I can now unfortunately not really justify spending the time to work on it. I have yet to decide whether I will continue with it next year to bring you all the exciting development news for PHP 8.2.

Derick Rethans  0:44  
	However, back to today, PHP eight one is going to be released today, November 25. In this episode, I'll look back at the previous episodes this year to highlight a new features that are being introduced in PHP 8.1. I am not revisiting the proposals that did not end up making it into PHP 8.1 feature two features I will let my original interview speak. I think you will hear Nikita Popov a lot as he's been so prolific, proposing and implementing many of the features of this new release. However, in the first episode of the year, I spoke with Larry about enumerations, which he was proposing together with Ilija Tovilo. I asked him what enumerations are.

Larry Garfield  1:26  
	Enumerations, or enums, are a feature of a lot of programming languages. What they look like varies a lot depending on the language, but the basic concept is creating a type that has a fixed finite set of possible values. The classic example is booleans. Boolean is a type that has two and only two possible values true and false. Enumerations are way to let you define your own types like that, to say this type has two values Sort Ascending or Sort Descending. This type has four values for the four different card suits, and a standard card deck. Or a user can be in one of four states pending, approved, cancelled or active. And so those are the four possible values that this variable type can have. What that looks like varies widely depending on the language. In a language like C or C++, it's just a thin layer on top of integer constants, which means they get compiled away to introduce at compile time, and they don't actually do all that much they're a little bit to help for reading. On the other end of the spectrum, you have languages like rust or Swift, where enumerations are a robust, advanced data type and data construct of their own. That also supports algebraic data types. We'll get into that a bit more later. And is a core part of how a lot of the system actually works in practice, and a lot of other languages are somewhere in the middle. Our goal with this RFC is to give PHP more towards the advanced end of enumerations. Because there are perfectly good use cases for it, so let's not cheap out on it.

Derick Rethans  3:14  
	In the next episode, I spoke with Aaron Piotrowski about another big new feature: fibres.

Aaron Piotrowski  3:20  
	A few other languages already have Fibers like Ruby. And they're sort of similar to threads in that they contain a separate call stack and a separate memory stack. But they differ from threads in that they exist only within a single process and that they have to be switched to cooperatively by that process rather than pre-emptively by the OS like threads. And so the main motivation behind wanting to add this feature is to make asynchronous programming in PHP much easier and eliminate the distinction that usually exists between async code that has these promises and synchronous code that we're all used to.

Derick Rethans  4:03  
	I also asked Aaron about small PHP I actually have a slightly related question that pops into my head as like. There's also something called Swoole PHP, which does something similar but from what I understand actually allows things to run in threats. How would you compare these two frameworks or approaches is probably the better word?

Aaron Piotrowski  4:25  
	Swoole is they try and be the Swiss Army Knife in a lot of ways where they provide tools to do just about everything. And they provide a lot of opinionated API's for things that in this case, I'm trying to provide just the lowest level just the only the very necessary tools that would be required in core to implement Fibers.

Derick Rethans  4:48  
	Although I discussed several deprecations from Nikita and the last year, I only want to focus on the new features. In episode 76. I spoke with him about array unpacking, after talking about changes to Null in internal functions.

Nikita Popov  5:01  
	The old background is set we have unpacking calls. If you have the arguments for the call in an array, then you write the free dots and the array is unpacked intellectual arguments. Now what this RFC is about is to do same change for array unpacking, so allow you to also use string keys.

Derick Rethans  5:24  
	In another episode, I spoke with David Gebler on a more specific addition of a new function fsync. David explains the reason why he wants to add this to PHP.

David Gebler  5:34  
	It's an interesting question, I suppose in one sense, I've always felt that the absence of fsync and some interface to fsync is provided by most other high level languages has always been something of an oversight in PHP. But the other reason was that it was an exercise for me in familiarizing myself with PHP core getting to learn the source code. And it's a very small contribution, but it's one that I feel is potentially useful. And it was easy for me to do as a learning exercise. 

Derick Rethans  5:58  
	And that is how things are added to PHP sometimes, to learn something new and add something useful at the same time. After discussing the move of the PHP documentation to GIT an episode 78, in Episode 79, I spoke with Nikita about his new in initializers RFC. He says:

Nikita Popov  6:15  
	So my addition is a very small one, actually, my own will, I'm only allowing a single new thing and that's using new. So you can use new whatever as a parameter default, property default, and so on.

Derick Rethans  6:29  
	The addition of this change also makes it possible to use nested attributes. Nikita explains:

Nikita Popov  6:34  
	I have to be honest, I didn't think about attributes at all, when writing this proposal. What I had in mind is mainly parameter defaults and property defaults. But yeah, attribute arguments also use the same mechanism and are under the same limitations. So now you can use new as an attribute argument. And this can be used to effectively nest attributes.

Derick Rethans  6:59  
	Static Analysis tools are used more and more with PHP, and I spoke to the authors of the two main tools, Matt Brown, of Psalm, and Ondrej Mirtes of PHPStan. They propose to get her to add a new return type called noreturn. I asked him what it does and what it is used for.

Ondrej Mirtes  7:14  
	Right now the PHP community most likely waits for someone to implement generics and intersection types, which are also widely adopted in PHP docs. But there's also noreturn, a little bit more subtle concept that would also benefit from being in the language. It marks functions and methods that always throw an exception. Or always exit or enter an infinite loop. Calling such function or method guarantees that nothing will be executed after it. This is useful for static analysis, because we can use it for type inference.

Derick Rethans  7:49  
	Beyond syntax, each new version of PHP also adds new functions and classes. We already touched on the new fsync function, but Mel Dafort proposed to out the IntlDatePatternGenerator class to help with formatting dates according to specific locales in a more specific way. She explains:

Mel Dafert  8:07  
	Currently, PHP exposes the ability for locale dependent date formatting with the IntlDateFormat class, it says basically only three options for the format long, medium and short. These options are not flexible in enough in some cases, however, for example, the most common German format is de dot numerical month dot long version of the year. However, neither the medium nor the short version provide and they use either the long version of the month or a short version of the year, neither of which were acceptable in my situation.

Derick Rethans  8:40  
	And she continues with her proposal:

Mel Dafert  8:42  
	ICU exposes a class called DateTimePatternGenerator, which you can pass a locale and so called skeleton and it generates the correct formatting pattern for you. The skeleton just includes which parts are supposed to include it to be included in the pattern, for example, the numerical date, numerical months and the long year, and this will generate exactly the pattern I wanted earlier. This is also a lot more flexible. For example, the skeleton can also just consist of the month and the year, which was also not possible so far. I'm proposing to add IntlDatePatternGenerator class to PHP, which can be constructed for locales and exposes the get best pattern method that generates a pattern from a skeleton for that locale.

Derick Rethans  9:26  
	Locales and internationalization have always been an interest for me, and I'm glad that this made it into PHP 8.1. I spoke at length with Nikita about his property accessors RFC, in which he was suggesting to add a rich set of features with regard to accessibility of properties, including read only, get/set function calls, and asymmetric visibility. He did not end up proposing this RFC, which he already hinted that during our chat:

Nikita Popov  9:53  
	I am still considering if I want to explore the simpler alternatives. First, there was already a proposal, another rejected proposal for Read Only properties probably was called Write Once Properties at the time. But yeah, I kind of do think that it might make sense to try something like that again before going to the full accessors proposal, or instead.

Derick Rethans  10:18  
	He did then later proposed a simpler RFC read only properties, which did get included into PHP eight as a new syntax feature. He explains again:

Nikita Popov  10:27  
	This RFC is proposing read only properties, which means that a property can only be initialized once and then not changed afterwards. Again, the idea here is that since PHP 7.4, we have Type Properties. Remaining problem with them is that people are not confident making public type properties because they still ensure that the type is correct, but they might not be upholding other invariants. For example, if you have some, like additional checks in your constructor, that a string property is actually a non empty string property, then you might not want to make it public because then it could be modified to an empty value. For example, one nowadays fairly common case is where properties are actually only initialized in the constructor and not changed afterwards any more. So I think this kind of mutable object pattern is becoming more and more popular in PHP.

Derick Rethans  11:21  
	Nikita, of course, meant this kind of immutable object pattern, which we didn't pick up on during the episode. Another big change was the PHP type system, where George Peter proposed out pure intersection types. He explains what it is:

George Peter Banyard  11:35  
	I think the easiest way to explain intersection types is to use something which we already have, which are union types. So union types tells you I want X or Y, whereas intersection types tell you that I want x and y to be true at the same time. The easiest example I can come up with is a traversable that you want to be countable as well.

Derick Rethans  11:54  
	To explain our pure George Peter says:

George Peter Banyard  11:58  
	So the word pure here is not very semantically, it's more that you cannot mix union types and intersection types together.

Derick Rethans  12:06  
	Just after the feature freeze for PHP 8.1 happened in July, another RFC was proposed by Nicolas Grekas to allow the new pure intersection types to be nullable as well. But as that RFC was too late, and would change the pure intersection type to just intersection types, it was ultimately rejected.

Derick Rethans  12:23  
	The last feature that I discussed in a normal run of the podcasts was Nikita's first class callable syntax support. He explains why the current callable syntax that uses strings and arrays with strings has problems:

Nikita Popov  12:35  
	So the current callable syntax has a couple of issues. I think the core issue is that it's not really analysable. So if you see this kind of like array with two string signs inside it, it could just be an array with two strings, you don't know if that's supposed to actually be a static method reference. If you look at the context of where it is used, you might be able to figure out that actually, this is a callable. And like in your IDE, if you rename this method, then this array should also be this array elements will also be renamed. But that's like a lot of complex reasoning that the static analyser has to perform. That's one side of the issue. The second one is that colour bulls are not scope independent. For example, if you have a private method, then like at the point where you create your, your callable, like as an array, it might be callable there, but then you pass it to some other function, and that's in a different scope. And suddenly that method is not callable there. So this is a general issue with both the like this callable syntax based on arrays, and also the callable type, is callable at exactly this point, not callable at a later point. This is what the new syntax essentially addresses. So it provides a syntax that like clearly indicates that yes, this really is a callable, and it performs the callable culpability check at the point where it's created, and also binds the scope at that time. So if you pass it to a different function in a different scope, it still remains callable.

Derick Rethans  14:08  
	This new feature is a subset of another RFC called partial function applications, which was proposed by Paul Crovella, Levi Morrison, Joe Watkins, and Larry Garfield, but ultimately got declined. So there we have it, a whirlwind tour of the major new features in PHP 8.1. I hope you will enjoy them. As I said in the introduction, I'm not sure if I will continue with the podcast to talk about PHP 8.2 features in 2022 due to time constraints. Let me know if you have any suggestions.

Derick Rethans  14:41  
	Thank you for listening to this installment of PHP internals news, a podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening and I'll see you next time.



Show Notes
----------

- Episode `#73 <https://phpinternals.news/73>`_: `Enumerations <https://wiki.php.net/rfc/enumerations>`_
- Episode `#74 <https://phpinternals.news/74>`_: `Fibers <https://wiki.php.net/rfc/fibers>`_
- Episode `#76 <https://phpinternals.news/76>`_: `Array Unpacking <https://wiki.php.net/rfc/array_unpacking_string_keys>`_
- Episode `#77 <https://phpinternals.news/77>`_: `fsync function <https://wiki.php.net/rfc/fsync_function>`_
- Episode `#79 <https://phpinternals.news/79>`_: `New in Initialisers <https://wiki.php.net/rfc/new_in_initializers>`_
- Episode `#81 <https://phpinternals.news/81>`_: `noreturn type <https://wiki.php.net/rfc/noreturn_type>`_
- Episode `#85 <https://phpinternals.news/85>`_: `Add IntlDatePatternGenerator <https://wiki.php.net/rfc/intldatetimepatterngenerator>`_
- Episode `#86 <https://phpinternals.news/86>`_: `Property Accessors <https://wiki.php.net/rfc/property_accessors>`_
- Episode `#88 <https://phpinternals.news/88>`_: `Pure Intersection Types <https://wiki.php.net/rfc/pure-intersection-types>`_
- Episode `#90 <https://phpinternals.news/90>`_: `Readonly Properties <https://wiki.php.net/rfc/readonly_properties_v2>`_
- Episode `#92 <https://phpinternals.news/92>`_: `First-Class Callable Syntax <https://wiki.php.net/rfc/first_class_callable_syntax>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
