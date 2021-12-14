PHP Internals News: Episode 96: User Defined Operator Overloads
===============================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-12-16 09:24 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin096
   :Enclosure: media/pin-096.mp3

In this episode of "PHP Internals News" I chat with Jordan LeDoux
(`GitHub <https://github.com/JordanRL/>`_) about the "User Defined Operator
Overloads" RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-096.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:14
	Hi, I'm Derick. Welcome to PHP internals news, a podcast dedicated to explaining the latest developments in the PHP language. This is episode 96. Today I'm talking with Jordan, about a user defined operator overloads RFC that he's proposing. Jordan, would you please introduce yourself?

Jordan LeDoux  0:33
	My name is Jordan LeDoux. I've been working in PHP for quite a while now. This is the second time I have ventured to propose an RFC.

Derick Rethans  0:44
	What was the first one?

Jordan LeDoux  0:45
	The first one was the "never for parameter types", which was much more exploratory. And we talked about it a little bit. And it generated a lot of good discussion that contributed to kind of the idea formation, which was what I hope to get out of it.

Derick Rethans  1:01
	Okay, but that didn't end up making it into a PHP release. As far as I understand, right?

Jordan LeDoux  1:07
	No, I withdrew it actually, it was clear that the better way to approach the problem it was trying to solve was with a much more comprehensive solution. That particular solution was something that only required a seven line change to the engine. So I wanted to see if it was something people were okay with, or thought was a decent idea for that particular problem, much more comprehensive, like template classes, or something like that is probably the better route to go.

Derick Rethans  1:35
	Well, I think the RFC that we're talking about today, is going to require quite a bit more than seven lines of code?

Jordan LeDoux  1:41
	Quite a bit more. Yeah.

Derick Rethans  1:42
	So what is this RFC that we're talking about today?

Jordan LeDoux  1:45
	Well, user defined operator overloads is a way for PHP developers to define the ways in which objects interact with specific operators. So for instance, the plus operator, the plus sign. It's a way for those objects to kind of define their own logic as far as how that's handled, which right now, as of PHP 8.0, those were all switched to type errors. So it's not possible currently to write any code that doesn't result in a fatal error, where objects are used with operators.

Derick Rethans  2:25
	Usually, I ask about every RFC, what problem are you trying to solve this? So what problem are you trying to solve this RFC?

Jordan LeDoux  2:31
	The biggest problem that this solves is that objects contain, so objects in most programs represent a value or multiple values that have a program context. That's the most powerful thing about objects is they're contextual, and they understand the state, they understand what state the object is in, and sometimes even what state the whole program is in. And that's necessary for a lot of things. Like for instance, if you're tracking a distance, you know, you might measure that meters, and that would have a number you might have 30 meters of distance, but it also has a unit of meters. You could just represent that as an int. And then the program just knows internally, hey this is always in meters. But if you need to convert that to a different unit, then that becomes: Okay, well, now I need a special case some things, or I need a function just for converting, and I need to remember which unit my number is in. In a lot of cases, you handle that with objects because objects understand state, and they understand state transitions, which is what a lot of methods are about; transitioning the state of the object from one state to another. Operators are also about state transitions. And they're about very specific kinds of state transitions. It's natural in a lot of ways to think that you, you should be able to define how those two things interact. But currently, it's just not possible within PHP.

Derick Rethans  4:00
	Well, does them this magic operator overloading?

Jordan LeDoux  4:04
	It allows PHP developers to define an implementation logic, which is much like you define a function body that describes how does this object interact with this operator. That's essentially it. There's a lot of other details as to how it does that and what are the restrictions, but that's really the core of the idea.

Derick Rethans  4:26
	And in what kind of situations would you use that?

