PHP Internals News: Episode 51: Object Ergonomics
=================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-04-30 09:14 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin051
   :Enclosure: media/pin-051.mp3

In this episode of "PHP Internals News" I talk with Larry Garfield
(`Twitter <https://twitter.com/crell>`_, `Website
<http://www.garfieldtech.com/>`_, `GitHub <https://github.com/Crell>`_)
about a blog post that he was written related to PHP's Object Ergonomics.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-051.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:16  
	Hi, I'm Derick. And this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 51. Today I'm talking with Larry Garfield, not about an RFC for once, but about a blog post that he's written called Object Ergonomics. Larry, would you please introduce yourself?

Larry Garfield  0:38  
	Hello World. My name is Larry Garfield, also Crell, CRELL, on various social medias. I work at platform.sh in developer relations. We're a continuous deployment cloud hosting company. I've been writing PHP for 21 years and been a active gadfly and nudge for at least 15 of those.

Derick Rethans  1:01  
	In the last couple of months, we have seen quite a lot of smaller RFCs about all kinds of little features here and there, to do with making the object oriented model of PHP a little bit better. I reckon this is also the nudge behind you writing a slightly longer blog post titled "Improving PHP object ergonomics".

Larry Garfield  1:26  
	If by slightly longer you mean 14 pages? Yes.

Derick Rethans  1:29  
	Yes, exactly. Yeah, it took me a while to read through. What made you write this document?

Larry Garfield  1:34  
	As you said, there's been a lot of discussion around improving PHP's general user experience of working with objects in PHP. Where there's definitely room for improvement, no question. And I found a lot of these to be useful in their own right, but also very narrow and narrow in ways that solve the immediate problem but could get in the way of solving larger problems later on down the line. So I went into this with an attitude of: Okay, we can kind of piecemeal and attack certain parts of the problem space. Or we can take a step back and look at the big picture and say: Alright, here's all the pain points we have. What can we do that would solve not just this one pain point. But let us solve multiple pain points with a single change? Or these two changes together solve this other pain point as well. Or, you know, how can we do this in a way that is not going to interfere with later development that we've talked about. We know we want to do, but isn't been done yet. So how do we not paint ourselves into a corner by thinking too narrow?

Derick Rethans  2:41  
	It's a curious thing, because a more narrow RFC is likely easier to get accepted, because it doesn't pull in a whole set of other problems as well. But of course, as you say, if the whole idea hasn't been thought through, then some of these things might not actually end up being beneficial. Because it can be combined with some other things to directly address the problems that we're trying to solve, right?

Larry Garfield  3:07  
	Yeah, it comes down to what are the smallest changes we can make that taken together have the largest impact. That kind of broad picture thinking is something that is hard to do in PHP, just given the way it's structured. So I took a stab at that.

Derick Rethans  3:21  
	What are the main problems that we should address?

Larry Garfield  3:24  
	So the ones that identify that people have been talking about are the following. One is constructors are just way too verbose. If you've looked at almost any PHP class, in almost any framework, the most common pattern is: you start with a class, you declare three to five properties that are private or protected. Then you have a constructor that takes three to five parameters and assigns each of those to those properties. Usually the names match all the way through, types match all the way through. It's all it's doing is shoving those parameters into properties. Right now, you have to repeat each property name four times total. It's just way too verbose. It's just more typing than we should be doing. And so there have been various proposals for ways to have to type less to do that.

Derick Rethans  4:11  
	We'll get to the solutions in a moment, I'm sure.

