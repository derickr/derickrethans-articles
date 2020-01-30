PHP Internals News: Episode 40: Syntax Tweaks
=============================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-02-13 09:03 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin040
   :Enclosure: media/pin-040.mp3

In this episode of "PHP Internals News" I chat with Nikita Popov (`Twitter
<https://twitter.com/nikita_ppv>`_, `GitHub <https://github.com/nikic/>`_,
`Website <https://nikic.github.io/>`_)
about a bunch of smaller RFCs.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-040.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:16
	Hi, I'm Derick. And this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 40. Again, I'm talking with Nikita. Perhaps we should rename this podcast to the Derick and Nikita Show at some point in the future. This time we're going to talk about a bunch of smaller RFC that he produced related to tweaking PHP syntax for PHP 8. Nikita, would you please introduce yourself?

Nikita Popov  0:42
	Hi, I'm Nikita and I do PHP core developement on behalf of JetBrains. We have a couple of new and not very exciting RFCs to discuss.

Derick Rethans  0:53
	Sometimes non not exciting is also good to talk about. Anyway, the first one that caught my eye was a RFC called static return type. So we have had return types for well, but what is special about static?

Nikita Popov  1:07
	So PHP has three magic special class names that's self, referring to the current class, parent referring to the well parent class, and static, which is the late static binding class name. And that's very similar to self. If no inheritance is involved, then static is the same as self introducing refers to the current class. However, if the method is inherited, and you call this method on the child class, then self is still going to refer to the original class, so the parent. While static is going to refer to the class on which the method was actually called.

Derick Rethans  1:51
	Even though the method wasn't overloaded

Nikita Popov  1:54
	Exactly. In the way one can think of static as: You can more or less replace static with self. But then you would have to actually copy this method inside every class where.

Derick Rethans  2:09
	You have not explained the difference between self and static. Why would you want to use static as a return type instead of self?

Nikita Popov  2:17
	There are a couple of use cases. I think the three ones mentioned in the RFC are. The first one is named constructors. So usually in PHP, we just use the construct method. Well, if we had to give this method, a type, a return type, then the return type will be static. Because of course, the constructor always returns while the class you're actually constructing, not some kinda parent class. And named constructors are just a pattern where you use a static method instead of a constructor, for example, because you have multiple different ways of constructing an object and you want to distinguish them by name.

Derick Rethans  2:57
	Could we also call those factory methods?

Nikita Popov  3:00
	Yeah, that's also related pattern. So for named constructors, you usually also want to return the object that it is actually called on.

Derick Rethans  3:09
	It makes sense attached there because of that then creates a contract that you know that is named constructor is going to return that same class and not something else. Because there's no requirements that would otherwise require that same class, like you'd have to construct.

Nikita Popov  3:22
	Exactly, yeah. The other pattern. These I think maybe that popularised by PSR, maybe 7 or something, the HTTP request object interface, the object is actually immutable. And the way you change it is by calling it with something method. And this method is going to return you a new object with this particular bit of information replaced. And again, usually for these kinds of API's, you also want to, you want to return the class that the methods actually call them. So if you extend this kind of API, you don't want to get objects of the parent class back. Right the third way, and I think the like by a pretty large margin, the most common one, is just normal fluid mthods. So where each method returns this. This is always an instance of static. So if you extend the class then this is going to be the extending class of the parent class. For this particular case, in PHP there is also a different convention where instead of returning static, you actually need right, return $this. So you use $this as a one word type, a special types indicates this type of method. So static would be a slightly weaker form of that. But we might still add the special $this case in the future.

Derick Rethans  4:51
	Because static would only enforce it's the same class but not to the same object.

Nikita Popov  4:56
	Exactly.

Derick Rethans  4:56
	Are you intending to add that to this RFC?

Nikita Popov  4:59
	I like to keep RFCs like different issues separated.

Derick Rethans  5:03
	It makes it easy to talk about them and get them accepted or not.

Nikita Popov  5:06
	I'm not totally convinced on the $this thing, because static is in the end. I mean, we already allow self return types, we allow parent. It make sense to allow static. But this is not really a type or some kind of extra contact contract on top of the type system. And I'm not sure it makes sense to open this position.

Derick Rethans  5:28
	Okay, in which position would you be able to use the static keyword? You've already mentioned the return types, there are other places as well?

Nikita Popov  5:36
	No, you can only use it in return types, it would simply not be sound. So it would violate the liskov substitution principle in any other place. The reason why you can use static in return types, is that static is basically a restriction on each inheriting class. So in your original class, static is the same as self. Then in the inheriting class, static is again the same as self, but in the inheriting class. And the inheriting class is a subclass or a subtype of the parent class. So this is allowed by the liskov substitution principle or by our variance rules. If you do the same things for parameters, you would also go from having a parameter for the parent class to parameter for the child class. So you would restrict the amount of inputs that are allowed in this parameter. And that's invalid. And the same argument also goes for properties.

