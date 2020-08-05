PHP Internals News: Episode 65: Null safe operator
==================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-08-06 09:28 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin065
   :Enclosure: media/pin-065.mp3

In this episode of "PHP Internals News" I chat with Dan Ackroyd
(`Twitter <https://twitter.com/MrDanack>`_, `GitHub
<https://github.com/danack>`_) about the Null Safe Operator RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-065.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:18
	Hi, I'm Derick, and this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 65. Today I'm talking with Dan Ackroyd about an RFC that he's been working on together with Ilija Tovilo. Hello, Dan, would you please introduce yourself?

Dan Ackroyd  0:37
	Hi Derick, my name is Daniel, I'm the maintainer of the imagick extension, and I occasionally help other people with RFCs.

Derick Rethans  0:45
	And in this case, you helped out Ilija with the null safe operator RFC.

Dan Ackroyd  0:50
	It's an idea that's been raised on internals before but has never had a very strong RFC written for it. Ilija did the technical implementation, and I helped him write the words for the RFC to persuade other people that it was a good idea.

Derick Rethans  1:04
	Ilija declined to be talking to me.

Dan Ackroyd  1:06
	He sounds very wise.

Derick Rethans  1:08
	Let's have a chat about this RFC. What is the null safe operator?

Dan Ackroyd  1:13
	Imagine you've got a variable that's either going to be an object or it could be null. The variable is an object, you're going to want to call a method on it, which obviously if it's null, then you can't call a method on it, because it gives an error. Instead, what the null safe operator allows you to do is to handle those two different cases in a single line, rather than having to wrap everything with if statements to handle the possibility that it's just null. The way it does this is through a thing called short circuiting, so instead of evaluating whole expression. As soon as use the null safe operator, and when the left hand side of the operator is null, everything can get short circuited, or just evaluates to null instead.

Derick Rethans  1:53
	So it is a way of being able to call a methods. A null variable that can also represent an object and then not crash out with a fatal error

Dan Ackroyd  2:02
	That's what you want is, if the variable is null, it does nothing. If a variable was the object, it calls method. This one of the cases where there's only two sensible things to do, having to write code to handle the two individual cases all the time just gets a bit tedious to write the same code all the time.

Derick Rethans  2:20
	Especially when you have lots of nested calls I suppose.

Dan Ackroyd  2:25
	That's right. It doesn't happen too often with code, but sometimes when you're using somebody else's API, where you're getting structured data back like in an in a tree, it's quite possible that you have the first object that might be null, it's not null, it's going to point to another object, and the object could be null so and so so down the tree of the structure of the data. It gets quite tedious, just wrapping each of those possible null variables with a if not null.

Derick Rethans  2:55
	The RFC as an interesting example of showing that as well. Is this the main problem that this syntax or this feature is going to solve?

Dan Ackroyd  3:03
	That's main thing, I think there's two different ways of looking at it. One is less code to write, which is a thing that people complain about in PHP sometimes it being slightly more verbose than other languages. The thing for me is that it's makes the code much easier to reason about. If you've got a block of code that's like the example in the RFC text. If you want to figure out what that code's doing you have to go through it line by line and figure out: is this code doing something very boring of just checking null, or is it doing something slightly more interesting and, like maybe slipping in a default value, somewhere in the middle, or other possible things. Compare the code from the RFC, to the equivalent version using the null safe operator, it's a lot easier to read for me. And you can just look at it. Just see at a glance, there's nothing interesting in this code going on. All is doing is handling cases where some of the values may be null, because otherwise you can just look at the right hand side of the chain of operators, see that it's either going to be returning null, or the thing on the right hand side. So for me, it makes code a lot easier to reason about something. I really appreciate in new features that languages acquire.

Derick Rethans  4:17
	From what I can see it reduces the mental overhead quite a bit. As you say, the full expression is either going to be the null, or the return type of the last method that you call.

Dan Ackroyd  4:29
	There's nothing of value in between. So all of that extra words is wasted effort both writing it, and reading it.

Derick Rethans  4:37
	Okay, you mentioned short circuiting already, which is a word I stumble over and had to practice a few times. What is short circuiting?

