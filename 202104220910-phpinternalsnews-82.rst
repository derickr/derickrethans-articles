PHP Internals News: Episode 82: Auto-Capturing Multi-Statement Closures
=======================================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-04-22 09:10 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin082
   :Enclosure: media/pin-082.mp3

In this episode of "PHP Internals News" I chat with Larry Garfield (`Twitter
<https://twitter.com/Crell>`_) and Nuno Maduro (`Twitter
<https://twitter.com/enunomaduro>`_, `GitHub
<https://github.com/nunomaduro>`_, `Blog <https://nunomaduro.com>`_) about the
"Auto-Capturing Multi-Statement Closures" RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-082.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:14  
	Hi, I'm Derick. Welcome to PHP internals news, a podcast dedicated to explaining the latest developments in the PHP language. This is episode 82. Today I'm talking with Nuno Maduro and Larry Garfield. Nuno, would you please introduce yourself?

Nuno Maduro  0:30  
	Hi PHP developers. My name is Nuno Maduro, and I am software engineer at Laravel, the company that owns the Laravel framework, and I have created multiple open source projects for the PHP community, such as Pest PHP, Laravel zero, collusion and more.

Derick Rethans  0:48  
	Alright, and Larry, could you please follow up on that.

Larry Garfield  0:51  
	Hello world, so I'm Larry Garfield. You may know me from several past podcasts here, various work in the PHP fig, and all around gadfly and nudge of the PHP community.

Derick Rethans  1:03  
	Good to have you again Larry and good to have you here today, Nuno. The RFC, that we're talking about here today is to do with closures, and the title of the RFC is auto capturing multi statement closures, which is quite a mouthful. Can one of you explain what this RFC is about?

Nuno Maduro  1:20  
	As you said, the RFC title is indeed auto capturing multi statement closures. But to make it simple, we are really talking about adding multi line support to the one line arrow functions that got introduced it, in PHP 7.4. Now, this new multi line arrow functions have exactly the same features as the one line arrow functions, so they are anonymous, locally available functions; variables are auto captured lexically meaning that you don't actually need the use keyword to manually import the use of variables, they just get auto captured from the outer scope. And the only difference really is one line arrow functions have a body with a single expression. This RFC actually allows you to use a full statement list that possibly ends with a return.

Derick Rethans  2:18  
	Excellent, what the syntax that you're proposing here?

Nuno Maduro  2:22  
	Well, as you may know, one line arrow functions have the syntax, which is fn, parameter list, and then that arrow expression thing, and this new RFC proposes that, optionally, developers can pass a curly brackets with statements, instead of having that arrow expression syntax. Now, this curly brackets with statements, simply denotes a statement list that potentially ends with a return. Concerning the Auto Capture syntax, we will be just reusing the Auto Capture syntax and feature that already exists on one line arrow functions, meaning that you don't need the use keyword to manually import variables. And of course, the syntax itself, in the in the feature, works the exactly same way. Concerning the syntax, it's also important to mention that this RFC was done in combination with the short functions RFC from Larry, but I think I'm going to let Larry speak about that later on this episode.

Derick Rethans  3:26  
	What's the main idea behind wanting to introduce this auto capture multi statement closures. Because from what I understand the arrow is now gone, and it's been replaced by just a function body within curly braces. But why would you want to extend a single expression to like multiple statements?

Nuno Maduro  3:44  
	Well, we all know that long closures in PHP can be quite verbose, even when you want to perform a simple operation. And this is largely due to the syntax syntactic boilerplate that you need to use when using long closures to manually import all the variables with the use keyword. Now, while one line arrow functions solve this problem to some extent, there are also a few cases that you might want to leverage the simplicity of auto capturing, but using two or three lines in a statement list. And one example I can think of is that when you are within a class method with multiple arguments on it, you might want to just return a closure, using all the arguments of that method, and actually using the use keyword in least all of the arguments is in this case, redundant and even pointless. It is also some use cases for example with array filter or similar functions, where they use keyword just adds some visual complexity to the code. We believe that the massive majority of PHP developers are really going to appreciate this RFC, as it makes just the code more simpler and shorter. The community loved these changes really, as proven on property promotions, one line arrow functions, or even the null safe operator.

