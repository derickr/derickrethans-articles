PHP Internals News: Episode 92: First Class Callable Syntax
===========================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-07-22 09:20 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin092
   :Enclosure: media/pin-092.mp3

In this episode of "PHP Internals News" I chat with Nikita Popov (`Twitter
<https://twitter.com/nikita_ppv>`_, `GitHub <https://github.com/nikic/>`_,
`Website <https://nikic.github.io/>`_) about the "First Class Callable Syntax"
RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-092.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:14  
	Hi, I'm Derick. Welcome to PHP internals news, the podcast dedicated to explaining the latest developments in the PHP language. This is Episode 92. Today I'm talking with Nikita Popov about a first class callable syntax RFC that he's proposing together with Joe Watkins. Nikita, would you please introduce yourself?

Nikita Popov  0:36  
	Hi, Derick. I'm Nikita and I am still working at JetBrains. And still working on PHP core development.

Derick Rethans  0:43  
	Just like about half an hour ago when we recorded an earlier episode. 

Nikita Popov  0:47  
	Exactly. 

Derick Rethans  0:48  
	This RFC has no relation to read only properties. What is the first class callable syntax RFC about?

Nikita Popov  0:55  
	The context here is that PHP has the callable syntax based on literals, which is that if you just use a plain string, it's interpreted as a function name, and an array where the first element is an object, and the second one is a method name, that's methods. Or the first element is the class name, and the second one is method name, that's a static method.

Derick Rethans  1:17  
	I would consider this concept a bit of a hack, especially the the one with the arrays, and I reckon you feel similar and hence this RFC?

Nikita Popov  1:27  
	Yes, I do. So the current callable syntax has a couple of issues. I think the core issue is that it's not really analysable. So if you see this kind of like array with two strings inside it, it could just be an array with two strings, you don't know if that's supposed to actually be a static method reference. If you look at the context of where it is used, you might be able to figure out that actually, this is a callable. And like in your IDE, if you rename this method, then this array should also be this array element will also be renamed. But there's like a lot of complex reasoning that the static analyser has to perform. That's one side of the issue. The second one is that callables are not scope independent. For example, if you have a private method, then like at the point where you create your callable, like as an array, it might be callable there, but then you pass it to some other function. And that's in a different scope. And suddenly that method is not callable there. So this is a general issue with both like this callable syntax based on arrays, and also the callable type. It's a callable at exactly this point, not callable at a later point. This is what the new syntax essentially addresses. So it provides a syntax that like clearly indicates that yes, this really is a callable, and it performs the callable callability check at the point where it's created, and also binds the scope at that time. So if you pass it to a different function in a different scope, it still remains callable.

Derick Rethans  3:01  
	And it's guaranteed to always be callable.

Nikita Popov  3:03  
	Yeah, exactly.

Derick Rethans  3:04  
	What does the syntax like? 

Nikita Popov  3:06  
	The syntax is the funny bit. As a bit of context. This proposal was created as an alternative or as a subset of the partial function application RFC.

Derick Rethans  3:17  
	That is just as hard to pronounce as first class callable syntax RFC. 

Nikita Popov  3:21  
	Yes, that's why we say PFA. The PFA RFC has a more general feature. It also allows you to create a reference to a callable as a side effect. But more generally, it allows you to also bind some of the arguments to a fixed value. And has like finer control over for example, you can create a callable that has three required parameters, by passing three question mark arguments. While the new syntax only allows you to use the signature of the original function. But the syntax between both of those is compatible. So the new RFC is a subset of PFA. And that's why it uses the syntax where you do a normal function call, but then pass three dots or an ellipsis as arguments.

Derick Rethans  4:08  
	Instead of passing the function's or method's normal arguments, you use the three dots.

Nikita Popov  4:14  
	I think like the way to think about the syntax is that this is similar to like a variadic argument, or to the argument unpacking syntax, just that the arguments haven't yet been provided, they will be provided during the actual call. But I think the syntax was definitely the most contentious bit in the discussion of the RFC. I think this is mainly related to the fact that if you the see this code snippet, it looks a bit like, like the example code where the arguments haven't been filled in. While now this is like actual syntax.

Derick Rethans  4:44  
	I'm sure there's quite a few tutorials out there explaining how PHP works by using dot dot dot. That is not something you can avoid.

Nikita Popov  4:54  
	Well, we can avoid it, but it's fairly tricky question. I mean, the reason for this dot dot dot syntax, on one hand, this the compatibility with partial functions. I mean, the PFA, RFC has recently been declined. But in the future, we could extend the current syntax to full partial functions. And we would not end up with two different ways. So that's one benefit of the syntax. But the other part is that PHP has different symbol tables for different kinds of symbols. People often ask, why can't you just write like strlen as a plain name, not inside a string, and have that be treated as a reference to this function? And the answer to that is that we can't do that because you can't have a constant that's called strlen. Normally, that would be reference to constant and the same actually applies to all other callable types as well. So if you have something like methods, like object or method name, that would right now be interpreted as a property access. And for static methods, it will be interpreted as a as a class constant access. So we have this ambiguity here. Even if we add an additional symbol to this, for example, like for classes, we have the syntax, class name, and then scope operator class, that gives you the class name. We could do something like strlen, scope operator function, or fn, or whatever, and have that return the callable. That would work, but it also has some ambiguities. For example, if you have something like object, arrow methods, and then scope operator fn, you have this ambiguity. Is this referencing the method of that name? Or is it referencing a callable stored inside the property of that name? This is like fundamentally ambiguous. The way we would resolve it is we will just say that this index is only usable with real simple, so it will always refer to a method, and you couldn't use the syntax to convert the callable stored in a property into a proper callable. I'm actually not sure how I should distinguish these two concepts, because we have the existing callable, strings and arrays, and the first class callables, which are really closure objects.

