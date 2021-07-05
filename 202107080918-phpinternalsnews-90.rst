PHP Internals News: Episode 90: Read Only Properties
====================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-07-08 09:18 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin090
   :Enclosure: media/pin-090.mp3

In this episode of "PHP Internals News" I chat with Nikita Popov (`Twitter
<https://twitter.com/nikita_ppv>`_, `GitHub <https://github.com/nikic/>`_,
`Website <https://nikic.github.io/>`_) about the "Read Only Properties" RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-090.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:14  
	Hi, I'm Derick. Welcome to PHP internals news, a podcast dedicated to explaining the latest developments in the PHP language. This is Episode 90. Today I'm talking with Nikita Popov about the read only properties version two RFC that he's proposing. Nikita, would you please introduce yourself?

Nikita Popov  0:33  
	Hi, Derick. I'm Nikita and I do PHP core development work by JetBrains.

Derick Rethans  0:39  
	What does this RFC proposing?

Nikita Popov  0:41  
	This RFC is proposing read only properties, which means that the property can only be initialized once and then not changed afterwards. Again, the idea here is that since PHP 7.4, we have typed properties. A remaining problem with them is that people are not confident making public type properties because they still ensure that the type is correct, but they might not be upholding other invariants. For example, if you have some, like additional checks in your constructor, that string property is actually a non empty string property, then you might not want to make it public because then it could be modified to an empty value for example. One nowadays fairly common case is where properties are actually only initialized in the constructor and not changed afterwards any more. So I think this kind of mutable object pattern is becoming more and more popular in PHP.

Derick Rethans  1:35  
	You mean the immutable object?

Nikita Popov  1:37  
	Sorry, immutable. And read only properties address that case. So you can simply put a public read only typed property in your class, and then it can be initialized once in the constructor and you can be... You don't have to be afraid that someone outside the class is going to modify it afterwards. That's the basic premise of this RFC.

Derick Rethans  1:57  
	But it also means that objects of the class itself can modify that value any more, either.

Nikita Popov  2:01  
	Exactly. So that's, I think, a primary distinction we have to make. Genuinely, there are two ways to make this read only concept work. One is like actually read only or maybe more precisely init once, which is what this RFC proposes. We can only set that once and then even in the same class, you can't modify it again. And the alternative is the asymmetric visibility approach where you say that, okay, only in the public scope, the property can only be read, but in the private scope, you can modify it. I think the distinction there is very important, because read only property tells you that it's genuinely read only, like, if you access a property multiple times in sequence, you will always get back the same value. While the asymmetric visibility only says that the public interface is read only, but internally, it could be mutated. And that might like be, you know, intentional, just that you want to like have your state management private, but that the property is not supposed to be immutable.

Derick Rethans  3:05  
	How's this RFC different from read only properties, version one?

Nikita Popov  3:09  
	Read only properties version one was called write once properties. I think the naming is kind of one of the more important differences. The new RFC is also effectively write once, but I think it's really important to view it from an API perspective as read only because that's what the user gets to see. While write once gives you this impression, that is know that you can externally from outside the class like passing the value once I know like dependency injection, that is what they would think of when they hear write ones. And from the technical site, there is a related difference. And that difference is that new RFC only allows you to initialize read only properties inside the class scope. That means if you do something really weird, like leaving a property uninitialized in the constructor, it's not possible for someone outside the class to initialize it instead, just like an extra safety check. Of course, you can, as usual bypass that, like if you're writing a serializer or hydrator, you can use reflection to initialize it outside the class. But normally you won't be able to.

Derick Rethans  4:13  
	Does that mean that these read only properties can also be initialized from a normal method instead of just from the constructor?

Nikita Popov  4:19  
	That's true. Yes, that's possible.

Derick Rethans  4:21  
	So the RFC talks about that a read only property cannot be assigned from outside a class. Does that mean it can be set by a different object of the same class?

Nikita Popov  4:29  
	Yeah, that's how scoping works in PHP. Scoping is always class based, not object based, it's a common misconception that if you have like private scope, you can access different objects of the same class.

