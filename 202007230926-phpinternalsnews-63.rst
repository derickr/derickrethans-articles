PHP Internals News: Episode 63: Property Write/Set Visibility
=============================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-07-23 09:26 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin063
   :Enclosure: media/pin-063.mp3

In this episode of "PHP Internals News" I talk with André Rømcke
(`Twitter <https://twitter.com/andrerom>`_, `GitHub
<https://github.com/andrerom>`_) about an RFC that he is working on to get
asymmetric visibility for properties.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-063.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:16
	Hi, I'm Derick, and this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 63. Today I'm talking with André Rømcke, about an RFC that he's proposing titled property write/set visibility. Hello André, would you please introduce yourself?

André Rømcke  0:38
	Hi Derick, I'm, where to start, that's a wide question but I think I'll focus on the technical side of it. I'm been doing PHP for now 15 plus years. Actually started in, eZ systems, now Ibexa, met with you actually Derick.

Derick Rethans  0:56
	Yep that's a long time ago for me.

André Rømcke  0:58
	And well I came from dotnet and front and side of things. Eventually I learned more and more in PHP and you were one of the ones getting me deeper into that, while you were working on eZ components. I was trying to profile and improve eZ publish which what's left of the comments on that that point. A long time ago, though. A lot of things and I've been working in engineering since then I've been working now actually working with training and more of that, and the services and consulting. One of the pet peeves I've had with PHP has been properties for a long time, so I kind of wanted several, several years ago to look into this topic. Overall, like readonly, immutable. To be frank from our side that the only thing I really need is readonly, but I remember the discussions I was kind of involved for at least on the sideline in 2012/ 2013. Readonly and then there was property accessors. And I remember and there was a lot of people with different needs. So that's kind of the background of why I proposed this.

Derick Rethans  2:04
	We'll get back to these details in a moment, I'm sure. The title of the RFC is property write/set visibility. Can you give me a short introduction of what visibility actually means in this contract, in this context?

André Rømcke  2:16
	Visibility, I just use the word visibility because that's what doc usually in php.net says, but it's about access the property, so to say, or it being visible to your code. That's it on visibility but today we can only set one rule so to say, we can only say: this is either public, private, or protected, and in other languages, there are for good reasons, possibilities to have asynchronous visibility. So, or disconnected or whatever you want to call it, between: Please write and read. And then in, with accessors you'll also have isset() and unset() but that's really not the topic here at this point.

Derick Rethans  2:56
	PHP sort of supports these kind of things, with it's magic methods, like with __get() and __set(). And then also isset() and unset(). You can sort of implement something like this, of course, in your own implementation, but that's, of course, ugly, I mean, ugly or, I mean there was no other way for doing it and I remember, you made have a user that in eZ Components that you mentioned earlier. What is the main problem that you're wanting to solve with what this RFC proposes?

André Rømcke  3:25
	the high level use case is in order to let people, somehow, define that their property should not be writable. This is many benefits in, when you have multiple API's, in order to say that I have this property should be readable. But I don't want anyone else about myself to write to it. And then you have different forms of this, you have either the immutable case were you literally would like to only specify that it's only written to in constructor, maybe unset in destructor, may be dealt with in clone and so on, but besides that, it's not writable. I'm not going into that yet, but I'm kind of, I was at least trying to lay the foundation for it by allowing the visibility or the access rights to be asynchronous, which I think is a building block for moving forward with immutability, we only unintentionally also accessors but even, but that's a special case.

Derick Rethans  4:24
	What is the syntax that you're suggesting to implement this, this feature?

André Rømcke  4:30
	Currently in the RFC there's two proposals. There's one where you have, for instance public:private. First you define the read visibility and then you run the find the write visibility. That was based on the email list back in 2012 proposal made back then. The one is inspired by Swift, the apple language, where they actually have a concept for this. So they have public than being the read access and then they have private and in the parenthesis they have (set). Basically this is then set. Benefit of this maybe it will probably be a better match for our reflection API, in terms of getting the modifier names and stuff like that. Secondly, the terminology with set they're kind of just nicely with accessors, so that's why I added this alternative suggestion to the syntax.

Derick Rethans  5:27
	Would either of these would it be possible to extend this to get actual setters and getters later?

André Rømcke  5:32
	Yes. Well this is a good question actually and Larry brought this up on the mailing list. In terms of how this actually should align up to a potential future accessors, if that is ever added. Made a good point, we started to rethink this a bit so I think I might have to discuss with him and also Nikita on this, but his point was: okay so if we add this now, how will this look, if you add accessors later to your code. The mindset of thinking I was thinking, this would be an either or. Oh, this would be kind of a shorthand for what you can specify in much more details with accessors. So once you use accessors you would remove this word, because it doesn't have a meaning any more. It's basically language syntax sugar, so to say, or short hand.

