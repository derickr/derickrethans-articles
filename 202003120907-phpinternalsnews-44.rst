PHP Internals News: Episode 44: Write Once Properties
=====================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-03-12 09:07 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin044
   :Enclosure: media/pin-044.mp3

In this episode of "PHP Internals News" I chat with Máté Kocsis (`Twitter
<https://twitter.com/kocsismate90>`_, `GitHub <https://github.com/kocsismate>`_,
`LinkedIn <https://www.linkedin.com/in/kocsismate90/>`_)
about the Write Once Properties RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-045.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:16
	Hi, I'm Derick. And this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 44. Today I'm talking with Máté Kocsis about an RFC that he produced called write only properties. Hello, Máté. How's it going?

Máté Kocsis  0:34
	Yeah, fine. Thanks.

Derick Rethans  0:36
	Would you mind introducing yourself a moment?

Máté Kocsis  0:38
	My name is Máté Kocsis and I'm a software engineer at LogMeIn. I've been using PHP for 15 years now. And after having followed the mailing list for quite some time, I started contributing to the project last October, and now Write Once properties is my first RFC.

Derick Rethans  0:58
	What is the concept of Write Once Properties?

Máté Kocsis  1:00
	Write Once Properties can only be initialised, but not modified afterwards. So you can either define a default value for them, or assign them a value, but you can't modify them later. So any other attempts to modify, unset, increment, or decrements them, would cause an exception to be thrown. Basically, this RFC would bring Java's final properties, or C#'s, read only properties to PHP. However, contrary how these languages work, this RFC would allow lazy initialization. It means that these properties don't necessarily have to be initialised until the object construction ends, so you can do that later in the object's life cycle.

Derick Rethans  1:48
	PHP already has constants, which are pretty much write only properties as long as they're being defined in a class definition. How does differ?

Máté Kocsis  1:58
	Yeah, it's it's the difference because, so you can assign these properties value in the constructor or anywhere. You don't don't have to define them a default value.

Derick Rethans  2:12
	Okay, and of course constants have the other problem is that you can only set its values to constants, not necessarily to any sort of expressions, or the result of other method calls.

Unknown Speaker  2:22
	So you can use objects, resources, any kind of property value here.

Derick Rethans  2:28
You mentioned C#'s read only properties. And you sort of mentioned them in the same breath as write ones properties for PHP. These seem like opposite things

Máté Kocsis  2:39
	Not quite opposite, but there's some distinction between the two. C sharp requires these properties to be initialised until the object construction ends. And this is very difficult to achieve in PHP. And now I'm using Nikita's words: Object construction is a fuzzy term and you can be sure if, if the contractor is involved at all. For example, if you are using Doctrine or proxy manager, so we decided to allow lazy initialization, which means that you don't have to assign these properties a value, you are free to do anytime when you want.

Derick Rethans  3:22
	What happens if you read them without them having being set yet?

Máté Kocsis  3:27
	Initially, when I started working on this proposal, I faced the problem because untyped properties have an implicit default value in the absence of an explicit default value. That's why you just can't really use them with the write once properties. Either you have a default value or you can do anything with them. That's why we we had to only allow typed properties with the write once properties and typed properties are in an uninitialised state by default. You can't read them until you first assign them a value.

Derick Rethans  4:04
	Because in PHP 7.4 that will throw a type error. So that actually ties in really nicely with PHP 7.4's initialise concept for the type hinted properties.

Máté Kocsis  4:14
	Yes.

Derick Rethans  4:15
	One thing that is slightly skipped over is which keyword does the RFC produced, because you mentioned final for Java and read only for C sharp, which one of you picked for PHP?

Máté Kocsis  4:25
	So there were plenty of possibilities considered. The first one was the final keyword. At first, it seemed to be the obvious choice for me, but after thinking about it, I turned out that it's not not the right candidate because currently it affects inheritance rules in PHP. And now we are talking about mutability rules. We had sealed which comes from C sharp and the problem is the same because it also affects inheritance rules, so we shouldn't reuse it for different purposes. We also consider immutable. It's one I like. But it might be a little bit misleading because the usage of immutable data structures, like objects or resources are not restricted at all. Then there's locked, which is a bit too abstract or vague name.

	We also have writeonce as well. And technically, it's the most accurate term. But from the user's point of view, it could be a bit confusing because they are not expected to write them at all, only the read these properties. And now we have readonly and probably this keyword get the most traction so far. And it's good. It's a good name because it refers to what users should generally do with these properties. However, there's also a slight problem that users can, or in some circumstances can, write these properties too. But that's not the general use case.

