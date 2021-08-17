PHP Internals News: Episode 93: Never For Parameter Types
=========================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-08-19 09:21 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin093
   :Enclosure: media/pin-093.mp3

In this episode of "PHP Internals News" I chat with Jordan LeDoux
(`GitHub <https://github.com/JordanRL/>`_) about the "Never For Parameter Types"
RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-093.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:14  
	Hi, I'm Derick. Welcome to PHP internals news, a podcast dedicated to explaining the latest developments in the PHP language. This is Episode 93. It's been quiet over the last month, so it didn't really have a chance to talk about upcoming RFCs mostly because there were none. However, PHP eight one's feature freeze has happened now, a new RFCs are being targeted for the next version of PHP eight two. Today I'm talking with Jordan LeDoux, about the Never For Parameter Types RFC, the first one targeting this upcoming PHP version. Jordan, would you please introduce yourself?

Jordan LeDoux  0:50  
	Certainly. And thanks for having me. My name is Jordan. I've worked as a developer for about 15 years now. Most of my career has been spent working in PHP. Although professionally, I've had experience working in C#, Python, TypeScript, mostly in the form of JavaScript, but a little bit of Node and, you know, a variety of other languages that I haven't spent enough time in to really be proficient in any real way. But recently, I decided to do something that I have thought about doing for many years, but never actually jumped into which is exploring the PHP engine itself and how I could possibly contribute to it.

Derick Rethans  1:32  
	And here we are, but your first our thing.

Jordan LeDoux  1:35  
	Yeah, it's exciting.

Derick Rethans  1:36  
	What is this RFC about, what does it propose?

Jordan LeDoux  1:39  
	Well, this RFC proposes allowing the never type, which was added in 8.1 as a return value, to parameters for functions and methods on objects. The main idea behind that is that when never was proposed as a return type, it was meant to signal that the function would never return. Not that it returns void, which of course, void signifies which is returning no value or returning, returning without any specified information. And never return signifies that the function will never return, which is a concept that exists in many other languages. And for that purpose in other languages, what's usually used is something called a bottom type. And that's what never ended up being. And I'm proposing that we extend the use of that bottom type to other areas where the type may be helpful.

Derick Rethans  2:38  
	So a bottom type, that might be a new term for many people, it will certainly for me when I looked at the never RFC for as return types. Can you sort of explain what a bottom type is, especially thinking about object oriented theory with something that we'd like to call the Liskov Substitution Principle? And also, how does it apply to argument types?

Jordan LeDoux  2:59  
	Let's start with the Liskov Substitution. The general idea behind Liskov Substitution is that if A is a subtype of B, then anywhere that A exists, you should be able to substitute B. It has to do with when you have a class hierarchy in in an object oriented language, that that class hierarchy guarantees certain things about substitutionality, like whether or not something can be substituted for something else. That affects language design in ways that a lot of programmers are kind of intuitively familiar with, but maybe not familiar with the theory and the ideas behind it more concretely. But LSP is the principle in SOLID, that's the L and SOLID. And it represents a portion of the whole idea of object oriented programming in PHP. Part of being able to substitute one object for another, based on their class hierarchy, and what they implement, and what they provide is there part of their ability to be substituted is whether or not they can fulfil the same kind of contractual requirements of typing. And with Liskov, that means that preconditions can never be strengthened, and post conditions can never be weakened. So a precondition would be a parameter type requirement. If you require that a parameters accepts an object, for instance, in PHP, you can't strengthen that requirement beyond just any object to a particular object. But you can weaken it from an object to an object or an integer with with unions. That's an example of the precondition side of it. The post condition side of it is that you can't, you can't weaken it. So if you have have, you know, if you have a return type of int, you can't have an inherited implementation return int or float, because that broadens the possible return types. They go in opposite directions. And one of them is covariance and one of them is contravariance.

Derick Rethans  5:18  
	I can never remember which one is which.

Jordan LeDoux  5:21  
	Yeah, basically contravariance go up the tree and covariance go down the tree. If you're thinking about widening, or sorry, narrowing. If you're thinking about narrowing, then covariance go down the implementation tree and contravariance go up the implementation tree.

Derick Rethans  5:40  
	Okay, so how does the bottom type fit in here?

