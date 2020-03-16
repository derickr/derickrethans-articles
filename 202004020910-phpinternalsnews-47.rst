PHP Internals News: Episode 47: Attributes v2
=============================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2019-04-02 09:10 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin047
   :Enclosure: media/pin-047.mp3

In this episode of "PHP Internals News" I chat with Benjamin Eberlei (`Twitter
<https://twitter.com/beberlei>`_, `GitHub <https://github.com/beberlei>`_,
`Website <https://beberlei.de>`_)
about an RFC that he wrote, that would add Attributes to PHP.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-047.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:16  
	Hi, I'm Derick. And this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 47. Today I'm talking with Benjamin Eberlei about the attributes version 2 RFC. Hello, Benjamin, would you please introduce yourself?

Benjamin Eberlei  0:34  
	Hello, I'm Benjamin. I started contributing to PHP in more detail last year with my RFC on the extension to DOM. And I felt that the attributes thing was the next great or bigger thing that I should tackle because I would really like to work on this and I've been working on this sort of scope for a long time.

Derick Rethans  0:58  
	Although RFC startled attribute version two. There was actually never an attribute version one. What's happening there?

Benjamin Eberlei  1:05  
	There was an attributes version one.

Derick Rethans  1:07  
	No, it was called annotations?

Benjamin Eberlei  1:08  
	No, it was called attributes. There were two RFCs. One was called annotations, I think it was from 2012 or 2013. And then in 2016, Dmitri had an RFC that was called the attributes, original attributes RFC.

Derick Rethans  1:25  
	So this is the version two. What is the difference between attributes and annotations?

Benjamin Eberlei  1:30  
	It's just a naming. So essentially, different languages have this feature, which we probably explain in a bit. But different languages have this. And in Java, it's called annotations. In languages that are maybe more closer home to PHP, so C#, C++, Rust, and Hack. It's called attributes. And then Python and JavaScript also have it, that works a bit differently. And it's called decorators there.

Derick Rethans  1:58  
	What are these attributes or annotations to begin with?

Benjamin Eberlei  2:01  
	They are a way to declare structured metadata on declarations of the language. So in PHP or in my RFC, this would be classes, class properties, class constants and regular functions. You could declare additional metadata there that sort of tags those declarations with specific additional machine readable information.

Derick Rethans  2:27  
	This is something that other languages have. And surely people that use PHP will have done something similar already anyway?

Benjamin Eberlei  2:35  
	PHP has this concept of doc block comments, which you can access through an API at runtime. They were originally I guess, added as part or of like sort of to support the PHP doc project which existed at that point to declare types on functions and everything. So this goes way back to the time when PHP didn't have type hints and everything had to be documented everywhere so that you at least have roughly have an idea of what types would flow in and out of functions.

Derick Rethans  3:07  
	Why is that now no longer good enough?

Benjamin Eberlei  3:09  
	Essentially, user land developers use doc blocks to put metadata in there, and you could access them through an API. We had two sort of standards, or we still have two standards that use this. The documentation standard coming from the PHP documentor community. And then mostly runtime use case that exists now is covered by the doctrine annotations library, which, incidentally, I have also worked on a lot. It is used, for example, by the Symfony community, by the Drupal community, and by a few other communities as well that are smaller that wanted to go into the direction of using annotations in this case or attributes.

Derick Rethans  3:53  
	What would doctrine use an annotation for?

Benjamin Eberlei  3:55  
	I said before that annotations, add metadata to declarations. So let's say you have in your code, for example, classes that you want to store in the database. So you need to map PHP classes to database tables and back. Usually, you would do that using some kind of configuration. And configuration can be many folds. So the easiest way would be to write this in PHP, say, this is the column name, this is the field name, this is the class name and then store and use this information. And then you can go and store this in ini files, yaml files, XML files. The problem with this kind of approach is often that you have the configuration file and you have the class, and they are totally separate from each other, usually in very different places of the codebase. This is not some kind of configuration that is fluid. It's very, very static configuration that depends on the class. And it will not really change unless the class also changes. So changes are usually done together. In this case, it might make sense to put the configuration on to the class. Because then you see the declaration, you see it's configured in some way. And then you can more easily understand that changes affect each other in some way. And it leads to less mistakes, in my opinion. And it makes it a little bit more obvious that the class is used in some configured way. 

Derick Rethans  5:26  
	We've had a quick look at what annotations are. The RFC introduces them in a different way, the attributes that you're not proposing, how are they different from the doc block comments?