Dan Ackroyd  4:45
	Very simple way of explaining it is this is similar to an electronic circuit when it's short circuited the rest of the circuit stops operating. It just the signal stop just returns back to earth. The null safe short circuiting, it means that when a null is encountered the rest of the chain of the code is short circuited and just isn't executed at all.

Derick Rethans  5:08
	Okay, what you're saying is that if you have a variable containing an object or working objects, you call a method that returns a null. And then, because it's null it won't call the next method on there any more.

Dan Ackroyd  5:20
	Yes, it won't execute the rest of the chain of everything after a short circuit, takes place.

Derick Rethans  5:27
	The RFC describes multiple way of short circuiting in, to be more precise, it's talks about three different kinds of short circuiting. Which ones are these and which ones have been looked at?

Dan Ackroyd  5:37
	``$obj = null; 	$obj?->foo(bar())->baz();``

	There's apparently three different ways to do short circuiting. And the way this has been implemented in PHP is the full short circuiting. One of the alternative ways was short circuiting for neither method arguments, nor chained method calls. Imagine you've got some code that is object method call foo. And inside foo there's a function called bar and then after the method call to foo, there's a another method call to baz. First way that some other languages have is short circuiting when basically no short circuiting. So, both the function call would be called, and also then the method call baz, would also be called. And this option is quite bad for both those two details, having the function call happen when the result of that function call is just gonna be thrown away is pretty surprising. It just doesn't seem that sensible option to me, and even worse than that, the chaining of the method call as is pretty terrible. It means that the first method call to foo could never return null, because the, there's no short circuiting happening. Each step then need to artificially put short circuiting after the method call to foo, which is a huge surprise to me. This option of not short circuiting either method, arguments, or chained method calls seems quite bad.

Derick Rethans  7:04
	If one of the methods set wouldn't have been called normally because the objects being called is null, and the arguments to that function are, could be functions that could provide side effects right. You don't know what's the bar function call is going to do here for example.

Dan Ackroyd  7:18
	It just makes the whole code very difficult to reason about and quite dangerous to use short circuiting in those languages.

Derick Rethans  7:25
	It's almost equivalent that if you have a logical OR operation right. If the first thing is through evaluates to true and you do the OR the thing behind the OR it's never going to be executed, it's a similar thing here and I suppose, except the opposite.

Dan Ackroyd  7:40
	It's very similar in that people are used to how short circuiting works in OR statements. And for me, similar sort of short circuiting behaviour needs to happen for null safe operator as well for it to be not surprising to everybody.

Derick Rethans  7:55
	This is the first option short circuiting never for neither method arguments, for chained methods calls. What was the second one?

Dan Ackroyd  8:02
	So the second one is to short circuit for method arguments but not for chaining method calls. This scenario, the call to bar wouldn't take place, but the call to the subsequent method call of baz would still take place. This is slightly less bad, but still, in my opinion, not as good as for short circuiting, again because even if the method call foo should never return null, cause the null propagates through the chain in the expression, then have to artificially use a another null safe operator to avoid having a can't call methods on parent.

Derick Rethans  8:41
	And then the third one, which is the short circuiting for both method arguments and chained method calls.

Dan Ackroyd  8:47
	That's the option has been implemented in PHP. This is the one that is most sensible, in my opinion. Soon as the short circuit occurs, everything in the rest of the chain of operators that applies to that objects, get short circuited. To me is the one that is least surprising and the one that everyone's going to expect for it to work in that way.

Derick Rethans  9:08
	So I've actually I have a question looking at the code that you've prepared here where it says: object, question mark, arrow, foo, which is the syntax. We didn't mention the syntax yet. So it is object, question mark, arrow, foo. And the question mark being the addition here. Would it make sense that after foo, instead of using just the article baz to use question mark baz. It'd be a reason why you want to do that?

Dan Ackroyd  9:33
	There are only for languages that don't do full short circuiting. For the languages that don't do full short circuiting and the null makes its way through to then have baz called on it, you have to use another null safe operator in there, just to avoid another error condition happening.

Derick Rethans  9:51
	Very well. Which other languages, actually have full short circuiting?