Jordan LeDoux  4:28
	A lot of them are situations where you're doing very complicated mathematics, or scientific computing or machine learning or things of that nature, where you are going to routinely encounter numbers that have state to them or that have multiple dimensions to them. So for instance, vector mathematics is one where the way that vectors interact with a lot of the operators that we're familiar with, like the multiplication sign is very different than how the number five interacts with the multiplication sign. Complex numbers is another one, you know, to multiply two complex numbers together, you have to treat it like a polynomial where you're multiplying it with the FOIL method: first, outside, inside, last. You know, there's a lot of those sorts of circumstances. But it also could potentially be very useful for some things that are not really mathematical but more quality of life for PHP developers. For instance, scalar objects is something that a lot of developers in PHP have, you know, wanted for a while. It's a thing that's a little more difficult to pin down, how exactly would you go about doing this within the engine, and it's a thing that the engine would kind of have to be very opinionated about by its nature. PHP developers can't provide their own scalar objects. And the main reason for this is that scalars interact with operators and objects can't. So simply allowing PHP developers to define a way for objects to interact with operators would allow user land to develop their own scalar object replacements. It wouldn't make every scalar that object; scalar objects within the engine still has, it's a separate feature. And it's still a thing that would be desirable, probably to a lot of people. But it gets quite a bit of the way there.

Derick Rethans  6:20
	It is always interesting that people come up with the example of complex numbers, because I'm not sure how useful that is in a PHP user land context. And then beyond the scalars, I then sometimes struggle to see where this could be used. With the only exception is probably doing calculations with money related issues. The moment you bring up operator overloading, you'll also get people to say that this is going to get abused. Examples of that, in my opinion at least, is where in C++ you have like the << operator to put things into the stream and stuff like that. What answer would you have to kind of comments?

Jordan LeDoux  6:58
	Abuse of operator overloads to do things that can create unmaintainable code, because that's really the concern for developers is, does a language feature promote code that's difficult to maintain, that's difficult to understand, that's difficult to follow, and develop, and you know, work with. The RFC, the way that I've gone about this implementation, has had that in mind, because I also have experienced that. This is not a thing where I coming down from the academic high tower with, you know, whatever my my concept of this is, and no, no real world experience with these things. I share a lot of those concerns. Actually, I think this is a very useful feature that has a lot of applications I've encountered. I have had to work with matrix maths, I have had to work with complex numbers, I've had to work with arbitrary precision numbers, and all of those situations would have been served so much better by having operator overloads. I was fighting with the language the entire time, I was trying to do those. But I understand you know, in a lot of web applications, those are not common problems to encounter. My experience of that isn't typical. The thing about the way that it's done is it tries to head off a lot of the ways that it could be misused. An example of that is that the RFC requires typing of the parameters. You can't define an operator method and leave the types blank. If you do, then you get a fatal error during compile. It tells you you must explicitly define a type. And the reason for this is that blank types are assumed to be mixed. So it's the same as putting mixed for the type within the engine. And a mixed type says I can take anything, it doesn't matter what you give me, I can take anything. But that simply isn't true for operators. It's never true. Because even if you think hey, I can accept floats, ints, I can accept any objects, I can figure something out with them. You know, even if you think that's true, what happens when somebody passes you a stream resource? I mean, that's part of mixed. Any implementation that says mixed is probably lying. This RFC requires you to document what are the types that you know how to interact with for this operator. And that's the thing that that developers are kind of going to be forced to think about when they implement this. You know, and that's one example. But there's several other things within the RFC that kind of try and take that concern very seriously. And say, what are the strategies we could design something that is going to be used correctly, most of the time, just by design.

Derick Rethans  9:42
	Would just not then create an inconsistency in the language where for some methods, you simply have to type the arguments.

Jordan LeDoux  9:50
	So yes, it's it is different than how other functions are defined. And methods are defined on classes, but that's one of the reasons that I believe very strongly that using a keyword other than function is a good idea. That's one of the other things that this RFC proposes is, instead of saying function plus or whatever, you say, operator plus. One of the things that that does is that signals to the developer, this is a different thing. That's not a trivial aspect of the RFC. It's not something that can just kind of be thrown away. It's like, oh, that sugar. In a very real way communicates to the developers, this is not like other functions, this is a different thing. It is a function internally within the engine. But that's because that's faster to do it that way. And it's a better way to implement it internally, within core. Developers should not be treating it in PHP as a function, it shouldn't be used that way. It's an engine hook.