Larry Garfield  4:14  
	The next one is what I've called the bean problem. So I've referenced to Java beans. For those who have not worked with Java before. And I haven't worked with it in a long time. But when I last did, this was standard, you'd have what's called a Java bean, which is just a Java class that has a bunch of properties that are private, and then a getter and a setter for every single one of those properties. PHP, you see the same pattern a lot, especially in ORMs. Largely that comes down to this makes serialisation and deserialization straightforward because you can access properties through a method, you know, the names, automatic naming and so on. But that's again, an awful lot of typing to bypass the private and protected keyword. So how can we reduce the mental overhead of that and just have access to what we need to with less work. That relates to a lot of the reasons for that is immutable objects. So it's been increasingly popular in PHP in recent years to have objects that even though the language doesn't support immutability are effectively immutable, in that the object doesn't give you a way to change its properties. But it gives you a way to create a new object that is the same, but with certain changes. Think DateTimeImmutable in PHP core, or it has a modify() method, which doesn't change the objects in place. You see, if you call a DateTimeImmutable object, call it with the modify() method with a parameter of plus one week you get back a new DateTimeImmutable object, that is the timestamp one week later. That pattern is increasingly common. PSR-7, the HTTP messages spec uses that a lot of other packages have started doing it. The way that usually ends up working is these wither methods. It's with some value, with some some property name and so on, similar to a setter, but it returns a new object and there's a common pattern for that now. Another problem is materialised values, where you have something that conceptually is a property. And to a outside caller, it really should just be a property. But you want to not have it be a full property itself. The example I use the kind of the canonical example is you have a first name property and a last name property and you want to format a full name property. There's a lot of cases like that. Right now, you do that as a method, and you have some kind of static cache internally. Which works. It's just: Can we make that better? And can we not make it worse with any of these other changes? A lot of this comes down to how do we make not make any of these problems worse. Another problem is, for lack of better term, and what I call the documented property problem, where if you have a large constructor, then you're going to pass in a bunch of different values because they all map to properties, but you need to keep track of: Okay, which one of these is which? And especially comes up for value options, rather than service objects. Were introduced in C, or Rust or Go would just be a bare struct, essentially, which PHP doesn't have. And we can get to why I think that's okay, we don't have. But objects where you really just have a combination of properties, and that's okay. But you still need to keep track of them, you want to be able to create an object that has only some of them. And if you have eight optional properties, and you want to just set the last one, right, now you have a bunch of nulls or question marks, or empty quotes, or zeros, or whatever default value, and again, it's just very cumbersome. And so the kind of the question I was looking at is, how can we make all of these better and not make any of them worse? That's kind of the problem space. I think most people can relate to, at least most of these. 

Derick Rethans  7:46  
	I would think so to certainly in some of my code, where that's been the case. Hopefully, that was all the problems you found.

Larry Garfield  7:53  
	I think I got all of them. 

Derick Rethans  7:55  
	As I alluded to, in the introduction, there have been quite a few smaller RFCs already to address some of the problems that you just mentioned. Which you list and as well as others in things that you have found that multiple people currently already do. Should we have a quick look at what these things are?

Larry Garfield  8:15  
	One of the proposals that I looked at was writeonce properties, as we are recording this, there's an RFC for that that's in voting. Although it looks like it's probably not going to pass that the vote stays where it is. Now, the idea there is allow typed properties to have a read only marker on them just like the type or public or private, and then they can only be written to once if they're uninitialised you can write to them, after that they're just stuck that way. The advantage is that would make them safe to expose publicly. And so you can have a property that you can expose to the world just access a property but not be concerned about someone changing it out from under you. The downside of that mainly comes down to that evolvable immutable object where that with method then becomes a lot harder, because you can't say: clone this object and change this one property because well, you can't change this one property, you'd have to fully construct a new object. There's also two different proposals that have been floated recently for compact object property assignments. I think they have different names for the same basic idea. Basically, if an object has public properties, being able to write to those in one shot in a code block, along with the constructor in a named fashion. It's essentially there's a common pattern now where you pass an associative array to a function which has a bunch of named properties, and then you can put them in whatever order you want. And then you know, dissect those and map those to properties internally. It's essentially taking that idea and baking it into the syntax, which does help and gives you when you have a lot of properties that are optional. It makes it a lot easier to you have a lot of properties defined or a lot of parameters defined it makes it a lot easier to piecemeal select them. The downside is all of those proposals to date only work on public properties, which have a long list of challenges with them. It also means you're bypassing any kind of validation around this property is only valid if this property is set, or this property has to be less than this property, and so on. Those are too limiting, but definitely they're trying to solve a real pain point.

Derick Rethans  10:19  
	Nor can you enforce types through that, of course.

Larry Garfield  10:21  
	Some of them I think, might be able to

Derick Rethans  10:23  
	I meant associative arrays.

