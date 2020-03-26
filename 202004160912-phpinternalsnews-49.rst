PHP Internals News: Episode 49: COPA
====================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-04-16 09:12 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin049
   :Enclosure: media/pin-049.mp3

In this episode of "PHP Internals News" I converse with Jakob Givoni
(`LinkedIn <https://www.linkedin.com/in/jakob-givoni-173319/>`_) about
the "Compact Object Property Assignment", or COPA for short, RFC that he
is proposing for inclusion in PHP 8.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-049.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:16  
	Hi, I'm Derick. And this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 49. Today I'm talking with Jakob Givoni about an RFC that is made with a very long name, the compact object property assignment RFC or COPA for short. Jakob, would you please introduce yourself?

Jakob Givoni  0:39  
	Yes, my name is Jakob. I'm from Denmark, and I've been working programming in PHP for 20 years now. I work as a software engineer for a company in Barcelona that's called Vendo. I got inspired to get involved in PHP internals after I saw you as well as Rasmus and Nikita in a PHP conference in Barcelona last November.

Derick Rethans  1:00  
	there was a good conference, I always like going there. Hopefully, they will run it this year as well. What I'd like to talk to you about today is the COPA RFC that you've made. What is the problem that this is trying to solve?

Jakob Givoni  1:14  
	Yes, I was puzzled for a long time why PHP didn't have object literals. And I looked into it. And I saw that it was not for lack of trying. Eventually, I decided to give it a go with a different approach. The basic problem is simply to be able to construct, populate, and send an object in one single expression in a block, also called inline. It can be like an alternative to an associative array. It gives the data a well defined structure, because the signature of the data is all documented in the class.

Derick Rethans  1:47  
	Of course, people abuse associative arrays for these things at a moment, right? Why are you particularly interested in addressing this deficiency as you see it?

Jakob Givoni  1:57  
	Well, I think it's a common task. It's something I've been missing, as I said inline objects, obviously literals for a long time, and I think it's a lot of people have been looking for something like this. And also, it seemed like it was an opportunity that seemed to be an fairly simple grasp.

Derick Rethans  2:14  
	What kind of solutions do people use currently, instead?

Jakob Givoni  2:18  
	I think, very popular one is the associative array where you define key value pairs as an array. The problem with that is that you don't get any help on the name of the indexes nor the types of the values.

Derick Rethans  2:33  
	I mean, it's easy to make a typo in the name, right? And it just either exists in the array suddenly, if you set it or you just get a random null value back. As you said, yeah, there's no way of enforcing the type here, of course. COPA compact object property assignment is a mouthful, and it is a new bit of syntax to the PHP language. What is this new syntax going to look like?

Jakob Givoni  2:55  
	While it looks just like when you assign a value to a property, but here you can add several comma separated lines of property name equals value inside a square bracket block, which is coming after the array and the array arrow operator. The syntax shouldn't really conflict with anything else we have at the moment. 

Derick Rethans  3:17  
	Because that's becoming more and more of a problem, right? Finding new bits of characters to use for new syntax. It is something that came up with annotations or attributes as well. 

Jakob Givoni  3:27  
	And then to start talking about, does this look like typical PHP? Or do you just like this syntax? Or do you hate it? It becomes a taste based thing. For me, the important thing is that if it works, and if it's fairly trivial to implement, I don't have a problem with it.

Derick Rethans  3:43  
	There was a related RFC early in the year which was called the object initializer RFC. How is your proposal different from that one?

Jakob Givoni  3:51  
	The object initializer is a new concept. Mine is different in in that I didn't want to introduce any new concepts. My approach was focused on pragmatism. In that other RFC, the initialization is done at the construction time. And you can kind of do it without even having to define your constructor. And one of the most important aspects of that one was to enforce that all the mandatory properties have been initialised. Because you can have type properties in PHP 7.4. If they don't have a value, then there is introduction of this new state of uninitialized properties. And the author of that RFC wanted to make sure that once the object was ready was fully constructed, it would validate that there was nothing missing there. So it has like six out of seven characteristics in common with mine, and one characteristic that is different. I looked into this about the mandatory promises and I didn't find a simple way or an obvious way to handle it. I have one idea if this COPA should pass and I have another idea if it fails. I didn't want to include that it was not part of my main goals.

Derick Rethans  5:01  
	I'm looking at the syntax here for a bit. And it seems that way how you can do this COPA block. If you have an object, you use the arrow which is dash greater than sign square brackets, and then the list of properties that you want to assign values to. And the RFC shows that to be equivalent to doing each line manually yourself. Does that mean that it is only works for public properties? 

Jakob Givoni  5:31  
	No, it would work also, for what do you call it, virtual properties that don't actually exist, or if they're private, it would just invoke the magic set method in that case. The same thing would happen as if you were to do the assignment line by line as in the example.

Derick Rethans  5:48  
	Without there being the underscore underscore set method set, it means that you can only really set the public properties in that case. 

Jakob Givoni  5:56  
	You won't be able to set private or protected properties directly unless the magic method does that.