Derick Rethans  6:37
	The RFC also talks a little bit about variance and subtyping. How is static considered here differently from self, or if you just explained exactly that?

Nikita Popov  6:46
	static is considered a sub type of self. If you have a parent method that uses a self return type, you can have a child method that uses a static return type, because static is ta further restriction. So self allows, still allows you to return the parent class, while static does not allow it. So you restrict the amount of return values and that's valid. While going the other direction. So replacing the static type and the parent method ,with the self type in the child method that would not be valid. Because, you make the amount of low values larger.

Derick Rethans  7:21
	And that is exactly the same as the other variance rules that we have since PHP seven for of course. The the last thing the RFC mentions or actually don't quite remember whether it mentions is, is whether you can also use static as part of a union type.

Nikita Popov  7:35
	So yes, you can.

Derick Rethans  7:37
	Okay, that's the simple answer. I like simple answers.

Nikita Popov  7:40
	Together with the other restrictions. So that union type has to be in the return type position. But apart from that, you can.

Derick Rethans  7:47
	That's good to hear.

Nikita Popov  7:48
	There is actually one more tricky thing regarding the property types. Without a lot of static and property types because as I mentioned, it would violate our variance rules. But unfortunately we have the extra issue that we also have static properties. So if you write public static foobar, then is that static for a static property or for a static type?

Derick Rethans  8:14
	Right, because we don't enforce that a static goes or goes before or behind public, private, or protected.

Nikita Popov  8:21
	Yeah.

Derick Rethans  8:21
	At least not in the syntax. I mean, I think coding standards actually do most of the time require the static to be before.

Nikita Popov  8:27
	Even the coding standards they would require you to write it as public static, not this static public.

Derick Rethans  8:35
	Oh, really? Okay. I thought was the other way around. Yeah, that is difficult. Because then you don't know which static is meant here.

Nikita Popov  8:41
	Yes, and we just allow on the, disallow it on the grammar level. It's actually a bit ugly, because we have to like duplicate the whole type grammar two times, once to include static, once to not include it, just to deal with this ugly of conflict.

Derick Rethans  8:56
	That's what happens when you come with something clever. You need clever workarounds.

Nikita Popov  9:00
	So it's unfortunate that the static keyword has like three or maybe four completely different meanings in PHP. Simply I think, simply because people wanted to re use a keyword, instead of introducing a new one

Derick Rethans  9:15
	Because introducing new keywords might end up meaning breaking people's code.

Nikita Popov  9:19
	On the downside, reusing keywords makes code confusing, because well, at least I got the impression that some people find the use of static for late static binding somewhat confusing. And I can also see if you see methods that has signature public static, whatever and return static, and you're not like super familiar with what all of that means.

Derick Rethans  9:46
	And that is quite a common pattern because this named constructors are static methods that return static. Let's move on to the next one, which is a tiny RFC that you came up with, which is the Class Name Literal on objects. What does this do?

Nikita Popov  10:03
	The syntax where you write a class name, then the double colon class. And that just returns you the fully qualified class name. For example, have a use statement for that class, you get back the full name instead of the short name. I think we've had this since PHP 5.5. And it's a great feature because it's like makes it clear where you're referencing the class and not just some random string. And that means, for example, that that IDE refactorings could work better and so on.

Derick Rethans  10:35
	Okay.

Nikita Popov  10:36
	The actual RFC is very simple. Currently, the class syntax is only allowed on like literal class names, but you can take an object variable and get the class of that object using the syntax.

Derick Rethans  11:48
	However, PHP has a function for that already, which is called get_class() right?

Nikita Popov  10:52
	Exactly. This is essentially just syntax sugar for get_class(). The reason why we want to have the syntax sugar is really not so much that writing get_class() is particularly hard, but just that people expect it to be there. This class syntax looks a lot like a constant access, like a class constant access. So it looks like every class has a magic constant called class. Usually you are able to access class constants on objects. So you can write object, the double colon, and the constant thing. And that works. So in that case, we just take the class of the object and access it on that class. For consistency reasons, it only makes sense that you can do the same with this particular magic concept as well. There's really all the motivation

Derick Rethans  11:43
	Originally the class literal colon colon class is resolved at compile time. Of course, that can't happen on object colon colon class. Is that still true or no longer?