Jordan LeDoux  5:43  
	The bottom type in any type system represents like the base type that all types originate from. And the best way in my mind to think about it is kind of just integer math. It's a thing that every programmer is going to be familiar with. And it fits, it fits all right. So you can think of the bottom type as zero integers, a lot of people would think of null as zero if they're thinking about a type system, but null is more like negative one. It's like the entire negative side of the integer system. We could say that the string type is one, and the integer type is two. And the float type is three, and maybe int or float, the union of them is five, which would be the two numbers added together. And if you describe type systems this way, then you can say, hey, if I take any type and add the value that represents the other type, then I get my result type. So if I take zero, the bottom type, and I add one, the string type, what I end up with is one, still the string type. So the bottom type is whatever type system or whatever, whatever type, when you add any other type to it, you get the type you added to it and nothing else, you just get your original thing. That's why it's called the union identity for the type system. The top type in PHP is mixed. And that's the opposite side of it. It's just like zero is the additive identity. One is the multiplicative identity. If you multiply anything by one, you're going to get what you originally had. And if you add anything to zero, you'll get what you originally had. So mixed ends up being, or the top type in general, ends up being the intersection identity, and the bottom type, or never, in PHP's case, ends up being the union identity. And all this is like deep type theory, but most programmers don't have to interact with it. It's more something that affects language design usually.

Derick Rethans  7:48  
	Could you think of mixed as being infinity?

Jordan LeDoux  7:50  
	That's actually with my with my crude integer analogy, yeah, it would be like all types, all possible types are added together.

Derick Rethans  7:59  
	That makes sense then. Okay, so we have explained what the bottom type is, but, and never being the bottom type. So why is it useful to use the bottom type, or never, as this RFC proposes, as a method argument type for parameters?

Jordan LeDoux  8:14  
	The largest benefit has to do with what we were talking about when it comes to covariance versus contravariance. You know, can you strengthen the requirements? Or can you weaken the requirements? When you inherit a method in a system that preserves Liskov Substitution, the parameters can be widened, they can accept more things. If I had an interface that said, it has one parameter, and that parameter is typed as int, then in any implementation, I could say, okay, but actually, the parameter type is int or float, I'm going to accept both. And I could do that in the implementation. But I would have to accept int, because that's part of the contract. That's part of the interface. So I have to accept whatever type is in the interface. I can just additionally add things on top of that. If I make my original definition, my root definition, the bottom type, then I can add any type to it. And I will just get that type. From never, if I had an interface with, you know, a method foo, and it has one argument, and that argument is typed never, I can re-type that argument as int, or I could re-type that argument as string, and all of them would be valid inheritances.

Derick Rethans  9:38  
	So it's a way of getting around having at least one concrete type like int in an interface.

Jordan LeDoux  9:44  
	Right. And in fact, never is a concrete type. It's a concrete type, that means this code can never be called. For return type, it means this type will never return. But if you're, if what you're trying to do is call it then you can never call it.

Derick Rethans  10:01  
	Which is then why never actually makes sense as a type name, because one of my further questions is going to be how does never as a name make sense? But you've now explained it in such a way that it actually does make sense. So there we go.

Jordan LeDoux  10:15  
	One of the questions that did come up in the internals discussion on this so far, has been a round that choosing never, and because a lot of the examples for use cases are around inheritance, it makes some kind of intuitive sense, if you're describing the inheritance behaviour, to name it something like any or anything, or you know, something about its how permissive it is in its inheritance definition. But the type should really describe the code that it's actually written in. You know, getting outside the idea of an interface or an abstract or something like that. If you have a function, and that function types it as never, or whatever you name, the bottom type. There's no data that can satisfy that. Because any data will have some type other than never. It'll have string, or int, or something. Even null has the null type. You can't provide any data to a function that requires never that will satisfy its type requirement. So that code can never be called. It's really when you start considering how does this affect inheritance that you start getting into this concept of Oh, maybe a different word makes sense. But then that doesn't really reflect the opposite side on the return side, when you're talking about covariance instead of contravariance. And that's why the bottom type for most languages is something like never, or nothing, or nil, something along those lines.

Derick Rethans  11:48  
	So if you type an argument to a method as never, the engine will, of course, enforce that you can't call it with any data because it wouldn't satisfy. But would it also automatically make a method abstract so that inheritance inheriting classes have to widen it or not?

