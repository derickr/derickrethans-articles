PHP Internals News: Episode 100: Sealed Classes
===============================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2022-03-24 09:04 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin100
   :Enclosure: media/pin-100.mp3

In this episode of "PHP Internals News" I talk with Saif Eddin Gmati (`Website
<https://les-tilleuls.coop>`_, `Twitter <https://twitter.com/azjezz>`_,
`GitHub <https://github.com/azjezz>`_) about the "Sealed Classes" RFC that he
has proposed.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-100.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:14
	Hi, I'm Derick. Welcome to PHP internals news, the podcast dedicated to explaining the latest developments in the PHP language. This is episode 100. Today I'm talking with Saif Eddin Gmati about the sealed classes RFC that they're proposing. Saif, would you please introduce yourself?

Saif Eddin Gmati  0:31
	Hello, my name is Saif Eddin Gmati. I work as a Senior programmer at Italy. I'm an open source enthusiast and contributor.

Derick Rethans  0:39
	Let's dive straight into this RFC. What is the problem that you're trying to solve with it?

Saif Eddin Gmati  0:43
	Sealed classes just like enums and tagged unions allow developers to define their data models in a way where invalid state becomes less likely. It also eliminates the need to handle unknown subtypes for a specific model, as using sealed classes to define models gives us an idea on what child types would be available at run time. Sealing also provides us with a way for restricting inheritance or the use of a specific trait. For example, if we look at logger trait from the PSR log package that could be sealed to logger interface. This way, we ensure that every use of this trait is coming from a logger not from any other class.

Derick Rethans  1:24
	I'm just reading through this RFC tomorrow, again, and something I didn't pick up on reading to it last time. It states that PHP already has sort of two sealed classes.

Unknown Speaker  1:35
	Yes, the throwable class in PHP can only be implemented by extending either error or exception. The same applies for DateTime interface, which can only be implemented by extending DateTime class or DateTime Immutable class.

Derick Rethans  1:52
	Because PHP itself doesn't allow you to implement either throwable or DateTimeInterface. I haven't quite realized that that these are also sealed classes really.  What is sort of the motivation behind wanting to introduce sealed classes?

Unknown Speaker  2:06
	The main motivation for this feature comes from Hack the programming language. Hack contains a lot of interesting type concepts that I think personally, PHP could benefit from and sealed classes is one of those concepts.

Derick Rethans  2:18
	What kind of syntax are you proposing?

Saif Eddin Gmati  2:21
	The syntax I'm proposing actually there is three syntax options for the RFC currently, but the main syntax is inspired by both Hack and Java. It's more similar to the syntax used in Java as Hack uses attributes. Personally, I have been I guess, using attributes from the start as I personally see sealing and finalizing similar as both effects how inheritance work for a specific class. Having sealed implemented as an attribute while final uses a keyword brings more inconsistency into the language which is why I have decided not to include attributes as a syntax option.

Derick Rethans  2:56
	In my opinion, attributes shouldn't be used for any kind of syntax things. What they should be used for is attaching information to already existing things. And by using attributes again, to extend syntax, you sort of putting this syntax parsing in two different places , right? You're putting it both in the syntax as well as in attributes. I asked what the syntax is, but I don't think he actually mentioned what the syntax is.

Saif Eddin Gmati  3:20
	The syntax the main set next proposed for the RFC is using sealed and permit as keywords we first have the sealed modifier which is added in front of the class similar to how final or abstract modifiers are used. We also have the permit clause which is basically a list allows you to name a specific classes that are able to inherit from this specific type.

Derick Rethans  3:43
	So when you say type here, is that just interfaces and classes or something else as well?

Saif Eddin Gmati  3:48
	It's classes interfaces and traits. Traits are allowed to add sealing but they are not allowed to permit. Okay for example, an interface is not allowed to permit a trait because a trait cannot implement an interface

Derick Rethans  4:03
	In the language itself, when does this get enforced?

Saif Eddin Gmati  4:06
	This inheritance restriction gets enforced when loading a class. So let's say we are loading Class A currently if this class extends B, we check if B is sealed. And if it is we check if B allows A to extend it. But when loading a specific sealed class, nothing gets actually checked. We just take the permit clause classes and store them and move on.

Derick Rethans  4:32
	It only gets checks if you're trying to implement an interface.

Saif Eddin Gmati  4:36
This gets enforced when trying to implement an interface, extend that class, or use it trait.

Derick Rethans  4:41
Okay. What are general use cases for this feature?

Saif Eddin Gmati  4:45
General use cases for a feature are for example, implementing programming concepts such as Option which is a type that can only have two subtypes. One is Some, other is None. Another concept is the Result where only two subtypes are possible, either success or failure. Another use case is to restrict inheritance. As I mentioned before, for example, logger trait from the PSR log package is a trait that implements some of the method methods in logger interface, and expects whoever is using that trait to implement the rest. However, there is no restriction by the language regarding this, we can seal this trait to a logger interface ensuring that only loggers are allowed use this trait.

Derick Rethans  5:34
When you say that Option has like the value Some or None, just sound like an enum to me. How should I think differently about enums and sealed classes here?

Saif Eddin Gmati  5:43
	Enums cannot hold a dynamic value. You can have a value but you cannot have a dynamic value, however, tagged unions will allow you to implement option the same way. Tagged unions are that useful only for this specific case, there is some other cases such as the one I mentioned for traits that cannot actually be implemented using the tagged unions. There is also the I don't know how to say this. Let's say we have a type A that sealed and permitting only B and C. And this case A on itself, as long as it's not an abstract class, is by itself a type. Can be used as a normal class, you can create an instance and use it normally. However with tagged unions, the option itself would not be a type, you either have some or none. That's the main difference between tagged unions until classes

