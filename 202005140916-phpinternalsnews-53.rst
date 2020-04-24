PHP Internals News: Episode 53: Constructor Property Promotion
==============================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-05-14 09:16 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin053
   :Enclosure: media/pin-053.mp3

In this episode of "PHP Internals News" I chat with Nikita Popov (`Twitter
<https://twitter.com/nikita_ppv>`_, `GitHub <https://github.com/nikic/>`_,
`Website <https://nikic.github.io/>`_)
about the Constructor Property Promotion RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-053.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:16  
	Hi, I'm Derick. And this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 53. Today I'm talking with Nikita Popov about a few RFCs that he's made in the last few weeks. Let's start with the constructor property promotion RFC.

Nikita Popov  0:36  
	Hello Nikita, would you please introduce yourself? Hi, Derick. I am Nikita and I am doing PHP internals work at JetBrains and the constructor promotion, constructor property promotion RFC is the result of some discussion about how we can improve object ergonomics in PHP.

Derick Rethans  0:56  
	Object economics. It's something that I spoke with Larry Garfield about two episodes ago, where we discuss Larry's proposal or overview of what can be improved with object ergonomics in PHP. And I think we mentioned that you just landed this RFC that we're now talking about. What is the part of the object ergonomics proposal that this RFC is trying to solve?

Nikita Popov  1:20  
	I mean, the basic problem we have right now is that it's a bit more inconvenient than it really should be to use simple value objects in PHP. And there is two sides to that problem. One is on the side of writing the class declaration, and the other part is on the side of instantiating the object. This RFC tries to make the class declaration simpler, and shorter, and less redundant.

Derick Rethans  1:50  
	At the moment, how would a typical class instantiation constructor look like?

Nikita Popov  1:55  
	Right now, if we take simple examples from the RFC, we have a class Point, which has three properties, x, y, and Zed. And each of those has a float type. And that's really all the class is. Ideally, this is all we would have to write. But of course, to make this object actually usable, we also have to provide a constructor. And the constructor is going to repeat that. Yes, we want to accept three floating point numbers x, y, and Zed as parameters. And then in the body, we have to again repeat that, okay, each of those parameters needs to be assigned to a property. So we have to write this x equals x, this y equals y, this z equals z. I think for the Point class this is still not a particularly large burden. Because we have like only three properties. The names are nice and short. The types are really short. We don't have to write a lot of code, but if you have larger classes with more properties, with more constructor arguments, with larger and more descriptive names, and also larger and more descriptive type names, then this makes up for quite a bit of boilerplate code.

Derick Rethans  3:16  
	Because you're pretty much having the properties' names in there three times. 

Nikita Popov  3:20  
	Four times even. One is the property name and the declaration, one in the parameter, and then you have to the assignment has to repeat it twice.

Derick Rethans  3:30  
	You're repeating the property names four times, and the types twice.

Nikita Popov  3:34  
	Right.

Derick Rethans  3:36  
	What is the syntax that you're proposing to improve this?

Nikita Popov  3:39  
	The syntax is to merge the constructor and the property declarations. So you only declare the constructor and you add an extra visibility keyword in front of the normal parameter name. So instead of accepting float x in the constructor, you accept public float x. And what this shorthand syntax does is to also generate the corresponding property. So you're declaring a property public float x. And to also implicitly perform this assignment in the constructor body. So to assign this x equals x, and this is really all it does. So it's just syntax sugar. It's a simple syntactic transformation that we're doing. But that reduces the amount of boilerplate code you have to write for value objects in particular, because for those commonly, you don't really need much more than your properties and the constructor.

Derick Rethans  4:40  
	Besides public, I suppose you can also use protected and private there as well.

Nikita Popov  4:45  
	That's right. So you can use all the visibility modifiers. Well, public protected private, static does not really make sense. But if we add other modifiers in the future, then those could be used there as well for example, if we add support for read only properties, then of course, you could also write public readonly float x or something.

Derick Rethans  5:09  
	The RFC talks about desugaring. How's this implemented? Is this transformation on in the  AST, or in another way?

Nikita Popov  5:17  
	This is not an AST transform, but I would say close enough. So we just generate the corresponding property declarations and assignments in the compiler. If you inspect the AST with an extension like PHP AST, you will see the code as written. So with the public in front of the parameter name, but if you inspect the code in reflection, then it will look as if you declared the property explicitly.

Derick Rethans  5:48  
	So the RFC talks about a few constraints and what you can and cannot do with those promoted properties. One of the things it talks about is nullability.

Nikita Popov  5:58  
	Well, we have two different nullability semantics in PHP for historical reasons. One is in parameters, where we say, if you use a type that is not explicitly nullable, but you have a null default value, then we make the type implicitly nullable. While for property types, which are newer, we no longer have this implicit behaviour. So if you want to have a nullable property, you do need to explicitly mark it as nullable. Just using a null default value on will result in an error. And the handling is the same here. So if you want to have a nullable promoted property, you have to mark it as nullable