Derick Rethans  6:03  
	So does that mean that it is pretty much only something that happens in syntax, and it doesn't have any other side effects or any other functionality that you wouldn't already be able to do?

Jakob Givoni  6:15  
	Yeah, it's just a new syntax for that. The emphasis here was pragmatism. So not introducing any new concepts.

Derick Rethans  6:23  
	What would use cases for this be?

Jakob Givoni  6:25  
	Typically, as I mentioned, they're data transfer objects, value objects. Those simple associative arrays that are sometimes used as argument backs to constructors, when you create objects. Some people have given some examples where they would like to use this to dispatch events or commands to some different handlers. And whenever you want to create and populate and and use the object in one go, the COPA should help you.

Derick Rethans  6:58  
	I suppose COPA would also work for standard class objects?

Jakob Givoni  7:02  
	It's an object just like anything else. So yeah, yes, there shouldn't be any surprises.

Derick Rethans  7:07  
	But of course, it doesn't really make a lot of sense to use standard class because then again, of course, you don't have the benefits of checking your property names or types, again, of course. Are the other use cases you can think of?

Jakob Givoni  7:19  
	Why don't have anything else in mind.

Derick Rethans  7:22  
	I remember quite a long time ago, because this is a subject that comes up quite a bit. That's pretty much people that write PHP code abuse associative arrays so much. Just like the object initializers RFC, as well as your COPA RFC, try to use objects in a different way to be able to prevent developers from abusing associative arrays, pretty much as more stricter data types. In languages like C, there's a distinct datatype for this is called a struct. Do you think it would make sense that instead of trying to overload our object semantics, then in stats use, or introduce something like a struct concept of that C or other kind of statically typed languages have?

Jakob Givoni  8:10  
	As I understand it, a struct is basically the same thing as structured as what I'm talking about structure set of data. However, I'm not sure if it's worth it to introduce a new concept. I don't know if it's necessary if it's possible to reuse the things that we already have enough familiar with. I think I would prefer that you call it overloading the object. But I don't see a lot of problems with having an object that is simply a list of properties with values. It's a very basic object. An object doesn't need to have any methods, it's possible to use that. Every time we add a new concept like struct would be, I feel that it would lead to a combinatorial explosion of implications that later you need to assess every time you want another future change. I haven't seen any RFCs that have specifically mentioned structs. But it is a very related concept.

Derick Rethans  9:08  
	I'm just asking because I spent a lot of time in C where we have structs. But we don't really have objects or classes to begin with. It's more familiar for me to use that. And the other reason why I was asking is that perhaps it would be possible to create like a slightly more natural syntax, because, in my opinion, I think the one that you currently have chosen isn't particularly the most friendly one, but that's my own opinion here. 

Jakob Givoni  9:33  
	There might be a window of opportunity, because curly brackets after the variable is going to be deprecated as a way as an array access. So maybe that could be used just curly brackets and dropping the arrow itself. That would look a lot more like like an object, I think, and it would also be shorter.

	Right. I mean, PHP 7.4 deprecated these.

	So the question is just how soon can we remove it and replace it to mean something else completely?

Derick Rethans  10:03  
	Yeah, that's a good question. I don't think I have the answer either. I guess it can be introduced as long as syntax that existed previously would now not do something different. And I think you would actually be okay here. 

Jakob Givoni  10:15  
	I'm pretty sure it would throw a syntax error. If you try to run this code in a previous version. 

Derick Rethans  10:21  
	I meant saying if you would reuse the curly braces, because as you said, they have been deprecated in PHP 7.4.

Jakob Givoni  10:28  
	I mean, if someone were not to follow that deprecation notice, that is now in place and would continue to keep their the code. If we change the implementation, it's better to get a clear, fatal error than to just have something really spurious happening.

Derick Rethans  10:45  
	Yes, absolutely, I definitely agree. Now, that's sort of what I was trying to get at, but you explained it more eloquently than I did. The RFC lists a few special cases. It talks about execution order and exceptions. I think some, somebody brought up somewhere that what happened If we're trying to set multiple properties through COPA and say the second out of three throws an exception. What would be the end state of the object for example? Could you talk a little bit through that?

Jakob Givoni  11:11  
	Regarding exceptions being thrown in any of those expressions where you are assigning, it's important to understand that the block of code that is COPA is not an atomic operation. Anything that happened before the exception will still have happened. And everything anything that happens after won't happen. Exactly like what you would expect if you were doing it line by line. Or if you were using method chaining to do several things on an object. I think it's going to happen what you would expect to happen unless for some, I think it might be unintuitive, that it's not an atomic operation. But it's just important to keep that in mind. That's why I listed it under special cases. And there's something similar with the execution order, in that you can list the properties in any order you like. It doesn't necessarily mean that you're going to get the same result if you change the order because you will be able to use the value of a previous assignment in the next one. Again, not 100% intuitive, but I think it might be worth the trade off in implementation and flexibility.