Derick Rethans  6:37
	A tagged union PHP doesn't have them. So how does a tagged union relate to enums?

Saif Eddin Gmati  6:43
	With tagged unions as the, there is an RFC that's still in draft, I suppose that uses actually it is built on top of enums that that's why.

Derick Rethans  6:55
	I reckon once that gets closer to completion, I'll end up talking to the author of that RFC. So something I'm wondering, can a sealed type permit only one other type? Or does it have to be more than one?

Saif Eddin Gmati  7:10
	No, it can permit only one type. Let's say we have class A that only permits B. However, another thing is class B does not actually have to extend A, like if A is permitting B, B does not actually have to implement A. It's still useful because another class called C can extend B and implement A, so an instance of A B can still exists.

Derick Rethans  7:36
	I'm not quite sure whether I understood that. If you have an interface that says A permits B, then B is not required to implement A, mostly because the moment you loads class B, you don't even know it exists, right? Because it doesn't refer to it.

Saif Eddin Gmati  7:54
	Yes.

Derick Rethans  7:55
	It's just going to break anything?

Saif Eddin Gmati  7:57
	Hopefully not. The only break would be in the new reserved keywords which are sealed and permits. So those cannot be used as identifiers any more, but depending on the syntax choice, if for example, the second syntax choice wins which that would only take the permits keyword. If the third syntax choice is chosen then no new reserved keywords will be introduced so there will be no breaks.

Derick Rethans  8:29
	From what I see in the RFC the first syntax is using both sealed in front of a as a marker and then using permits. With the second syntax, you don't use seal but you infer that it is sealed from the permits keyword I suppose. And then in the last option you use the for keyword instead of permits and also don't use sealed yet?

Saif Eddin Gmati  8:51
	The third syntax choice is will be the one with no breaks as we will not be introducing any new keywords; for is already a reserved keyword in PHP.

Derick Rethans  9:02
	What is your preference?

Saif Eddin Gmati  9:03
	Personally I prefer the first syntax choice as it's the most explicit. When you start reading the code you can tell from the start this is a sealed class without having to continue reading until you reach permits.

Derick Rethans  9:15
	I think I agree with you there. Beyond the syntax is there anything else that needs to be changed in PHP itself?

Saif Eddin Gmati  9:22
	The only other change that will be introduced in PHP is in reflection class. A new method called isSealed will be added to reflection method, which allow you to check if a class the class being reflected is sealed. Another method will be added called getPermittedClasses which returns the list of class names provided in the permits clause. Also a new constant should be added to reflection class that is is_sealed constant which exposes the bit flag used for sealed classes. Some changes will happen to the getModifiers method in reflection class. This method will return the bit flag is sealed set, if the class being reflected is sealed. The getModifierNames method will also return the string sealed if the bit is set, that should be about it.

Derick Rethans  10:12
	Basically everything that you need in reflection to find out whether it's a sealed class and other permits.

Saif Eddin Gmati  10:18
	Yes.

Derick Rethans  10:20
	See, I see the name of getPermittedClasses has to use, has the word classes in it. Does that mean that the types after permits have to be classes?

Saif Eddin Gmati  10:32
	No, they can be either classes or interfaces. But PHP refers to both classes and interfaces as classes in the reflection. So we have a reflection class, but that's actually a reflection trait class interface. And basically everything is class-ish.

Derick Rethans  10:47
	Class-ish. I like that. Did you look at some other alternatives to implementing the same feature or just the three syntax choices that you came up with?

Saif Eddin Gmati  10:56
	I did not consider any other alternatives precisely as the alternatives might be type aliases, tagged enums, package visibility. But I think each of these RFCs focused on a specific problem and expanding that area, while sealed classes focuses on all the problems mentioned on in this RFC tries to solve them in a minimal way. But only in relation to inheritance in classes, interfaces, and traits.

Derick Rethans  11:24
	Keeping it short and sweet. What has the feedback been so far?

Saif Eddin Gmati  11:29
	The feedback has been pretty mixed. Some people are against adding more restriction to types and inheritance. But in my opinion, this is not about adding restriction, but rather providing the user with the ability to add restrictions. And we already have final classes, which a lot of people seem to dislike.

Derick Rethans  11:48
	I don't understand why. But fair enough.

Saif Eddin Gmati  11:51
	I have created a community poll a couple of weeks ago to gather feedback on Twitter. The results were 60% for with over 150 participants. Another poll was created by Peter on Facebook ended with 54 of people voting yes. However, such polls that do vary depending on the audience. So it can be really an accurate representation of the PHP community.

Derick Rethans  12:15
	Polls on Twitter are never scientific, or they? I see that the RFC is in voting already. So for people listening to this, and if you have voting rights, then you have until when exactly?

Saif Eddin Gmati  12:28
	Until the end of the month.

Derick Rethans  12:30
	March 31. It says yes. Okay. Well, thank you very much for taking the time today Saif about sealed classes.

Saif Eddin Gmati  12:37
	Thank you for having me. Hopefully, I get to be here another time in the future.

Derick Rethans  12:42
	I hope so too. Thank you for listening to this installment of PHP internals news, a podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next time.



Show Notes
----------

- RFC: `Sealed Classes <https://wiki.php.net/rfc/sealed_classes>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
