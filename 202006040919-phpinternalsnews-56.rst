PHP Internals News: Episode 56: Mixed Type v2
=============================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-06-04 09:19 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin056
   :Enclosure: media/pin-056.mp3

In this episode of "PHP Internals News" I chat with Dan Ackroyd
(`Twitter <https://twitter.com/MrDanack>`_, `GitHub
<https://github.com/danack>`_) about the Mixed Type v2 RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-056.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:20  
	Weekly a podcast dedicated to demystifying the development of the PHP language. This is Episode 56. Today I'm talking with Dan Ackroyd about an RFC that he's made together with Mate Kocsic it's called the mixed type version two. Hello, Dan, would you please introduce yourself?

Dan Ackroyd  0:38  
	Hi Derick. So my name is Dan Ackroyd, also known as Dan Ack online. I maintain the PHP image extension. And I also contribute to PHP internals illegitimate by maintaining some documents that called the RFC codecs that are a set of notes of why certain ideas haven't reached fruition in PHP core, and occasionally I help other people write RFCs.

Derick Rethans  1:04  
	Continuing with the improvement of PHP type system in the last few releases. And we've seen a few more things coming into PHP eight but union types. For a long time, there has been an issue with PHP's internal functions that the type that a return cannot necessarily be represented in PHP type system because they do strange things. It is RFC building more on top of PHP's type system. What is this is trying to solve?

Dan Ackroyd  1:29  
	There's a couple of different problems that's trying to solve. The one I care more about is userland code, I don't actually contribute that much to internals code so I'm not that familiar with all the problems that has. The reason I got involved with doing the mixed RFC was: I had a library for validating parameters, and due to how that library needs to work the code passes user data around a lot internally, and then back out to whether libraries return the validators result. So I was upgrading that library to PHP 7.4, and that version introduced property types, which are very useful things. What I was finding was that I was going through the code, trying to add types everywhere occurred. And there's a significant number of places where I just couldn't add a type, because my code was holding user data that could be any other type. The mixed type had been discussed before, an idea that people kind of had been kicking around but it just never been really worked on. That was the motivation for me, I was having this problem where I couldn't upgrade my library, as I wanted to, I kept forgetting has this bit of code here, been upgraded. And I just can't add a type, or is it the case that I haven't touched this bit of code yet. So coincidentally, I saw that Mate was also looking at picking up the RFC, and he had copied the version that Michael Moravec had been working on previously. I want as I mentioned earlier, I help people write RFCs is for a lot of people where English isn't their first language, it's a difficult thing to do writing technical documents in English. I also think that writing RCFs in general is slightly harder than people really anticipate. Each RFC needs to present clearly why something's a problem, why the proposed solution would work, snd, at least to some extent why other solutions wouldn't work. Looking at the text from the previous version I could see the tool though, I understood, all of the parts of that RFC, I don't think that it made the case for why mixed was the right thing to do in a very clear way. So I spent some time working with Mate to redraft the RFC, discussing it between ourselves and going through a few of the smaller issues before presenting it to internals, for it to be officially discussed as an RFC.

Derick Rethans  3:51  
	Where does the name mixed actually come from?

Dan Ackroyd  3:54  
	So, mixed is actually a very old concept in PHP it's been used in the docs for multiple decades. I think we have multiple core contributors who are younger than the mixed type, which is an interesting situation for a language to be in. It had been used in the documents, all over the place. It has been used to show that the type of a parameter, or return type from functions was quite complicated. It's actually slightly different from how people might use it in userland code. A lot of the places where it's used in the docs would now use a union type there instead of the mixed type. But there are still places where mixed is the correct type to use in the documents.

Derick Rethans  4:40  
	This being an RFC, you're proposing something to do in it. What are you proposing to introduce into PHP?

Dan Ackroyd  4:46  
	To be precise, the RFC proposes being able to use the word mixed as a type to be used for parameter types, return types, and property types and mixed is really a shortcut for something that can be done in Union types, mix is the equivalent of writing array or blue, or callable or int, or float, or no object or resource or string. One of the benefits of mixed is that it's much shorter to type but the full equivalent to that.

Derick Rethans  5:18  
	And you'd have to do is every time you use it.

Dan Ackroyd  5:20  
	It's particularly hilarious when you've got a function that accepts any type of parameter, and then returns that parameter, that's been modified. So you have mixed on the way in, and mixed on the way out, having all of those words on the same line of code is just too much.

Derick Rethans  5:35  
	Does the mean that makes is pretty much implemented as a union type?

Dan Ackroyd  5:39  
	I have no idea. I'd have to refer you to the actual implementation which I can't recall the details off, off the top of my head. The actual internal type checking in PHP is not as clean as you might imagine, from userland, particularly around things like callable, that's not, it's not a straightforward path of code for tracking, whether something's callable. It works as union type, but how it is actually implemented internally, is probably more detailed than that.

