PHP Internals News: Episode 39: Stringable Interface
====================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-02-06 09:39 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin039
   :Enclosure: media/pin-039.mp3

In this episode of "PHP Internals News" I chat with
Nicolas Grekas (`Twitter <https://twitter.com/nicolasgrekas>`_,
`GitHub <https://github.com/nicolas-grekas/>`_,
`LinkedIn <https://www.linkedin.com/in/nicolasgrekas/>`_,
`Symfony Connect <https://connect.symfony.com/profile/nicolas-grekas>`_)
about the new "Stringable Interface" that Nicolas is proposing, as well as
about voting rights (on RFCs).

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-039.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:16
	Hi, I'm Derick. And this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. Hello, this is Episode 39. Today I'm talking with Nicholas Grekas about an RFC that he's produced called stringable interface. I already spoke with Nicholas last year about the work that Symfony does the new PHP versions come out to look at deprecations and to make sure that versions of Symfony work with new versions of PHP. But this time Nicholas came up with his own RFC called the stringable interface. Nicholas, could you explain what streamable is?

Nicolas Grekas  0:54
	Hello, and Stringable is an interface that people could use to declare that they implement some the magic toString() method.

Derick Rethans  1:02
	Because currently there's not necessary to implement an interface, and PHP's internals will always use toString if it is available in a class, right?

Nicolas Grekas  1:10
	Yeah, absolutely.

Derick Rethans  1:11
	What is true reason why you would want to have a stringable interface.

Nicolas Grekas  1:16
	So the reason is to be able to benefit from union type in PHP 8. Right now, if you want to accept a string as an argument, it's pretty easy. You just add the string type, right? Let's say now you want to accept a string or a stringable object, stringable an object being something that implements this method. If you want to do that, you can not express the type using types today.

Derick Rethans  1:42
	Because if you choose string, and then the name of an object that would only do that specific object.

Nicolas Grekas  1:47
	Yes, there are some cases in Symfony especially because this is where work and I do open source. Where we do want to not call toString method until the very latest moment. after example is in the code: one is from Drupal. Drupal computes some constraint validation messages, lazyly, and it's pretty important to them because computing the message itself is pretty costly. They don't need to compute it all the time. Actually, we added the type, the string type in Symfony five, before it was released and Drupal came and say: Oh, this is breaking our code and our features, what should we do now? And we removed the type and we replaced it by some annotation saying: Okay, this is a string or a stringable object. So in the future, when will add up PHP 6 would like to be able to express that using a type of real one,

Derick Rethans  2:41
	PHP 6?

Nicolas Grekas  2:42
	No, PHP 8, that's true. Strings and PHP 6.

Derick Rethans  2:49
	Yay.

Nicolas Grekas  2:51
	Another example is also is pretty similar, actually. It's in the symfony auto wiring system. We have services that we wire and sometime we can not; the auto wiring logic is broken doesn't work because some class cannot be at a wet. So in this case, we have a lazy message, because sometime of service while it's not auto wireable, it's going to be removed later on because we removed, Symfony removes, unused services. So instead of computing ahead of time and error message that is heavy to compute, and that we might just trash because the service is going to be removed. We have this lazy thing because yeah, it's heavy to cook with that. So real world use cases.

Derick Rethans  3:32
	I think the intention by by having a stringable interface actually makes sense. What are the concerns for for adding this to your own code, are issues with backwards compatibility, for example?

Nicolas Grekas  3:43
	That's another goal of the RFC. The way I have designed it, is that I think the actual current code should be able to express the type right now, using annotations of course. So what I mean is that the interface, the proposal, the stringable is very easily polyfilled. So we just create this interface into global namespace, the declarative method, and done. So we can do that now. We can improve the typings now, and then in the future, we'll be able to turn that into an actual union type.

Derick Rethans  4:16
	You'd be able to do that almost immediately. Well, you would be able to do that in PHP 8.

Nicolas Grekas  4:21
	Yeah.

Derick Rethans  4:21
	Without it being a problem. And of course, in that case, you can remove to polyfilled stringable interface.

Nicolas Grekas  4:27
	Yeah, absolutely.

