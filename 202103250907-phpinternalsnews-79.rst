PHP Internals News: Episode 79: New in Initialisers
===================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-03-25 09:07 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin079
   :Enclosure: media/pin-079.mp3

In this episode of "PHP Internals News" I chat with Nikita Popov (`Twitter
<https://twitter.com/nikita_ppv>`_, `GitHub <https://github.com/nikic/>`_,
`Website <https://nikic.github.io/>`_) about the "New in Initialisers" RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-079.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:14  
	Hi, I'm Derick. Welcome to PHP internals news, a podcast dedicated to explain the latest developments in the PHP language. As you might have noticed, the podcasts are currently not coming out once every week, as there are not enough RFCs being submitted for weekly episodes. I suspect that this will change soon again. This is episode 79. In this episode, I speak with Nikita Popov, about a few more language additions that he's proposing. Nikita, how are you doing today?

Nikita Popov  0:43  
	I'm doing well, Derick, how are you doing?

Derick Rethans  0:45  
	I'm pretty good as though, I always much happier when it's sunny outside.

Nikita Popov  0:48  
	Yeah, for us to weather also turned today. Yesterday it was still cold. 

Derick Rethans  0:53  
	We're here to talk about a few RFCs. The first one, titled "New in Initializers". What is this RFC about?

Nikita Popov  1:00  
	The context is that PHP has basically two types of expressions: ones, the ones used on normal code, and the other one in a couple of special places. These special places include parameter default values, property default values, constants, static variable defaults, and finally attribute arguments. These don't accept arbitrary expressions but only a certain subset. So we call those usually constant expressions, even though they are not maybe constant in the most technical sense. The differences really that these are evaluated separately so they don't run on the normal PHP virtual machine. There is a separate evaluator that basically evaluates an abstract syntax tree directly. They are just like, have different technical underpinnings.

Derick Rethans  1:49  
	Because it is possible to for example, define a default value to seven plus 12?

Nikita Popov  1:54  
	Exactly. It's possible to define it to seven plus 12, but maybe not to seven plus variable A, or seven plus some function call or something like that.

Derick Rethans  2:03  
	I guess the RFC is about changing this so that you can do things like this. What are you proposing to add?

Nikita Popov  2:09  
	Yes, exactly. So my addition is a very small one, actually. I'm only allowing a single new thing and that's using new, so you can use new, whatever, as a parameter default, property default, and so on.

Derick Rethans  2:23  
	In this new call, pretty much a constructor call for a class of course, can arguments to be dynamic, or do they need to be constant as well?

Nikita Popov  2:33  
	Rules are always recursive, so you can like pass arguments to your constructor, but they also have to follow the usual rules. So again, they also have to be constant expressions, or after this RFC, they can also include new, but they cannot include variable references and so on.

Derick Rethans  2:50  
	Is this something that is being defined at the grammar level or somewhere else?

Nikita Popov  2:55  
	We actually used to define that at the grammar level back in PHP five, but nowadays we just accept all expressions, and then we print you a bit nicer error message if it turns out it's not supported in this context.

Derick Rethans  3:08  
	And that happens when the AST is created. 

Nikita Popov  3:10  
	Yeah, exactly. 

Derick Rethans  3:11  
	The new syntax additions is to allow new in default values in places. The RFC talks a little bit about the evaluation order, and what is this and why is is important.

Nikita Popov  3:23  
	This is really the critical part. Right now, the kinds of things you can use in constant expressions are well as the name says, these can supposed to be constant, and not dynamic. This is not entirely true because, for example, you can like reference a class constant and referencing a class means that you call an autoloader. And that can run arbitrary code. Or you can trigger a notice that triggers an error handler and that also runs arbitrary code, like more like at the design level at the conceptual level, these really are constant expressions that are not supposed to have any side effects.  Allowing new changes that in the big way, because new calls constructors and constructors can do whatever they want. I think it would be unusual to print out a string in the constructor, but some people might want to create a database connection there. The question of where exactly these expressions get evaluated becomes more important now, so we have to think about that a bit more carefully. Previously this was never really formally specified, how the evaluation works. There are some cases where it's pretty clear cut, for example, parameter default value of course gets evaluated when you call the function, and that parameter hasn't been passed. Similarly if you define a global constant, then the value is evaluated when you define the constant. And then there are the problematic cases, that I'm actually still not sure what to do about. The problematic cases are class constants, and static properties. Because the way these work right now is that they are evaluated lazily, so when you declare the class, they are not evaluated, and then later if you use the class, at least most uses of the class, they will be evaluated.