Derick Rethans  6:07  
	I'll have a good book, a little bit later than. As, you set a sort of acts as a union. But Union types, and variance are quite tricky. And then I spoke with Nikita about union types, it wasn't the clearest explanation because it's a really difficult concept, right. So how does the mixed type interact with variance in either arguments or return types properties?

Dan Ackroyd  6:30  
	I agree completely. Variance's complicated thing, and liskov substitution principle is a reasonably complicated thing. Full disclaimer here, I am not a computer scientist, I didn't study computer scientists in University. I studied chemistry and molecular physics, and the only formal education I've had in programming, was a single 10 hour course that taught us how to use Fortran 77, which is a lovely language for the 70s, not quite so good for the 1990s when I was learning it. I think people concentrate too much on the theory behind computer science. If I read out the general rule of LSP or liskov substitution principle. It says: For each object O1 of type S, there is an object of type T, such that for all programs P defined in terms of T, the behavior of P is unchanged. When O1 is substituted for O2 and S is a subtype of T. I don't fully understand that. I mean I can go through it and understand it in principle, but I don't understand it. I don't grok it at a fundamental level when I'm writing code, for me a better way of thinking about LSP is to simply say that: if your code follows LSP, then it's probably not going to blow up. If you violate LSP, your code has a very good chance of blowing up. For both parameter types and return types, the way that PHP implements the type checking through variantce, the type checking is done to make it conform with LSP, but the simplest way of putting it is: make sure that your codes not going to blow up on bad assumptions about the types that being passed around.

Derick Rethans  8:17  
	Because PHP does it adhere to LSP your lovely new mixed type does have to adhere to it. How does your lovely new mixed type tie in with LSP and variance specifically because mixed is a little bit special. In some cases, because at the moment PHP if you have a method. And you return nothing from it, sort of acts like mixed. So I saw that in the RFC there is a specific handling of having no arguments going to mixed and then back to no type.

Dan Ackroyd  8:48  
	The RFC; one of the details, is when no type is present for a functional term the signature checks for inheritance are done as if the parameter had a mixed, or void type, so that's a union type of mixed and void. That's the correct thing to do. It makes the code work as you'd expect it to do, and avoids any possible scenarios where you'd make an assumption about the method in the parent class, and that assumption not being true in the child class. I think this is one of the areas where PHP's special behaviour, shines through. This might not be an acceptable solution to people who work in languages that have a cleaner type system, but they probably stay well clear of PHP to begin with, but the details of how it works means that the code behaves as you'd expect it to and doesn't blow up. 

Derick Rethans  9:42  
	Well, that's the reason why void isn't part of the mixed union?

Dan Ackroyd  9:47  
	Mixed and void are related, but quite different from each other. Mixed is a guarantee that for return types. It's a guarantee that a parameter will be returned, but you can't, we can't give you any more details of what the type of that parameter will be. Void, is a guarantee, in quotes, that no value will be returned. I actually strongly regret void being present in PHP. I think it was a mistake. One of the very nice things about PHP is the way that every function returns null, even if you don't have a return statement in that function. This is something that's quite different to a lot of other languages where it's common to have functions declared as void return type, so there's no return value at all. Because PHP always return null, it allows you to do things like var dump, then put a function inside var dump bracket, and that's always guaranteed to not blow up.

	I would have strongly preferred us to introduce the null type to PHP, and for people to use that, when they're not returning a more semantically meaningful value from their function. I think that would actually be a lot better into the PHP type system, and make it a lot easier to write code, that's chainable.

Derick Rethans  11:10  
	The only real locations where it can't return any values is a constructor and a destructor in PHP.

Dan Ackroyd  11:16  
	It would still have a use for functions that never return. So like continual loops, and also functions that only ever exit for by throwing an exception. I think TypeScript has this, I think they call it none. I can't remember the details but it has its uses but the way that most people are using it in PHP is wrong, in my opinion. The reason I still get a little bit worked up about this is because people are still suggesting that we should change the behaviour of the language to match the void return type. I.e. make it so that if you try and use the return value from a function that has a return type of void that PHP should blow up. I just strongly disagree with that, I think, returning null so that functions can be chained together. Even if there's no semantically useful information there is preferable to having code blow up through trying to read the result of a function.

Derick Rethans  12:12  
	Because it's a bit different than in statically typed or compiled languages where you can do all these checks in the compiler right? And never had runtime, whereas in PHP these checks always have to happen at runtime.

