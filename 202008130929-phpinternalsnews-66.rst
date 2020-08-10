PHP Internals News: Episode 66: Namespace Token, and Parsing PHP
================================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-08-06 09:28 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin066
   :Enclosure: media/pin-066.mp3

In this episode of "PHP Internals News" I chat with Nikita Popov (`Twitter
<https://twitter.com/nikita_ppv>`_, `GitHub <https://github.com/nikic/>`_,
`Website <https://nikic.github.io/>`_) about his Namespaced Names as a Single
Token, and Parsing PHP.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-066.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:16  
	Hi, I'm Derick, and this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 66. Today I'm talking with Nikita Popov about an RFC that he's made, called namespace names as token. Hello Nikita, how are you this morning?

Nikita  0:35  
	I'm fine Derick, how are you?

Derick Rethans  0:38  
	I'm good as well, it's really warm here two moments and only getting hotter and so.

Nikita  0:44  
	Same here. Same here.

Derick Rethans  0:46  
	Yeah, all over Europe, I suppose. Anyway, let's get chatting about the RFC otherwise we end up chatting about the weather for the whole 20 minutes. What is the problem that is RFC is trying to solve?

Nikita  0:58  
	So this RFC tries to solve two problems. One is the original one, and the other one turned up a bit later. So I'll start with the original one. The problem is that PHP has a fairly large number of different reserved keyword, things like class, like function, like const, and these reserved keywords, cannot be used as identifiers, so you cannot have a class that has the name list. Because list is also a keyword that's used for array destructuring. We have some relaxed rules in some places, which were introduced in PHP 7.0 I think. For example, you can have a method name, that's a reserved keyword, so you can have a method called list. Because we can always disambiguate, this versus the like real list keyword. But there are places where you cannot use keywords and one of those is inside namespace names. 

	So to give a specific example of code that broke, and that was my, my own code. So, I think with PHP 7.4, we introduced the arrow functions with the fn keyword to work around various parsing issues. And I have a library called Iter, which provides various generator based iteration functions. And this library has a namespace Iter\fn. So Iter backslash fn. Because it has this fn keyword as part of the name, this library breaks in PHP 7.4. But the thing is that this is not truly unnecessary breakage. Because if we just write Iter backslash fn, there is no real ambiguity there. The only thing this can be is a namespace name, and similarly if you import this namespace then the way you actually call the functions is using something like fn backslash operator. Now once again you have fn directly before a backslash so there is once again no real ambiguity. Where the actual ambiguity comes from is that we don't treat namespace names as a single unit. Instead, we really do treat them as the separate parts. First one identifier than the backslash, then other identifier, then the backslash, and so on. And this means that our reserved keyword restrictions apply to each individual part. So the whole thing. 

	The original idea behind this proposal was actually to go quite a bit further. The proposal is that instead of treating all of these segments of the name separately, we treat it as a single unit, as a single token. And that also means that it's okay to use reserved keywords inside it. As long as like the whole name, taken together is not a reserved keyword. The idea of the RFC was to, like, reduce the impact of additional reserved keywords, introduced in the future so in PHP eight we added the match keyword, which can cause similar issues, and in PHP 8.1 maybe we're going to add an enum keyword, and so on. Each time we add a keyword we're breaking code. The idea of this RFC was to reduce the breakage, and the original proposal, not just allowed these reserved keywords inside namespace names, but also removed the restrictions we have on class names and function names and so on. So you would actually be able to do something like class Match. Then, if you wanted to use the class, you would have to properly disambiguate it. For example, by using a fully qualified name, starting with a backslash or by well or using any other kind of qualified name. So you wouldn't be able to use an isolated match, but you could use backslash match, or some kind of things they just named backslash match. However, in the end, I dropped support for this part of the RFC, because there were some concerns that would be confusing. For example if you write something like isset x, that would call the isset on built in language construct. And if you wrote backslash isset x, that would call a user defined, isset function, because we no longer have this reserved keyword restriction. While this is like an ambiguous from a language perspective, the programmer might get somewhat confused. So if we want to go in this direction we probably want to add more validation about which reserved keywords are allowed in certain positions and I didn't want to deal with that at this point.

Derick Rethans  5:33  
	Also by removing it, you make the RFC smaller which gives usually a better chance for getting them accepted anyway. 

Nikita  5:40  
	Yes. 

Derick Rethans  5:41  
	You mentioned in the introduction that originally you tried to solve the problem of not being able to use reserved keywords in namespace names. It ended up solving another problem that at the moment you wrote this, you wasn't quite aware of. What is this other problem?

Nikita  5:57  
	The other problem is the famous or infamous issue of the attributes syntax, which we are having a hard time solving. The backstory there is that originally the attributes introduced the PHP eight use double angle brackets, or shift operators as delimiters.

Derick Rethans  6:18  
	I've been calling it the pointy one. 

