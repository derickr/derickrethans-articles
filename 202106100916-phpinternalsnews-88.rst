PHP Internals News: Episode 88: Pure Intersection Types
=======================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-06-10 09:16 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin088
   :Enclosure: media/pin-088.mp3

In this episode of "PHP Internals News" I talk with George Peter Banyard
(`Website
<https://gpb.moe>`_, `Twitter
<https://twitter.com/Girgias>`_, `GitHub <https://github.com/Girgias>`_,
`GitLab <https://gitlab.com/Girgias>`_)
about the "Pure Intersection Types" RFC that he has proposed.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-088.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:14  
	Welcome to PHP internals news, a podcast dedicated to explaining the latest developments in the PHP language. This is Episode 88. Today I'm talking with George Peter Banyard about pure intersection types. George, could you please introduce yourself?

George Peter Banyard  0:30  
	Hello, my name is George Peter Banyard. I work on PHP code development in my free time. And on the PHP Docs.

Derick Rethans  0:36  
	This RFC is about intersection types. What are intersection types?

George Peter Banyard  0:40  
	I think the easiest way to explain intersection types is to use something which we already have, which are union types. So union types tells you I want X or Y, whereas intersection types tell you that I want X and Y to be true at the same time. The easiest example I can come up with is a traversable that you want to be countable as well. So traversable and countable. Currently, you can do intersection types in very hacky ways. So you can either create a new interface which extends both traversable and countable, but then all the classes that you want to be using this fashion, you need to make them implement the interface, which might not be possible if you using a library or other things like that. The other very hacky way of doing it is using reference and typed properties. You assign two typed properties by reference, one being traversable, one being countable, and then your actual property, you type alias reference it, with both of these properties. And then my PHP will check: does the property respect type A those reference? If yes, move to the next one. It doesn't respect type B, which basically gives you intersection types.

Derick Rethans  1:44  
	Yeah, I saw that in the RFC. And I was wondering like, well, people actually do that?

George Peter Banyard  1:49  
	The only reason I know that is because of Nikita's slide.

Derick Rethans  1:51  
	The thing is, if it is possible, people will do it, right. And that's how that works.

George Peter Banyard  1:56  
	Yeah, most of the times.

Derick Rethans  1:57  
	The RFC isn't actually called intersection types. It's called pure intersection types. What does the word pure do here?

George Peter Banyard  2:05  
	So the word pure here is not very semantic. But it's more that you cannot mix union types and intersection types together. The reasons for it are mostly technical. One reason is how do you mix and match intersection types and union types? One way is to have like union types take precedence over intersection types, but some people don't like that and want to explicit it grouping all the time. So you need to do parentheses, A intersection B, close parentheses, pipe for the union, and then the other type. But I think the main reason is mostly the variance, like the variance checks for inheritance are already kind of complicated and kind of mind boggling.

Derick Rethans  2:44  
	I'm sure we'll get into the variance rules in a moment. What is it actually what you're proposing to add here. What is the syntax, for example?

George Peter Banyard  2:52  
	So the syntax is any class type with an ampersand, and any other class type gives you an intersection type, which is the usual way of doing and.

Derick Rethans  3:01  
	When you say class types, do you also mean interfaces?

George Peter Banyard  3:04  
	Yes, PHP has a concept of class types, which are mostly any class in any interface. There's also a weird exception where parent and self are considered class types, but those are not allowed.

Derick Rethans  3:20  
	Okay, so it's just the classes that you've defined and the class that are part of the language but not a special keywords, self and parent and static, I suppose?

George Peter Banyard  3:28  
	Yes, the reason for that is standard types are not allowed to be part of an intersection, because nothing can be an integer and a string at the same time. Now, there are some of the built in types, which can be kind of true. You could have a callable, which is a string, because callables can be arrays, or can be a closure. But that's like very weird and not very great. The other one is iterable. If when you expand that out, you get redundant types, which we can talk about later. And the final thing is parent, self, and static, just makes for some very weird design questions, in my opinion, like, if you ask for something to be an intersection with itself, you basically can only enforce conditions on subclasses. You have a class and you say: Oh, I want it to return self, but also be countable for some reason, but I'm not countable. So if you extend me, then you need to be countable, but I'm not. So it's very weird. parent has kind of the very same weird semantics where you can ask a parent, but it's like, if the base class doesn't support it, and you ask for a parent to be an intersection, then you basically need the child to implement the interface and then a child to return the first child. If you do that main question. Why? Because I don't see any good reasons to do it. And it just makes everything harder.

Derick Rethans  4:40  
	You've only added for the sake of completeness instead of it being useful. Let's move on birds. You've mentioned which types are supported, which is class names and interface names. You already hinted a little bit at redundant types. What are redundant types?

George Peter Banyard  4:56  
	Currently, PHP already does that with union types. If you repeat the type twice in a union, you'll get a compile error. This only affects compiled time known aliases. If you use a use statement, then PHP knows that you basically using the same type. However you use a runtime alias, then it can't detect that.

Derick Rethans  5:13  
	A runtime alias, what's that? 

George Peter Banyard  5:15  
	So if you use the function class_alias.