Dan Ackroyd  12:23  
	They do but I think it's at a different level than that it's just does, being able to define the fact that we're reading from a particular function should make the program blow up. Is that a useful thing to do or not? This is quite similar to another discussion that pops up every now and again, of whether to make PHP blow up if too many parameters are passed to functions. There's people who strongly feel that this is a terrible thing to allow, that we need to punish anybody who has extra parameters, being passed around. I actually find having extra parameters be a useful debugging technique very occasionally. Imagine scenarios, in scenarios where you've got an interface that comes from a library that's implemented in 10 different classes in your code, but you want to debug one particular implementation. Just being able to temporarily add on some extra parameters to a method call, and have that just work allows you to do some debugging techniques that just wouldn't be possible if PHP blew up when extra parameters get passed.

	This is similar, really similar to the void discussion where people have very strong feelings about, we need to punish people who are writing code wrongly, we need to stop that code from working. The other way that yeah it's not great code, and maybe they might want to refactor their code to not do that, but I can't see any benefit in making PHP blow up.

Derick Rethans  13:49  
	In my opinion, this is I think that belong in project's coding standards, and their static analysers that they run over the code to make sure that they do all our stylistic choices correct, and not having too many arguments to methods is exactly belongs in that category. Right. 

Dan Ackroyd  14:05  
	I agree completely.

Derick Rethans  14:06  
	There's a few more things that I'd like to poke your mind about. The mixed type does not include null, is there a reason for that?

Dan Ackroyd  14:14  
	We discussed this a reasonable amount when drafting the RFC, there's reasons to allow nullability, but what we couldn't see was a clear strong need of why nullability would be required. The mixed type includes null as one of the types and the union of the types of represents. So, adding nullability doesn't actually add any more, more information to the mixed type, because by definition, it's already can be null. It's always possible to add more to PHP core but removing features is really difficult. So we decided to leave it out, for now, just because we can't think of a really strong reason to add it. If someone finds a really clear compelling argument to allow mixed to be nullable, I would definitely be in support of that so long as there was a reasonable reason to have it. What I probably prefer before that, though, is it's kind of odd that the null type isn't usable as a type in PHP by itself. I think that's unfortunate because for union types, imagine you've got some code that can, it's going to return either a float or int, and then you find a reason why it might need to return null. Changing the definition from float or int, to float or int or null, is easier to read for me than question mark, float or int. So I think that might be another RFC that pops up on the radar in the not terribly distant future.

Derick Rethans  15:38  
	Time is running out for PHP eight little bit of course. So resource is part of mixed, but resource as a type you can't use as a type hint anywhere in PHP. So what's going on here?

Dan Ackroyd  15:51  
	Resource is more of a pseudo type, then a real type in PHP. It comes from code that was written before PHP even had classes is my understanding. Though obviously that's from the dawn of time so it's hard to figure out where. When people started writing PHP, they used resource, as we use classes now to represent a complicated bit of state that needs to be passed around from one piece of code to another. The problem with resource as a type, is that it doesn't really tell you that much about the type. If something is a resource, it could be a file handle, a curl handle, a GD image, an XML parser, or any of the other things that are called resource types. It's an ongoing piece of work to slowly refactor resource types away and replace them with classes wherever possible. An example of that is the hash context, used to be a resource type in PHP and I think since PHP 7.2 that's been changed to a class. Work's ongoing, and eventually hopefully most of the other resources will go away, and made into more specific types, but in the meantime resource still exists in PHP. The reason that's included in the mixed definition is because it's a reasonable thing to do to pass a file handle around. And so if you've got a parameter type of mixed. It's absolutely fine to pass in a file handle to that piece of code. Excluding the resource type would make the mixed type be too annoying to deal with because your, your code would then deal with all the other types, except resource.

Derick Rethans  17:21  
	That make sense. As I mentioned in the introduction mixed is already something that's used in a PHP documentation for a long time, and the RFC talks about stubs in PHP. This is something that is going to be introduced with PHP eight as well, what are these stubs.

Dan Ackroyd  17:38  
	I haven't contributed to any of this work so I apologize to anybody who has been doing this piece of work if I get any of the details wrong. One of the problems with PHP core was that for a long time, the information that was used to generate the reflection information was done on a very ad hoc basis. Some of the information was incorrect, and keeping the reflection information up to date with the actual definitions of how the functions work was annoying, to say the least. It's been an effort by a number of the core contributors to set up a system of file stubs, that allow people to write PHP code that defines a stub for each of the internal functions. So that's just like literally a PHP file that has a stub version of the function that just defines the parameter types, parameter names, and the return types. My understanding is that that information is then used internally by the PHP eight build process to generate the reflection information extract the parameters where appropriate, and could be used for features like named parameters where the name of a parameter in those stubs, the name would be coming from the stub file, rather than some random C file in the middle of the PHP core code.

Derick Rethans  18:53  
	And the stubs at the moment can't represent mixed. There's still a hold on, with comments.