Derick Rethans  4:28
	This is going to impact extensions, as well, because extensions, I mean, PHP, internal functions, they often accept strings. I don't actually remember but if you use a scaler type hint string for an internal method than PHP or internal function, this is actually called a toString interface on objects. Like if you would call strlen() on an object that implements toString would actually call toString and return the length to that result.

Nicolas Grekas  4:53
	Yes, absolutely.

Derick Rethans  4:54
	So that wouldn't impact that specific case then.

Nicolas Grekas  4:57
	About extension because that's the current state of the implementation of extension, there was a discussion we're going to talk a bit later about, I think. The current state of the art say is that the interface declares the method that just run right, it declares the written type. It's colon string. So the declaration is public function "toString : string". The very first version didn't have the written type, because it's easier for backward compatibility. Because the current code doesn't need the written type. So by not adding it to the interface, we don't break backward compatibility, which is another critical lighting designer feature that I want at least to have. And so feedback came on the first pull request and said okay, we need the written type. So, the way I implemented that is that now in the RFC actually, the written type is implicit. toString, if you declare it, whether you type ": string" or not, it's there. If you do some reflection later on an instance of something that that then the reflection will tell: Yes, there is a written type and it's string,

Derick Rethans  6:01
	Whether you have defined it or not in your class. So that's a little bit of magic that gets added on.

Nicolas Grekas  6:07
	So it doesn't break any semantics because the written type is already in force: you cannot return anything else than the string right now.

Derick Rethans  6:14
	Yeah, that's true. So that means that automatically toString methods will in return type hints require string to be returned.

Nicolas Grekas  6:21
	Yes.

Derick Rethans  6:22
	And that tweak was necessary to make sure that an older backward compatibility was being broken.

Nicolas Grekas  6:27
	Yes

Derick Rethans  6:28
	Does that also extends to extension that no part that are not part of the PHP core distribution, do they need to be changed as well?

Nicolas Grekas  6:35
	So right now, in the current implementation, yes, they need to be changed. If they declare the toString method, they need to change the type basically, to declare that they return the string explicitly in the C code. So that the current state it's pretty easy on the implementation, implementation side to ask that to the extension authors, right? I think it is doable, but Nikita today posted proposal to improve and go to the next level of the RFC. And the next level would be to have the same magic for the declaration of the interface itself. So it would mean if you declare a toString method, then you implement the stringable interface without having to explicitly declare it in the class.

Derick Rethans  7:22
	I think that actually makes quite a bit of sense because that is pretty much how toString is used already. Anyway, the PHP engine enforces it has to be a string that's being returned.

Nicolas Grekas  7:31
	Yeah, that's very interesting in that would make the type as a typehint much more useful because any pre existing code would just work with the type and pass the type into the written type and so on. So that would be great. So the link with the extension is that maybe we should have the same automatic declaration implicit declaration applied to extensions. So then extension to boodle have to do to do anything and done. That would declare both the written type and the interface.

Derick Rethans  8:03
	That makes sense. You mentioned that Nikita just suggested something to tweak this RFC. I reckon this RFC is still open for discussion and voting hasn't started on it yet.

Nicolas Grekas  8:13
	Yes.

Derick Rethans  8:13
	Do you have any sort of idea for a timeframe where you think this will be finished?

Nicolas Grekas  8:17
	The earliest is on February 6, because we know we need to wait two weeks. So I opened that so we go. I don't know how to write the last part of what we discussed. So Nikita's suggestion. So I'm asking him to some help. As soon as it's ready. I think it can be open for voting. So it can be 10 days. So it didn't trigger much discussions on internals, which I don't know. Maybe it's a very, it's a good point. Or maybe it's like people will vote against without expressing why, I don't know. I hope it's a good thing.

Derick Rethans  8:50
	Sometimes people just start paying attention and there's a new vote.

Nicolas Grekas  8:53
	Yeah.

Derick Rethans  8:54
	So there wasn't a lot of controversy about stringable as you just said, but there was some controversy about you actually apply for voting rights, I remember what happened there?

Nicolas Grekas  9:03
	So yes, I applied for voting. Because of my implication, I think I'm an active PHP contributor to internals in not on not on the C-side, but Okay, so since I wanted to open this RFC, I said: Okay, now it's time to do the bureaucratic steps to get a vote, right?

Derick Rethans  9:23
	Yep.