Derick Rethans  6:43  
	And you cannot just rely on setting the default to null?

Nikita Popov  6:46  
	Exactly, but I think it's like detail. And really this could go either way. I just prefer the explicit nullability because this seems like the direction we are going to in the future. I don't know if we will ever remove this implicit behaviour. Maybe not. But I think nowadays explicit one is preferred. 

Derick Rethans  7:10  
	Less magic is better. 

Nikita Popov  7:11  
	Less magic, exactly.

Derick Rethans  7:13  
	The RFC also has like constraints in there. You can also define a constructor in traits and abstract classes. Can you also use a constructor property promotion there as well?.

Nikita Popov  7:23  
	In traits? Yes, I mean in traits, using it will be a little bit weird. But there is no reason why it can't work. After all traits can have a constructor that will be used in the using class. And traits can also have properties that get imported. So the same mechanism works there as well. It does not work for abstract constructors or constructors in interfaces. The syntax also implies that you have some assignments inside the body of the constructor, and if we have an abstract constructor, then we could not emit these assignments anywhere. We could support it as a special case, like saying that it only declares the properties but skips those assignments. But I know how often you've used abstract constructors, I probably used them like maybe once or twice in all my time working with PHP. So either they really need extra support in that area. 

Derick Rethans  8:25  
	It would also then introduce an inconsistency were promoted properties in abstract classes or abstract class constructors if that's the thing, would be different from normal class constructor property promotion. How does the inheritance work? Is the working in the same way or is there no specific difference in it?

Nikita Popov  8:44  
	Based on like discussion feedback, I think inheritance is the largest point of confusion with this syntax. The thing is that does not really have any special interaction with inheritance. So you can just follow this like syntactical transformation it does, which does not have any impact on inheritance. But the thing is, if you just look at the code, and you see you have the parent class defining the constructor, and the child class defining the constructor, and then you're wondering, well, is there some kind of connection between the parameters? The promoted parameters declared in one constructor and the other one? And the answer is simply: No, there isn't. Those have nothing to do with each other. And even more generally, constructors are a bit of a special case where inheritance is concerned. So usually, we say that methods always have to be compatible with the parent method. So the signature has to be compatible, the return type has to be well not match, but be contravariant. And similar for the argument types, but this rule does not apply for the constructor. So the constructor really belongs to a single class, and constructors between parent and child class do not have to be compatible in any way.

Derick Rethans  10:09  
	Are there any types that you can't use for constructor property promotion?

Nikita Popov  10:14  
	Just callable. Because callable is not a valid property type. Well, there is one more thing that you can't use a variadic argument. Well, if you write a variadic argument, you write something like int, dot, dot, dot, whatever. But the type you're actually writing is int, because that's the type of each individual argument. But all of that gets collected into an array. So the type of the corresponding property would have to be array. So we would have to do an extra transform that's maybe not super obvious. And so I've left this part out. 

Derick Rethans  10:50  
	And also PHP's type system doesn't support defining an array of integers. It only supports describing an array. At a time we're talking about is, at the end of April, this hasn't gone up for a vote yet. When do you think this will happen?

Nikita Popov  11:05  
	The RFC will need one small adjustment because the attributes RFC is currently in voting and it very much looks like it's going to be accepted. We will need to also consider support for attributes on the promoted properties. I think the only small question there is, what does the attributes apply to? Because this could apply to the parameter or to the property, or both.

Derick Rethans  11:34  
	How would you actually set these attributes because from what I understand docblocks, you can only use in front of a method name or a property declaration. How would you define a different attribute for each of the promoted properties?

Nikita Popov  11:48  
	I believe that the attributes RFC already supports attributes on parameters, so that shouldn't be a problem. 

Derick Rethans  11:55  
	So it allows for setting a specific attribute for each of the arguments coming into the constructor. But that didn't quite answer the question. When do you think we'll be voting on this?

Nikita Popov  12:05  
	Maybe in a week or so. 

Derick Rethans  12:06  
	By the time this podcast comes out?

Nikita Popov  12:09  
	Well, we have had a lot of activity recently in PHP internals. So I guess we are one of the few places that benefit from the Coronavirus, because people now have time to work on PHP.

Derick Rethans  12:24  
	Yeah, I mean, I'm looking at so much extra code now. Interestingly, when going to the RFC, and as a side note, it mentioned somewhere that when defining more properties, the line length goes too long, because you now have this extra keyword in there. And that could benefit from then separating the constructor arguments over multiple lines. And that that raises the point is that you can use a trailing comma in arrays when you call functions, but not in argument lists. And I saw that you've also made another RFC for adding the trailing commas in the parameter lists.