Jordan LeDoux  12:04  
	So that was part of my original idea behind the RFC, was forcing something that implements it or something that inherits it, to widen it. However, there were a lot of good arguments about why that may not be a good idea. One is that in PHP, an empty type isn't an empty type, it's mixed, not widening, would actually just be saying the type is mixed, which is valid, you can go from the bottom type to the top type in a contravariant way, that's a totally valid way to do it. The problem is that PHP has a weakly enforced typing system. Like that's only a problem in this context, it's actually a very powerful feature of the language in a lot of other contexts. So we don't necessarily want to get rid of that. In addition to that, the actual mixed type as a literal that can be used in the language was only added in 8.0. It would kind of represent a much larger backwards compatibility break to require it to be explicitly widened. That was part of my original concept for a lot of the reasons that you were just talking about. But for PHP, specifically, it would probably present more problems than it would provide kind of solutions and utility. I've kind of been convinced off that point a little bit by the arguments of others.

Derick Rethans  13:22  
	Would it be possible to instantiate a class that has a method with its argument typed as never?

Jordan LeDoux  13:28  
	Yes, as long as he never called that argument directly.Using a never type in a constructor would definitely be a definitely be a No, no, that would result in a type error. As soon as you try to instantiate the class. 

Derick Rethans  13:41  
	It would basically make the constructor private. 

Jordan LeDoux  13:43  
	Yeah, you would get a slightly less useful error. 

Derick Rethans  13:47  
	Yes. 

Jordan LeDoux  13:48  
	Then you do if you make the constructor private, and then try and call it.

Derick Rethans  13:53  
	It makes no sense to do it. The RFC slightly touches on generics. And it goes in a way talking about why this is sort of slightly like generics. Could you explain the interaction between these two concepts?

Jordan LeDoux  14:07  
	Generics is a feature obviously, that a lot of PHP developers want. And it's also a very complicated feature to do. Way outside of what I was willing to consider for, for my first, my first attempt at something useful. One of the most common ways that generics are used, is within an inheritance structure to allow something similar to type widening, particularly for parameters. Being able to say, I want the type to be able to be widened, but I don't know exactly how it will be widened. That's something that generics offer. Generics offer many other features as well and many other capabilities. But that particular one is something that this can do. It comes with a cost though, because this isn't generics. It is not really the right way to do that. It can be done without generating compile errors now, if you use never, that's the main difference. You still would encounter errors in static analysis and IDE hints, for instance. The IDE, he wouldn't be able to tell if you typed against a interface that had never as a parameter type, it wouldn't be able to tell what sorts of types the implementers have. Because that's not really the point of accepting an interface as a type for a parameter, or for a function call, or something like that. The point is that any implementer of this will be an acceptable type. But that means that from a static analysis perspective, it won't know what the type requirements are, because it won't know what concrete implementations are being provided in the code. So obviously, this is a limitation. And this limitation does not exist in a in an actual generics implementation of some kind. I do view it as an improvement personally. And the main reason is that before, if you tried to do something like this, then the errors that you generated and the problems that you caused, were in your code, in its actual execution. Now, the errors and the problems that you need to solve are going to be in your static analysis or in your IDE. And that's, that's something that's a lot safer for the code in general. It can present some maintainability challenges, it can be annoying for developers to deal with, all of that's absolutely true. And it doesn't provide all the things that you would want from being able to do that. It moves the where the safety problems are from being in code execution, where it can cause real problems into code writing, which is where you have the opportunity to kind of think about it, reason about it and catch it. 

Derick Rethans  16:54  
	From what I understand is that if you have a class doing implementing some kind of generics, then you'd often expect that where this generic type is used in either argument parameter, or return value, that'd be the same for all the methods, whereas with never, you can of course not enforce it, that it isn't being the same type being used, in all the other places where you would otherwise expect or enforce with  generics to be the same type. So I reckon that's one of the differences. I think that was a better explanation that what I read from the RFC.

Jordan LeDoux  17:25  
	That was a explanation and in argument that I wasn't forced to actually articulate until I presented it to other people, because I went through several days of research before even writing the RFC. And sometimes when you do that, particularly for you know, for programming related things, you just absorb the information and you kind of forget about what did I, which things were new, like which things that I just learned, and which things that I already know. You just integrate it all into your new programming knowledge. And so that was definitely something that when I wrote the first draft of the RFC, I didn't, didn't articulate particularly well, because I it just made sense in my head. And I had forgotten: Hey, this is something I didn't know a week ago, I should probably explain it to others.

Derick Rethans  18:10  
	That's why we have the RFC process, right? So that other people also voice the opinion, and perhaps all the slightly confused language that make perfect sense to the author, but not necessarily to other people that read it, right? 

Jordan LeDoux  18:23  
	Yes. 

Derick Rethans  18:24  
	The RFC kindly mentioned that there's no backwards incompatible changes. So that's always good news, that makes it a little bit easy to accept. What was sort of the biggest pushback against the RFC?