Derick Rethans  10:51
	When you're writing the code. If you do operator plus, for example, then at that point, it's clear what the plus does, but not necessarily, when you read the code, and you see the plots, you don't necessarily know what it means, right? Which I think is one of the bigger criticisms of having operator overloading support. But then you can also make the argument saying that well, operators they have a specific meaning in normal language, right. The plus means adding two things. So the argument would be that only use the plus operator for adding things together, not for example, adding a comment to a blog post, which you technically could do, right?

Jordan LeDoux  11:25
	You could.

Derick Rethans  11:26
	I definitely say that is something you should definitely not do, which you could, for example.

Jordan LeDoux  11:30
	That's another reason to kind of not treat them as functions in the syntax. You know, I think that having that operator keyword there really communicates that strongly to PHP developers. You know, when you look at a line of code, that's variable A plus variable B, and you're sitting there thinking: Hmm, I wonder if there's an operator overload involved here, because that might be a thing you do have to think about if this were included in core. While that's an additional thing that might have to be investigated, you know, by developers, and that that's not a trivial thing, I completely acknowledge that. It's also not a thing that would happen by accident, it would have to be intentional, because all objects error, if they're used with an operator currently, and after this is introduced, all objects will continue to error unless they define their own overload within the class that's being called, or one of its parents obviously, because inheritance is respected. It's not a thing that would happen by accident, there's no code that's going to accidentally inject an object into an operator, and all of a sudden, PHP makes wild assumptions and your code is spitting out a number that doesn't make sense, or something like that, because it's simply going to error. This is going to error very early. So you're going to get that feedback from the engine right away, when you do something like that. Maybe you didn't intend or that maybe was ambiguous.

Derick Rethans  12:55
	I've just realized that in languages like C++, you can define multiple versions of the same operator, because you can have method overloading. This is not something you can do in PHP with normal methods either. So do I understand correctly that you can't do that in this case, either it, you need to accept multiple types in the overloaded operator, and then make a decision yourself.

Jordan LeDoux  13:17
	It was suggested to me by a couple of people who gave me very early feedback that, hey, C++ accomplishes this with method overloading, you should do method overloading. And I took one look at that and said: One, I'm already doing a lot of work for this, that sounds like double the work. And two, I'm not convinced that's the best way to do it. Three, that's a huge separate change, that should probably be considered separately. And four, I don't think it's necessary. You can accomplish it with Union types, which we have. And that's another thing that maybe this is a guardrail for PHP developers using it incorrectly. If you're unioning, eight different types, and maybe you're not using it correctly. I mean, that'll look ugly. And I'm people might complain: hey, I don't want to have to Union all these things. I want to be able to overload the method directly with multiple versions. Having that feedback, right in your code that: Hey, this looks ugly. Maybe I'm doing it wrong. I see it as a positive thing, in a lot of ways.

Derick Rethans  14:19
	I agree. First of all, it's a separate subject that should be discussed separately. Now, so far, we've only mentioned the operator keyword, but we haven't spoken about the rest of the syntax yet. So how would you define an overloaded operator?

Jordan LeDoux  14:33
	As we were discussing, there's the keyword operator. So you would define it very similar to how you would define a function. You can give it a visibility, but it can only accept the visibility public, you can omit that if you want. But it can be abstract or final. So you can have an abstract class that forces an implementation, or you can have a class that disallows overriding of the method. You use the keyword on operator, and then where the function name would go for any other function, you use the symbol that you want to overload, so you don't name it the English word plus, you use the actual symbol '+'. And then the rest of it is the way you would define any other function or method because it has a lot of the same concerns that functions do. But it visually looks very different, which I think is another good guardrail. Another good bit of feedback to developers.

Derick Rethans  15:28
	What are the arguments that the overload is operating methods need to accept?

