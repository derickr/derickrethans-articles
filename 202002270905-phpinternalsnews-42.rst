PHP Internals News: Episode 42: Userspace Operator Overloading
==============================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-02-27 09:05 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin042
   :Enclosure: media/pin-042.mp3

In this episode of "PHP Internals News" I chat with Jan Böhmer (`GitHub <https://github.com/jbtronics>`_,
`LinkedIn <https://www.linkedin.com/in/jan-boehmer>`_)
about the Userspace Operator Overloading RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-042.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:16  
	Hi, I'm Derick. And this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 42. Today I'm talking with Jan Böhmer about Userspace Operator Overloading. Jan, would you please introduce yourself?

Jan Böhmer  0:33  
	Hi, my name is Jan Böhmer. I'm a physics student from Germany. And I'm the author of the Operator Overloading RFC.

Derick Rethans  0:40  
	What brought you to writing this RFC?

Jan Böhmer  0:42  
	Mostly because I have worked with monetary objects in the past. And it was a bit tedious to work when it comes to calculating. And whenever you have to want to calculate something, you have to call functions on objects. This is not possible to call, just use operators like with normal values like floats or integers. 

Derick Rethans  1:06  
	Because the monetary objects themselves had multiple things embedded in there or something like that?

Jan Böhmer  1:11  
	Yes, they describe mostly a value and a currency. And together they are saved in an object.

Derick Rethans  1:18  
	Okay that that seems like a reasonable thing to do, right? I mean, other times people say the same thing about doing complex numbers or something like vectors. The RFC is called Userspace Operator Overloading. What is operator overloading? 

Jan Böhmer  1:31  
	Yeah. Basically, is the idea that you can define operators, like addition or subtraction, or the string concatenation for objects

Derick Rethans  1:43  
	Does PHP already have something like this?

Jan Böhmer  1:45  
	Actually, yes. Objects can have something that calls do operation handler. This is called whenever PHP encounters an object, but if used with an operator. The problem is that this handler is only available for PHP internally. So if you want to use it, you have to write an extension.

Derick Rethans  2:06  
	So it will be possible to have in an extension a Monetary class with its own operators already defined on it.

Jan Böhmer  2:14  
	Exactly PHP extension GMP uses this as already. The problem is that it's not very flexible, you already have to know, be familiar with C, you have to be able to compile that. You have to contribute it to whatever system you want to use it. Since we have the foreign function interface since PHP 7.4 we can implement many things without to actually have an extension. But this operator overloading is something that's not possible yet inside from PHP.

Derick Rethans  2:47  
	So it wouldn't have been possible to write the GMP extension which is of course a wrap around libgmp with FFI, because there's no operator overloading available in PHP. 

Jan Böhmer  2:59  
	Not in that comfortable way. You could use this way with functions but it would a bit more tedious then using just operators.

Derick Rethans  3:09  
	You've mentioned the Monetary object as a good use case. What other use cases can you think of?

Jan Böhmer  3:15  
	Higher mathematical objects like complex numbers, vectors, or something like tensors, maybe something like the string component of Symfony. That's you can simply concat this string objects with a normal string using the concat operator and doesn't have to use the function to call that, because basically, this should behave similar to a basic string variable, and not, like something completely different.

Derick Rethans  3:45  
	What is the syntax you're proposing for implementing this?

Jan Böhmer  3:49  
	My idea was similar to Python to use special metric function, the methods for every operator you can overload. So if you want to overload the addition operator, you would implement function called, a static function called __add, for example. This offer this function takes both operands, the left hand operand and the right hand operand. So you can decide if your current, this object is on the right or the left hand. That is important to determine something like one divided for zero, or one divided through two, or two divided through zero. There are two complete different cases and you have to be able to differentiate between the two cases.

Derick Rethans  4:39  
	And wouldn't that not be possible to do in non static functions?

Jan Böhmer  4:43  
	Another problem with non static functions such as possible access to this variable. If you modify an object from inside an operator handler, this can lead to very, very strange behaviour. Because normally operations doesn't change the object itself, but rather you should return a new value. The problems such as asked us to this, it is very easy to accidentally change the this object. If you only pass both objects like via a static methods, it is a bit more clear that you have to create an all new object

Derick Rethans  5:24  
	Would a type hint enforce that you return a object to the same class?

Jan Böhmer  5:29  
	Not an all case you want to return an object of the same class. For example, takes a dots product of vectors. So you take two vectors, multiply it in some way as you return to normal float value.

Derick Rethans  5:43  
	Of course, yes.

Jan Böhmer  5:44  
	If you were to enforce that, but would always to be the same types as those limits the use cases, in my opinion too much. 

Derick Rethans  5:52  
	But you could of course type hint the __add operator yourself?

Jan Böhmer  5:57  
	It's always typehints in arguments, in my observation are used as a hint which type are supported for the operand handler. If you for example, vector plus an integer, and your operator handler only declares vector vector as a parameter types, then this operator will not be called, and it will tried to be called on the second object.

Derick Rethans  6:24  
	So it won't the called and instead it falls back to the second object to be called on.