Derick Rethans  5:10  
	And I think this is something that the PHP language itself is moving forwards to, right. I see in the RFC that you're, you're trying to make sure that syntax stays consistent with itself and you use a word called sigils here for some of these things. What were the important parts of making sure that the syntax stays the same?

Larry Garfield  5:30  
	So this actually relates to the short functions RFC. So the short functions RFC, as we discussed in previous episode, I'm trying to make it possible to write single line expression named functions in a more compact way. And that is a different problem with different syntax than auto capturing closures, which is why we needed to work together on these, because we want to make sure that the syntax for the various different kinds of functions that PHP supports, are all consistent with each other, and this piece of syntax indicates this thing consistently, rather than: in some cases this syntax means this, and other cases that means this other thing. Didn't want to end up with that kind of mess. After the short functions RFC, one part of the feedback was, there are people working on auto capturing long closures. Y'all should work together and make sure that the syntax plays nicely. And, for various reasons I just sat on it for a while, before finally getting hold of Nuno earlier this year, and we got together and talked and made sure that what he was doing and what I was doing, the syntax complemented each other. We ended up with in our discussion and analysis is that you can have a function that is named or anonymous. It can have Auto Capture or not. And it can either be a single expression with no return statement because it just evaluates to that value, or it can be a list of statements, one of which could be returned, potentially multiple could be returned. And right now we have syntax in PHP to support three possible combinations. But there's actually eight possible ways to combine those two, those three variants. We looked at all right, we have three of the eight, which of the others makes sense to have, and the two that makes sense to have are: short functions named function, no closure, and I say an expression body is what my RFC does. And then, anonymous Auto Capture statement list, which is what Nuno's RFC does. That rounds out that list, and the other combinations, I'm not sure actually have a purpose. Technically could exist and this also then means, if in the future we wanted to add those, then we know exactly what the syntax is going to be for those and what they would mean it's all just following that established pattern, so it makes it really easy for people to learn and understand. So what we end up with, out of these two RFCs together, which can stand alone, and they're going to be put to votes separately, but as I said complement each other. If you have: something, double arrow, expression, that consistently throughout the language ends up meaning evaluates to this expression. Short lambda means: you know, this function evaluates to this expression, a named function, double arrow expression, array key double arrow expression, a match statements, and so on and so on and so on, double arrow always means evaluates to expression. And the key word function means declaring a function named or anonymous, that does not auto capture anything. The case of a named function, there's nothing to capture; in the case of a closure, you'd have the manual capture with a use statements exactly like we've had since 5.3. And the FN keyword means, this is a function that is going to Auto Capture. But oh fn statement list with curly braces. I know that means: Auto Capture, statement list, or keyword function with an arrow: I know that means not auto capturing expression, and all these combinations then just make sense and they're easy to learn and they fit together well. That's really what we were trying to make sure, that between these two RFCs we ended up with that consistent set of rules around the syntax that was not designed that way originally but plays out in that way, and we can now make sure it stays playing out in that way that is internally consistent, and therefore easy to learn, easy to document and so on.

Derick Rethans  9:35  
	Because all the things that you just mentioned, they're already in place for existing syntax.

Larry Garfield  9:40  
	Correct. And so we're just taking existing syntax that means a thing, and combining those existing pieces of syntax in a new way. In some cases that syntax didn't necessarily mean that deliberately, for example, the FN keyword on short lambdas, was not added for the purpose of declaring Auto Capture. It was added because we needed to have some kind of syntax there to keep the lexer happy, fine, but now that it's there, we can say: all right, that is going to now mean auto capturing function, because that's something consistent with the language as it is and the language as it evolves into.

Derick Rethans  10:14  
	Is the intention of this new syntax to replace long closures?