Dan Ackroyd  18:58  
	That's correct. This is similar to what I was finding with my own libraries that there were just some things that you just can't currently, add type information for. And it was quite frustrating having to, oh no somebody hasn't missed this one it's just not expressible. Another reason for having mixed is that although generics are going to be still quite a long way off from arriving in PHP. If you wanted to express just a generic array that can contain any possible value. That's another case where the mixed keyword would be used.

Derick Rethans  19:29  
	I've saw some people ask why mixed was chosen here and not any. Is there any specific reason for that?

Dan Ackroyd  19:36  
	The very short reason is that it was easier. Mixed has had a mixed concept for multiple decades, mixed is used widely in PHP core code and documentation. It's also used widely in a community for tools like PHP Stan and Psalm where people use mixed in docblocks, or Psalm annotations to indicate any type. It's really widely established. We did discuss, using any instead. It just didn't seem worth the effort of trying to push it through, at least in part because there's so much legacy going on. Also it's just not clearly that much superior to mixed.

Derick Rethans  20:16  
	Very well. Are there any BC concerns by introducing the mixed keyword.

Dan Ackroyd  20:20  
	That's a small BC break, you can't use mixed as a class name or function name probably any more, but it's a pretty small one, and anybody using an IDE can just add as using a function called mixed in their code can right click on the function, rename, maybe go and get a cup of coffee if that IDE is slow. There is also tools in the PHP community. This is actually quite a surprising thing that PHP has one of the best refactoring tools out there in Rector. That's a tool that, because it understands the abstract syntax tree of PHP, it can understand that: Oh hey there's this new BC break in the next version of PHP. In this case, if you have some code that had a class name mixed it would understand this is going to break. They provide sets of tools for allowing you to upgrade your code automatically. It's a really awesome tool. It's slightly surprising to me that it's probably like one of the best code refactoring tools, if not the best, in any software language. I've looked at some other language's ecosystems, and I think one of the things about PHP is that because it's actually quite a diverse ecosystem, and people sometimes migrate from Symfony to Laravel, or want to upgrade a PHP 5.6 codebase to PHP seven, or those types of things to value in a refactoring tool is a lot higher. Somebody has gone out and done the work to make that tool, and it's really pretty good.

Derick Rethans  21:46  
	Sounds like something I should investigate a little bit then, because I actually had never heard of it. Also make sure to either link in the show notes to it. When you're introducing yourself, you mentioned that you're the maintainer of the image magic extension and PHP that you can use to manipulate images. What's going on with this extension? Is there going to be an upcoming release at some point?

Dan Ackroyd  22:05  
	I want to apologize to everybody for being very lazy and not doing a release, even though there's a small segfault, that happens occasionally, and it's which we have a fix for. To be honest, I don't really use the extension at all myself. And so, maintaining it is more source of stress rather than enjoyment. I know there's many, many things that could be improved for the project including doing releases on a timely basis, and improving the security of how it works, but it's just really hard to justify spending time working on it when it's just a source of stress for me, but it doesn't really provide any benefit to me. As an effort to make it be worth my time effectively or at least give me a gold focus on, I'm going to start asking people to donate money to the projects, to sponsor it, just that I can actually justify myself getting stressed out from trying to help people with impossible to solve bugs that only happened on their system, because otherwise it's just a bit too much stress for me to really want to spend any much, much more time working on it. 

Derick Rethans  23:08  
	Very well, do you have anything else to add?

Dan Ackroyd  23:10  
	Yes, I have a big request, and you've done this a couple of times during this interview. I'd very much appreciate it if everyone in the PHP community could refrain from using the word hints. When talking about types. It used to be that PHP type system was just hints where yeah the documentation says that this function takes an int, but that was just a hint, and it wasn't really enforced. The type system in PHP has evolved into an actual type system that is enforced at runtime, and although it's not a big deal. It does help when talking amongst ourselves as a community, but also when we're talking to people who don't do that much PHP, who are coming from other languages, where their type system is still just a set of hints. Using a slightly more precise language of the PHP type system and parameter types, return types, and property types. It avoids any confusion about what's actually happening in the engine. And if that is my windmill that I tilt at.

Derick Rethans  24:11  
	Alright, thank you, Dan for taking the time this afternoon to talk to me. And I will be looking forward to seeing mixed in PHP because it got accepted, just earlier, yesterday I think. And, yeah, part of PHP's improving type system again.

Dan Ackroyd  24:25  
	Thanks for having me on. It's been a pleasure.

Derick Rethans  24:28  
	Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool, you can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.


Show Notes
----------
i
- RFC: `Mixed Type v2 <https://wiki.php.net/rfc/mixed_type_v2>`_
- `Rector for refactoring <https://rectoring.com/>`_
- `RfcCodex <https://github.com/Danack/RfcCodex/blob/master/rfc_codex.md>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