Larry Garfield  10:25  
	Yeah, the associative array approach you can do now, which is really the only possible thing I can say in its favour is that it works today. Type enforcement isn't there, it's poor for documentation. Please don't do that. All these are dancing around names parameters, which is a different language feature that's been discussed on and off for many, many years. I don't know of any current RFCs on the table for this one, but it's come up many times. Number of languages have this Python has it for example, where give or take whatever syntax instead of specifying, call this function with parameters, one, seven and 19, and then you have to guess what those numbers mean, you can call a function with count equals one, order equals ASC, whatever. And then you can reverse the order, change the order around. It's essentially the same idea. But for function parameters rather than Object Properties. Again, there's implementation challenges there. But certainly there are languages that do it successfully. Another problem space people have been looking at is access control. So we mentioned the the read only property. In the discussion for that Nicholas Grekas, made a suggestion for having instead of having a read only flag, allow the access control on a property to be different for read and write. So you could have a property that is publicly readable but not writable. But private writable, or private and protected writable. That gives you many the same benefits as the read only flag would have, but without breaking some of the current patterns we have around cheap cloning of objects and so forth.

Derick Rethans  11:58  
	Because of course in PHP, PHP's object oriented system is based on classes, not on objects. You can access read and write private properties of other objects as long as they have the same class.

Larry Garfield  12:10  
	Correct. And that's something that we take advantage a lot of in cloning, to hold wither method style is based on that. If that feature of PHP went away, it would break an awful lot of code. So don't change that. Other things have been on the table. People have talked in the past about constructor promotion, which is a feature that a couple of languages have including Hack, which is the Facebook PHP fork. The basic idea there is, instead of repeating properties once for their declaration, once in the constructor, and then twice in an assignment, you just declare them as part of the constructor. And it becomes essentially a macro to expand that out to the same original code. Hack already has a syntax for that. This one actually has been a proposal for PHP before and it didn't pass.

Derick Rethans  12:57  
	Was it proposed in the exact same syntax as Hack? I don't believe so because Hack had types at the moment, and PHP did not.

Larry Garfield  13:05  
	The earlier syntax, I was just looking at that RFC earlier today, used public function constructs this arrow foo, comma, this arrow bar. And then you still had to declare the properties independently, so it only solves half the problem. And the syntax looked kind of weird. The Hack syntax just lets you put the entire property declaration in place of the parameter in the constructor line, and it fills in all of the other pieces. You have public function, construct, parentheses, private int, a number, private bar, some bar object, and so on. And it would automatically create that property on the class and take the parameter and promote it and do the assignment for you. So that's what Hack does. I believe TypeScript has something similar, although I haven't worked with it. It's again just simplifying that common case. Another non PHP place I look for inspiration is Rust, because Rust does immutable objects very well. And so I figured, alright, let's let's look what other languages are doing. What Rust does, they have objects that are more bare than PHP does, much like Go where it's really a struct to which you can attach methods rather than an enclosed object, but they let you create a new object. Here, the object constructor syntax is essentially named parameters already, you're essentially providing a Json like block of this property of this value, this property should have this value, similar to the object constructor proposals. But you can then say, dot dot some other object of the same type, which Rust reads as: and fill in anything I haven't specified with the values from this other object. The fallout of that is making new object that is the same as this other object, but for this one change really easy. Could we do something like that either using Rust syntax or something else just conceptually, would that work to make with the with style methods easier, possibly would it help bypass the problems with a read only flag and so on. Finally, kind of the granddaddy of them all proposal in PHP from a couple of years ago is property accessor methods. This is a very contentious RFC, it didn't pass mostly for performance reasons, as I understand it. But the idea here was you could declare a property to have a dedicated getter and setter method. And then when you try to read or write a property, that method gets called transparently in the background. It's essentially the same idea as the magic get and magic set methods on objects, but specifically for each property, which can then eliminate a lot of: if we're talking about this property, if we're talking about that property gives you a lot more flexibility. It also allows you to then, because those are methods, control the access of those methods separately for get and set. So you can have a public getter and private setter method. A number of other languages have this, Python does, JavaScript does. So I included that okay, this has been a proposal on the table before, I personally really like it. The only downside is the performance impact because since people can't really know in advance if a property it's going to be accessing is guarded by methods like this or not, it means every property access, therefore has an extra if statement around it in the engine. And the performance impact of that, well, small, individually, really adds up when you're talking about 10s of thousands of property accesses. As I understand that, that was the main reason that it didn't pass before. I don't have a good solution for the performance issue. Unfortunately, it would be delightful if you know the typing system would let us do that. Or if the JIT would do something there. I have no idea that's well out of my wheelhouse.