Nikita Popov  11:53
	So it really was true in the first place. For normal class names of cours is resolved at compile time. Actually one of the like gotchas with the syntax is that some people expected to validate that the class actually exists. So they expect that this gets auto loaded and they get an error if it doesn't exist, doesn't happen. So you can reference some non existing class with this syntax just fine. The usually your IDE is going to show a warning for that. I mean, as we just discussed, we also have a couple of magic class names. So we have self, parent, and static. The static one in particular, also always has to be resolved at runtime, because we don't know what the what class the method is actually going to be called on. Actually, self and parent also sometimes have to be resolved at runtime. And there are two cases where that can happen. One is if you use traits, because in that case self refers to the using class, not to the trait. So in closures the self class, refers to the bound scope. The bind to method, there is like the last argument on, is the scope you're using. So in those cases, it's already dynamically resolved.

Derick Rethans  13:09
	Okay. The RFC mentions one specific area where you can't use colon colon class. In which situation can you still not use colon colon class on objects?

Nikita Popov  13:20
	You can always use it on an object. I think what you're referring to is that normally, for normal class constants, you can also put the class name inside the string. I mean, put the string class name inside the variable and then access the constant on that variable.

Derick Rethans  13:38
	Oh, right. Yes.

Nikita Popov  13:39
	For the double colon class syntax, we don't want to allow that. Because, well, first this is kinda useless, because it will just return you back the same string you gave it. And I think in that case, the fact that the class name is not validated, this is especially confusing.

Derick Rethans  14:00
	Okay, that makes sense. So you can only call colon colon class on literal class names that you already could, as well as on variables that contain an object?

Nikita Popov  14:09
	That's right? Yeah.

Derick Rethans  14:10
	That sounds great. Does it show up differently in reflection?

Nikita Popov  14:13
	This magic class constant actually doesn't show up in reflection at all. It looks like a constant both it's really a special syntax that just happens to share the look with constants.

Derick Rethans  14:24
	Do you expect any controversies about this?

Nikita Popov  14:27
	I don't think so.

Derick Rethans  14:28
	I don't think so either. I can't really see anything that people could complain about too much. I think. I however, do think that for the next RFC that you came up with the variable syntax tweaks, there will be a little bit of haggling about whether this is good idea to do. In PHP seven, zero, we got this uniform variable syntax. Could you give a brief reminder of what it was about?

Nikita Popov  14:48
	That was about, well fixing a couple of syntax inconsistencies when it comes to variables syntax. So variable syntax in PHP is extremely, extremely magic. Like our expression, syntax nice and regular. But the variable syntax is a huge assortment of special rules and that RFC made those rules little bit less special at more regular.

Derick Rethans  15:17
	From what I understood we missed a few inconsistencies that we probably also should have addressed in that RFC. And that is what you know, trying to tweak again?

Nikita Popov  15:24
	All of these remaining consistencies are like really, really minor things and edge cases. But weirdly, all or at least most of them are something that someone at some point ran into, and either open the bug or wrote me an email or pinged me on Twitter. So people somehow managed to still run into these things.

Derick Rethans  15:52
	The RFC mentions four specific things that we've missed. What is the first one?

Nikita Popov  15:57
	Yeah, so it's probably going to be somewhat hard to talk about some of these examples.

Derick Rethans  16:02
	I know because I think some of them make no sense whatsoever.

Nikita Popov  16:05
	Yeah.

Derick Rethans  16:06
	Because how do you call a method on the string?

Nikita Popov  16:07
	Context for this one is I have a nice little extension called scalar objects, which allows you to more or less define methods on strings, on integers, on arrays and so on. In with the uniform variable syntax, we have allowed calling methods on string literals. That actually makes no real sense with baseline PHP. But if you're using scalar objects, then this is a useful feature because you can do something like take a string that rule and call length on it, while otherwise we'll have to wrap it in brackets.

Derick Rethans  16:45
	So it's just a syntax change pretty much.

Nikita Popov  16:48
	Well, what this one particular is about that right now, this works if it's a literal string, but if you have any variables inside it than suddenly stops to work, which is just a very.

Derick Rethans  16:59
	So it is the interpolated strings inside double quotes, the dollar variable name syntax. That's the problem that?

Nikita Popov  17:05
	Yeah.

Derick Rethans  17:06
	The second one is called constant derefenceability, which is a word I can't pronounce. And my text edit says it's not a word. So what do you mean by it?