Nikita  6:20  
	Okay, the pointy one. And then there was a proposal accepted to instead change this to double at operator at the start, because that's kind of more similar to what other languages do; they use one @, we will use two @s on to avoid the ambiguity with the error suppression operator. Unfortunately, it turned out that as initially proposed the syntax is ambiguous. All because of quite an edge case, I would say. So attributes in PHP are going to be allowed on parameters, and then parameters can have a type. You can have a sequence where you have @@, then the attribute name, then the type name, then the parameter name. And the problem is that because we treat each part of the name separately, between the backslashes, there can also be whitespace in between. So you can have something like a, whitespace, backslash, whitespace, b. Now the question is, in this case does the a\b belong to the attribute so is it an attribute name, or is only the a an attribute name, and \b is the type name of the parameter. Yeah that's ambiguous and we can't really resolve it unless we have some kind of arbitrary rule, and while the original proposal did introduce an arbitrary rule that the attribute name cannot contain whitespace, it will be interpreted in a particular way. And what this proposal, effectively does is to instead say that names, generally cannot contain whitespace.

Derick Rethans  8:01  
	Your namespace names as token RFC basically disallows spaces for namespace names which you currently can have?

Nikita  8:10  
	Right. So I don't think anyone intentionally uses whitespace inside namespace names. Or actually, you could even have comments inside them. But it is currently technically allowed, and one might introduce it as a typo. That means that this change is a backwards compatibility break. Because you can have currently whitespace in names, but based on static analysis of open source projects, we found that this is pretty rare. So I think we found something like five occurrences inside the top thousand packages. There is one other backwards compatibility break that is in practice I think much more serious. And that's the fact that it changes our token representation. So instead of representing just names as a string separates a string. We now have three different tokens, which is the one for qualified names, one for fully qualified names. So with everything backslash and one for namespace relative names, which is if you have namespace backslash at the start, and namespace I mean, here literally. So this is a very rarely used on PHP feature.

Derick Rethans  9:26  
	I did not know it existed.

Nikita  9:28  
	I also actually did some analysis for that I found very few uses in the wild. I think mostly people writing the static analysis tools know about that, no one else. But the other problem is that this breaks static analysis tools because they now have to deal with a new token representation.

Derick Rethans  9:47  
	We have been talking about these tokens. What are tokens on like the smallest level.

Nikita  9:51  
	PHP has, what's a three phase compilation process where first we convert the raw source code, which is just a sequence of characters into tokens, which are slightly larger semantic units. So instead of having only characters we recognize reserved keywords and recognize identifiers and operators, and so on. Then on the second phase, we have the parser which converts these individual tokens into larger semantic structures like addition expression, or an assignment expression or class expression and so on. Finally we convert the result of that which is a parse tree or an abstract syntax tree into our actual virtual machine instructions, our bytecode representation. 

Derick Rethans  10:41  
	And then PHP eight down to machine code, if the JIT kicks in. 

	I remember from a long time ago when I was in uni, is that there are different kinds of parsers and from what I understand PHP's parser is a context free parser. What is is a context free parser?

Nikita  10:57  
	Well a context free in particular is a term from the CompSci theory, where we separate parsers into four languages, into four levels, though I think this is not a super useful distinction, when it comes to language parsing. Because general context free parser has complexity of O(n^3). Like if you have a very large file, it will take very long to parse it could be with the general context free parser. So what we'll be do in practice are more like deterministic context free languages, which is a smaller set. The formal definition there are a set that can be parsed by deterministic push down automaton but I don't think you want to go there.

Derick Rethans  11:41  
	No, not today.

Nikita  11:44  
	Because in practice actually what we do in PHP is even more limited than that. So the parser we use in PHP is called LA/LR1 parser. So that's a look ahead, left to right, right most derivation parser

Derick Rethans  11:59  
	Also mouthful, but it's simpler to explain.

Nikita  12:02  
	I think really the most relevant parts of this algorithm to us is the "1". What the one means is that we can only use a single look ahead token. When we recognize some kind of structure, we have to be able to recognize it by looking at the current token, and that the next token. At each parsing step, those two things have to be sufficient to recognize and to disambiguate syntax. Actually even more restrictive than that but I think that's a good mental model. This is really, I think the core problem we often have when introducing new syntax, so this is the problem we had with the short arrow syntax, short closure syntax. There we would need not one token of look ahead but an infinite potentially in finite number of look ahead. 

Derick Rethans  12:55  
	In case the function keyword is still being used. 

Nikita  12:59  
	Yes, that's why we added the fn keyword to what the problem because with the fn keyword we can immediately say at the start of the start of the arrow function that this is an arrow function. And we will have similar problems if we introduce generics in the future, at which point we might actually just give up and switch to a more powerful parser. The parser generator we use, which is bison, has two modes. One is the LA/LR mode. And the other one is the GLR mode. They both use the same base parsing algorithm at first, but the GLR mode allows forking it. So if it encounters an ambiguity, it will actually like try to pursue both possibilities, which is of course a lot less efficient, but it allows us to parse more ambiguous structures.