Derick Rethans  4:42  
	That was a surprise to me the first time I ran into that, but once you know, it's obvious that it's should work all over the place right. Now, the RFC states that you can only use read only with typed properties. Why is that?

Nikita Popov  4:55  
	So this is related to the initialisation concept, that typed introduced. Typed properties start out, if you don't give them a default value, they start out in an uninitialized state. And we reuse that state for read only properties. You can only assign to the property while it's uninitialized. And once it's initialized, you cannot assigned to it any more or even unset it back to an uninitialised state. For non typed properties, you also can get into this uninitialized state by explicitly unsetting it. But the problem is that this is not the default state. Untyped properties always have no default value, even if you don't specify one, which effectively means that these properties are always initialized. So they will be kind of useless if you used read only with them. Which is why we make this distinction to avoid any confusion. And if you want to use an untyped read only property, you do that by using the mixed type, which is the same but has the initialisation semantics of typed properties.

Derick Rethans  5:56  
	What would that mean if you have say, a resource or class typed property with a read only keyword? Can you not read or write to a resource any more? Or modify properties on an object, that is the value of a read only typed property?

Nikita Popov  6:12  
	No, you can still modify those, because we have to distinguish the concepts of like exterior and interior mutability here. So objects and resources are... Well, I mean, we often say they are passed by reference, which is not strictly true, because those are not PHP references. But the important part is that they only pass around some kind of handle and you can still modify the inside of that handle. What you can't do is you can't reassign to a different resource or reassign to a different object, it's always the same object, the same resource, but the insides of the objects, those can change. Of course, if your object also only contains read only properties, then you won't be able to change this.

Derick Rethans  6:55  
	Okay, and that answered another question that I have: how it possible to make the whole object read only or on all of its properties? Where the answer is by setting all the properties to read only.

Nikita Popov  7:05  
	We could like add a read only class modifier that makes all the properties implicitly read only but maybe future scope.

Derick Rethans  7:13  
	Is there a reason why read only properties can't have a default value?

Nikita Popov  7:18  
	So this is, again, same issue with initialization. If they have a default value, then they're already initialized. So you can't overwrite it. Like we could allow it, but you just would never be able to change them from the default value, which is something we could allow it just wouldn't be very useful.

Derick Rethans  7:36  
	in PHP, eight zero PHP introduced promoted or constructor promoted properties, I think it's the full name of it. How does this read only property tie in with that? Because can you set the read only flag on a constructor promoted property?

Nikita Popov  7:49  
	Yeah, you can. So that works as expected. Important bit there is that with the promoted properties, you can set the default value. And the reason is that for promoted properties, the default value is not the default value of the property. It's the default value of the parameter. And that works just fine.

Derick Rethans  8:07  
	But again, it would be a constant.

Nikita Popov  8:10  
	No, in that case, it's not a constant, because it's just a default value for the parameter and the code we generate, we just assign this parameter to the property IN the constructor.

Derick Rethans  8:19  
	Which means that if you instantiate the class with different arguments, they would of course, override the default value of the constructor argument value, right?

Nikita Popov  8:28  
	That's right. If you pass it explicitly, then we use the explicit value. If you don't pass an argument, then we use the default of the argument, but it still gets assigned to the property through the like automatically generated code for the promotion.

Derick Rethans  8:42  
	Promotion isn't actually... there isn't actually really a language feature, but more of a copy and paste mechanism.

Nikita Popov  8:50  
	Yes, this is pure bit of syntax sugar.

Derick Rethans  8:53  
	Which sometimes can be handy. But all kinds of interesting things that we added to properties or type system in general, usually inheritance comes into play. Are there any issues here with read only and inheritance? What does it do to variants or traits or things like that, all the usual things that we need to take into consideration?