Jan Böhmer  6:29  
	Yes, the idea behind it, is that only one of the objects have to know about both classes. So if you want to combine, for example, two objects from different libraries, and library A doesn't know about library B then only objects of the second library have to know about object A. In C++ you can define supported type outside of classes. So you can define combinations between arbitrary objects. The problem is in PHP this was a bit complicated. And the best way to implement this handler in types or classes. So the class has to know about each other objects, it could be interact with possibly.

Derick Rethans  7:14  
	That makes sense to me. What happens if neither of the classes, or if one of them is a class, and the other one is just a scalar type, if none of the add methods fit, what would happen then?

Jan Böhmer  7:24  
	The operators implement an handler, then those doesn't support them, then an error would be thrown. 

Derick Rethans  7:32  
	And that is a type error like you'd normally get? 

Jan Böhmer  7:34  
	If the object doesn't implement operator at all, then a notice would be triggered. The idea's that in the moment, it is possible to write something like object plus one, this would be a fine expression in PHP, in the current PHP versions, the object could be interpreted as a one and just a notice would be thrown. For compatibility reasons, my RFC does the same behaviour if no operators are overloaded on objects. 

Derick Rethans  8:05  
	That seems like a reasonable compromise there. I remember from in the past, I think it was Sara Golemon that wrote an extension for using operator overloading. And I remember from the time that there is a problem with using the lesser than or greater than operators, because I think one of them gets flipped around automatically in the engine is being changed in PHP already, or are you running into the same problem? 

Jan Böhmer  8:28  
	I'm not sure about this. My RFC doesn't mention comparison operators like greater or less at all. Because comparison, handled differently internally of PHP. This doesn't work about this. This is mentioned do operator handler. It would be a bit other implementation to do this. Also, the comparison is a bit complicated on its own terms. Maybe it's more useful to use interfaces for, to implements this overloading, or to use. Also, there are some problems. Maybe we should only allow something like an compare operator that's resolved either, minus one, one, or zero. If object's lesser or equal, so that everything is defined together at once. So it's not possible to define an object that has maybe, for example, the lesser, but not the greater operator.

Derick Rethans  9:32  
	But this sounds like that's for a different RFC.

Jan Böhmer  9:35  
	Exactly. That's a bit complicated. If the current operator overloading RFC gets passed, then maybe a comparison operator overloading RFC would make sense.

Derick Rethans  9:46  
	From reading the RFC, I've noticed that you also won't be able to use a shorthand assignment operator. So for example, plus equals. What is the reason for that?

Jan Böhmer  9:56  
	So every shorthand operator becomes currently an assignment of A plus B. The do operation handler cannot decide if an shorthand operator or normal operator was called. Allowing to overloads the shorthand operators, would maybe allow some benefits for objects terms of memory optimization. If you call a short hand operator you can mutate the object itself doesn't have to create a new object which takes more memory, but I think with the garbage collector of PHP that is not such a big problem. And if that is really needed feature in the future, this could be edited in other, later version of PHP. 

Derick Rethans  10:41  
	Okay. 

Jan Böhmer  10:42  
	Many other languages doesn't allow to otherwise shorthand operators so I don't think that as too much need for.

Derick Rethans  10:49  
	Operator overloading sometime has criticisms directed at it. What are some of the criticisms you've heard about it?

Jan Böhmer  10:56  
	First of all, there are some criticisms about the operator overloading idea in general. So there's also some criticism could be abused for doing some very weird things with operator overloading. So as mentioned C++ there is a shift, left shift operator, is used for output in a stream to the console. Or you could do whatever you want inside this handler, so if somebody would want to save files or modify the file in inside operator overloaded handler, it would be possible, and it's in the most cases function would be more clear what it does.

Derick Rethans  11:35  
	Of course, in a function add(), if you implemented yourself, nothing stops you of course on writing to a file either.

Jan Böhmer  11:41  
	Operator overload issues, in my opinion only be used for things that's related to maths or creating custom types that behave similar to the built-in types.

Derick Rethans  11:52  
	Like complex numbers, or vectors, or monetary numbers. So far, we have been discussing this RFC for a few weeks now. What do you think the chances are of it being passing? 

Jan Böhmer  12:05  
	I'm not sure. I think the idea of operator overloading in general is accepted in the community, but doesn't hear so much backlash. There was some time discussion about how to do it. Some people think it's maybe better if you would implement operator overloading with interfaces, like with ArrayAccess, or to introduce some completely new keywords, like in other languages. In C++, or C#, there are a special keyword operator, that's marks an operator overloading function. So it is clear that is not a real function but special handled way.

Derick Rethans  12:49  
	Instead of using the underscore underscore in front of method names. When do you think you'll be ready to put this up or vote?

Jan Böhmer  12:56  
	Wasn't it busy last days, I will do some revises to my RFC, and polish my implementation.

Derick Rethans  13:06  
	Okay, thank you very much this morning for taking the time to talk to me Jan.

Jan Böhmer  13:10  
	Thank you very much for inviting me. 

Derick Rethans  13:13  
	Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language, I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening and I'll see you next week.


Show Notes
----------

- RFC: `Userspace Operator Overloading <https://wiki.php.net/rfc/userspace_operator_overloading>`_


Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