Nicolas Grekas  9:24
	And I think I'm the first person to actually get through some process for getting votes in itself. I mean, I think most people or maybe all people that have a vote, a vote as a side effect of of something else.

Derick Rethans  9:38
	Yeah, usually about contributing patches, either PHP itself, documentation or extensions.

Nicolas Grekas  9:43
	So I think there's there has been some confusion, but it's been sorted out pretty quickly. I think I'm going to be able to vote on the next RFC. I'll report back if I can.

Derick Rethans  9:54
	Okay, fair enough. Currently, we don't really have a process for this at all. I mean, you get to vote when you have a GIT account. Pretty much, or a PHP commit access in some form. And I don't think we've ever really thought about handing that out to people that have been contributing a lot. Right. So that's kind of an interesting thing to see. What we have seen in the past, is people wanting just saying: Yeah, I'd like to vote, or in other cases, or yeah can I have a php.net email address, right. So that also happened because that is a side effect of getting commit access.

Nicolas Grekas  10:23
	Okay.

Derick Rethans  10:24
	At the moment I what happened when you did it, it got immediately shut down. Probably a bit quicker than was nice without any discussion. But I think in the future, we do need to come with, come up with a plan and perhaps even think about how to approach voting for features for RFCs the first place because we don't really have a set guideline on who gets to do this and who doesn't get to do it and stuff like that.

Nicolas Grekas  10:49
	Yeah, it's pretty interesting. Nikita just after the or during the discussion at, he posted some stats on the number of people who can vote and I think the number is like 1900

Derick Rethans  10:59
	Yeah. There's quite a lot here.

Nicolas Grekas  11:01
	It's bit strange. And most people don't vote, I think, because they think they shouldn't. I don't know, something like that. But it's true. It's pretty strange. What I like about this situation is that it doesn't draw a strong line between people that contribute C code and people that write PHP code. And it's nice for PHP. I really think it's nice for PHP to have people that vote that don't do C code. But I think, of course, people that do C code must have the strongest voice, because at some point, the implementation decides.

Derick Rethans  11:35
	Well, that is a different right, the votes are usually on the idea, not on the implementation. But sometimes the implementation is so complicated that it's nearly impossible to implement, like, I've very briefly spoken with Nikita about generics. I'm sure we'll talk about that at some point, where I'm pretty sure that generics is an idea that simple, I mean, people will vote for it, but as an implementation it might not be that simple to do.

Nicolas Grekas  12:01
	Yeah.

Derick Rethans  12:02
	So what happens if you vote for the feature, but you can't come up with a good implementation?

Nicolas Grekas  12:06
	So I'm inside of thinking that people should vote on the implementations. I mean, people shouldn't be able to vote only on an idea. If there is an idea, it will be supported by an implementation that proves that we are talking about something real, no, just a fancy idea that might not work in black. So that's my opinion.

Derick Rethans  12:24
	That's a good point. But as you said, from the 1900 people, or or 1900 people plus, that's controlled, most of them are not familiar with a PHP internals whatsoever, because they tend to be contributions to the documentation. This is also very valuable, but it doesn't mean you know, and you don't necessarily know PHP internals,

Nicolas Grekas  12:40
	Yeah, sure.

Derick Rethans  12:41
	The oher way can be true as well right? You might know a lot about PHPs internals, but never really use PHP in real life, in your job, or anything like that.

Nicolas Grekas  12:48
	So it's also good to be able to team up with someone that knows how to code the C part, the internal part. So you have the idea you're you're the supporter part of the team and then someone - being able convince someone to do the implementation or to help you do it, is also proof of kind of interest. So starting small and bringing more people in the boat and making it happen as a thought.

Derick Rethans  13:12
	Yeah, and we saw some of that happening last year. I can't quite remember what feature it was or or exactly what it was. But I agree with you. I think that is important to do that you can at least somebody convinced to implement the feature before just voting on the idea. Thank you for taking the time with me this morning, Nicholas.

Nicolas Grekas  13:30
	And thank you Derick for having me again.

Derick Rethans  13:32
	It it continues like this I'm sure we'll speak again at some point in the future.

Nicolas Grekas  13:35
	Okay.

Derick Rethans  13:39
	Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening and I'll see you next week.


Show Notes
----------

- RFC: `Stringable Interface <https://wiki.php.net/rfc/stringable>`_


Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
