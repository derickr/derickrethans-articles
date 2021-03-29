PHP Internals News: Episode 80: Static Variables in Inherited Methods
=====================================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-04-01 09:08 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin080
   :Enclosure: media/pin-080.mp3

In this episode of "PHP Internals News" I chat with Nikita Popov (`Twitter
<https://twitter.com/nikita_ppv>`_, `GitHub <https://github.com/nikic/>`_,
`Website <https://nikic.github.io/>`_) about the "Static Variables in
Inherited Methods" RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-080.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:14  
	Hi I'm Derick, welcome to PHP internals news, the podcast dedicated to explain the latest developments in the PHP language. This is episode 80. In this episode I speak with Nikita Popov again about another RFC that he's proposing. Nikita, how are you doing today?

Nikita Popov  0:30  
	I'm still doing fine.

Derick Rethans  0:33  
	Well, that is glad to hear. So the reason why you saying, I'm still doing fine, is of course because we basically recording two podcast episodes just behind each other.

Nikita Popov  0:41  
	That's true.

Derick Rethans  0:42  
	If you'd be doing fine 30 minutes ago and bad now, something bad must have happened and that is of course no fun. In any case, shall we take the second RFC then, which is titled static variables in inherited methods. Can you explain what is RFC is meant to improve?

Nikita Popov  1:00  
	I'm not sure what this meant to improve, it's more like trying to fix a bug, I will say. This is a really, like, technical RFC for an edge case of an edge case, so I should say first, when I'm saying static variables, I'm not talking about static properties, which is what most people use, but static variables inside functions. What static variables do unlike normal variables, is that they persist across function calls. For example, you can have a counter static $i equals zero, and then increment it, and then each time we call the function, it gets incremented each time and the value from the previous call is retained. So that's just the context of what we're talking about.

Derick Rethans  1:43  
	Why would people make use of static variables?

Nikita Popov  1:46  
	I think one of the most common use cases is memoization. 

Derick Rethans  1:50  
	Can you explain what that is? 

Nikita Popov  1:51  
	If you have a function that that computes some kind of expensive result, but which is always the same, then you can compute it only once and store it inside the static variable, and then return it. Maybe possibly keyed by by the function arguments, but that's the general idea. And this also works if it's a free standing function. So if it's not in the method where you could store state inside the static property or similar, but also works inside a non method function.

Derick Rethans  2:22  
	The keyword here in his RFC's title is inherited methods, I suppose. What happens currently there?

Nikita Popov  2:29  
	There are a couple of issues in that area. The key part is first: How do static variables interact with methods at all? And the second part is how it interacts with inheritance. So first if you have an instance method, with a static variable, then some people expect that actually each object instance gets a separate static variable. This is not the case. The static variables are really bound to functions or methods, they do not depend on the object instance. Second problem is: What happens with inheritance? So you have a parent class with a method using static variables and then you have a child class that inherits this method. There are two ways you can interpret this, either this is still the same method, so it should use the same variables, or you could say okay the inherited method is actually a distinct method and should use separate variables. PHP currently follows the second interpretation.

Derick Rethans  3:24  
	And is this even the case, if it's overridden, or just when it's inherited? Because there's a difference there supposed as well.

Nikita Popov  3:30  
	Yeah, this is the basic model that PHP tries to follow but there are quite a few edge cases. The one that's what you mentioned if you override the method, then of course, you're calling the overridden method so the static variables don't even come in. But if you then call the parent method, then usually you would expect, if you do an override and just call the parent that the behaviour is exactly the same as if you didn't override it at all. That's not the case here, because now you're calling the parent method. So the problem you have here is that if you didn't override the method, then the child method and the parent method would have different static variables. Now if you call parent, then you're just calling the parent method, so you get back to one set of static variables, which are the same for both methods. You can see they're the same for both methods, but rather because you're calling the parent method, there is only one method involved, only one set of static variables involved. You can't really just like seamlessly extend a method that uses static variables, without changing the behaviour by accident.

Derick Rethans  4:37  
	This sounds all very complicated.

Nikita Popov  4:39  
	Yes I said, I did warn you that this is an edge case of an edge case.

Derick Rethans  4:44  
	But I think the whole idea behind the RFC is to make it less complicated.

Nikita Popov  4:48  
	Yes, this is like not the, the only issue you can run into. There is another one that we have actually addressed separately, but which still exists on earlier PHP versions, which is that the value of the static variables depend on the time of inheritance. Let me be a bit more explicit there. What we had in previous versions is that half your parent method was a static variables, then you call that parent method, static variables change, then you inherit it. And again, call the inherited method. In that order: first declare the parents, call it, declare the child, call it. In that case we will actually take the static variables at the time, where the inheritance actually happens. The first call onto the parent method modify the static variables, then we will use some modified variables. From that point on, it will have a separate copy, the child method, but it will like pick up these original modifications before inheritance happens. Now in PHP 8.1, we actually already fixed that so that we always use the original values, but this is like just one more thing to the list of weird things that happen, if you use static variables inside methods and you inherit them.

Derick Rethans  6:06  
	I think I understand more of it now.

Nikita Popov  6:08  
	I think you understand more than you ever wanted to know.