Larry Garfield  10:19  
	Not entirely. I suspect, a great many use cases of long closures could get away with then using the auto captured version, but there's no plan to remove the long closure version. Part because there are cases you do need the manual closure, particularly, it's still the only way to capture a variable by reference. All the other versions are by value, which 99% of the time is what you want, but in that other 1% If you do need to by reference, then you've still got the long version.

Derick Rethans  10:48  
	The long version that uses the use keyword.

Larry Garfield  10:51  
	Right, and then you're manually capturing things, are cases where you would want to, you know, use the same variable name inside and out but not refer to the same variable, so in that case you can use the long version, and then you don't have that collision. In practice however, the only languages I know of, that have explicit capture on closures are PHP and C++. As far as I know, every other language, including the other major scripting languages, Auto Capture. We're really the oddball out, and in practice, I think using Auto Capture is going to be fine. It's going to be easier, and we're not going to introduce any substantial amount of new bugs around it. The place where that might cause issues, is if you have a long function with a long anonymous function in the middle of it with Auto Capture and you can't keep track your variables, in which case you're already doing it wrong anyway, you should have shorter functions and shorter closures. A real use case for this is: I have a function that's a closure that's two or three lines long, because not everything in PHP can be an expression, sometimes it has to be a statement. So okay, this thing is going to be two lines long instead of one line long, but I don't want to have to convert to the super verbose long version and manually declare all of my imports, I just want to add a second line. And so this makes that use case, a lot easier and a lot more convenient.

Derick Rethans  12:11  
	I remember when discussing the match operator, which I think I also spoke to you about?

Larry Garfield  12:17  
	Match is the one you spoke to yourself about.

Derick Rethans  12:19  
	That's what it was, yes.

Larry Garfield  12:20  
	It was a fun episode.

Derick Rethans  12:22  
	When I discussed the match operator with myself, I think I looked at whether it was possible to extend a match expression with having multiple statements on the right hand side as well, where it is currently: it's the arrow with a similar expression. Is this something that you'd be looking at to tie this auto caption closure style into as well?

Larry Garfield  12:42  
	It's a related issue that has been discussed mainly around match, around supporting multi line expressions. And that'll be some kind of syntax which hasn't been defined yet to list a series of statements, which can then be wrapped up together and have a final statement that is returned, and then the whole thing gets evaluated, and can be used in place of an expression like in a match statement or a function body. If we had a syntax for multi line expressions, that would be an alternative way to get to the same place this RFC gets to, because you could say: FN, parameters, double arrow, multi line expression, with whatever syntax that ends up being. And that gets you essentially the same thing at the end of the day. Is that good or bad, I'm kind of torn on it. What we point out in the RFC, is this syntax for auto capturing multi line closures, gives us a kind of roundabout way to put a multi line body into a match arm, where what you, the single expression, that the match arm evaluates to, is a multi line closure with Auto Capture that you then immediately self execute. The syntax for that is a little bit quirky. It looks kind of like older style JavaScript. We have parenthesis, function definition, closed parenthesis, open paren, close paren, so it just executes immediately. It's not ideal for that use case. Personally I think multi line match expressions, or multi line match arms, are a rare enough need that on the rare occasion you need it, this would be good enough. And if it's not good enough, you really should break that logic out to a separate function anyway and just call that. Not everyone agrees with that. So that's more a more of an interesting side effect of this RFC than a goal per se. One have to use it in that fashion I probably not will not use that in that fashion, very often, but it's, we now have a solution for that use case, if you actually have that use case, and we don't need any dedicated syntax for that.

Derick Rethans  14:46  
	That could be part of a future RFC if people still feel inclined, that they need that. Talking about things in the future, is there any future scope with this RFC as well?

Nuno Maduro  14:57  
	There is really nothing planned on this RFC is future scope. Yet, there is something that I will like to personally explore in the future, that now we have this fn keyword, that means Auto Capture, or access to the outer scope. And I think something would be very cool, is to explore named functions in the way that they are declared globally, with something like fn get name, and then that function would be able to access the outer scope, but again this is something that personally I would like to explore but it's not included in this RFC. I just plan to explore this in the future now that we have this possible combinations that Larry just explained.