Nikita Popov  9:13  
	Most of our rules for properties, most of our inheritance rules for properties are invariant, which means that the property in the child class has to basically look the same as the property in the parent class. And this is also true for read only properties. So we say that if the parent property is read only the child property has to be read only, and the same the other direction and for the type as well. So we say that the type of the read only property has to match with the parent property. Those rules are very conservative and like they could theoretically maybe be relaxed in some cases. For read only properties on read only is like kind of return type and return types in PHP are covariant. So one could argue that the type should also be covariant here The problem is that like here, we get into this little detail that read only is really not read only, but init once. So there is this one assignment. And assignment is more like an argument. So is contravariant. So if we made them covariant, then we would basically lie about this one initializing assignment. And we don't like to lie about these things, at least without some further consideration. 

Derick Rethans  10:24  
	Read only doesn't change the variance rules at all, which is different from the property accessors RFC we spoke about a couple of months ago. Talking about that this is actually a competing RFC, or do they tie together? 

Nikita Popov  10:37  
	Well, I should first say that probably the variance stuff defined in the accessors RFC maybe isn't right, for the same reason and should also like just keep things invariant. But it's more complicated there because it also has abstract properties, or abstract accessors. And then the abstract case, that's where the different variance rules are safe. To get back to your question, they are kind of competing, but could also be seen as just complimentary. So read only is basically a small subset of the accessors feature. I mean, accessors also allow you to implement read only properties as part of a much more general framework. And read only properties cover like only this small but very useful corner in a way that's much simpler. And I think that's not just simpler from a technical perspective, but also simpler to understand for the programmer, because there are a lot of less edge cases involved.

Derick Rethans  11:32  
	Because we're getting pretty close to feature freeze now. Are you still intending to take the property accessors RFC forward for PHP eight one? 

Nikita Popov  11:41  
	No, definitely not.

Derick Rethans  11:43  
	But you have taken the read only properties one forwards because we're already voting on it. At the moment, it looks like the vote is going to pass quite easily. So there's that. We don't have to speculate about that, like we had to do with previous RFCs. Do you think I've missed anything talking about read only properties? Or have we covered everything?

Nikita Popov  12:01  
	We've missed one important bit. And that's the cloning. So the read only properties RFC is not entirely uncontroversial. And basically all the controversy is related to cloning. The problem are wither methods as they're used by PSR seven, for example, which are commonly implemented by cloning the original object, and then modifying one property in it. This doesn't work with read only properties, because you know, if you clone the object, then it already has all the properties initialized. So if you try to change something, then you'll get an error that you're modifying read only property. So these patterns are pretty fundamentally incompatible. There are possible solutions to the problem primarily a dedicated syntax that goes into the code name of cloneWith where you clone an object, but override certain properties directly as part of the syntax, which could bypass the read only property checks. The contention here is basically there are some people consider this clone support and these wither methods are very important. Well, I mean, read only properties are about immutable objects. And like PSR seven is a fairly popular read only object pattern, they think that without support for cloning or for withers, the feature loses all value. That's why the alternative suggestions are either to first introduce some more cloning functionality. So like this mentioned, cloneWith or instead implement asymmetric visibility, which does not have this problem because inside the class, you can always modify the property. So it works perfectly fine with cloning. My personal view on this is well I personally use cloning in PHP approximately never. So this is not a big loss for me, and I'm happy for this from my perspective, edge case, to be addressed at a later time. I can understand that other people put more value on this then I do.

Derick Rethans  13:55  
	There doesn't seem to be enough push against that for this RFC to fail. So there is that. Thanks Nikita for explaining the read only properties RFC which will very likely see in PHP eight one. Thanks for taking the time.

Nikita Popov  14:08  
	Thanks for having me Derick, once again.

Derick Rethans  14:13  
	Thank you for listening to this installment of PHP internals news, a podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast as well as the Xdebug debugging tool, you can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening and I'll see you next time.


Show Notes
----------

- RFC: `Read Only Properties <https://wiki.php.net/rfc/readonly_properties_v2>`_
- RFC: `Write Once Properties <https://wiki.php.net/rfc/write_once_properties>`_
- Episode #44: `Write Once Properties <https://phpinternals.news/44>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