Derick Rethans  16:34  
	That's lots of solutions that people have come up with in the past and haven't made RFCs for yet. Solving them all one by one, as you mentioned isn't particularly useful thing to do. Because, as you say, you end up in a jumbled mess of things. Your article continues to have an analysis section about all the different aspects of all the different problems and solutions that we've just mentioned here. What's your thinking here, how to join up all the dots?

Larry Garfield  17:00  
	My goal was alright, as I said, what's the minimum amount of change we can do, that gets us the maximum benefit and solve as many problems as possible without making anything worse? Is there a way that we can make some problems not their own problem, but the result of some other problem? Can we make one a degenerate case of another and thereby solve, kill multiple birds with one stone essentially? What I came up with was: one, constructor promotion on its own, I think is very useful. Let's do that. Named parameters on their own are very useful, let's do that. The combination of constructor promotion and named parameters together gives us the equivalent of a object initialization syntax. The specific symbology in the syntax may look slightly different. But essentially you get the same net effect where you could say, hey, new product object and pass it a series of key values and you're done. And the object itself is defined as just a bunch of key values in the construct statements, and no body, and that still gets promoted. So we end up with struct like, or record like objects with relatively little syntax as kind of a side effect of these two other changes that have good arguments for them on their own. 

Derick Rethans  18:14  
	And also without introduce a new concept such as struct. 

Larry Garfield  18:18  
	Exactly. There's also discussion about, should we just introduce a separate language construct for a struct or a record, that is just their properties, possibly some validation, they will pass by value instead of by reference, which makes immutability easier, to design those for immutability. I've toyed with that idea in the past. And every time I come down to eventually I'm going to want to do everything that classes do anyway. Or if they do something special, I'm going to want to do those in classes, except for the way they pass. Legitimately, there's cases where we would want to have a value object that passes in a more by value style instead of the pseudo reference that objects passed today. There are use cases for that, that's really the only difference. Everything else is essentially the same in both cases, it's more work than is needed to try and create a whole separate construct there. Instead, let's make this one construct flexible enough that we can use it in either way, at whatever use case makes sense. I think those two changes together give us the most bang for the buck and don't harm anything else.

Derick Rethans  19:16  
	Both of these two proposals help to solve the first problem that you have outlined, which is the problem with constructing objects. So the other problem that we spoke about is the value object and access to properties for example. Have you come up with a solution of which proposals would work towards solving that problem as well?

Larry Garfield  19:36  
	My proposal on that front, based on what's available, is so I like Nicholas's idea of separate access control for read and write. Okay, now what syntax can we use for that that is going to be self explanatory and readable and not block property accessors if we ever get to the point of figuring out how to do those performently. I don't think we can go all the way to property accessors right now, I would love to, but I don't think that's feasible. Instead, we can borrow some of the syntax from that proposal and let you declare hard to explain this in verbal format. It's like: string name, curly brace, public get, private set, curly brace. Which is essentially the syntax that the property accessor proposal RFC had, but with the method bodies removed, which that RFC actually supported anyway. And what that gives us is then a syntax to say, this property has different visibility for reading and writing, for get and for set, in a way where it's natural to be able to add in functionality to that later for getters and setter methods. If we figure out how to do it. There are probably other syntaxes that could do the same. I'm flexible. I think the key here is some sort of syntax that gives us that split visibility in a way that opens itself to future extension, rather than just throwing more keywords before a property and hoping it works out for the best. And once you've done that, then I think it's worth it to consider: could we do some kind of Rust like cloning or Rust like creation process? I don't know. It could be a variant on cloning. People have proposed a clone this with and then list of properties. And that, essentially de-sugars into creating that new object and then calling a bunch of property set commands. Maybe that's viable. Maybe it's not I'm not sure. Maybe using a syntax closer to what Rust has so that certain thing parameter lists can get auto populated, I don't know. But I think that's an area worth exploring, and would be a nice add on to these others, but it's not a prerequisite. The thing I like about what I'm proposing here, each of these individual pieces carries value on its own. And there's a good reason to vote for each of these on their own, but they dovetail together so that the whole is greater than the sum of the parts. And I think that's the mark of good design where you don't solve each individual problem. You have tools that together solve several problems. It just kind of falls out of the design.

Derick Rethans  22:06  
	Of course, at the moment you wrote this blog post, none of these proposals had more to it than your description in your article.