Derick Rethans  5:07  
	It would happened when you're done instantiate the class?

Nikita Popov  5:09  
	For example, yes. If you instantiate the class, if you add a static property and so on. But for example, not if you extend the class. In cases that like might potentially meet the evaluated initializers. The problem here is that, um, of course, if now your expressions can have side effects, then it's not great if you don't have like hard guarantee when it's actually going to be evaluated. On the other hand, what I actually implemented already, but I'm not sure it's a good idea is to change it to evaluate the expressions equally, so when you declare the class, we immediately evaluate all the static properties and constants.

Derick Rethans  5:47  
	I think I found a problem with that as well. For example, if one of the default values is doing new DateTime for example, if you rely on that happening when you instantiate the object, you will get a different time than when you declare a class.

Nikita Popov  6:01  
	I probably should have mentioned that explicitly. So when it comes to properties we'll always evaluate when you create the object. If you do a new DateTime then you will always get a new DateTime for each object, otherwise it wouldn't really make sense. The problem with the, with evaluation order for the static properties is, that if we evaluate immediately when they declare the class, then we can run into issues with dependencies. If you're using auto loading it's usually not a problem, but if you like declare classes manually, then you might have one class on using a constant from a class that you declare later on. Right now, that works fine, because the initializers are evaluated lazily, but if we evaluate them immediately then this is going to throw a fatal error because it says okay the class hasn't been declared yet. That's a backwards compatibility break, and there are also some other issues, for example with preloading, where it's not really clear when exactly we should be evaluating things in that context. So this is a point I'm undecided on, when things should evaluate. If we should stick with the current lazy evaluation or make it easier or possibly just limit the RFC, to not allow new inside static properties and class constants, because, at least for me personally, those are not the main use case. The three things that seem most important to me are parameter default, property default, I mean non static property default, and usage in attribute arguments.

Derick Rethans  7:29  
	When I was reading the RFC, I was realizing that if a constructor throws an exception, for example if it's a default argument to a method, then what would happen? Would the method fail, or, or would the method call fail or something else?

Nikita Popov  7:45  
	It would behave basically the same way as if you initialize the method argument, inside the method, you could do like you would do right now like with a null check maybe. So you would just get an exception thrown inside the method, and it would fail.

Derick Rethans  8:01  
	And is that the same if you would use it as the new new syntax as a default value to have constructor arguments as well?

Nikita Popov  8:09  
	Yeah, I think there's a situation would be the same, the one special case is if you use it as a property default, and then the instantiation fails, then we treat this basically the same way as the constructor having failed, which is a special situation, a slightly special situation, because we will also not call the destructor in that case. So we say that the object has been incompletely constructed so it will not get destructed.

Derick Rethans  8:38  
	And of course if this is a standard class property, then this happens on instantiation of the class. How would this work if it would be for a static property?

Nikita Popov  8:50  
	For a static property, well that, that depends again on the whole question of evaluation order. So for example the way things work right now, like without this proposal, is for example if you have a static property and you're referencing it constant that doesn't exist. Then, when you try to use the class you get an exception that okay undefined constant whatever. If you try to use it again, you still get the same exception, so you get this exception every time you use a class. This is what would happen on this case as well.

Derick Rethans  9:24  
	So it wouldn't happen on class declaration, but when you start using it? 

Nikita Popov  9:29  
	Depending on where we evaluate. Either only on use or the first time on declaration and then afterwards in each use. If you still try to use it despite the declaration having failed, which is an odd thing to do but you would have to counter it somehow. 

Derick Rethans  9:45  
	You know, if it is possible to do people will find a way how to do it.

Nikita Popov  9:48  
	Yes, certainly. 

Derick Rethans  9:49  
	Can you talk a little bit about recursion protection as well because the RFC talks about that?

Nikita Popov  9:54  
	Well that's another edge case. So if you create an, have an, for example Class A with a property that has an initializer new A. That means when you create an object of class A, and try to initialize it you have to create another object, and then another object, and another, and we have to detect that situation, or we do detect that situation and for nice exception instead of, resulting in a stack overflow.

Derick Rethans  10:20  
	Which is beneficial.

Nikita Popov  10:21  
	Yes, because most people do not know what to do when people went PHP throws a segmentation fault, so they do prefer exceptions, usually.

Derick Rethans  10:30  
	I would too. The RC also talks about, there are some issues around traits which I didn't quite fully understand, would you mind explaining that to me?