Derick Rethans  15:43  
	It's always interesting to see what people think of when you post RFC to the mailing list.  What sort of where the biggest arguments against introducing this new syntax?

Larry Garfield  15:54  
	It's interesting. For both of these RFCs both my short functions, and now the Auto Capture multi line, the feedback from the public community on Reddit, on Twitter and so forth has been extremely positive people love: Oh, I can write less syntax and get stuff. It's been not universally but overwhelmingly positive. The feedback on the mailing list has been decidedly mixed with some people saying, cool this you know I've been waiting for that, and others saying: Why? Been push back: If your capture statement is that complex you, you're doing it wrong anyway. Or, if you do have Auto Capture, rather than explicit, your odds of capturing something you don't intend to are higher. And so you can introduce weird bugs that way.

Derick Rethans  16:42  
	Which aren't really arguments against having a multi line out to capture closure, with two or three statements. I mean if you're putting 50 lines in there then sure, you can make that argument I guess.

Larry Garfield  16:53  
	Exactly, and that's kind of our response is: if you have a complex enough piece of code that Auto Capture becomes problematic. You have a complex enough piece of code that you really should be manually capturing, or just refactor your code so you don't have that much complexity. That since it kind of becomes a good indicator of need to refactor. Then there's always the argument of: why should you add more syntax for anything, you know, we've got one syntax let that rule everything and that's that comes up with every RFC. Points are valid, to an extent, but I think the convenience factor of being able to write code more naturally with less effort that does stuff that right now is just clunky, is a stronger argument, especially given that most other languages don't have manual capture and get along fine. People have mentioned JavaScript as an example where the Auto Capture used to be highly problematic, then resolved with an extra keywords you can declare a variable inside a closure with let, that is then locally scoped and overrides anything in a parent scope. I don't think PHP needs that in part because we don't use closures as overwhelmingly as JavaScript does, and honestly that problem has kind of gone away in JavaScript, as they've introduced real classes, and other more traditional techniques that obviate the need for those kind of closure inside closure inside closure inside the closure nonsense. Python doesn't actually have multi line lambdas as far as I'm aware, because they have named functions that are scoped local. Ruby, as far as I know just does Auto Capture and doesn't have any special syntax around it. So, I have not heard of them having any problems. As I said, C++ has manual capture and that's the only one I can think of that has it. I think looking at other languages, the problems people have pointed out are more hypothetical than real, and I'm hoping that, you know, voters on the list will see all right. This makes life easier and the problems with it are hypothetical, not real problems that we've seen in the wild. So let's just make life easier for people.

Derick Rethans  19:05  
	Is there any chance of this breaking BC, somehow?

Larry Garfield  19:08  
	It shouldn't, the syntax right now would be syntax error. I don't see any, any BC breaks possible.

Derick Rethans  19:16  
	That's always a good thing isn't it. 

Larry Garfield  19:18  
	Yes. 

Derick Rethans  19:19  
	You were talking about appealing to the voters on the mailing list, which have the right to vote on features usually. When do you think you will be putting this up for a vote?

Larry Garfield  19:29  
	Probably around the end of April. We can put probably both RFCs up for a vote, you know, let the chips fall where they may. As you said both RFCs are stand-alone. If one pass and the other fails everything still works. Obviously we both like both the pass.

Derick Rethans  19:43  
	And with both you mean: both this RFC, which is the output capturing multi statement closures RFC, as well as the short functions RFC that we spoke about in episode 69. 

Larry Garfield  19:53  
	Correct. 

Derick Rethans  19:54  
	Thank you for taking the time today to talk to me about the new RFC that you're proposing.

Larry Garfield  20:00  
	Thank you, Derick always good to talk.

Nuno Maduro  20:01  
	Yeah, thank you so much for having me.

Derick Rethans  20:06  
	Thank you for listening to this instalment of PHP internals news, a podcast, dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patron. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening and I'll see you next time.


Show Notes
----------

- RFC: `Auto-Capturing Multi-Statement Closures <https://wiki.php.net/rfc/auto-capture-closure>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