Larry Garfield  22:15  
	Some of them had old RFCs that had been proposed and either didn't make it to a vote or the vote gone slightly negative for various reasons. But yeah, I did not have any patches. My C skill is still extraordinarily limited. That this was a discussion starter, not a here's an RFC with code. 

Derick Rethans  22:32  
	Of course, we are no day and a half or two days later. And now there is of course, an RFC for one of them, which is the constructor promotion, which pretty much as we spoke about earlier, picks up Hacklang's syntax and ports it to PHP. 

Larry Garfield  22:47  
	Yes, I've concluded that my primary role in PHP internals is inspiring Nikita to go write things. 

Derick Rethans  22:53  
	And you were successful in this case. 

Larry Garfield  22:56  
	A year ago, I was on this podcast with you talking about comprehensions, when I was pushing for those, and those never happened. But out of that discussion, Nikita noticed, oh yeah, short lambdas I should go finish those and then went and finished that RFC. My role is convincing Nikita, he should do things. So I consider that a worthwhile contribution.

Derick Rethans  23:13  
	Fair enough. I agree. Anyhow, it would be interesting to see where this ends up going. We are about, what three, three months away from PHP 8.0's feature freeze. So there's plenty of time to look at these other three proposals that you concluded would be great to have altogether.

Larry Garfield  23:32  
	I'm happy to work with anyone who actually does know, working on internals on any of these. Personally, I think the asymmetric visibility is the next one after constructor promotion. That's straightforward to do. I know Levi Morrison on the lists has suggested that named parameters has a lot of other gotchas around it that I didn't get into here. And that is very likely. There may very well be implementation reasons why these are harder than I present them as. I fully acknowledge that. But again, if any of these individually, I think still moves the language forward in a way that doesn't close off future avenues.

Derick Rethans  24:07  
	Do you think you'll end up learning some C to be able to work on this yourself?

Larry Garfield  24:11  
	So I used to work in C briefly, 16 years ago. I had a very, very short career writing software for Palm OS. 

Derick Rethans  24:18  
	And I remember us talking about it, when we recorded episode last year.

Larry Garfield  24:22  
	And I did some C again, just recently, while playing with FFI. As we've discussed before, the PHP engine is not written in C, it's written in a macro language that is written in C. There's a learning curve there that I have yet to scale. 

Derick Rethans  24:34  
	Fair enough. 

Larry Garfield  24:35  
	If someone wants to mentor me in that while we work on one of these, I am very open to that. So putting that out there. 

Derick Rethans  24:40  
	You might be inundated by messages now, you never know.

Larry Garfield  24:43  
	Better that then getting ignored

Derick Rethans  24:45  
	Do you have anything else to at?

Larry Garfield  24:46  
	I think it's beneficial for PHP collectively to take this broader approach of, not just okay, what can solve this immediate problem in front of us, we can scratch this one itch, but what are all the itches that we have that need to get scratched? And how can we solve all of those in a way that is going to have the best bang for the buck. And let us do the least amount of work at the least amount of syntax, least amount of conceptual overhead, and yet give us the most flexibility. And there's been a lot of talk anytime we're talking about the PHP type system of we eventually want generics, generics are hard. But let's make sure that whatever we do, doesn't make generics even harder. I think that's good that we have this goal in mind. And we're: all right, what iterative steps get us closer to that without locking us, in without painting us into a corner. And that's kind of what I'm trying to do here. And I would very much encourage everyone working on PHP to take that approach of: don't solve the immediate problem, look at the broader picture, what will solve multiple problems, what will dovetail nicely with something else and what kind of big picture plan in architecture we can look at that ends up making the language better rather than just looking at our feet.

Derick Rethans  25:57  
	Well, thanks for taking the time this afternoon to come and talk about the object ergonomics. We'll see how much of it ends up in PHP eight.

Larry Garfield  26:05  
	Fingers crossed.

Derick Rethans  26:07  
	Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening and I'll see you next week.


Show Notes
----------

- Larry's Blog Post `Improving PHP's object ergonomics <https://hive.blog/php/@crell/improving-php-s-object-ergonomics>`_
- RFC: `Object Initialiser <https://wiki.php.net/rfc/object-initializer>`_
- RFC: `Compact Object Property Assignment <https://wiki.php.net/rfc/compact-object-property-assignment>`_
- Episode 30: `Object Initialiser <https://phpinternals.news/30>`_
- Episode 49: `COPA <https://phpinternals.news/49>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