Dan Ackroyd  9:55
	So the languages that have full short circuiting are C sharp, JavaScript, Swift, and TypeScript, and the languages that don't have for short circuiting are Kotlin, Ruby, Dart, and hack. I'm not an expert on those languages but having a quick look around the internet, it does seem to be that people who try to use null safe operator in the languages that don't implement full short circuiting are not enjoying the experience so much. To me it appears to be a mistake in those languages. I don't know exactly why they made that choice to imagine that is more a technical hurdle, rather than a deliberate choice of this is the better way. It does appear that implementing the full short circuiting is quite significantly more difficult than doing the other option, other types of short circuiting, just because the amount of stuff that needs to be skipped to the full short circuiting, so you've got to imagine that they thought it was going to be an acceptable trade off between technical difficulty and implementation. The, I think that's probably going to be useful enough. To me it just doesn't seem to be that great of a trade off.

Derick Rethans  11:07
	Short circuit circuiting happens when you use the null safe operator. So the null safe operator that syntax is question mark arrow. You mentioned that there is a chain of operators, what kind of operators are part of this chain or what is this chain specifically?

Dan Ackroyd  11:21
	So the null safe operator will by, and short circuit, a chain of operators, and it will consider any operators that look inside or appear in side and object to be part of a chain, and operators that don't look inside an object to be not part of that chain. So the ones that appear inside an object are property access, so arrow, null safe property access. So, question mark arrow. Static property access, double colon. Method call, null safe method call, static method call, and array access also. Other operators that don't look inside the object would be things like string concatenation or math operators, because they're operating on values rather than directly like pairing inside the object to either read a property or do mythical, they aren't part of the chain. They'll still be applied, and they will be part of the chain that gets short circuited

Derick Rethans  12:17
	Which specific chain is used as an argument in a method called as part of another, what sort of happens here?

Dan Ackroyd  12:27
	This is at the limit of my understanding, but the chains don't combine or anything crazy like that. It's only in the object type operators here inside the objects that might be null. That will continue a chain on a chain of operators is then used as a argument to another method call or function call. That's the end of that chain, and they'd be two separate chains, so for me there's no surprising behaviour around the result of a non safe operator being used as an argument to another function call, or another method call, which might have a separate null safe operator on the outside, those two things are independent. They don't affect each other.

Derick Rethans  13:09
	Yeah, I think that seems like a logical way of approaching this. Otherwise, I expect that the implementation will be quite a bit trickier as well.

Dan Ackroyd  13:17
	This is actually something I consider quite deeply when people come up with RFCs is, how would I explain this to a junior developer. If the answer is: I would struggle to explain this to a junior developer. That probably means that, one I don't understand the idea well enough myself, or possibly that the idea is just a bit too complicated for it to be used safely in projects. I mean, there's a difference between stuff being possible to use safely. And we've got things in PHP that are possible to use safely but aren't always going to be used safely, like eval and other questions which can be used quite dangerously, and the general rule for a junior developer would be: You're not allowed to use eval in your code, you need to have a deep understanding of how it can be abused by people. But something for like the null safe operator. It's got to work in a way that's got to be safe for everyone to use without having a really deep understanding of what the details are of its implementation.

Derick Rethans  14:14
	That makes a lot of sense yes. The RFC talks about a few possibilities that are not possible to do with a null safe operators. What are these things it talks about?

Dan Ackroyd  14:25
	Until right before the RFC went for a vote. There was, as part of the RFC there was quite a bit more complexity. And it was possible to use null safe in the write context. So you could do something like null safe operator bar equals foo, effectively assigning the string foo something that might or might not be null. I only learnt this recently. You can also foreach over an array into the property of an object, which I've never seen before in my 20 odd years writing PHP. It would be possible to use the null safe operator in there. And you'd be foreaching over an array into either a variable or into null, which is not a variable. The RFC was trying to handle this case, these cases are generally referred to as write contexts, as opposed to just read contexts where the value is being read. The RFC has a lot of work went into supporting the null safe operator in these write contexts. But luckily, somebody pointed out that this was just generally a rubbish idea, and hugely complex. It has a problem that you just fundamentally can't assign a variable or a value to a possibly non existent container. It just doesn't make any sense. So, a couple of days before it went to a vote, Ilija just asked the question, why don't we just restrict it to read contexts. That's where the value of the RFC is, making it easier to write code that's either going to read a value or call a method on something that might or might not be null. All of this stuff around write context was just added complexity that doesn't really deliver that much value. It's nothing to do with the using null safe operator in write context was removed from the RFC, which made it a whole lot simpler. It made it a lot less likely that people would like to code that doesn't do what they expected.