Benjamin Eberlei  5:37  
	The idea is that we introduce a new syntax that is independent of the doc block comments. Essentially, before each declaration, you can use the lesser than symbol twice, then the attribute declaration, and then the greater than sign twice. This is the syntax I've used from the previous attributes RFC. And Dmitri at that point used the syntax from Hack. And it makes sense to reuse this not because Hack and PHP are going in the same direction any more. But because Hack at that point they introduced it that they had the same problems with which symbols are actually still easy to use. And we do have a problem in PHP a little bit with the kind of sort of free symbols that we can still use at certain places. And lesser than and greater than at this point are easy to parse. There are a bunch of alternatives and one thing that I will probably propose is an alternative syntax where we start with a percentage sign, then the square bracket open and then a square bracket close. This is more in line with how Rust declares attributes. While Rust uses the sort of the hash symbol, which we can't use because it's a comment in PHP. 

Derick Rethans  6:54  
	And you don't want to use emojis. 

Benjamin Eberlei  6:55  
	Some crazy people propose to use emojis which would easily work in PHP, but I guess it would be hard to remember the number to get the Unicode sign. 

Derick Rethans  7:06  
	Within the two opening lesser than signs and two greater than signs to close it. What's in the middle?

Benjamin Eberlei  7:12  
	You declare an attribute name. And then you sort of have a parenthesis open, parentheses close, to pass optional arguments. You don't have to use them. So you can only use the attribute name. If you sort of want to tag something: just this is a validator, or this is an event listener, whatever you come up with, to use attributes for. But if you need to configure something in addition, then you can use. The syntax sort of looks like if you would construct a new class, except that you don't have to put the new keyword in front of it. 

Derick Rethans  7:45  
	It looks like function arguments pretty much. 

Benjamin Eberlei  7:47  
	Yes, exactly. Yeah. 

Derick Rethans  7:48  
	What kind of values can you use in the optional arguments to the attributes?

Benjamin Eberlei  7:53  
	The attributes are not really runnable code in a way. Since they are declarations, they don't allow arbitrary PHP code to run there. What is obviously allowed a simple literal values, so a number, or a fixed string, a fixed array declaration, and all this kind of things are possible. What is also possible is exactly the same expressions that you can also declare in class constants. So, in the class constants, you can do simple mathematical expressions, you can reference other constants. So, this is something that will be very interesting for attributes to do reference class names for example. 

Derick Rethans  8:34  
	What happens if you define an attribute on a declaration element? 

Benjamin Eberlei  8:38  
	What happens is that while the PHP script gets compiled, it will see that there are attributes declared and it will parse the attributes and similar to the doc block store them on the internal structure for future reference. Attributes are parsed in my current proposal in a way that you can have every attribute just once. This is something that is still under heavy discussion, because there are a few good ideas why you would need two, or multiple. Essentially similar to how a doc block is a string, we then store an array, which represents the attributes belonging to the class or the function or the constant. And this is something that the engine stores and also stores it in OPCache.

Derick Rethans  9:27  
	How would you access these attributes?

Benjamin Eberlei  9:28  
	Attributes are accessed through the reflection API. The reflection API also allows access to doc blocks. For attributes that would be a new function called getAttributes(). And it returns a list of all attributes using a new reflection class called ReflectionAttribute. There you can access what name does this attribute have? What are the arguments that are passed? And then this goes into one of the next features of this RFC proposal. You can also ask it to return this attribute as an object instance.

Derick Rethans  10:05  
	An object instance of which class though?

Benjamin Eberlei  10:07  
	Attributes, and this is something that is different to the initial version, the version one attributes RFC is, attributes names resolve to class names. That means if you declare an attribute, for example, Foo, and you have an import for our class, MyApplication/Foo, then during passing the attribute will be resolved to my attribute view name. It uses the same mechanism for class resolving that is used in every script. It reflects the use statements that are declared in the file. And you can use namespaces, namespace operators to reference the attributes as well. 

Derick Rethans  10:49  
	These are attributes not classes, so I don't quite see all the link between the attribute names in the classes is?

Benjamin Eberlei  10:55  
	One problem with the original doc block based system was that there are conflicts between attributes of different systems. One library would have a type annotation, or a var annotation, and some other library would also use it. This could lead to conflict if the syntax for them was slightly different. So this would lead to problems when multiple parses would use the same attribute. And they would parse them differently. And this could lead to errors. One problem that was mentioned in the initial attributes RFC and that, I think, if you vote us all so used as a reason for voting no is that there was no namespacing, which means that different libraries could clash and their use of attributes. My idea was we already have classes, we have namespacing. We can resolve this by using this mechanism. You declare an attribute and an attribute always resolves to a class. In the best case scenario, you would also declare this class in your code. Essentially, the attribute is not an attribute, but it's a special class that represents an attribute. This is also shown in the code that by having an additional interface, or a sort of a marker interface, that attributes can implement to make it obvious that they they are used as an attribute.

Derick Rethans  12:19  
	You mentioned that you could access the attributes through reflection API, and you can get them out as an object?

Benjamin Eberlei  12:25  
	Yes, this is why I mentioned before that the syntax sort of looks like constructing a new object, but without the new keyword. When you access the objects through the reflection API, it would essentially instantiate the class, and all the arguments that you put into the attribute declaration are passed into the constructor of the object. And this is why the connection is there between a class and an attribute. It directly goes to instantiating the attributes as an object using the arguments and giving the developer access to them.