Jordan LeDoux  15:33
	Most of them accept and actually require two arguments. The first is the corresponding operand. The things that are to the right and the left of your operator, they're called operands. And one of them will have this overload and the other one will be some kind of value. You need to accept the other value. And then the second parameter is the operand position, whether or not the operator overload being called; whether it's on the left side of the operator or the right side of the operator, because some some operations depend on whether or not it's on the left or right side.

Derick Rethans  16:13
	Would you say that most of the time, the operators will be used on two objects of the same class, in which case that doesn't really matter?

Jordan LeDoux  16:22
	A lot of the time, I think good implementations of this feature would involve objects that share a base class, share a parent class, or are the same class. I think it would be a very rare circumstance where a good usage of this feature would involve accepting a class that doesn't meet either of those criteria. Maybe it could happen, but I think in most situations, that would be another one of those things that kind of gives you you know, the code smell that a something may be wrong.

Derick Rethans  16:55
	Then of course, with the exception that, for example, vectors, you can multiply with a number. And I define number very loosely here. And then in that case, the order is important. So the RFC has a table of having a whole list of operators, but it doesn't include all of them. What kind of categories are included, which ones aren't?

Jordan LeDoux  17:12
	There's two main categories of operators that are proposed in this RFC, the mathematical operators, you know, your plus, minus, divide, multiply, the pow operator, and the modulo operator. And then the second class of operators are all the bitwise operators. So bitwise and, bitwise or, bitwise not, shift left, shift right, that kind of thing.

Derick Rethans  17:37
	And let's see in the table that It all says equals in the spaceship operators in there. But what I don't see in there, it's larger than, or smaller than operators.

Jordan LeDoux  17:46
	I made the decision very early when I was developing this RFC that I didn't want to support the comparison operators independently. And what I mean by that is, I didn't want to have an object that defined separate logic for the greater than sign than they did for the less than sign. That was mainly to avoid situations where reversing things would change the Boolean logic. Instead, there's a single operator, the comparison operator, or the spaceship operator, that allows you to overload all of them, but only in a way that's self consistent. By implementing that operator overload, you can cover all of the inequality operators, but it will always be consistent with its own output. It's never going to give you things that are logical contradictions with its own data.

Derick Rethans  18:43
	Would the overloaded spaceship operator implementation also be used for other comparisons, like greater than, less than and greater than equals?

Jordan LeDoux  18:52
	That's correct. Going into the implementation just a little bit. Internally, all of those operators, the greater than sign, the less than sign, greater than, and equals to, all of those are internally done as a comparison. That type of comparison where you're outputting, negative one, zero or positive one, they indicate, is it larger? Is it smaller? Is it equal? This actually keeps the PHP user land implementations more consistent with how things are done internally within the engine and makes it much easier to support all of those things, not just consistently, you know, without logical contradictions, but as far as how it gets done within the engine, it makes it much easier to handle those.

Derick Rethans  19:39
	Yeah, I see there's another few implied operators in there. For example, if you're like the -= operator, then that gets implied as $a = $a - $b and stuff like; that all seems to be fairly sensible there. And similar it like ++$a, you get $a = $a + 1, which is basically what that means. You mentioned the word implementation detail. And I have a question myself here is: The symbol tables contrary to support a plus or minus? So do they get transformed into a specific name, for example?

Jordan LeDoux  20:12
	Internally, the function name for a method on a class is stored as a Zend string, which can handle the symbols, it just doesn't. And that's mainly because the lexer can't; the parser is restricted from doing that, because it's kind of ambiguous in all contexts. For instance, outside of a class, following a function, using arbitrary symbols might cause some issues. But that's another thing that the operator keyword makes simpler. The operator keyword in the parser makes allowing the symbols much smaller implementation hurdle, I think that would be something that would be very difficult to do with the function keyword. But internally, it actually does get stored as the symbol. And then it gets put as a kind of an internal pointer with the other Magic Methods. Because internally, it's treated kind of like a magic method.

Derick Rethans  21:07
	Are they flagged with a specific flag or a bit, showing that they are overloaded methods?