Derick Rethans  7:11  
	Which actually sort of brings me to the next question which just popped in my head, which is: Does this first class scalable syntax, what is returned as return a closure or an existing callable type as we have now, with a callable type being a single string, or this array syntax that we now use. 

Nikita Popov  7:28  
	The syntax returns a closure. Actually, the syntax works essentially the same way as the closureFromCallable method. And we do need to return a closure otherwise, we don't get this behaviour where the scope is bound at the time where the callable is created, rather than called. I think maybe going forward, I would generally recommend that people use a closure type, instead of a callable type in type declarations. I mean, you already cannot use callable for property types. Exactly due to this problem that callability is context dependent. While we only forbid it in property types, the same general problem also exists for argument and return types. And especially with the new syntax being introduced here, I think it's best to use closure instead of callable in the future.

Derick Rethans  8:18  
	Does that sort of mean that first class scalable syntax is syntactic sugar? Or does it do more than the closureFromCallable method?

Nikita Popov  8:27  
	No, I think it's effectively just syntactic sugar for closureFromCallable.

Derick Rethans  8:34  
	I'm actually not sure whether Xdebug is be able to do anything with these closure from callable things to begin with. So that is something I'm going to have to investigate. 

Nikita Popov  8:44  
	Be able as in like, display that it actually refers to a specific method rather than just some kind of closure?

Derick Rethans  8:51  
	Yeah, because at the moment, it shows you the file name and the line numbers, it doesn't have a name right if you create normal closures, but in this case, it's important to know that it actually refers to specific methods, which is the same thing as the closureFromCallable syntax would also do, but I've never done anything with that.

Nikita Popov  9:10  
	But I think there is a way to get like the underlying prototype for the closure, and you should be able to determine it from there.

Derick Rethans  9:18  
	The first class callable syntax, are there situations where you can't use it?

Nikita Popov  9:22  
	One place where you don't want to use the new syntax as if you don't want to actually create a closure object, and validate callability at the point of creation. For example, creating this first class callable also implies that you have to autoload the class for a static method. If you have some kind of like large definition of of handler, of static handler methods for routes or something like that, then using the first class callable syntax would imply that you have to immediately create closure objects for all of these and immediately load all those classes. That's a use case where you might want to stick with the old syntax.

Derick Rethans  10:01  
	But wouldn't opcache resolve that issue really?

Nikita Popov  10:04  
	No, opcache is really exactly the reason why you wouldn't want to do that. For example, for my fastroute library, I cache all the data as a static array. And that's something that OpCache can cache very efficiently because it's in shared memory and accessing it is essentially zero cost. If you include something like first class callables in it, then those have to always be created at runtime, because we don't have concept like, like a persistent object. That means that this can no longer, I mean, the whole script can be in shared memory, but it still has to be executed always at runtime to construct the whole data structure. And that's going to be less efficient. To give a more clear answer to your question is that the first class callable syntax has a cost when creating the callable, and if you are in a situation where avoiding that cost is really critical for performance, that's why you wouldn't want to use it.

Derick Rethans  11:00  
	And instead you'd have to use the old scalable syntax that we already have.

Nikita Popov  11:03  
	Exactly. So for that reason, I think that the old syntax is not going to be removed in the near future at least, though maybe we can deprecate certain aspects of it. For example, the syntax also allows you to do highly context dependent things like referencing self, which is even worse than the situation with a private method, because self could refer to something different every time you call it. Those are some things we might want to deprecate early, but the main syntax itself was probably going to stay for a while.

Derick Rethans  11:34  
	Because callability is checked when you create the closures does that mean it also checks for strictness then? If your PHP file has been declared with strict types?

Nikita Popov  11:45  
	Strictness is handled the same as with closureFromCallable.The strictness is still determined at the time where the call is made, not where the callable was created, which actually, I am not a fan of how PHP handles strict types together with dynamic calls. But that's like a pre existing problem. And this isn't touching on this.

Derick Rethans  12:06  
	The language has many issues that probably could have been done better if it was designed from scratch. But that ship has sailed 26 years ago.

Nikita Popov  12:15  
	The strict types are not quite that old.

Derick Rethans  12:17  
	No, that is true. The language itself is of course.

Derick Rethans  12:23  
	Okay, thank you very much then, for taking the time this morning to talk to me about first class scalable syntax.

Nikita Popov  12:29  
	Thanks for having me, Derick.

Derick Rethans  12:30  
	Thank you for listening to this installment of PHP internals news, a podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening and I'll see you next time.



Show Notes
----------

- RFC: `First Class Callable Syntax <https://wiki.php.net/rfc/first_class_callable_syntax>`_
- RFC: `Partial Function Application <https://wiki.php.net/rfc/partial_function_application>`_
- Episode #89: `Partial Function Applications <https://phpinternals.news/89>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