Derick Rethans  12:19  
	As you mentioned, there's no new semantics in there. Talking a little bit about implementation here. As there is no patch available, is this something that you'd be interested in developing yourself? Or are you looking for somebody else to help you out on that?

Jakob Givoni  12:32  
	I actually haven't contributed any code before. I'm not familiar with C. But one reason that I chose this RFC and this approach is also that if I can't get any volunteers, I might be able to learn and to do it myself, since it seems like it's mostly a parser syntax thing, probably should be able to pick that up.

Derick Rethans  12:53  
	I would also think because there is no new semantics in here, that it would instead be something in between, probably just the lexer that we have, the parser, and then constructing an equivalent abstract syntax tree or AST segment out of that.

Jakob Givoni  13:12  
	I would be thrilled to collaborate with someone to do some pair programming in order to get started if anyone is up for it.

Derick Rethans  13:18  
	So if you're listening to this episode, and you want to help Jakob out, why not get in touch with him? His contact details will be in the show notes for sure. The RFC also lists a few things that you have thought about, but you have decided not to either pick up into the RFC or you don't think they are in scope. Would we'll talk about that a little bit? 

Jakob Givoni  13:36  
	There's some special things that you can do at the moment when you assign a value to a property. Things like using a variable to specify the property name, or to generate the property name from an expression using the curly brackets after the arrow. There's also array access directly on the properties, or increment, decrement, or nested object accesses. I don't think that these things are really essential. I've decided to probably leave it out of scope for now unless it's trivial. If it if it's trivial to implement that as well. It's okay with me. It's not deal breaker. But you have to do a cost benefit analysis. And I'm thinking that it could be a future scope. If there's a demand this can be addressed in a later RFC.

Derick Rethans  14:23  
	The RFC also talks about nested COPA. But it looks so complicated to me that I'm not sure whether it is actually something that we even should add to begin with.

Jakob Givoni  14:34  
	I don't think it's as complicated as it looks. So you can already already do nested COPA in if you create a new object inline as well as you of course, you can assign it to a property in the outer scope of the COPA. But if you want to over, to set just one property of a nested object, then you cannot do that directly. Well, you can do it actually if you access the previous one. Because you have access to the current property when you do their assignments. So you can see in my example that you can do it. But there might be a better syntax for doing that.

Derick Rethans  15:11  
	I'm happy to see that there's no backward incompatible changes. So that's always a win. What has been the feedback so far?

Jakob Givoni  15:17  
	Yeah, the feedback has been mixed bag as to say. There's some recognition that this has potential to be a useful feature. This is a critique of the syntax, as you also mentioned, and then about the missing functionality, like the mandatory properties and atomic operations. And then of course, named parameters always comes up. The PHP internals list. It's a tough crowd. I really enjoyed engaged in this project. So I don't mind it's part of it. I also really like this side discussion that we're having currently about ways to improve the way that we collaborate and make progress, especially on tough issues.

Derick Rethans  15:58  
That has definitely improved over the last five years to a decade, but it can always be improved more, I would say. What is your end goal with this RFC? I guess you would like to see this added to PHP at some point, are you targeting it for PHP eight?

Jakob Givoni  16:13  
	I would be extremely proud to see this added to PHP at some point. And if it can make it into PHP eight in the first release, that would be awesome. That's at least what I'm going for, for now.

Derick Rethans  16:25  
	The PHP project is looking for release managers for PHP eight zero, with feature freeze happening at the end of June somewhere. So there's lesser and lesser time available for doing these things. So I'm curious to see where this ends up.

Jakob Givoni  16:39  
	It's a race against time at the moment.

Derick Rethans  16:42  
	But that's always the case, isn't it? I think be interesting to see if, if somebody wants to help out to make the implementation of this, or rather, I'd be interested to see whether you'd be able to pick up that yourself actually. We can always do with more people that work on a PHP language. Do you have anything else to add yourself?

Jakob Givoni  17:00  
	I'd say that I spent a lot of effort researching and writing this. And I just hope that people will study the RFC properly and keep an open mind. I know it's probably going to be a hard sell. And that's okay. I just wanted to give it a go. And this is just just the beginning of my contributions, I hope.

Derick Rethans  17:19  
	I spoke with Mate a little bit a few episodes ago. He was getting worried about it not getting accepted at some point. And I pointed out to him that scalar type hints took about a decade and seven attempts to finally make it into PHP. So it helps to just persist I would say in times. 

Jakob Givoni  17:37  
	Times change and also you get new ideas and you evolve.

Derick Rethans  17:42  
	The language continues to improve and that's how I like it. Thanks, Jakob for taking the time to talk to me today. It was interesting to see what you're up to.

Jakob Givoni  17:51  
	My pleasure. Thank you so much Derick for having me.

Derick Rethans  17:56  
	Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.


Show Notes
----------

- RFC: `Compact Object Property Assignment <https://wiki.php.net/rfc/compact-object-property-assignment>`_
- RFC: `Object Initialiser <https://wiki.php.net/rfc/object-initializer>`_
- Episode 30: `Object Initialiser <https://phpinternals.news/30>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
