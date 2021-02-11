PHP Internals News: Episode 76: Deprecate null, and Array Unpacking
===================================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-02-18 09:04 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin076
   :Enclosure: media/pin-076.mp3

In this episode of "PHP Internals News" I chat with Nikita Popov (`Twitter
<https://twitter.com/nikita_ppv>`_, `GitHub <https://github.com/nikic/>`_,
`Website <https://nikic.github.io/>`_) about two RFCs: Deprecate passing null
to non-nullable arguments of internal functions, and Array Unpacking with
String Keys.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-076.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:14  
	Hi I'm Derick. Welcome to PHP internals news, a podcast dedicated to explain the latest developments in the PHP language. This is Episode 76. In this episode, I'm talking with Nikita Popov about a few more RFCs that he has been working on over the past few months. Nikita, would you please introduce yourself.

Nikita Popov  0:34  
	Hi, I'm Nikita. I work on PHP core development on behalf of JetBrains.

Derick Rethans  0:39  
	In the last few PHP releases PHP is handling of types with regards to internal functions and user land functions, has been getting closer and closer, especially with types now. But there's still one case where type mismatches behave differently between internal and user land functions. What is this outstanding difference?

Nikita Popov  0:59  
	Since PHP 8.0 on the remaining difference is the handling of now. So PHP 7.0 introduced scalar types for user functions. But scalar types already existed for internal functions at that time. Unfortunately, or maybe like pragmatically, we ended up with slightly different behaviour in both cases. The difference is that user functions, don't accept null, unless you explicitly allow it using nullable type or using a null default value. So this is the case for all user types, regardless of where or how they occur as parameter types, return values, property types, and independent if it's an array type or integer type. For internal functions, there is this one exception where if you have a scalar type like Boolean, integer, float, or a string, and you're not using strict types, then these arguments also accept null values silently right now. So if you have a string argument and you pass null to it, then it will simply be converted into an empty string, or for integers into zero value. At least I assume that the reason why we're here is that the internal function behaviour existed for a long time, and the use of that behaviour was chosen to be consistent with the general behaviour of other types at the time. If you have an array type, it also doesn't accept now and just convert it to an empty array or something silly like that. So now we are left with this inconsistency.

Derick Rethans  2:31  
	Is it also not possible for extensions to check whether null was passed, and then do a different behaviour like picking a default value?

Nikita Popov  2:40  
	That's right, but that's a different case. The one I'm talking about is where you have a type like string, while the one you have in mind is where you effectively have a type like string or null. 

Derick Rethans  2:51  
	Okay. 

Nikita Popov  2:52  
	In that case, of course, accepting null is perfectly fine.

Derick Rethans  2:56  
	Even though it might actually end up being different defaults.

Nikita Popov  3:01  
	Yeah. Nowadays we would prefer to instead, actually specify a default value. Instead of using null, but using mull as a default and then assigning something else is also fine. 

Derick Rethans  3:13  
	What are you proposing to change here, or what are you trying to propose to change that into?

Nikita Popov  3:18  
	To make the behaviour of user land and internal functions match, which means that internal functions will no longer accept null for scalar arguments. For now it's just a deprecation in PHP 8.1, and then of course later on that's going to become a type error.

Derick Rethans  3:35  
	Have you checked, how many open source projects are going to have an issue with this?

Nikita Popov  3:40  
	No, I haven't. Because it's not really possible to determine this using static analysis, or at least not robustly because usually null will be a runtime value. No one does this like intentionally calling strlen with a null argument, so it's like hard to detect this just through code analysis. I do think that this is actually a fairly high impact change. I remember that when PHP 7.2, I think, introduced to a warning for passing null to count(). That actually affected quite a bit of code, including things like Laravel for example. I do expect that similar things could happen here again so against have like strlen of null is pretty similar to count of null, but yeah that's why it's deprecation for now. So, it should be easy to at least see all the cases where it occurs and find out what should be fixed.

Derick Rethans  4:35  
	What is the time frame of actually making this a type error?

Nikita Popov  4:38  
	Unless it turns out that this has a larger impact than expected. Just going to be the next major version as usual so PHP 9. 

Derick Rethans  4:45  
	Which we expect to be about five years from now. 

Nikita Popov  4:49  
	Something like that, at least if we follow the usual cycle.

Derick Rethans  4:52  
	Yes. Are there any other concerns for this one?

Nikita Popov  4:55  
	No, not really.

Derick Rethans  4:57  
	Maybe people don't realize it.

Nikita Popov  4:58  
	Yeah, possibly. You can't predict these things, I mean like, this is going to have like way more practical impact for legacy code than the damn short tags. But for short tags, we get 200 mails and here we get not a lot. 

Derick Rethans  5:14  
	I think this low impact WordPress a lot.

Nikita Popov  5:17  
	Possibly but at least the thing they've been complaining about is that something throws error without deprecation, and now they're getting the deprecation so everyone should be happy.

Derick Rethans  5:28  
	Which is to be fair I think is a valid concern.

Nikita Popov  5:30  
	Yes, it is. I've actually been thinking if we should like backport some deprecations to PHP 7.4 under an INI flag. Not like my favourite thing to work on, but people did complain?

Derick Rethans  5:47  
	Which ones would you put in there?

Nikita Popov  5:48  
	I think generally some cases where things went from no diagnostics to error. I think something that's mentioned this vprintf and round, and possibly the changes to comparison semantics. I did have a patch that like throws a  deprecation warning, when that changes and that sort of something that could be included.

