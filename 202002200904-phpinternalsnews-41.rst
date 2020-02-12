PHP Internals News: Episode 41: __toArray()
===========================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-02-20 09:04 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin041
   :Enclosure: media/pin-041.mp3

In this episode of "PHP Internals News" I chat with Steven Wade (`Twitter
<https://twitter.com/stevenwadejr>`_, `GitHub <https://github.com/stevenwadejr/>`_,
`Website <https://www.stevenwadejr.com/>`_)
about the __toArray() RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-041.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:16  
	Hi, I'm Derick. And this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. Hi, this is Episode 41. Today I'm talking with Stephen Wade about an RFC that he's produced, called __toArray(). Hi, Steven, would you please introduce yourself? 

Steven Wade  0:35  
	Hi, my name is Steven Wade. I'm a software engineer for a company called follow up boss. I've been using PHP since 2007. And I love the language. So I wanted to be able to give back to it with this RFC. 

Derick Rethans  0:48  
	What brought you to the point of introducing this RFC? 

Steven Wade  0:50  
	This is a feature that I've I've kind of wish would have been in the language for years, and talking with a few people who encouraged it's kind of like the rule of starting a user group right? If there's not one and you have the desire, then you're the person to do it. A few people encouraged and say: Well, why don't you go out and write it. So I've spent the last two years kind of trying to work up the courage or research it enough or make sure I write the RFC the proper way, and then also actually have the time to commit to writing it and following up with any of the discussions as well. 

Derick Rethans  1:18  
	Okay, so we've mentioned the word RFC a few times. But we haven't actually spoken about what it is about. What are you wanting to introduce into PHP? 

Steven Wade  1:25  
	I want to introduce a new magic method. The as he said, the name of the RFC is the __toArray(). And so the idea is that you can cast an object, if your class implements this method, just like it would toString(). If you cast it manually to array then that method will be called if it's implemented. Or as, as I said, in the RFC, array functions will it can it can automatically cast that if you're not using strict types. 

Derick Rethans  1:49  
	Oh, so only if it's not strictly typed. So if its weakly typed would call the toArray() method if the function's argument or type hint array. 

Steven Wade  1:58  
	Yes, and that is actually something that came up during the discussion period, which is something again, this is why we have discussions, right? Is to kind of solicit feedback on things we don't think about it, we may overlook or, and so someone did point out that it is, you know, it would not function that way, or you would not expect it to be automatically cast for you, if you're using strict types. 

Derick Rethans  2:17  
	Okay. 

Steven Wade  2:18  
	The RFC has been updated to reflect that as well. 

Derick Rethans  2:20  
	So now the RFC says it won't be automatically called just for type hint. 

Steven Wade  2:24  
	Correct. 

Derick Rethans  2:24  
	Not everybody is particularly fond of magic methods. What would you say about the criticism that introducing even more of them would be sort of counterproductive, because you'll end up not necessarily knowing what happens if you start calling a method, when you do a cost, for example. 

Steven Wade  2:38  
	The beauty of PHP is in its simplicity. And so adding more and more interfaces, kind of expands class declarations enforcement's and in my opinion, can lead to a lot of clutter. And so I think PHP is already very magical. And the precedent has been set to add more magic to it with 7.4 with the introduction of serialize and unserialize magic methods, and so for me it's just kind of a, it's a tool. I don't think that it's necessarily a bad thing or a good thing. It's just another option for the developer to use. 

Derick Rethans  3:06  
	Two episodes ago, I spoke with Nicolas Grekas about a Stringable interface that he suggested to introduce, which is a little bit similar to sort of the casting with toArray(). And hence, do you think it would have make sense to have an __toArray() also happen if the class implements a interface with a typed function argument? 

Steven Wade  3:29  
	I think that would be two separate RFCs. I think the first one to kind of get it on par with what's what we have now in PHP would be to introduce the toArray(). And then a separate one would be if we wanted to follow suit with an arrayable interface. 

Derick Rethans  3:43  
	And which is the same thing that happens with the Stringable interface, right? We have had toString() for how many years, decades? But from what I understand, if you have a typed property "string", it would also call the toString() method when it's defined on an object that's being passed in, or do I misunderstand that, there are misremember that? 