Nikita Popov  17:14
	That's a good question. I think the term is more or less picked up from C, where we have pointers. And we can dereference pointers to access what the pointer points to. So that's the star operator in C. In PHP, we use the term dereference to also access some kind of structure in some way. For example, to access an array element, so array dereference, or to access an object properties, object reference and so on. That particular one is, I think two things. One is that you can, for example, access the first character of a constant. So read the constant name then brackets zero. Well, maybe even not the first time I can think of a better example. Um, you haven't the constant that contains an array, and you want to access a specific key on that area. That's something you can do already you right now. The same syntax does not work if the constant is in magic constant. What also doesn't work is if you use the our alternative array access syntax. So we have the square brackets, that's what people should use. And we have the curly braces, which is the alternative way to access arrays and which is actually deprecated as of 7.4. I'm not totally sure that that's going to be removed in PHP 8 or not. If it's going to be removed, then this part is a moot point. But yeah, this is again, I think, from a practical perspective, not really interesting. The only situation where I think this is useful is again, of course scalar objects, because it means you can call the methods on the constant

Derick Rethans  18:57
	Okay, which in syntax is the grammar currently disallowed doing that.

Nikita Popov  19:00
	Currently it's disallowed and that would allow it.

Derick Rethans  19:03
	A third one is related. I think it's a class constant dereferenceability.

Nikita Popov  19:07
	So someone complained about this one on Twitter. I don't know how they ended up trying to do this. Something you can do right now is, you can access a static property. And then you can interpret the content of that static property as a class name, and access another static property on that. So you can change chain these static property accesses. For some reason, the same does not work with class constant access. So static property accesses can be chained. But class constant accesses can't be. Again, for no particular reason, this change would allow that to happen as well.

Derick Rethans  19:41
	This is even a change that makes sense without having to use scalar objects.

Nikita Popov  19:45
	That one is. I wouldn't write that kind of code, but it logically makes sense.

Derick Rethans  19:50
	And then the last one is, in the RFC is called arbitrary expression support for new and instanceof.

Nikita Popov  19:56
	Yeah, so this is probably the only one that's actually useful for something. PHP has well, a bunch of places where usually you have to place either an identifier or a namespace name, but class name, method name, or a property name, and so on, or even the variable name. For all of these places, we usually support some kind of special syntax to instead use a general expression. For example, some of the variable with a static name, you can use curly braces to use a dynamic name instead.

Derick Rethans  20:26
	I think for new, we did it at some point already.

Nikita Popov  20:29
	For new, this doesn't exist yet. So you can use a variable as class name, but you can't actually compute the class name as part of the expression.

Derick Rethans  20:40
	I think what I was referring to, is you can use braces around the whole new class extension, so you can call methods. So that's that's what I meant, but this is specifically using an expression behind new.

Nikita Popov  20:51
	Yeah, so these are like two things. One is whether you use an expression for the new class name, and the others for the use the new itself as an expression. And yeah, the same, so yeah, right now, we don't support that for new. And we also don't support it for instanceof, so the the right hand side, which consists of that as the class new. The RFC just proposes to allow an expression and parenthesis in there. And this kind of stuff is, again, not well not particularly useful. But it is useful for things like code generation, where you may have to insert arbitrary expressions sets up your coins. And there are actually some nice hacks that you can use right now. So you can use a variable with a complex expression inside it, where you assign to the variable itself and then return its name.

Derick Rethans  21:42
	I don't think I understand this. You're saying you can construct a string with a complex expression in it.

Nikita Popov  21:48
	Not a string. You you write something like new variable, but with a curly brace syntax, and in there you return you start off with the string containing some kind of dummy variable name, and then you concatenate that with an empty string. But that empty string is computed by doing the assignments to the variable name that you're actually going to return.

Derick Rethans  22:12
	I still don't understand this. You know, what I'm going to do is I'm just going to link to a example for this in the show notes.

Nikita Popov  22:19
	It's not really important. You can just cut off this part.

Derick Rethans  22:22
	Yep sure, I can do that too-perfectly fine.

Nikita Popov  22:24
	Nice hack.

Derick Rethans  22:24
	But let's not teach too many hacks to people such I think. Thank you for taking the time with me today, Nikita to talk about a bunch of little RFCs that you've written. Perhaps by the time this podcast comes out, we've started voting on them and we'll see what happens to them.

Nikita Popov  22:37
	Thanks for having me once again.

Derick Rethans  22:41
	Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening and I'll see you next week.


Show Notes
----------

- RFC: `Static Return Type <https://wiki.php.net/rfc/static_return_type>`_
- RFC: `Class Name Literal on Object <https://wiki.php.net/rfc/class_name_literal_on_object>`_
- RFC: `Variable Syntax Tweaks <https://wiki.php.net/rfc/variable_syntax_tweaks>`_
- RFC: `Uniform Variable Syntax RFC <https://wiki.php.net/rfc/uniform_variable_syntax>`_


Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