Derick Rethans  5:16  
	It's new to me!

George Peter Banyard  5:18  
	it technically exists. It also doesn't guarantee basically that the type is minimal, because it can only see those was in its own file. For example, if you say I want A and B, but B is a child class of A, then the intersection basically resolves to only B. But you can only know that at runtime if classes are defined in different files. So the type isn't minimal. But if you do redundant types, basically, it's a easy way to check if you might be typing a bug.

Derick Rethans  5:46  
	You try to do your best to warn people about that. But you never know for certain. 

George Peter Banyard  5:51  
	You never know for certain because PHP doesn't compile everything into like one big program like in check. Static analyser can help for that.

Derick Rethans  5:59  
	Let's talk a little bit about technical aspects, because I recommend that implementing intersection types are quite different from implementing union types. What kind of hacks that you have to make in a parser and compiler for this?

George Peter Banyard  6:11  
	Our parser has being very weird. The parsing syntax should be the same as union types. So I just copy pasted what Nikita did. I tried it. It worked for return types without an issue. It didn't work with argument types, because bison, which is the tool which generates our parser, was giving a shift reduce conflict, which basically tells: Oh, I got two possible states I can go in, and I don't know which branch I need to go, because the PHP parser only does one look ahead. Because it was conflicting, the ampersand, either for the intersection type or for to mark a reference. Normally, if the paster is more developed, or does more look ahead, it is not a conflict. And it shouldn't be. Ilia managed to came up with this ingenious idea, which is just redefine the ampersand token twice and have very complicated names, and just use them in different contexts. And bison just: now I have no issue. It is the same token, it is the same character. Now that you have two different tokens it manages to disambiguate, like it's shift produce. So that's a very weird.

Derick Rethans  7:17  
	I'll have a look at what that actually does, because I'm curious now myself. Beyond the parser, I think the biggest and most complicated part of this is implementing the variance rules for these intersection types. Can you give a short summary of what a variance rules are, and potentially how you've actually implemented them?

George Peter Banyard  7:38  
	Since PHP seven point four, return types and up covariant, and parameter types are contravariant. Covariant means you can like restrict, we can be more specific. And contravariance means you can be broader or like more generic. Union types already gives some interesting covariance implications. Usually, you would think, well, a union is always broader than a single type, you say: Oh, I want either a traversable or accountable, it seems that you're expanding the type sphere. However, a single type can have as a subtype, a union type. For example, you say,:Oh, my base type is a Class A, and I have two child classes, which are B and C. I can type covariantly that I want either B or C, because B or C is more specific than just A. That's what union types over there allows you to do. And the way how it's implemented. And how to check for that is you traverse the list of child types, and check that the child type is an instance of at least one of the parents types. An intersection by virtue of you adding constraints on the type itself will always be more specific than just a single type. If you say: Oh, I want a class A, then more specifically, so I want something of class A and I want it to be countable. So you're already restrict this, which gives some very interesting implications, meaning that a child type can have more types attached to itself than a parent type. That's mostly due how PHP implements its type system, to make the distinctions, basically, I've added the flag, which is either this is a union, meaning that you need to check it is part of one, or it's an intersection. The thing with intersection types is that you need to reverse the order in how you check the types. So you basically need to check that the parent is at least an instance of one of the child types, but not that none of the child types is a super type of the parent type. Let's say you have class C, which extends Class B and Class B extends Class A. If I say let's say my base type is B to any function, and I give something which is a intersection T, any interface, this would not be a valid subtyping relation to underneath B. Because if you looked it was a Venn diagram in some sense, you've got A which is this massive sphere, you've got B which is inside it, and C which is inside it. A intersection something intersects the whole of A with something else, which might also intersect with B in a subset, but it is wider than just B, which means like the whole variance is very complicated in how you check it because you can't really reuse the same loop.

Derick Rethans  10:13  
	I can't imagine how much more complicated this gets when you have both intersection and union types in the same return type or parameter argument type.

George Peter Banyard  10:22  
	One of the primary reasons why it's currently not in the RFC, because it is already mind boggling. And although I think it shouldn't be that hard to like, add support for it down the line, because I've already split it mostly up so it should be easy to check: Oh, is this an intersection? Is this a union? And then you need to branch.

Derick Rethans  10:42  
	Luckily because standard types aren't included here, you also don't really have to think about coercive mode and strict mode for these types. Because that's simply not a thing. 

George Peter Banyard  10:50  
	That's very convenient. 

Derick Rethans  10:52  
	Is the future scope to this RFC?

George Peter Banyard  10:54  
	The obvious future scope is what I call composite types, is you have unions and intersections available in the same type. The main issue is mostly variance, because it's already complicated, adding more scope to it, it's going to make the variance go even harder. I think with most programming languages, the variance code is always complicated to read. While I was researching some of it, I managed to hit a couple of failures, which where with I think was Julia and the research paper I was it was just like focusing on a specific subset. And like, basically proving that it is correct. It's not a very big field. Professors at Imperial, which I've talked to, have been kind of helpful with giving some pointers. They mostly work with basically proper languages or compiled languages, which have this whole other set of implications. Apparently, they have like a bunch of issues about how you normalize the types like in an economical form, to make it easier to check. Which is probably one of the problems that will need to be addressed, when you get like such a intersection and union type. First, you normalize it to some canonical form, and then you work with it. But then the second issue is like how do you want the composite types to actually be? Is it oh, you have got parentheses when you want to mix and match? Or can you use like union precedence? I've heard both opinions. Basically, some people are very dead against using Union as a precedent.