Jordan LeDoux  21:13
	Yes, there's a new flag that's added as part of this. That's only for methods, ZEND_ACC_OPERATOR.

Derick Rethans  21:21
	Which I think becomes important if you start looking things like reflection. Because if you list all the methods on a class on the reflection class, then you sort of need to know, what are the already overloaded operator methods or normal methods?

Jordan LeDoux  21:37
	Yes, that's, that is something that became very important when I went into do the reflection implementation for this, which has also been completed at this point. As part of reflection, actually, I very much didn't want to return the operators with other methods. Because again, I don't think that developers should be encouraged to think of these as methods, in most circumstances. That having the flag there made that a very simple change. It was like three or four lines of code per implementation per method that was affected on the reflection classes, check the flag, and then we're done. We're out.

Derick Rethans  22:13
	In addition to that, of course, you gets operator specific reflection methods, right? Because you do want to check whether you have them.

Jordan LeDoux  22:20
	For normal methods, you have getMethod, getMethods, and hasMethod. And so there's three additional methods that are added to reflection class, getOperator, getOperators, and hasOperator, and they behave exactly the same way as the corresponding method ones, but they only deal with the operators.

Derick Rethans  22:43
	The RFC is talking about it an operator methods will be represented by reflection methods, which makes sense, but as you indicate there aren't really methods. And you shouldn't really think of them as methods. So would it not make sense to have a reflection operator method perhaps?

Jordan LeDoux  22:59
	I did consider that. So when I was looking at the implementation for ReflectionMethod, I was looking at the methods that you have on that. And I was saying to myself, is this something that shouldn't be there for operators that not only, you know, maybe it doesn't provide useful information, like for instance, isPrivate will always be false for operators because you can't make operators private, but it doesn't break for operators, it still works. And all of the methods on ReflectionMethod were of that nature. Some of them were not super useful for operators, but none of them were things that were broken, or that were totally didn't make sense. And so because of that, I thought, well, maybe it's better to just have ReflectionMethod and just use that again, instead of creating a separate one that doesn't really have any additional functionality. It's just a copy, essentially, so that they don't have to be maintained separately.

Derick Rethans  23:57
	I see in the RFC, that you're also adding the isOperator methods to reflection methods, so that you can distinguish between normal methods and operator overloaded methods, right, which is then I suppose the alternative to having a different instance class that represents either the method or the operator?

Jordan LeDoux  24:15
	So that was the only thing that I really saw as being necessary, necessarily different, is being able to tell is my instance of ReflectionMethod a normal method or an operator method. That could be solved by having a child class instead, that would be another way to do it, I can definitely see advantages of doing it that way. And I thought about doing it that way. It's already a very big RFC. I kind of wanted to reduce the amount of things that people had to think about or that people had to say, well, this is something different. This is already very different from a lot of things in PHP. And it was one of those things where I was like, that seems like a place where it's not necessary for me to create something new for people to consider.

Derick Rethans  24:56
	As you say, this is quite a long and complicated RFC. What's been the feedback been so far?

Jordan LeDoux  25:02
	A lot of the feedback so far has revolved around the new keyword, the operator keyword. You know, questions about why is this necessary, as opposed to using the function keyword, which we talked about already a little bit. And kind of going through, what are the implications of that, not just within PHP, but also downstream for tooling to things like Psalm, Rector, tools that PHP developers use IDEs, PhpStorm, you know, what are they going to have to do to handle this? And is that more difficult or less difficult with a keyword? Depending on what the answer to that is? Is that trade off worth it?

Derick Rethans  25:41
	Has there been any of the expected feedback saying: Oh, this is just going to be abused by users all over the place?

