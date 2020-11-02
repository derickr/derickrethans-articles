PHP Internals News: Episode 69: Short Functions
===============================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-11-05 09:32 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin069
   :Enclosure: media/pin-069.mp3

In this episode of "PHP Internals News" I talk with Larry Garfield
(`Twitter <https://twitter.com/crell>`_, `Website
<http://www.garfieldtech.com/>`_, `GitHub <https://github.com/Crell>`_)
about a new RFC that's he proposing related to Short Functions.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-069.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:15  
	Hi, I'm Derick, and this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. 

Derick Rethans  0:24  
	Hello, this is Episode 69. Today I'm talking with Larry Garfield, about an RFC that he's just announced called short functions. Hello Larry, would you please introduce yourself?

Larry Garfield  0:35  
	Hello World, I'm Larry Garfield, the director of developer experience at platform.sh. These days, you may know me in the PHP world mainly from my work with PHP FIG. The recent book on functional programming in PHP. And I've gotten more involved in internals in the last several months which is why we're here.

Derick Rethans  0:57  
	I'm pretty sure we'll get back to functional programming in a moment, and your book that you've written about it. But first let's talk about short functions, what are short functions, what is the problem that are trying to solve?

Larry Garfield  1:11  
	Well that starts with the book actually. Oh. Earlier this year, I published a book called Thinking functionally in PHP, on functional programming in PHP, during which I do write a lot of functional code, you know, that kind of goes with the territory. And one of the things I found was that the syntax for short functions, or arrow function, or can be short lambdas, or arrow functions, you know whatever name you want to give them, was really nice for functions where the whole function is just one expression. Which when you're doing functional code is really really common. And it was kind of annoying to have to write the long version with curly braces in PSR 2, PSR 12 format for functions that I wanted to have a name, but we're really just one line anyway does return, blah blah blah. It worked, got the job done. 

Larry Garfield  2:13  
	Then hanging around with internals people, friend of the pod Nikita Popov mentioned that it should be really easy. Now that we've got short functions, or short lambdas, do the same thing for named functions. And I thought about. Yeah, that should be doable just in the lexer, which means, even I might be able to pull it off given my paltry miniscule knowledge of PHP internals. So, I took a stab at it and it turned out to be pretty easy. Short functions are just a more compact syntax for writing functions or methods, where the whole thing is just returning an expression. 

Derick Rethans  2:56  
	Just a single expression?

Larry Garfield  2:58  
	Yes. If your function is returning two parameters multiplied together, it's a trivial case but you often have functions or methods that are doing. Just one expression and then returning the value. It's a shorter way of writing that. Mirrored on the syntax that short lambdas use. It doesn't enable you to really do anything new, it just lets you write things in a more compact fashion. But the advantage I see is not just less typing. It lets you think about functions in a more expression type way, that this function is simply a map from input to this expression, which is a mindset shift. So yes it's less typing but it's also I can think about my problem as simply an expression translation. And that's very very common in functional programming, also it helps encourage writing pure functions which functional programming is built on, and is just general good practice anyway, pure functions have no side effects. Take no stealth input from globals or anything like that. And their only output is their return value. You want most of your codebase to be that, and a short function is really hard to not make that, you can but it's hard not to. In one sense it's just syntax saving, in another sense it's enabling a more functional mindset, as you're writing code, which I'm very much in favor of. 

Derick Rethans  4:27  
	What is it that you're proposing then?

Larry Garfield  4:29  
	The specific syntax is the same function, signatures we have now: function name, open paren, parameters, closed paren, optionally colon return type. But then instead of open curly brace, it's just a double arrow expression semi colon, the exact same body style as a short lambda has. 

Derick Rethans  4:52  
	Or as a match expression?

Larry Garfield  4:54  
	Or a match expression or, you know, just array values, you know they're all examples of double arrow expression, which means this thing becomes that thing, wraps that thing. And so it's a very familiar syntax, and it can all go on on one line or you can drop it to the next line and indents depending on how long your code is. I've done both with short lambdas, they both read just fine. That is the exact same semantic meaning as open curly brace return that expression closed curly brace.

Derick Rethans  5:28  
	Yeah, and that wouldn't allow you to allow multiple statements in that case, because the syntax just wouldn't allow for it.