Derick Rethans  13:50  
	Wouldn't that not cause a problem that has some cases because every token will be ambiguous, it will explode in like an exponential way?

Nikita  13:57  
	Yes, I mean it's worst case exponential. But if you like have a careful choice of where you are ambiguous, then you can see that okay with this particular ambiguity, you can actually get worse still linear for valid code.

Derick Rethans  14:14  
	That makes a decision between two possibilities pretty much at that stage.

Nikita  14:18  
	But I think the danger there is that one might not notice when one introduces ambiguities, or maybe not real syntax ambiguities, you do tend to notice those. But ambiguities on the parser level where the parser cannot distinguish possibilities based on the single token for get.

Derick Rethans  14:41  
	Was there in the past being ambiguities introducing a PHP language, that we fail to see? 

Nikita  14:46  
	I don't think so. I mean, because we use this parser generator. It tells us if we have ambiguities, so either in the form of shift/reduce or reduce/reduce conflicts. So we cannot really introduce ambiguities. We can it's possible to suppress them, and we actually did have a couple of suppressed conflicts in the past, this was for one particular very well known ambiguity, which is the dangling elseif/else. Basically that the else part of an ifelse. So if you have a nested if without braces. Then there is an else afterwards, the else could ever belong to the inner if or to the outer if, but this is like a standard ambiguity in all C like languages that allow omitting two braces.

Derick Rethans  15:35  
	That's why coding standard say: Always use braces. The patch belonging to this RFC has been merged. So it means that in PHP eight we'll have the longer or the new token names. Do you think that in the future we'll have to make similar choices or similar adjustments to be able to add more syntax?

Nikita  15:56  
	Yes. As I was already saying before, I think these syntax conflicts, they crop up pretty often. This is like very much not an isolated occurrence. If it's really fundamental. If there are any fundamental problems in the PHP syntax where we made bad choices. I think it's pretty normal that you do run into conflicts at some point. So, especially when it comes to generics using angle brackets, in pretty much all languages I deal with my major parsing issues when it comes to that. For example Rust has the famous turbo fish operator, where you have to like write something like colon colon angle brackets to disambiguate things in this specific situation. 

Derick Rethans  16:49  
	I just listened to a podcast on the Go language where they're also talking about adding generics, and they had the exact same issue with parsing it. I think they ended up going for the square bracket syntax instead of the angle brackets, or the pointies, as I mentioned. 

Nikita  17:04  
	Everyone uses the pointy brackets syntax, despite all those parsing issues because everyone else use it. For PHP for example, we wouldn't be able to use square brackets, because those are used for array access, and that would be completely ambiguous, but we could use curly braces. Well, people have told me that they would not appreciate that. Let's say it like this.

Derick Rethans  17:31  
	Is it's finally time to start using the Unicode characters for syntax? 

	No!

Nikita  17:38  
	Although I did see that, apparently, some people use non breaking spaces inside their test names, because it looks nice and is pretty crazy.

Derick Rethans  17:49  
	I like using it in my class names as well instead of using study caps.

Nikita  17:52  
	I hope that's a joke, you can ever tell.

Derick Rethans  17:54  
	I like using emojis instead. That's also a joke, but I do use it in my presentations and my slides just to jazz them up a little bit. 

Nikita  18:01  
	This is an advantage of PHP, because our Unicode support in identifiers works because the files are in UTF-8. That means that all the non ASCII characters have the top bit set, and we just allow all characters with a top bit set as identifiers. And that means that there is no validation at all that identifiers used contain only like identifier or letter characters, so you can use all of the emojis and whitespace characters and so on, you won't have any restrictions.

Derick Rethans  18:36  
	And that was possible for as long as PHP 3 as far as I know. It's a curious thing because this is something that popped up quite a lot when we're discussing, or arguing, about Unicode. You shouldn't allow Unicode because then you can have funny characters in your function names like accented characters and stuff like that, and then also always fun to point out that yeah you could have done that since 1997. Anyhow, would you have anything else to add?

Nikita  19:01  
	I don't think so.

Derick Rethans  19:02  
	Well thank you very much for having a chat with me this morning. And I'm sure we'll see each other at some other point in the future. 

Nikita  19:09  
	Thanks for having me once again Derick.

Derick Rethans  19:12  
	Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool, you can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.

Show Notes
----------

- RFC: `Treat namespaced names as single token <https://wiki.php.net/rfc/namespaced_names_as_token>`_
- `GLR Parser <https://en.wikipedia.org/wiki/GLR_parser>`_
- `LALR(1) Parser <https://en.wikipedia.org/wiki/LALR_parser#Overview>`_
- `Iter Library <https://github.com/nikic/iter>`_
- RFC: `Match expression <https://wiki.php.net/rfc/match_expression_v2>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