Steven Wade  4:00  
	I haven't followed that one too closely. I've kind of been catching up on some of the discussion today. But and yeah, I don't know off the top of my head what that would do. 

Derick Rethans  4:07  
	I didn't mean with the ori.. with the newly suggested Stringable interface with adults we currently have. 

Steven Wade  4:12  
	I'm not sure how that would work. 

Derick Rethans  4:13  
	I don't know, either. That's what I'm asking you.

Steven Wade  4:15  
	With the array and with the typed properties? That's a good question. That's again some feedback, we kind of need to that I need to think through 

Derick Rethans  4:21  
	Because I think it would make sense to at least behave the same and I don't particularly mind which way it goes. Me that's, that's a personal opinion here. 

Steven Wade  4:28  
	And that's a great idea I need to haven't played with 7.4 too much, I need to pull it down and try and just see what the behaviour of string is because that's the main goal of this is to try and just get this on a parity, functionality parity with with what's toString() will do. And so if that is how it handles it with typed properties and I would want to implement that as well. 

Derick Rethans  4:47  
	In a similar way. I don't also know what happens if if you have toString() available in a class and you pass it in as an argument that is typed as string.

Steven Wade  4:54  
	Even though at least when my test was weak types, it will actually cast that for you. If you have that. String argument type hint, it will cast it and then that will be a copy. So it will actually just be the result of that cast to string. I do not think I think it throws an error if you have a strict type set.

	No, I think it'd be very similar, right. It's just how you want to use it in user land, you know, the __toArray() is you're going to you could cast it yourself ,or you can with weak types PHP could cast for you in the appropriate circumstances. If you want the same functionality. In some for now, you would need to call, you know, the __serialize() yourself with the toArray(). In the future, you could implement the toArray() and then your serialize could actually just cast this object to array, and then that should actually convert that for you. And then serialise will then return array so you're not duplicating how you want that object represented when it's an array. 

Derick Rethans  6:00  
	So the RFC mentions that when you do a print_r of person is called __toArray(). But that's not particularly a cast. So why would it do it here, but not for method arguments, for example?

Steven Wade  6:11  
	That is a product of this being my first time and that was a mistake that was thankfully pointed out during the discussion period and has been corrected.

Derick Rethans  6:19  
	I read this RFC a week or two ago or so. And I haven't.. I should have reread it this morning that. I did not so my apologies for not being fully up to date here. There's some array functions in PHP like sort() that operate on an array as a reference right? That can't particularly work if you first have to cast to an array, which is what your current RFC now just. I mean, toArray() only gets called when you cast to an array or when it's a weakly typed argument. But how would it work for methods or functions that accept an array by reference?

Steven Wade  6:49  
	At least the way I proposed it, they would throw an error as it currently does. Again for my test and trying to keep this within parity with the toString. I don't believe there are many functions that will operate on toString on, on a string by reference, as there are with arrays. From what I can recall is that it would throw an error. If you try to operate by reference on an object that implements toString, it will throw an error. 

Derick Rethans  7:10  
	And it wouldn't just fall back to using an object because that'd be very strange behaviour in that case, I suppose. 

Steven Wade  7:15  
	Basically, if it's if it's not something that can be cast or converted to an array through this method, and it's just going to be the same functionality you have in current PHP, which will be throw an error.

Derick Rethans  7:24  
	Going to go for the principle of least astonishment or something.

Steven Wade  7:27  
	Yeah, I don't want to introduce too many changes to it. I just want to be able to cast.

Derick Rethans  7:31  
	I think that is a great idea. Actually, I mean, the same thing I've spoken with Nikita about, that introducing features step by step makes it a lot easier for people to comprehend what you actually end up doing. And there's also less, less chance of people getting bogged down in liking a specific aspect of the RFC but not of the other RFC parts. And we end up not merging the whole thing with the sub part of it.

Steven Wade  7:54  
	And that's why I was very purposeful and not including any kind of write. You write, you cannot write to a class that implements toArray(). You know, as you will with array ArrayObject, because that we have that for a reason. So this is different functionality, we just wanted to keep it small, and just have this little helper

Derick Rethans  8:11  
	I read in the RFC, something called get_mangled_object_vars(), but I didn't quite understand what it was. 