Larry Garfield  5:34  
	Correct. There's just like short lambdas. If you have multiple statements we'll get to that, that's not a thing. There is discussion of having short lambdas now take multiple lines. I haven't weighed in on that yet I'm not getting into that, at this point. 

Derick Rethans  5:50  
	Where came up more recently again was with the match expression for PHP 8.0, where, because you're limited to this one expression on the right hand side, there was originally talk of extending that to multiple lines, but then again that wouldn't met, that wouldn't match with the short lambdas that we have, or your short functions if they make it into the language. 

Larry Garfield  6:11  
	Right, all of that comes down to the idea of block expressions, which would be a multi lines set of statements that gets interpreted as an expression, which doesn't exist in PHP today, is how something like Rust works, but lifting that into PHP while there might be advantages to it is a big lift, and that needs a lot of thinking through to make it actually makes sense, it may not make sense. Again, I have not dug into that in much detail yet, other than to say, we really need to think that through before doing anything about it.

Derick Rethans  6:45  
	Yeah, especially about what kind of return value you'll end of returning from it because a return without the return keyword is tricky to. It's tricky to create semantic reasoning for and how to do that right.

Larry Garfield  6:56  
	Yeah, there's a lot of semantic trickery involved there so I explicitly avoiding that in this RFC, it's a nice simple surgical change.

Derick Rethans  7:05  
	You mentioned some use cases in the form of, they're useful in functional programming, but most people don't use functional programming with PHP, or maybe in your opinion don't use it yet. Would would be use cases for non functional programming with PHP for this new syntax?

Larry Garfield  7:22  
	Even if you're not doing formal functional programming. There's still a lot of cases where you have a function that just ends up being one line because that's all it needs to be. Especially if you're doing object oriented code. How many classes have you written that have, they're an entity class and they have eight properties, which means you have eight getter methods on them, each of which does nothing except return this arrow, you're removing three lines out of each one of those again using standard syntax conventions. By using a short function for that. You may also have a lot of refactoring techniques encourage producing single line functions or single line methods. For example, if you have an if statement, or a while statement or some other kind of check and check is A and B, and or C equals D, or some kind of complex logic there. Very common recommendation is alright break that out to a utility method, or utility function that can give a name to and becomes more self documenting. This is a good refactor and giving you a bunch of single line functions that are just really an expression. So, I'll write structure them as just an expression. My feeling is is more advantageous with standalone functions than with methods, but most of the logic applies to both equally well. 

Derick Rethans  8:51  
	In the case of setters and getters, that actually makes quite a bit of sense righ? 

Larry Garfield  8:55  
	It's just for getters for setters generally you set something, and then return this or return null, or something like that and that is a different statement, so it wouldn't work for setters. There are ways to click around that by calling a sub function there which I'm not actually going to encourage but you can do.

Derick Rethans  9:15  
	Yeah, I guess you could create a lambda.

Larry Garfield  9:17  
	And actually what you do is have a function that just takes a parameter and ignores and returns a second parameter. And then the body of your setter is calling that function with the Sep sabyinyo this row foo equals whatever, then your second parameter is this, and it ends up working. I don't actually suggest people do that.

Derick Rethans  9:39  
	It's also too complicated for me to understand what you're trying to say there so let's, let's not encourage that use of it.

Larry Garfield  9:46  
	I did it just to see if I could not because it's a good idea. 

Derick Rethans  9:50  
	Okay, your conclusion was, it's not a good idea. 

Larry Garfield  9:54  
	Cute hack. 

Derick Rethans  9:55  
	I saw in a discussion on the mailing list, some people talking about why is this using function and not fn. What is your opinion about that?

Larry Garfield  10:04  
	Mainly because using function was easier to implement initially. So that's what I went with; just the way some of the lexer rules are structured, it was a bit trickier to use, to do fn. And I figured go with the easy one. That said, Sara Goleman gave a patch that's takes care the FN part. So there are patches available for both. Personally I moderately prefer function, I think it has less confusion. And if you have to convert a function from one to the other, it's less work then. But I don't really care all that much so if the consensus is we like the feature but we want to use fn I'm good with that too. There's some interesting discussion around, we were saying, there are some people trying to push right now to have short lambdas also take multiple statements, or to have long lambdas, anonymous functions, do auto capture. And so this question is now okay, the double arrow versus the FN, which one means auto capture, which one means single expression. I haven't weighed in on that yet. It'd be sense to sort all of it out and make it all, logically consistent, but as long as things are consistent I don't particularly care which keyword gets used where.