Nikita Popov  10:38  
	The issue here is that traits can have properties and or rules. A rule is that if you have two traits, used in the same class, and declaring the same property, they have to be compatible. And compatible means effectively they have to be exactly the same. So, same visibility and same default value. The trouble here is that if we are dealing with an instance property, which has a new expression as a default value, then we have to somehow check that these are the same. It would be not great if we actually had to evaluate the initializer to do that because, I mean it's okay if it's just you know, with initializer something like one plus two, but if it's an actual new expression we don't want to create objects which again might have side effects and so on. What I'm specifying is that if you have a trait property with this kind of dynamic initializer, so using the new expression, than we will always consider it not compatible.

Derick Rethans  11:36  
	Would it currently be compatible with one of those trait properties, it says seven plus three, for example?

Nikita Popov  11:42  
	That will be compatible, which is actually, I think relatively new thing. We used to not evaluate initializers and traits at all, and say those are incompatible and that changed at some point and seven point, I don't know which version. But in this case we would go back to saying it's incompatible, because at least I don't see a good way to make it compatible and I don't think it's particularly important to support that case.

Derick Rethans  12:10  
	Do you have any information about how much traits are actually used?

Nikita Popov  12:15  
	Well, I know that Laravel uses them. But I have no idea how much.

Derick Rethans  12:22  
	One last thing I think RFC mentioned, is that it also has an effect on attributes, that it sort of gets nested attributes in by the back door. How does that work?

Nikita Popov  12:33  
	I wouldn't call it the back door. Exactly. I have to be honest, I didn't think about attributes at all when writing this proposal, what I had in mind is mainly parameter defaults, and property defaults. But yeah, attribute arguments also use the same mechanism and are under the same limitations. So now you can use new as an attribute argument. And this can be used to effectively nest attributes, so the example I've seen from Symfony is that they have, for example, assertions. They have an assert all attribute which has the which accepts, which wants to accept a list of assertion attributes. And now you can actually do that because you can, um, create these attribute objects recursively. The example from the RFC is assert all, then new assert not null, new assert length max six.

Derick Rethans  13:26  
	That's actually kind of neat, that is just ends up starting to work on right?

Nikita Popov  13:30  
	Yeah, I mean, I read the thread for Symfony how they are trying to work around that. They have various ideas of how to do it and it's all pretty ugly. So I think it's nice to have a more or less proper solution for that.

Derick Rethans  13:45  
	They'll just have to wait until PHP 8.1.

Nikita Popov  13:48  
	Yes, that is the disadvantage.

Derick Rethans  13:51  
	Out later this year.

Derick Rethans  13:53  
	Are there any backwards incompatible changes?

Nikita Popov  13:56  
	That again comes back to the evaluation order the problem. Originally I had intended to this, this to be compatible. Now if we change evaluation order then it is breaking, depending on that, the answer is yes or no, I am still not sure on that one.

Derick Rethans  14:11  
	Because I think PHP eight one already has a breaking changes in there where the order of declaration of properties is now different.

Nikita Popov  14:19  
	Yeah, that the change, though I hope that does not affect people too much because it's mostly about debugging functionality, which of course you are kind of interested in. 

Derick Rethans  14:29  
	Yep, it broke my tests, which is a good thing because it means that my tests cover all the edge cases as well. I think we sort of done discussing this RFC, is there anything else that might ends up being added here in the future, or what still needs to be hammered out before you can put it up to vote?

Nikita Popov  14:47  
	Apart from the evaluation order question that I have been continuously mentioning, the future scope would be to extend this to not just new expressions, but also for example static method calls, popular alternative pattern is to not use constructors, but named constructors, which are implemented as static methods, and similarly also function calls for example so you can use something like strlen() or count inside an initializer.

Derick Rethans  15:13  
	Isn't strlen a language construct now?

Nikita Popov  15:15  
	No it isn't. It has an optimized implementation in the virtual machine, but it's still technically a normal function call.

Derick Rethans  15:23  
	Because I remember that, breaking tests in Xdebug as well at some point, because it suddenly didn't suddenly was no longer a function call.

Nikita Popov  15:30  
	Things do tend to break in Xdebug.

Derick Rethans  15:34  
	Okay, I'm used to it. Thank you, Nikita for taking the time to talk about your new in initializers RFC.

Nikita Popov  15:40  
	Thanks for having me.

Derick Rethans  15:45  
	Thank you for listening to this instalment of PHP internals news, a podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supports of this podcast as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening and I'll see you next time.


Show Notes
----------

- RFC: `New in Initialisers <https://wiki.php.net/rfc/new_in_initializers>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