Derick Rethans  6:12  
	I would say that if we were in January 2020 here, when these things popped up, then probably would have made sense to add these warnings and deprecations behind the flag for PHP seven four, but because we've now have done 15 releases of it, I'm not sure how useful this is now to do.

Nikita Popov  6:30  
	I guess people are going to be upgrading for a long time still. I don't know I actually not sure about how, like distros, for example Ubuntu LTS update PHP seven four. If they actually follow the patch releases, because if they don't, then this is just going to be useless.

Derick Rethans  6:48  
	Oh there's that. Yeah. 

Derick Rethans  6:50  
	There is one more RFC that I would like to talk to you about, which is the array unpacking with string keys RFC. That's quite a mouthful. What does the background story here?

Nikita Popov  7:00  
	The background is that we have unpacking in calls. If you have the arguments for the call in an array, then you write the three dots, and the array is unpacked into actual arguments.

Derick Rethans  7:14  
	I'd love to call it the splat operator.

Nikita Popov  7:16  
	Yes, it is also lovingly called the splat operator. And I think it has a couple more names. So then, PHP 7.4 added the same support in arrays, in which case it means that you effectively merge, one array to the other one. Both call unpacking and array unpacking, at the time, we're limited to only integer keys, because in that case, are the semantics are fairly clear. We just ignore the keys, and we treat the values as a list. Now with PHP 8.0 for calls, we also support string keys and the meaning there is that the string keys are treated as parameter names. That's how you can like do a dynamic named parameter call. Actually, this probably was one of the larger backwards compatibility breaks in PHP eight. Not for unpacking but for call_user_func_arg, where people expected the keys to be thrown away, and suddenly they had a meaning, but that's just a side note. 

Derick Rethans  8:21  
	It broke some of my code. 

Nikita Popov  8:23  
	Now what this RFC is about is to do same change for array unpacking. So allow you to also use string keys. This is where originally, there was a bit of disagreement about semantics, because there are multiple ways in which you can merge arrays in PHP, because PHP has this weird hybrid structure where arrays are a mix between dictionaries and lists, and you're never quite sure how you should interpret them.

Derick Rethans  8:54  
	It's a difference between array_merge and plus, but which way around, I can ever remember either.

Nikita Popov  9:00  
	What array_merge does is for integer keys, it ignores the keys and just appends the elements and for string keys, it overwrites the string keys. So if you have the same string key one time earlier and again later than it takes the later one. Plus always preserves keys, before integer keys. It doesn't just ignore them, but also uses overriding semantics. The same is the other way round. If you have something in the first array, a key in the first array and the key in the second array, then we take the one from the first array, which I personally find fairly confusing and unintuitive, so for example the common use case for using plus is having an array with some defaults, in which case you have to, like, add or plus the default as the second operand, otherwise you're going to overwrite keys that are set with the defaults which you don't want. I don't know why PHP chose this order, probably there is some kind of idea behind it.

Derick Rethans  10:01  
	It's behaviour that's been there for 20 plus years that might just have organically grown into what it is.

Nikita Popov  10:07  
	I would hope that 20 years ago at least someone thought about this. But okay, it is what it is. So ultimately choice for the unpacking with string keys is between using the array_merge behaviour, the behaviour of the plus operator, and the third option is to just always ignore the keys and always just append the values. And some people actually argue that we should do the last one, because we already have array_merge and plus for the other behaviours. So this one should implement the one behaviour that we don't support yet.

Derick Rethans  10:40  
	But that would mean throwing away keys.

Nikita Popov  10:43  
	Yes. Just like we already throw away integer keys, so it's like not completely out there. So yeah, that is not the popular option, I mean if you want to throw away keys can just call array_values and go that way. So in the end, the semantics it uses is array_merge

Derick Rethans  10:58  
	The array_merge semantics are..

Nikita Popov  11:01  
	append, like ignore integer keys just append, and for string keys, use the last occurrence of the key.

Derick Rethans  11:07  
	So it overwrites.

Nikita Popov  11:08  
	It overwrites, exactly. Which is actually also the semantics you get if you just write out an array literal where the same key occurs multiple times. Unpacking is like kind of a programmatic way to write a function call or an array literal, so it makes sense that the semantics are consistent.

Derick Rethans  11:26  
	I think I agree with that actually, yes. Are there any changes that could break existing code here?

Nikita Popov  11:32  
	Not really because right now we're throwing an exception if you have string keys in array unpacking. So it could only break if you're like explicitly catching that exception and doing something with it, which is not something where we provide any guarantees I think. So generally I think that, removing an exception doesn't count as a backwards compatibility break.

Derick Rethans  11:55  
	I think that's right. Do you have anything else to add here?

Nikita Popov  11:59  
	No, I think that's a simple proposal.

Derick Rethans  12:02  
	Thank you, Nikita for taking the time to explain these several RFCs to me today.

Nikita Popov  12:07  
	Thanks for having me Derick.

Derick Rethans  12:11  
	Thank you for listening to this instalment of PHP internals news, a podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next time.


Show Notes
----------

- RFC: `Array unpacking with string keys <https://wiki.php.net/rfc/array_unpacking_string_keys>`_
- RFC: `Deprecate passing null to non-nullable arguments of internal functions <https://wiki.php.net/rfc/deprecate_null_to_scalar_internal_arg>`_


Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