Nikita Popov  12:58  
	So there's like a super simple RFC, just allow that extra comma. This has actually already been discussed a couple of times in the past, and has not, has been declined that point.

Derick Rethans  13:13  
	I'm just having a quick look at it. Because this RFC is already voting to see what the current votes are, and it's 58 for and one against.

Nikita Popov  13:21  
	I think like the main counter argument people have against this kind of trailing comma stuff is, well, doesn't that mean that it encourages writing methods with a lot of parameters, which is a bad style. I don't think it does. And I think that even if you don't have a lot of parameters, it's fairly easy to run into line length limitations, because nowadays like to use expressive long parameter names, and expressive long type names, so even without adding an extra protected in front of all of that, you can really easily get signatures that split across multiple lines. In which case having the trailing comma is nice, mainly because we already write it everywhere else.

Derick Rethans  14:12  
	Except for in arguments to methods, because you can't.

Nikita Popov  14:17  
	Well, there are also a couple of other places where you can't. For example, like if you have a class implements, and then implements many interfaces, then you can't put a trailing comma after the last interface. And this is something we could also allow. But I think the relevant distinction there is that this is kind of a freestanding list. Um, it's not wrapped inside brackets, or parentheses. So it kind of looks a little bit weird if you have a trailing comma there, which is possibly also why previous RFC on that simply allowed trailing comma everywhere did not pass.

Derick Rethans  14:58  
	As I said, it looks likely that will pass.

Nikita Popov  15:01  
	Yes, I think it's unlikely that we're going to get 13 new no votes.

Derick Rethans  15:07  
	What I also find interesting is that an RFC that you've mentioned earlier in the episode is that attributes are going to pass as well. At the moment, there's only one no votes there as well, which surprised me because the last time attributes was discussed was very much not going to pass whatsoever.

Nikita Popov  15:27  
	Yeah, this is an interesting effect. It's hard to say why it happens. Probably, well, part of the reason is just that issues that were raised on previous proposals have been addressed. For example, the last one by Dmitri had the very controversial aspects where it's exposed the AST. The abstract syntax tree representation of the attributes, which has gone from this one, and thus removes one of the contentious issues. But I think another part is just that sometimes it takes multiple proposals to really get an idea through internals. We have this situation pretty commonly that though the first RFC fails, second RFC fails, and then the third one does pass.

Derick Rethans  16:18  
	It's also it's taken five years or so. And people's opinions might just change about these things. 

Nikita Popov  16:23  
	Exactly. The previous proposals might just have been before their time.

Derick Rethans  16:29  
	I saw you had made one other tiny RFC, which is the stricter type checks for arithmetic slash bitwise operators. What is that about? 

Nikita Popov  16:40  
	Very simple. So if you're write, well, x minus y, and x is an array. And y is a resource, like what do you expect the outcome to be? There is really no reasonable way that can work. So this RFC proposes to make the arithmetic and the bitwise operators, when working on arrays, when working on objects, and working on resources, simply throw an exception. And the motivation for that was the operator overloading RFC, which has in the meantime been declined. But still, this was a concern raised there that while you can overload operators for objects, but you still get pretty weird behaviour if an overloaded operator is missing, because we currently handle that with just a otice and assuming that the object is equal to one, which is usually not a useful or desired behaviour.

Derick Rethans  17:39  
	There is of course, one exception where you can still use an arithmetic operator, which is the plus between arrays.

Nikita Popov  17:46  
	That's right, yeah. So array plus array is similar to an array merge operation. And that one is of course, well defined and remains supported

Derick Rethans  17:55  
	Whereas things like true divided by 17, although not sensible, it'll continue to work.

Nikita Popov  18:00  
	Right, that also. Yeah, so because this is simply a much more contentious issue whether, like implicitly treating true as one is a good idea or not. Personally, I know I have written code where I, for example, add up booleans. Just as a count of how often something is true. This is like maybe maybe, style wise it would be better to write an explicit integer cast. But the code is also not really wrong. This may be as a discussion for another time.

Derick Rethans  18:33  
	As we've said before, the smaller the RFCs, the easier it is to get them passed as well. Alright, Nikita, thanks for taking the time this morning to talk to me about constructor property promotion RFC, and a few others. We'll see whether they get passed for PHP eight. 

Nikita Popov  18:48  
	Thanks for having me Derick, once again.

Derick Rethans  18:52  
	Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.



Show Notes
----------

- `Constructor Property Promotion <https://wiki.php.net/rfc/constructor_promotion>`_
-  `Allow trailing comma in parameter list <https://wiki.php.net/rfc/trailing_comma_in_parameter_list>`_
- `Stricter type checks for arithmetic/bitwise operators <https://wiki.php.net/rfc/arithmetic_operator_type_checks>`_


Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