Derick Rethans  16:17
	I also think it would be a lot less likely to have been accepted in that case, as it stands the vote for this feature was overwhelmingly in favour. Did you think it was going to be so widely accepted or did you think the vote was going to be closer?

Dan Ackroyd  16:31
	So I thought it was gonna be a lot closer. There are quite a few conversations on the internet, where people raise a point that one I don't fully agree with it was a valid point to make. They were concerned that people use null safe operator in places were having null, rather than an object might represent an error. And if people are just using the null safe operator to effectively paper over this error in their code, then it would make figuring out where the null came from, and what error had occurred to cause it to be there. I think the answer to that is this feature isn't appropriate use in those scenarios. If a variable being null, instead of an object represents an error in your code, then you shouldn't be using this null safe operator to skip over that error condition. You need to not use it and watch a bit more code that explicitly defines, or checks that error, handles it more appropriately, rather than just blindly using this feature, without thinking if they pick your use case.

Derick Rethans  17:30
	Or of course thrown exception, which is what traditionally is done for this kind of error situations right?

Dan Ackroyd  17:35
	be a thing so it shouldn't be null, but there's very small chance it is, but only is in situations where you're not aware and throwing an exception so you can error out, and then debug it is the correct thing to do, rather than just silently have errors in your application.

Derick Rethans  17:50
	Would you have anything else that to the null safe operator?

Dan Ackroyd  17:52
	Not really. Except say it's quite interesting that quite a few of the new features for PHP eight are, don't technically allow anything new to be done, there just remove quite a bit of boilerplate. For me, it'll be interesting to see the reaction to that from the community because this is something people have criticized PHP for for quite a long time, they're being very verbose language, particularly compared to TypeScript, where a lot of very basic creating objects can be done in very few lines of code. So between the null safe operator, and the object property promotion, that's for some projects, which use a lot of value types, or data from other services where you don't have control over how it's structured, I think these two features are going to remove a lot of boilerplate code. So I think this might improve people's experience of PHP quite dramatically.

Derick Rethans  18:40
	That ties back into this sort of idea that Larry Garfield has with his object ergonomics, right, especially with the data values that the document was referring to mostly.

Dan Ackroyd  18:50
	Yeah.

Derick Rethans  18:51
	I've another question for you, the last one. Which is: What's your favourite new feature in PHP eight?

Dan Ackroyd  18:56
	Generally the improvements to the type systems are going to cheat by giving two answers. Union types. This will actually be really nice for the imagick extension just because a huge number of the methods in there, accept either string or an imagick pixel object which represents colour internally to the imagick library. Have been able to have correct types on all the methods that currently don't have the right, don't have the correct type information, will be very, very nice. Doesn't make anything new possible. It just makes it easier to reason about the code, so it's easier for tools like PHP Stan to have the correct type of information available rather than having to look elsewhere for the correct type information. Again the mixed types, is very small improvements to the type system in PHP, but it's another piece that helps complete the puzzle of the pulling out the type system to make it be closer to being a complete a type system that can be used everywhere, rather than having to have magic happening in the language.

Derick Rethans  20:02
	It also will result in more descriptive code right because all the information that you as well as PHP Stan need to have to understand what this is saying or what I was talking about. It's all right there in the code now.

Dan Ackroyd  20:15
	Yeah, and having the information about what code's doing in the code, rather than in people's heads, makes it easier for compilers, and static analysers to do their jobs.

Derick Rethans  20:26
	Thank you, Dan for taking the time this afternoon to talk to me about the null safe operator.

Dan Ackroyd  20:31
	No worries Derick, very nice to talk to you as always.

Derick Rethans  20:35
	Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.


Show Notes
----------

- RFC: `Null Safe Operator <https://wiki.php.net/rfc/nullsafe_operator>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