Derick Rethans  13:00  
	Does it only do something like this when you use the getObject() on the reflection arguments? Or is it also possible that I don't care about these classes things whatsoever, and I can just get a list of attributes and their optional values that are associated with them? 

Benjamin Eberlei  13:16  
	You don't have to have a class, and the class name resolving in PHP is independent of classes actually existing. The attributes RFC respect that. You can just import anything that is not a class and use an import statement to shorten the attribute usage, or you can use the absolute namespace syntax to put a fully qualified attribute name into your code. And it wouldn't fail. The fail would only happen when you call the method on ReflectionAttribute to get the attribute as an object. So this is something the RFC is also in flux with and about to change it. The first version mentioned that attributes will always be auto loaded when they are declared at compile time. This would essentially treat attributes similar to base classes or interfaces, in a way that they are always resolved, they're always checked. However, this is a little bit overkill for userland attributes. And a lot of feedback was related to this should only happen when the reflection API is used. So I'm going to change this. One thing that we do need to handle in a way is a built in attributes. One reason why I want to add this RFC as well is that there are a few use cases coming up in PHP itself, that could benefit a lot if we had built in attributes. Since we don't have a clear path forward there. But Nikita has published his ideas on editions. So there's some paths forward to having PHP code work slightly differently depending on what developers want. Attributes could be helpful there. Other things for example, the JIT. JIT has features where you can at the moment use doc block comments to declare methods as always JIT-able or never JIT-able. Dmitri used doc block comments to check for JIT or no JIT tag in there. This is essentially something that attributes should be used for because should be machine readable. Then there's a lot of other stuff that for example, Rust also put forward that PHP is struggling with: conditional declarations of functions. For example, Symfony has a polyfill library that adds functions that are in higher languages, re implements them in a way that they're also available in lower versions where they don't exist in core. There are a lot of hacks around the sort of conditional declaration of functions and classes and stuff that make it difficult for OPCache to actually cache the files. I believe there are also even more problems if you use these kind of fights with pre loading. Essentially what could be done with attributes would be something like conditionally declared as function only if it's on PHP 7.3 and lower something like this.

Derick Rethans  16:13  
	You just mentioned using JIT or no JIT as an annotation. Does that also mean that extensions have easy access to these attributes?

Benjamin Eberlei  16:21  
	OPCache's not a PHP core functionality. It's still its own extension. The idea is that extensions have access to attributes in a very simple way. So there will be a Zend API, sort of an internal name for an API that the Zend engine provides to extensions and extensions will be able to access attributes and make decisions based on this. Extensions can already hook into the compile step of PHP and there's a hook called zend_ast_process. During AST processing, you can do stuff. That would be one way to, for extensions to look at attributes and maybe change code if they want. Then the engine obviously has tonnes of other hooks where the declarations are available in the data structure that the Zend engine provides. So there's zend_class_entry, for example, where you could look into the attributes as an extension and make decisions.

Derick Rethans  17:20  
	This is a pretty new RFC, and hence there're always going to be few open issues. Because we like to argue about stuff. What are the open issues on this RFC? 

Benjamin Eberlei  17:29  
	This is the seventh RFC on this topic. So there has been a lot of discussion. I guess this feature is, in a way quite controversial because of the implementation details. A lot of my work now will be to find the best implementation that can actually make this feature part of core by getting enough votes for it. And so I gathered a lot of feedback from the community; also talked a lot to contributors. Changes that I will be probably doing is allowing multiple attributes. What I said before, the auto loading has to be clarified. There has to be some distinction between internal attributes and user land attributes in a way that doesn't require auto loading. Hack, for example, has __ as a magic prefix, which I want to avoid, because it puts up all this magic methods, sort of argument back on the table. We need to have something to make a distinction between userland and internal attributes, because the internal attributes need to be validated very strictly at compile time. And the userland attributes need to be validated only when you call the getAsObject() method on the reflection API.

Derick Rethans  18:42  
	How long do you think there'll be before you put this RFC up for a vote?

Benjamin Eberlei  18:46  
	It's a bit tricky because this issue is so controversial. I don't want to invest month of work and then get a no vote. And so I do want to have some feedback quite quick enough. I do realise that the first draft needs some work and clarifications that would otherwise lead to no votes from contributors. So I hope to get this done in, let's say, two to four weeks of additional work.

Derick Rethans  19:09  
	All right, Benjamin. That was a great explanation of the attributes version two RFC.

Benjamin Eberlei  19:16  
	Thank you for having me, and I really appreciate it again.

Derick Rethans  19:21  
	Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.


Show Notes
----------

- RFC: `Attributes v2 <https://wiki.php.net/rfc/attributes_v2>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