Derick Rethans  6:10
	It's a curious thing. I remember we had a PHP developers meeting back in 2000, let's say 2008. But it could as well have been 2005, where we also actually spoke about read only properties, but I'm going to have to dig up the notes for that to see what it said there. Maybe you find it interesting to read to see what the history said about this.

Unknown Speaker  6:31
	I'm curious. The question is open, so I plan to put it to vote.

Derick Rethans  6:37
	When do you think you're putting it up for a vote?

Unknown Speaker  6:39
	I think it should be close now. I will answer the mail, which came from Nicholas. I don't know if there is no more problems than we could do it this week or early next week.

Derick Rethans  6:54
	As the properties are write once, how will she implement lazy loading with that? In order to do the lazy loading, you need to first figure out whether the property is already set. How will you know that it's already set? How can you check for that?

Máté Kocsis  7:07
	I think generally you don't have to worry whether a property's write once or or not. Since mainly, we are talking about private or protected properties in the most cases. However, if you need this information, then you will be able to use reflection. I've already added support for method in in ReflectionProperty for this purpose.

Derick Rethans  7:31
	Let me ask a little bit more about that. You mentioned that this is meant for lazy loading. I understand lazy loading is something that you do well, you're executing and all the methods. For example, on an object, you do get something and that needs to fetch things from a database. Because those write once properties are private or protected, most of the time, the code that fetches the things from the database that does the lazy loading still needs to know whether the properties already been written to. Because if it would attempt it again, you'd potentially get an exception. So how would it know it's already been written to?

Máté Kocsis  8:03
	Good question. I was talking with with Marco Pivetta. His use case with proxy manager is to unset these properties in advance and then it can use the get or set or I don't know which magic methods.

Derick Rethans  8:28
	I saw that the RFC mentioned a few other alternative approaches for this feature. And the headlines in the RFC say: read only semantics, write before construction semantics, and property accessors. Would you mind explaining these and why they haven't made the final RFC?

Máté Kocsis  8:44
	The first one was to follow Java and C sharp, and require all write once properties to be initialised until the object construction ends. And this is what we talked about before. The counter arguments were that it's not easy to implement in PHP. This approach is unnecessarily strict. The other possibility is to let our limited writes to these properties until object construction ends and then do not allow any writes. But positive effect of this solution is that it plays well with bigger class hierarchies, where possibly multiple constructors are involved, but it still has the same problems as the previous approach. Finally, the property accessors could be an alternative to write once properties, although in my opinion, these two features are not really related to each other. But some say that property accessors could alone prevent some unintended changes from the outside and they say that maybe it might be enough. I don't share this sentiment. So in my opinion, unintended changes can come from the inside, so from the private or protected scope. And it's really easy to circumvent visibility rules in PHP. There are quite some possibilities. That's why it's a good way to protect our invariants.

Derick Rethans  10:15
	What was the most criticism you got on the mailing list about his proposal?

Máté Kocsis  10:18
	As far as I remember, the property accessor. The biggest criticism was that we don't really need this term, but we could use property accessors.

Derick Rethans  10:29
	We have spoken a little bit about what this feature is. We went into a few use cases with lazy loading. What would other use cases for this be?

Máté Kocsis  10:38
	I think it's really suitable for domain driven design, or working with value objects, and I'm a great fan of DDD. The problem is PHP can't guarantee any immutability for our objects. Just one example. You can invoke the object constructor as many times as you wish, which overrides all your properties.

Derick Rethans  11:04
	I had not thought about that you can actually call the constructor yourself. And of course you can.

Máté Kocsis  11:08
	Yes, me neither. I just saw somewhere probably in a previous discussion about immutable objects. That's the advantage of having write once properties. You could by using write once properties, yeah, you can prevent accidental modifications from the outside or from the inside too. And that's the main purpose.

Derick Rethans  11:32
	Your main purpose wasn't lazy loading but more immutable value objects.

Máté Kocsis  11:36
	Yes, yes. Right. I proposed right fans properties first, to pave the road for immutable objects because this is my main goal.

Derick Rethans  11:46
	Okay, but you're going step by step. I think that's actually a wise way and Nikita have said something similar that it is nicer to take things little by little so that it is easier to convince people that this is a good feature or not.

Unknown Speaker  11:59
	Actually it was Nikita's idea to split the two proposals.

Derick Rethans  12:03
	That make sense. Okay, Máté, thank you for taking the time this morning to talk to me.

Máté Kocsis  12:08
	Thank you for having me.

Derick Rethans  12:11
	Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening and I'll see you next week.


Show Notes
----------

- RFC: `Write Once Properties <https://wiki.php.net/rfc/write_once_properties>`_
- `PHP Developers Meeting Notes <https://derickrethans.nl/files/meeting-notes.html>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