Derick Rethans  11:20  
	By adding this feature to PHP, the syntax feature, is there a possibility for backward compatibility breaks?

Larry Garfield  11:27  
	I don't believe so. The syntax I'm proposing would be a syntax error right now, so there shouldn't be any backward compatibility issues. Other use case I forgot to mention before, is if you're doing functional style code. Then, very often want to branch, your logic. Very concisely without full if statements, people are used to Haskell, I use the pattern matching and stuff like that, I'm not proposing that here. But the new match statement in PHP eight zero is a single expression. That gives you a branching capability. And so that dovetails together very nicely to say: okay, here's a function branch, using a match statement based on its inputs. And it maps to a single expression. Could be a call to another function, could be just a single expression. It's just another place where it doesn't make possible, anything you couldn't do before. It just makes certain patterns more convenient.

Derick Rethans  12:25  
	So no BC breaks, but more use cases. Can you see what else could be added to this kind of style functionality for the future?

Larry Garfield  12:35  
	I think this syntax itself is very easily self contained, does one simple thing and does it well and there's not much room to expand on it. There's no reason to have closures on named functions that's not really a thing. One of the things I like about it though, is it dovetails nicely with some other things that are in flight. A teasier for future episodes I suppose I'm collaborating with Ilya Tovolo on enumerations that hopefully will support methods on enumeration values. It is an excellent case for single line expressions because there's not much else to do in a lot of cases. So you can end up writing a very compact enumeration that has methods that are single expression, and boom, you've got a state machine. I've been working on a pipe operator that allows you to chain functions together. You can now have a single line, single expression, function that is just: take input and pipe it through a bunch of other functions. And now you have a complex pipeline that is just one single expression function. Hopefully these things will come to pass. I still got quite a bit of time before eight one's feature freeze, so we'll see what happens, but to me all of these things dovetail together nicely. And I like it when functionality dovetails together nicely. That means you can have functions or functionality that has benefit on its own. But in tandem with something else they're greater than the sum of their parts and that's to me the sign of good language design and good architecture.

Derick Rethans  14:09  
	And we have been getting to some of that that PHP 8.0 with having both named arguments and promoted constructor arguments for example. 

Larry Garfield  14:17  
	Exactly. You talked about that several episodes ago. 

Derick Rethans  14:20  
	Would you have anything else to add? 

Larry Garfield  14:22  
	Like, the. A lot of the changes that have happened in PHP eight that have made, 7.4 and eight, that have made functional style code more viable and more natural. And that's the direction that I'm hoping to push more with my limited technical skills in this area, and better design skills and collaborating with others, but there's a lot of targeted things we could do in PHP 8.1 to make functional style code easier and more viable, which works really well in a request response environment. Which PHP's use cases are very well suited to functional programming. I'm hoping to have a lot of these small targeted changes like this that add up to continuing that trend of making PHP a more functional friendly language,

Derick Rethans  15:11  
	But you're not proposing to turn into a functional language altogether?

Larry Garfield  15:15  
	A strictly functional language? No, that would not make any sense. A language in which doing functional style things is easier and more natural? Absolutely. I think there's a lot of benefits to that, even in a world where people are used to doing things in an OO fashion. Those are not at odds with each other. Functional programming and object oriented code. A lot of the principles of functional programming are also principles of good OO code, like stateless services. That's a pure function by a different name. Ways to do a lot of those things more easily. I think is a benefit to everyone. Those who agree with me, I could use help on that, volunteers welcome.

Derick Rethans  15:54  
	As always, as always. Okay, thank you, Larry for taking the time this morning slash afternoon to talk to me about short functions.

Larry Garfield  16:02  
	Thank you, Derick and take care of PHP world.

Derick Rethans  16:06  
	Thanks for listening to this installment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me slash patron. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.


Show Notes
----------

- RFC: `Short Functions <https://wiki.php.net/rfc/short-functions>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
