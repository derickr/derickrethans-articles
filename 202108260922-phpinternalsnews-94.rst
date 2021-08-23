PHP Internals News: Episode 94: Unwrap Reference After Foreach
==============================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-08-26 09:22 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin094
   :Enclosure: media/pin-094.mp3

In this episode of "PHP Internals News" I chat with Nikita Popov (`Twitter
<https://twitter.com/nikita_ppv>`_, `GitHub <https://github.com/nikic/>`_,
`Website <https://nikic.github.io/>`_) about the "First Class Callable Syntax"
RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-094.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:14  
	Hi, I'm Derick. Welcome to PHP internals news, the podcast dedicated to explaining the latest developments in the PHP language. This is Episode 94. Today I'm talking with Nikita Popov about the unwrap reference after foreach RFC that he's proposing. Nikita, would you please introduce yourself?

Nikita Popov  0:33  
	Hi, Derick. I'm Nikita and I work at JetBrains on PHP core development. 

Derick Rethans  0:38  
	So no changes compared to the last time.

Nikita Popov  0:41  
	Yes, at the time before that.

Derick Rethans  0:43  
	So what is the problem that is RFC is going to solve?

Nikita Popov  0:46  
	Well, it's really a very minor thing. I think it's a relatively well known problem for the more experienced PHP programmers. It's like a classic example, you have a foreach loop by reference. So foreach array as value by reference, and then you do a second loop after that, foreach array as value at the same it's by value. So without the reference sign. The result of that is that your last two array elements are going to be the same, which is kind of unexpected. If you're not familiar with how references in PHP work and scoping in PHP works. So I think it's worth explaining what's going on there.

Derick Rethans  1:27  
	Can you quickly explain the scoping or rather the lack of it, I suppose?

Nikita Popov  1:31  
	Yeah, it's really the lack of PHP really only has function scoping. So if you have a foreach array as value, then the value variable is going to stay alive, even after the foreach loop. And usually, that won't make much of a difference. So you will just have like reference to the last element of the array, might even be useful for some cases, you know, before we added the array, I think, array_key_last function. If the last element now is a reference, so if you have a reference to the last element, then you're write into that variable is also going to modify the last element of the array. So if you now have a second foreach loop, using the same variable, that's actually not just modifying that variable, but it's also always modifying the last element of the array.

Derick Rethans  2:15  
	Okay, just to clarify, it isn't necessarily the last element in the foreach loop. It's the last one that's been assigned to?

Nikita Popov  2:22  
	Yeah, that's, that's true.

Derick Rethans  2:24  
	Is this not something that people actually use for some useful reasons?

Nikita Popov  2:28  
	As mentioned before, technically, you could use it to get a reference to the last element and then modify the last element outside the foreach loop. I don't think this is a particularly common use case. But I'm sure people have used in here there. This is a use case we would break with the proposed RFC.

Derick Rethans  2:47  
	I think it is one I have used in the past, it's probably not how I would do it now. But I'm pretty sure I have some point in the past. What are you proposing to change with this RFC?

Nikita Popov  2:57  
	The change is pretty simple. And that's to unwrap or to break the reference after the loop. You will still have like after the loop, the variable will still contain the value of the last element, or of the last like visited element, but it will no longer be a reference to it. If you write into the variable, it will not modify the original array. And if you have a second loop that writes into the variable that also doesn't modify the original error any more.

Derick Rethans  3:25  
	At which point and how is this reference broken?

Nikita Popov  3:29  
	It's at the end of the foreach loop, or as you say, if you break out too early, then of course, it would also get broken. So it's referenced inside the foreach loop and stops being referenced outside the loop.

Derick Rethans  3:41  
	And that would happen also, if I would use a goto for example?

Nikita Popov  3:45  
	Oh, that that's a trick question, actually, yes, it should happen. But now that you have mentioned it, I think my current implementation does not handle that particular case, I will have to double check it. But that should happen, yes.