Jordan LeDoux  25:47
	There's been one or two so far, you know, I think operator overloading as a concept as a feature in programming. And this isn't restricted to PHP as a language. This is something that comes up in other languages, too. I think, as a concept, this feature is something that's always kind of been that way to a lot of languages. There's very few languages where people don't have strong opinions about it. Even in those languages, people don't really encounter that often. But it's the kind of thing that people feel strongly about. So I would always imagine that there are going to be people who, quite rightly, from their own experience, believe that this is just a bad idea. And I can understand why they would think that. I disagree, but I can understand why they would think that. I think about the only language I'm aware of that doesn't have that kind of thing going on is maybe R, but R is a language that's kind of designed around nothing but mathematics. So the idea of being able to control operators is kind of central to what the language does. So it's maybe the only example I can think of, but the rest of them, you know, it is somewhat controversial. And I think it kind of always will be, even if it gets accepted.

Derick Rethans  26:54
	Talking about that. When do you think you'd be opening voting for this?

Jordan LeDoux  26:58
	I'm thinking more along the lines of early January. I think holding the vote two weeks after I announced it on internals a second time, it would be right almost on top of Christmas, I think that would also kind of be a bit unkind, and also may not serve the RFC well. So I think waiting till January is probably the right idea.

Derick Rethans  27:18
	I think that's the nicer way of doing it as well. Yes. Do you have anything to add that we forgot to speak about?

Jordan LeDoux  27:25
	I wanted to mention going back to the operator keyword, and kind of the discussion around that. And the feedback that's been generated so far on that, a really good way to think about it is that the operator keyword is very similar to the enum keyword. Enums are classes, they simply are, but they're classes with very specific restrictions on them. The operator is a function, but it's a function with very specific restrictions on them. And it's for a lot of the same reasons. Enums are intended to be used for a very specific purpose. Operator overloads are also intended to be used for a very specific purpose. And that's one of the reasons that I think it's not not as bad of a thing. And I think that people really should be thinking about it more in terms of why we have the enum keyword instead of terms like, why don't we just use another magic method or something like that? You absolutely could do it that way, the same way that you could do enums it's just classes, but there's value there and doing it with its own keyword, I think.

Derick Rethans  28:29
	Well, thank you, Jordan for taking the time this morning or your night, to talk about the operator overloads proposal.

Jordan LeDoux  28:35
	Yeah, thank you for having me.

Derick Rethans  28:41
	Because I've been on hiatus for a while I wanted to jump in with a few newsworthy items. First of all, I would like to thank Nikita for the many years he worked on PHP, while being an employee of JetBrains. He has decided that he wants to work on something else besides PHP and choose to leave JetBrains to work on LLVM. This means that I will be speaking to him on this podcast a lot less, if at all.

	With Nikita's departure the PHP protect now has nobody working full time on it, as it is desirable for the continuation Nikita's old employer, JetBrains, has banded together with members of the PHP community, including core contributors, companies and sponsors to set up a foundation to fund contributors to work on PHP. Once this is up and running, I will make sure to dedicate an episode to this exciting new development. I have included a link to the foundation on Open Collective in the show notes.

	Just before Nikita left the project two more RFCs were passed. The first one was to move the PHP bug tracker from https://bugs.php.net to https://github.com/phps/php-src repository now accepts your bug reports, whereas the bugs.php.net system has been largely retired. We still accept security bugs on the old issue tracker because we can discuss these in private there before making them public.

	The second RFC implemented the deprecation of dynamic properties with PHP 8.2. Instead of allowing codes to define a rights to undeclared properties, they will now need to be defined in your class definition, otherwise, you will get a deprecation warning. I have included the link to this RFC in the show notes as well. I'm not sure whether I will produce a specific episode on the subject.

	With all the news out of the way, I'd like to thank you for listening to this installment of PHP internals news, a podcast dedicated to demystifying development of the PHP language. I maintain a Patreon for an account for sponsors of this podcast as well as the Xdebug debugging tool. You should sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next time.


Show Notes
----------

- RFC: `User Defined Operator Overloads <https://wiki.php.net/rfc/user_defined_operator_overloads>`_
- RFC: `Deprecate Dynamic Properties <https://wiki.php.net/rfc/deprecate_dynamic_properties>`_
- `PHP Foundation <https://opencollective.com/phpfoundation>`_ on Open
  Collective

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