Jordan LeDoux  18:36  
	Initially, actually, the most pushback that I got, when I presented it, was a round choosing never as the name, which I thought would, in my mind, I thought would be something that was barely discussed, actually. But I mean, that again, just like you said, that's why this process exists. So that everybody can actually understand, you know, the things that go into it. I did do the research into that prior, but I couldn't find a single language that had more than one bottom type. The concept of more than one bottom type itself, if you if you go back to the integers concept, that'd be like having what positive zero and negative zero or something like that. It's a concept that just intuitively when you, when you understand how the type system itself works, you feel like okay, so there's probably something wrong from a design perspective, if you have more than one bottom type. It's very easy to not be able to see that intuitively. If you don't go in and take a really deep look at how the type system works or, or how types in general work and what they mean and stuff like that. That was the biggest pushback that I got initially. That mostly just involved explaining the things that I just said about like, why is that the case? Why does that make sense? What are some examples of other languages? Just going into that kind of information. The biggest blocker at the moment, really is about, does it make sense to provide never as a parameter type? If you can't use that statically? If you can't use it in a static analysis situation, then does it make sense to ever use it? And if it doesn't make sense to ever use it, then even if it provides contravariance, is it worth adding? And that's the main discussion that's being had right now, that's not entirely resolved. I think that there is still value in doing that. I think that that argument would carry a lot more weight with me personally, if PHP had a better way of handling the situations where that might happen. That's a much larger undertaking, which I would be interested in, but is also not really not really something that would be as kind from a backward compatibility standpoint.

Derick Rethans  20:54  
	Which we are quite keen on.

Jordan LeDoux  20:57  
	Right, exactly. I don't personally see a way that this type of functionality could be provided, that could satisfy that concern, and not also invalidate enormous amounts of code that currently exist. You kind of have to choose one or the other from my understanding. I will be very pleased if somebody is able to provide me a way to, even if it's a lot of work, provide me a way that that can be accomplished without breaking a lot of code. That was one of my goals is I don't want this, I want this to be an addition to what currently exists, not not something that breaks the current typing that we have, or the way that PHP developers currently interact with the type system or anything like that. That's the current discussion. And I think the thing that is most unresolved.

Derick Rethans  21:46  
	I think you'll have a hard time trying to introduce breaking changes into the language. And I also think rightfully so.

Jordan LeDoux  21:54  
	Yeah, it's it's a good safeguard. And it's a good principle to have in general, I think. I think the way that I described it is that this type of breaking change, the type that would be necessary, wouldn't really be like the type of breaking change you expect in a major version, it would more be like a parallel language syntax. It'd be more like rewriting PHP into a different language with very similar semantics, but very different idea of what the language means underneath. Because fundamentally, in order to do that, typing could no longer be optional anywhere. It would have to be every variable, every piece of data, every function, would have to have an explicit type that the developer is able to control and modify and mutate as they want. I don't even know if type juggling would be possible with the kind of change that would be necessary. And that's, that's half of what PHP consider PHP, so.

Derick Rethans  22:55  
	Definitely not requiring types and places. Because I think, as I said, I think you'll have a hard time convincing people to go that way. 

Jordan LeDoux  23:02  
	Which is the reason I didn't consider going that way, really, so.

Derick Rethans  23:06  
	Would you have anything else to add that I'm, that we missed discussing this RFC?

Jordan LeDoux  23:10  
	This RFC is really interesting to me personally, just from an intellectual perspective. I think that for a lot of users, there's not a lot of use cases where you would use it in your own programs. If you did, it would end up being in like the very core base systems, and only in a few places. In those places, it might really shine, it might be something that's absolutely incredible. Most places in most programs you would never be using, well, you would never be using this type. It's much more critical for some of the internal features things like array access, that interface, and the typing that it requires for parameters. The typing for that's pretty broken. This could be a way that we could fix that possibly, and a couple of the other internal engine features that are implemented through interfaces and things like that would also probably be helped quite a bit by this.

Derick Rethans  24:10  
	Thank you very much for taking the time today to talk about the Never For Parameter Types RFC.

Jordan LeDoux  24:16  
	Thank you for having me.

Derick Rethans  24:20  
	Thank you for listening to this installment of PHP internals news, a podcast dedicated to demystifying the development of a PHP language. I maintain a Patreon account for supporters of this podcast as well as the Xdebug debugging tool, you can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening and I'll see you next time.



Show Notes
----------

- RFC: `Never For Parameter Types <https://wiki.php.net/rfc/never_for_parameter_types>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