Derick Rethans  4:00  
	It's good to know that you've thought about it then.

Nikita Popov  4:02  
	Well, I didn't think about it. Because I mean, I guess I can mention it here, the way this works is that well, at the end of the foreach loop, we have like an instruction that frees the loop variable. And I can just add an additional one that breaks reference. But if you use things like goto or multi level breaks, or something like that, then we insert these clean-up instructions before the jump. We have to make sure to actually insert the reference breaking instruction there as well. So it's like not automatically handled.

Derick Rethans  4:38  
	Is this going to be a separate instruction or as we tend to call them opcodes?

Nikita Popov  4:43  
	I'm using a separate one, but one could run it as a flag into the instruction that frees the loop variable, but I think it's cleaner to have a separate instruction for it. Like technically one could optimize it away in some cases, like I wouldn't bother but it's like semantically a  different thing.

Derick Rethans  5:01  
	I think it'd be nicer result, because it makes it easier to visualize what's happening, right? 

Nikita Popov  5:06  
	Yeah, it is.

Derick Rethans  5:07  
	Did you actually check whether some code uses this construct?

Nikita Popov  5:10  
	I have to admit, I tried checking it using a very basic approach, just look at foreach loops by reference. And then if the variable is used after that. But that kind of primitive approach has way too many false positives, for example, you have a foreach loop inside, and if, and then the variable is reused inside an else. So it like wouldn't flow from the if into the else. So you would have to do some kind of more sophisticated control flow analysis. It's something that can be done, but I didn't bother doing it for a one off backwards compatibility check. So I don't have any hard data on how much code is actually using something like this. 

Derick Rethans  5:51  
	So this is where I'm a little bit on the fence about this change, because it is changing behaviour, that's going to be pretty hard to figure out what is actually going to affect your codebase.

Nikita Popov  6:01  
	It should be possible to very reliably detect that. It's just something you have to actually implement. But you're right now there is no easy way to check that. 

Derick Rethans  6:13  
	It's something that static analysers could probably have a look at.

Nikita Popov  6:16  
	Yeah, expect that maybe Psalm or PHPStan, something like that will be easier to implement, because they already have control flow information.

Derick Rethans  6:23  
	You don't really know how impactful this, which is, in my opinion, a bit of the scary bit. How important do you think you'll find it to have this RFC going through and implemented?

Nikita Popov  6:33  
	I don't think it's super important. It's mostly like, small quality of life fix for newer developers . People who have already encountered this issue once won't forget about it again. In fact, it's somewhat common recommendation that you should always unset the loop variable after a foreach by reference loop. So I've seen that as like a policy some people use, that could be avoided. So yeah, I don't think it's a critical feature, just a small improvement.

Derick Rethans  7:08  
	Would it be an alternative idea to instead deprecate the foreach by reference?

Nikita Popov  7:14  
	Okay, that's the radical approach. Everything is possible. I think that foreach by reference is relatively, I mean, I think it's one of the most common uses of references we have, and one of the most reasonable ones. I mean, the alternative is search into by value loop, and then you modify it by looking up the element by key again, which is a bit more ugly, I would say. I think we shouldn't deprecate foreach by reference, though it would be kind of nice to have a different way to achieve the same. One other unfortunate thing about foreach by reference is that it leaves behind references in the array. The case I'm looking at here is this reference to the last element, where you have like reference structure that's pointed to both from inside the array, and from this loop variable. The other thing that foreach by reference does is that for all the other array elements, you will actually leave behind the reference wrapper that's just used in this one single place for this single array element. Essentially, you are wasting memory, because we will leave behind this that reference wrapper. So after you do the foreach by reference loop over the array, the array will actually grow larger. So if you're storing like integers, and it may grow significantly larger, like from a technical perspective, foreach by references, also not great. But like from a usability perspective, it's nicer then modifying values by key lookup.