Derick Rethans  6:21
	Besides the two that are in the RFC. Did you consider any other syntaxes that you discarded?

André Rømcke  6:26
	Yeah I actually had a previous RFC where I, which is linked to in this one, where I was looking into how it might look like with the new attributes, instead for directly for the read only and immutable keywords for instance. Some might find it strange that I'm using referring to both, keywords, because in some languages they mean one and the same. That first draft explored a bit on the attributes syntax how that could work. I like that a lot actually. But it made me realize that underneath, what we're actually dealing with, is asynchronous visibility. So I don't think necessarily people agree with me but I think from at least in how we expose this in a reflection API. It's much nicer to work with if it's purely about, okay, do I have write access or not, as opposed to having to deal with a checking if there is an attribute called readonly in the reflection cote, which for me, looks like code smell or. It looks like it's solving the wrong problem or, I don't know how to phrase it.

Derick Rethans  7:32
	And I think I agree with you there. I'll always consider attributes as something that modifies the attribute, but visibility would be something that is so inherent of a property that it doesn't really make a lot of sense to do it that way, but that's my feeling about it. I mean, it makes sense to have an attribute for JIT or no JIT because that's some contextual meaning to something else. Similarly, like if you have the ORM attributes or column attributes that people have been suggesting and mainly estate, they convey information about the properties to some other third party tool, not necessarily to PHP itself. Because PHP doesn't care about its JIT or not. It's OPCache cares about it and this would be a similar way right?

André Rømcke  8:13
	So in this case, it might have been better as a keyword, but I was, so I was exploring that as well and that's what's been done in the past in 2012 and then later in 2018 with that immutable RFC, and then again in 2020 with the writeonce RFC which is doing this, different semantics and focusing more on the write once but anyway those three cases were keywords. I would guess that you would prefer rather a keyboard on for this kind of things.

Derick Rethans  8:42
	Oh you have keywords already right, I mean private and public are keywords, or the alternatives that you have with having a modifier on when this keyword exists. I mean, is a similar idea of doing it. I mean, I haven't really thought about which ones I prefer more than the other but I would definitely prefer something like this over an attribute.

André Rømcke  9:00
	Either way, the benefit of doing adding a keyword or attributes for this is that you could, you could also allow them on the class level so you could allow it to be okay, unless anything else is to find then, please let this be the default but all properties be, for instance, immutable, unless they say otherwise. They would need the keyword on for not immutable or something like that. In a immutable case, if you say on the class it's immutable, the whole class should be immutable, and I'm sorry.

Derick Rethans  9:27
	I don't think it makes sense to that subdivided does it?

André Rømcke  9:30
	For me at least that distinction, which could exist between readonly and immutable. Readonly, if you look away from how C sharp is defining it and rather look to Rust. In Rust, readonly is basically the modules can read it, but not write it, your module can write to it. So that's the definition of readonly and I think there are use cases for it. But, Marco is challenging me on that so I need to find.

Derick Rethans  10:00
	It's often good when Marco challenges people on these things because I would trust him with making that kind of semantic choices over many other of the people that are really good working on PHP engine, for example. You slightly touch reflection, that has to be modified to support it, well either keyword or keyword modifier ,or whatever you want to call it in the end. Is there is something else that needs to be changed besides Reflection, or how does reflection, need to be changed?

André Rømcke  10:29
	Reflection needs to one way or the other expose, at least this RFC, expose the fact that you have now disconnected or asynchronous read and write visibility. But other than that, I don't see a big problem for reflection because, at least, in most cases, code tend to use setAccessible(). This will work as before, basically, you will get access in the instance of the reflection property. If there's any code out there that is checking isPrivate(), isProtected(), isPublic(), they might need to check if the read or write access visibility is as they expect. I haven't never had the need to use this myself but there's probably use cases out there for code dealing with unknown objects, for instance.

Derick Rethans  11:20
	And I saw in the RFC that you're suggesting to slightly change what isAccessible() does, but also add new methods to specifically check for the setability and getability. What are the names of these methods that you're wanting to add because I can't quite remember?