Derick Rethans  6:12  
	You've mentioned the edge cases. What is the result, going to be once this RFC passes, which I'm going to think is quite likely

Nikita Popov  6:21  
	The result is, hopefully, simpler than what we have, namely that static variables are really bound to a specific function or method declaration. If you have one method using static variables, then you have only one set of static variables ever. If it's inherited, you still reuse the same static variables because there is no separate inherited method. It's just the same method in the child class. That's the concept.

Derick Rethans  6:52  
	And if you override it in an inherited class?

Nikita Popov  6:55  
	If you override it, and you call the parent method, then the behaviour is unchanged because you still have just a single set of static variables, so there is no edge case here any more because the child, the child method never had a separate set.

Derick Rethans  7:10  
	But if the overridden method also defines its own static variable with the same name?

Nikita Popov  7:17  
	That's possible. In that case, once again this rule is that each method has its own static variables and methods can have static variables and the child method can have them as well, if they are overridden and there are no name clashes between them.

Derick Rethans  7:32  
	Because they are going to be totally separated, meaning that any code you run in the inherited methods will only affect its static variables, and any code that runs in the original methods only affects the static variables that are bound to that specific method.

Nikita Popov  7:50  
	Exactly. I mean, in the end, static variables are really the type of global state, just a type that is kind of isolated to a specific namespace and doesn't cause clashes, so in that sense, it's important that these things are isolated.

Derick Rethans  8:06  
	And that would also make the behaviour, a lot more easier to explain than it currently is. Because every methods, has its own set of static variables.

Nikita Popov  8:15  
	Yes. 

Derick Rethans  8:15  
	Or I should say, every declared methods, has its own set of static variables.

Nikita Popov  8:20  
	I guess that is an important distinction. If you can see the methods inside your code and see the static variable inside it, then that is a distinct one. If we ignore the exception of traits.

Derick Rethans  8:34  
	You're going to have to explain that as well.

Nikita Popov  8:37  
	Well traits are always a special snowflake. Our general model for traits is that they are compiler assisted copy and paste. So a trait should roughly behave as if you just copied all the methods into the class that's using the trait. And in that sense, if you are actually copying the code of your method with a static variables, then it should also use a distinct set of static variables for each use. And that is also how it is proposed to behave. So that is like the one exception where you have a single method declaration in your code, but each using class will get a separate set of static variables.

Derick Rethans  9:15  
	Because the code is copied in place, instead of linked, or used in place. It's also the case for all the methods declared in traits, they're also copied into the same symbol table as the methods belong to a class. 

Nikita Popov  9:32  
	Yeah, that's right. 

Derick Rethans  9:33  
	Should be reference counted in some way because you probably won't duplicate the exact data.

Nikita Popov  9:38  
	We of course don't actually copy the methods, or at least most of the methods, but from the programmer perspective that's how it works

Derick Rethans  9:46  
	Why do you say most of the methods, and not all the methods?

Nikita Popov  9:50  
	We separate two things there. There is the method itself. So the op-array, and then there is all the stuff it uses like the opcodes the like arguments information and so on. What they do for traits, is we share all the data, and only create a separate op-array. Reason is that there are some differences. For example, we have to adjust the scope, we have to adjust possibly the function name if aliases are involved, and we have to adjust the static variables. So it's like kind of a partial copy we do.

Derick Rethans  10:22  
	Which is probably the most efficient way of doing it?

Nikita Popov  10:24  
	Yes.

Derick Rethans  10:25  
	Because this RFC is changing behaviour due to bug fixes, I would probably argue, what kind of backwards compatibility issues are there? And have you looked at how much code that actually impacts?

Nikita Popov  10:39  
	I haven't looked how much code it impacts because this seems like pretty hard to really analyse. I mean I guess something we could easily check is how much static variables are used in methods at all. But it would be hard to distinguish whether this change. I mean how to distinguish in a completely automated way. Whether this change makes behavioural difference for a particular use case or not. So I can't really give information on that, though I would expect that impact is relatively low because the common use cases, things like memoization, they aren't affected by it, or they are only affected by it in the sense that: Then you will memoize a value only once for the whole class hierarchy instead of memoizing it once for each like inherited class.

Derick Rethans  11:32  
	So, it's going to improve the situation there as well, is pretty much what you're saying?

Nikita Popov  11:36  
	Yeah, but I'm sure there are also cases where the previous behaviour was like intentionally used. I mean it was never documented, but you know if some behaviour exists, people will always make use of it in the end, but I can't really say exactly how much impact this would have.

Derick Rethans  11:55  
	Do you have anything else to add, discussing this RFC?

Nikita Popov  11:59  
	No, I think that's it.

Derick Rethans  12:00  
	Then I would like to thank you for taking the time today, again, to talk to me about static variables in inherited methods.

Nikita Popov  12:08  
	Thanks for having me once again.

Derick Rethans  12:16  
	Thank you for listening to this instalment of PHP internals news, a podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening and I'll see you next time.


Show Notes
----------

- RFC: `Static Variables in Inherited Methods <https://wiki.php.net/rfc/static_variable_inheritance>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