Derick Rethans  8:53  
	I guess it's going to depend on how big the array is, right? I mean, if it's a few elements, it probably doesn't matter.

Nikita Popov  8:58  
	But if you have like a 100,000 element array, then you paying for 100,000 reference wrappers that you don't need afterwards any more.

Derick Rethans  9:07  
	In that case, it's rather better to just modify it through the key that you obtained by doing foreach key as value.

Nikita Popov  9:14  
	Right. But it's also worth noting that foreach reference actually has different semantics then foreach value, because foreach by value works on the copy of the array. Like it's not an actual copy just like semantically. If you modify the array inside the foreach by value loop, then we will copy the array. Doing the modification with a separate key lookup and foreach by value loop will actually copy the array at that point, while foreach by reference takes account modifications of the array. So even if you like add or remove elements in the array in the foreach by reference loop, it will try on the like best effort basis to still iterate on in a reasonable way on the modified array. It's like not a straightforward replacement.

Derick Rethans  10:00  
	It all depends on what people intended to do with it. Right? Do you think there are any further situations that are a bit strange? That could benefit from having some subtle changes to the language semantics?

Nikita Popov  10:13  
	Nothing can who comes to mind immediately.

Derick Rethans  10:16  
	Yeah, I can't think of any either. But I thought maybe maybe have something in the pipeline. Would you have anything else to add to this RFC?

Nikita Popov  10:23  
	Well, one more thing that's discussed in the RFC is the case of complex variables. A little known fact, in the foreach loop, you don't have to assign to a simple variable, you can also assign to something like an object property, or an object property on the result of a function call that that means that in the loop, this function is getting called on every iteration, and then you assign it to a property on the result. So you can do that kind of weird stuff, we allow it.

Derick Rethans  10:52  
	And does it the work without any weird side effects?

Nikita Popov  10:56  
	Depends on what you consider weird, but basically does what you expect as if you had written an explicit assignment to the complex variable.

Derick Rethans  11:04  
	I reckon that's how it's instructed out in the oparray then as well.

Nikita Popov  11:07  
	Yeah, exactly. As far as this RFC is concerned, the problem there is that to unwrap the reference of the loop, we actually have to evaluate the variable again. And if it's a complex variable that might have side effects, for example, the function call. And that's why the RFC says that if the variable is complex, we are not going to do that, like that's probably going to be more unexpected than leaving a reference wrapper around. So we have this extra weird edge case. In the internals discussion, some people already suggested that maybe we should just deprecate support for these kind of complex assignments. One could also mention that an alternative that has been suggested is to actually make the loop variable, scoped to the foreach loop. So we could unset it entirely after the loop, rather than just breaking the reference, which is, of course, a larger change, larger backwards compatibility break. It also doesn't really align with PHP semantics of only having function scope and not block scope.

Derick Rethans  12:06  
	I probably agree without, it's too much of a change to do that. Because then you sort of expect that all the language constructs should have a scope. I mean, it needs to be either one or the other.

Nikita Popov  12:15  
	Yeah, I mean, other languages like JavaScript have solved that by introducing a separate way to declare scoped variables. So that will be "let", just changing the behaviour in one place is probably not a good idea.

Derick Rethans  12:30  
	I probably agree with you though. It was a bit of a shorter RFC this time. That's okay with me.

Nikita Popov  12:35  
	Yes, I used that as an excuse to discuss some foreach behaviour details.

Derick Rethans  12:40  
	Fair enough. Thank you for taking the time this morning to come and talk to me about the references after foreach RFC.

Nikita Popov  12:47  
	Thanks for having me, Derick, once again.

Derick Rethans  12:53  
	Thank you for listening to this installment of PHP internals news, a podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening. I'll see you next time.



Show Notes
----------

- RFC: `Unwrap Reference After Foreach <https://wiki.php.net/rfc/foreach_unwrap_ref>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