André Rømcke  11:37
	So the names of the reflection properties suggested currently in RFC is isSetPrivate(), isSetProtected(), isSetPublic() and then similar for the gets so isGetPrivate(), isGetProtected(), and isGetPublic(). And this would be, in addition to the current isPrivate(), isProtected(), isPublic(), which would be then have to be just a tiny bit to rather affect the visibility of both, so to say. So in the case of public both actually needs to be public for this to return true. Same with protected could also be protected:public or in the case of the other syntax protected, but public set. So, a kind of a write only property, then it would be considered protected, along with the private.

Derick Rethans  12:34
	So this brings me actually to point that, when seeing that your second syntax which has this set in between parenthesis, to getter didn't have anything within parenthesis. Could it make sense to have, get in parenthesis there instead of having it not marked at all?

André Rømcke  12:51
	It could be, but this, this this kind of brings us to maybe our downside with the syntax. And also, one, one point that Larry was making in terms of how this would fit with accessors, because it was basically proposing to reverse it. So it would be set colon private for instance, so that the order would be more in line with what might come with accessors later. So, Nikita basically proposed that we move this to a PHP 8.1 and even though I haven't updated the RFC yet I agree. So, I expect us to do more discussions around the syntax and maybe look at this as Nikita proposed on a higher level. Look at how this different needs can be aligned in a better way.

Derick Rethans  13:37
	You mentioned PHP 8.1, if that's the case, then this is the first RFC where it's discussing for for PHP 8.1, which is kind of nice because of course feature freeze is coming pretty soon now. And that's even closer to when this podcast comes out at the end of July. Is there any potential for breaking backwards compatibility with the addition of the syntax that you're proposing?

André Rømcke  13:58
	There's a few things which I haven't explored yet and defined in RFC and that is with this RFC, the probably needs., well, there definitely needs to be the definition on how property exists and these kind of things should work, and also other methods that check about. There's also isset() and unset(). Needs to be defined how this should work, so I don't know currently about any clear BC breaks per se, currently I'm only aware of it's being the case once people use it. For existing code I haven't seen any BC breaks yet, but for any code that starts to use it. You will need to adapt your, your reflection code potentially, and we'll need to look into how propertyExists(), and isset() and so on will behave.

Derick Rethans  14:41
	What's the feedback been so far?

André Rømcke  14:43
	From others in Symfony slash CMS is part of the realm, it's kind of positive. They would very much like this kind of features to easily more granularly say how this property should be should be accessed. So that's a definite need out there. There are a few language purists maybe, or whatever we should call it, that are clearly against this and allowing this kind of pragmatic access to define whatever you want. While I kind of understand it, and to some extent agree with it, I still think there's usefulness in defining the underlying semantics, on the, on the access to the properties. Yes for immutable, it's not right there yet, you could see the future where we have, if we use the words from from C sharp, it could be public:init for instance, or something like that, whatever the syntax is to define that this is public property, but it's only writable in constructor and potentially destructor and so on. So there's definitely missing a lot here to enable the full immutability semantics. Personally, I think this could be the underlying semantics of immutability, even though that would be that maybe the keyword we expose some promote more to use for for entities and grant uses.

Derick Rethans  16:05
	I don't have to ask you then when you think you'll be putting up this to vote if if you end up retargeting this for PHP 8.1, because you have a full year to go for that.

André Rømcke  16:14
	To be honest, I'm not a C developer. So, even though I think I got some pointers that you for trying to debug the session code in PHP but didn't go beyond that. So, unless anyone volunteers to code the patch for this in the next few weeks I don't see us moving this to that point right now.

Derick Rethans  16:34
	I mean, this could be a good starting point for exploring how to implement all the concepts that you were talking about such as immutability and readonly concepts and things like that. In that that's always we're doing it right and it's not a bad thing that takes a longer time than just the minimum discussion period of two weeks. All these complex interactions are always going to be very difficult to sort out anyway.

André Rømcke  16:56
	On one side of course I would like to have this as soon as possible, but I've been waiting since 2011 so I can wait another year.

Derick Rethans  17:03
	What is a year on a decade. Alright André would you have anything else to add?

André Rømcke  17:08
	No. One thing to add this, I, no matter what we end up with here there is definite need to avoid the boilerplate code, dealing with magic methods and also the performance it of using those. It would be great and simpler to get into the language without this stuff.

Derick Rethans  17:25
	Thank you, André for taking the time this morning to talk to me, as I said, I think this will be a good starting point for a somewhat longer discussion.

André Rømcke  17:32
	Thank you as well this was a great opportunity to meet you again of course, and also great to be on your podcast. It's a nice podcast to follow.

Derick Rethans  17:43
	Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool, you can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.



Show Notes
----------

- RFC: `Property write/set visibility <https://wiki.php.net/rfc/property_write_visibility>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