Steven Wade  8:16  
	So that was actually a function introduced in 7.4, as a direct result of my original proposal trying to see what people thought in the internals and in the community of this feature. Sometime in spring, last year 2019, I began this discussion, and there was some initial feedback with folks saying that it would cause some breaking changes in their libraries or their code, because they are overloading the casting. Right now, if you cast an object, I guess you get insight into the object's internals without any side effects. And so I think that's how Symfony's var dumper works. And that's how they're able to display some of that information. So that was concern by introducing this, that functionality would break. And so to introduce a method that would give you the same benefits without overloading the casting, the get_mangled_object_vars() was introduced and accepted and implement in 7.4.

Derick Rethans  9:04  
	And that returns the object properties with their special characters in place. Because PHP internally, if you have a private method, the name for both methods and property is done by doing a null character, the name of the class, a null character then the property name. So that's what that would return, I suppose. 

Steven Wade  9:22  
	I believe so. 

Derick Rethans  9:22  
	I ran into a similar issue in Xdebug, because in some cases, you want to call get_debug_info, which is what people implement for getting debug info for their objects. But in other cases, you don't want to do it because you want to see everything that happens internally, or you want to see all the properties that exist. So there's kind of a tricky one. And I think at some point with toArray also happening, I might actually end up adding the output of both toArray() and get_debug_info separate sort of fake properties into the Xdebug output. But of course that only works if toArray() has no side effects. I don't think there's any way of preventing that in the toArray method that you can now implement that it doesn't change any information in normal properties, for example, right?

Steven Wade  10:12  
	And that's kind of some of the internals of it that I'm not fully familiar with. With it, I'm hoping to kind of, you know, the discussion period will help eliminate some of that.

Derick Rethans  10:20  
	I don't think you'd be able to actually.

Steven Wade  10:22  
	Just recently, we were able to throw an exception from the toString. I don't know if you can actually do any kind of operations, write operations on the object within the toString? I do? That's a good question. And I do look that up. And whatever that behaviour is, we'd want to mimic here as well.

Derick Rethans  10:34  
	I believe you can. It's normal PHP code, right? And if you don't want to do it, you need to clone it first, which is something you could choose as an implementation, right? You could first clone the class and then call the toArray method on the cloned object. I don't think we have any protection for that. The RFC is currently in the discussion phase. At the time of recording, we're talking about the discussion period. When I sort of thinking of ending that and going for vote?

Steven Wade  10:58  
	I think this is actually going to be probably a longer period of discussion. And I think most RFC is most fleshed out just because of the nature of it. I am a full time employee full time, father, husband, and also student, as well. And so I don't have a lot of time to do this. And I want to do it right. I want to be able to respond to this. And so the discussion opened up a week ago, and this morning is the first time I've had to be able to respond to that and update the RFC. And so I because I really care about this and would love this feature to go in. I want to continue to solicit discussion and advice and questions and to be able to answer them all and do that. So however long it takes. Ideally, I would love it to be closed, voted on, accepted and implemented in time to be able to get in for the feature freeze for 8.0. 

Derick Rethans  11:40  
	For that you have about four months. Would you have anything else to add that I forgot? Or you want to add that you think it's interesting to know about this RFC? 

Steven Wade  11:50  
	Yeah, the only thing I would add is I've seen discussion, someone posted the RFC on Reddit and I've seen discussions with people like it, people hate it. They want to move one way or the other again, it's just It's a small feature, it's a helper. It's a tool that you can use. Is it perfect? No. Is it going to satisfy everybody? No. You've got the people who are want more functional and procedural you got people who want more OOP. I think it's just another helpful tool that could be in your tool belt. If you use it great. If you don't, you don't have to touch it.

Derick Rethans  12:19  
	Very well. Thank you, Steven, for taking the time to talk to me this afternoon. I'm looking forwards on this coming to vote at some point.

Steven Wade  12:27  
	Thank you for having me on the show. And let me explain the purpose and the reasoning behind this RFC. And thank you very much for giving a voice to those looking to improve the language.

Derick Rethans  12:35  
	You're most welcome. Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions feel free to email them to derick@phpinternals.news. Thank you for listening and I'll see you next week.

Show Notes
----------

- RFC: `__toArray() <https://wiki.php.net/rfc/to-array>`_


Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