Derick Rethans  12:14  
	My question is going to be, is this actually something people would use a lot?

George Peter Banyard  12:21  
	I don't think it would be used a ton. The moment you want to use it, it is very useful. One example is with the PSRs, the HTTP interfaces. Or if you want the link interface. Combining these multiple things gets it convenient. One of the reasons why I personally wanted as well, it's for streams. So currently, streams don't have any interface, don't have any classes. PHP basically internally checks when you call like certain string methods. For example, if you try to seek and you provide a user stream, it basically checks if you implement a seek method, which should be an interface. But you can't currently do that. Ideally, you would want to stream maybe like a base class, instead of having like a seekable stream, and rewindabe stream, or things like that. You basically just have interfaces. And then like if somebody wants a specific type of stream, just like a stream, which is seekable, which is rewindable. And other things. We already have that in SPL because there's an iterator. And we have a seekable iterator interface, which basically just ask: Oh, this is there's a seek method. I think it depends how you program. So if you separate the many things into interfaces, then you'll probably use intersections types a lot. If you use a maybe a more traditional PHP code base, which uses union types a lot. Union types are like going to be easier. And you want to reduce that.

Derick Rethans  13:32  
	Would you think that lots of people already use union types because it's pretty new as well. Isn't it?

George Peter Banyard  13:38  
	Union types are being implemented in various different libraries. PSRs are updating the interfaces to use union types. One use case, I also have a special method, which was taken the date, it takes a union of like a DateTime interface, a string or an integer. Although intersections types are really new, you hear people when union types were being introduced, you heard people saying, I would promote bad cleaning habits, you shouldn't have one specific type. And if you're using a union, you have a design issue. And I had many people complaining to me why and intersection types of see? Why they haven't intersection types being introduced first, because intersection types are more useful. But then you see other people telling us like, I don't see the point in intersection types. Why would you use an intersection type, just use your concrete class, because that's what you're going to type anyway.

Derick Rethans  14:21  
	I can give you a reason why union types have implemented first, over intersection types, I think, which is that it's easier to implement.

George Peter Banyard  14:28  
	It's easier to implement. And it's more useful for PHP as a whole, because PHP functions accepts a union or return a union. Functions return false for error states instead of null. It makes sense why union types were introduced first, because they are mostly more useful within the scope of what PHP does.

Derick Rethans  14:46  
	Do you think you have anything else to add about intersection types? At the moment, it's already up for voting, when is that supposed to end?

George Peter Banyard  14:54  
	So the vote is meant to end on the 17th of June.

Derick Rethans  14:57  
	At the moment I see there's 15 votes for and two against so it's looking good. What's been your most pushback on this? If there was any at all?

George Peter Banyard  15:05  
	Mostly: I don't see the point in it. However, I do think proper reasons why you don't want it, compared to like some other features where it's more like have thoughts on what you think design wise. But it is undeniable that you you add complexity to the variance. And to the variance check. It is already kind of complicated. I have like a hard time reading it initially. There's the whole parser hackery thing, which is kind of not great. It's probably just because we use like a restricted parser because it's faster and more efficient.

Derick Rethans  15:36  
	I think I spoke with Nikita about parsers some time ago and what the difference between them were. If I remember which episode it was all the to the show notes.

George Peter Banyard  15:44  
	And I think the last reason against it is that it only accepts pure intersections. You could argue that, well, if you're adding intersections, you should add the whole feature set. It might impact the implementation of type aliases, because if you type alias T to be a union of A and B, and then you use type T in an intersection, you basically get a mixture of unions and intersections, that you need to be able to work with. The crux of this whole feature is the variance implementation. And being able to rationalize the variance implementation and been to extend it, I think it's the hardest bit.

Derick Rethans  16:18  
	I guess the next thing still missing would be type aliases, right? Like names for types, which you can't define just yet, which I think you also mentioned in the RFC is future scope. 

George Peter Banyard  16:29  
	Yeah. 

Derick Rethans  16:30  
	Thank you, George, for taking the time today to talk to me about pure intersection types.

George Peter Banyard  16:36  
	Thanks for having me on the show.

Derick Rethans  16:41  
	Thank you for listening to this installment of PHP internals news, the podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening and I'll see you next time.


Show Notes
----------

- RFC: `Pure Intersection Types <https://wiki.php.net/rfc/pure-intersection-types>`_
- Episode #66: `Namespace Token, and Parsing PHP <https://phpinternals.news/66>`_
- `GLR Parser <https://en.wikipedia.org/wiki/GLR_parser>`_
- `LALR(1) Parser <https://en.wikipedia.org/wiki/LALR_parser#Overview>`_
- `Iter Library <https://github.com/nikic/iter>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
