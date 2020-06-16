PHP Internals News: Episode 59: Named Arguments
===============================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-06-25 09:22 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin059
   :Enclosure: media/pin-059.mp3

In this episode of "PHP Internals News" I chat with Nikita Popov (`Twitter
<https://twitter.com/nikita_ppv>`_, `GitHub <https://github.com/nikic/>`_,
`Website <https://nikic.github.io/>`_) about his Named Parameter RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-059.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:18  
	Hi, I'm Derick, and this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 59. Today I'm talking with Nikita Popov about a few RFCs that he's produced. Hello Nikita, how are you this morning?

Nikita Popov  0:35  
	Hey Derick, I'm great. How are you? 

Derick Rethans  0:38  
	Not too bad, not too bad today. I think I made a decision to stop asking you to introduce yourself because we've done this so many times now. We have quite a few things to go through today. So let's start with the bigger one, which is the named arguments RFC. We have in PHP eight already seen quite a few changes to how PHP deals with set up and things like that we have had an argument promotion in constructors, we have the mixed type, we have union types, and now named arguments, I suppose built on top of that, again, so what are named arguments? 

Nikita Popov  1:07  
	Currently, if you're calling a function or a method you have to pass the arguments in a certain order. So in the same order in which they were declared in the function, or method declaration. And what named arguments or parameters allows you to do is to instead specify the argument names, when doing the call. Just taking the first example from the RFC, we have the array_fill function, and the array_fill function accepts three arguments. So you can call like array_fill( 0, 100, 50 ). Now, like what what does that actually mean? This function signature is not really great because you can't really tell what the meaning of this parameter is and, in which order you should be passing them. So with named parameters, the same call would be is something like: array_fill, where the start index is zero, the number is 100, and the value is 50. And that should immediately make this call, like much more understandable, because you know what the arguments mean. And this is really one of the main like motivations or benefits of having named parameters.

Derick Rethans  2:20  
	Of course developers that use an IDE already have this information available through an IDE. But of course named arguments will also start working for people that don't have, or don't want to use an IDE at that moment. 

Nikita Popov  2:31  
	At least in PhpStorm, there is a feature where you can enable these argument labels for constants typically only.  This would basically move this particular information into the language, but I should say that of course this is not the only advantage of having named parameters. So making code more self documenting is one aspect, but there are a couple couple more of them. I think one important one is that you can skip default values. So if you have a function that has many optional arguments, and you only want to say change the last one, then right now you actually have to pass all the arguments before the last one as well and you have to know: Well, what is the correct default value to pass there, even though you don't really care about it.

Derick Rethans  3:19  
	If I remember correctly, there are a few functions in PHP's standard library, where you cannot actually replicate the default value with specifying an argument value, because they have this really complex and weird kind of behaviour.

Nikita Popov  3:33  
	That's true, but that's something we're trying to eliminate in PHP eight mostly.

Derick Rethans  3:39  
	And of course additional you'd never have to remember, whether in_array and array_search have needle or haystack first, which is also beneficial. 

Nikita Popov  3:46  
	That's true. Yeah.

Derick Rethans  3:48  
	You mentioned that there are a few other benefits as well. You mentioned self documenting and the skipping of arguments, what other benefits are there?

Nikita Popov  3:54  
	The other part is that you can also reorder the parameters. So this varies a little bit by language. In some languages you're required to still pass the arguments in the same order. They were declared, even if you're using name parameters. But for the purposes of PHP, you would allow passing them in arbitrary order. Just like you said you don't have to remember if the haystack is first, or the needle comes first. And I think one case where all of these benefits, play together particularly well, is when it comes to object construction. So you already mentioned that we have the constructor promotion RFC in PHP eight, which makes it pretty simple to declare value objects. So you just list all the available properties and their default values and types, the constructor and you're done. But when you actually instantiate the object, you still have to, their ergonomics are not particularly good, because you have to remember in which order you have to pass the parameters, don't really know which parameters which just looking at the call. And once again, you have to specify everything and you can't just skip a few of them with default values. And if you have like a constructor with maybe five or six arguments coming in, which is maybe unusual for normal methods, but I think somewhat normal for constructors in particular, then the current development experience there is just not very nice. And named parameters would essentially provide us something akin to an object initialization syntax which is available in many other languages, and which has also been proposed for PHP, previously. But you would get this just as a side effect of combining constructors and named parameters, without having to define any kind of special semantics for how object construction works, and how initializer syntax interacts with constructors and so on.

Derick Rethans  5:55  
	That ties in again with the object ergonomics that I spoke about with Larry earlier this season as well.

Nikita Popov  6:01  
	Yeah, I believe that this combination of ,constructor promotion and named parameters for constructors was one of the things.

Derick Rethans  6:10  
	We've spoken a little bit about what it is. Now, how would you use this in PHP, what is the syntax for that you're proposing?

Nikita Popov  6:18  
	I mean syntax is always bike shedding question. The particular one, I am proposing for now is to save the parameter name as literal, so no dollar in front of it or something. And the colon and the value you want to pass. 

Derick Rethans  6:35  
	Is there any precedence for this syntax already, either in PHP or outside of PHP? 

Nikita Popov  6:41  
	In PHP, not really. I mean, PHP, we usually use the double arrow to have any kind of key value mapping. This is sort of key value mapping. In other languages, yes the syntax does exist. I'm actually not sure which languages exactly use it. Probably C sharp and Kotlin. Python uses just an equal sign. Well, there are a couple who use it. I actually initially use the double arrow syntax because it's more familiar with PHP, but I found that it's, there's not really read as nicely. And I also have some ideas on how we can, like, integrate this colon syntax, into the language in a more consistent way.

Derick Rethans  7:27  
	I think I saw in the RFC that the only said the only way how you can do the keys is by literal and not by a variable.

Nikita Popov  7:34  
	That's right. This is mainly just to avoid confusion. Well if you allow specifying a variable, then the question is, well, is this variable just the parameter name? Because I mean the signature, you also write this as a variable, or is it the variable that contains the parameter name like variable variables in PHP. So I think to sidestep that confusion, we just allow identifiers, but you can still use a variable parameter names from the argument unpacking syntax. 

Derick Rethans  8:04  
	How does that work? 

Nikita Popov  8:05  
	So PHP supports the three dots, the ellipsis operator, both in the function declaration, and for function calls. The declaration that just means collect all the trailing arguments. And the call, at the call, means that you get an array, and the elements of this array should be interpreted as function arguments. And parameters extend that by also allowing array keys. And if you unpack an array with string keys then those will be interpreted as parameter names, and we'll use the usual named parameters passing semantics.

Derick Rethans  8:47  
	Interesting. I actually missed that, while reading the RFC. To be fair, I skimmed it, not really read tit. Yeah it's good to see that actually. Now people currently use positional arguments and not named arguments. How would these two interact.

Nikita Popov  9:01  
	Mostly, the named parameters are just syntax for positional arguments, so we perform an internal transformation to convert named parameters into positional parameters. As far as both the engine is concerned and the callee is concerned. They don't really know about parameters that's all. They see usual positional call where all the missing arguments have been filled in with default values. I think the only part to watch out for there is exactly this case of variadics, because previously, the variadic parameter could only contain a list of arguments, and now it can also have string keys, or like left over named parameters. So which did not have a matching argument in the function signature so both will now get collected to the variadic parameter. Think that's like the only case where I know that the calling convention really changes for the recipient of the arguments.

Derick Rethans  10:02  
	Because otherwise got a normal array they now get a bunch of things with potentially having keys in there as well. What would happen if I specify a named argument by name and also include it into the variadics?

Nikita Popov  10:15  
	So generally the rule is always you can pass a parameter at most once you can have the situation where you first pass some positional arguments, and then you pass named arguments. If you do that this named argument cannot clash with the previous past positional argument, if you run in this kind of situation we will always throw an exception at that point. So you're not allowed to overwrite the previous argument, or something like that.

Derick Rethans  10:42  
	Same would work that if a method would collect named arguments and also have the variadics array. In case you specify more arguments then the function would take. And, in the variadics you'd have that name again that would have already clashed before it even gets turned into variadic. Are the names that she gives to named arguments are case sensitive or case insensitive?

Nikita Popov  11:04  
	They are case sensitive. Because the parameters you specify in the function are just variables and variables in PHP are case sensitive as well.

Derick Rethans  11:14  
	At the moment if you inherit a method in a inheriting class, then it doesn't particularly matter what the names of these method arguments are. When you get now named arguments, is this going to change, because at the moment PHP doesn't enforce that the names of inheriting methods are of course clashing, or the same as the ones that are overriding in the parent class?

Nikita Popov  11:37  
	This is one of the bigger open questions we have. The problem is that if you call a method with the names from the parent class, and the child class change them, then you'll get an error because this named parameter just doesn't exist in the child class. And there are a couple of ways to approach that one is to forbid during inheritance, any kind of parameter name changes, which would be a fairly significant backwards break because well, it never mattered in the past and based on some cursory analysis, this is like parameter name changes, somewhat common in code right now. The other possibility is to just ignore this issue, expect that a lot of code is never going to use name parameters. So using the parameters only makes sense with some types of methods. If you have a method that only accepts one argument can be pretty sure that no one's going to call it that has a name parameter, and there is the option of just ignoring this issue and fixing it as it comes up, more or less. Which is maybe not the most principled approach. But if we look at other languages that do make heavy use of parameters for example like Python. And we see that they also just ignore the problem. So it looks like in practice this does work out. Of course, a significant difference there is that Python has had in parameters for a long time already. We will be retrofitting them on an old language. So the situation is somewhat different and probably rather than more dangerous for us. 

Derick Rethans  13:14  
	This is something of course that static analysis tools can check for quite easily and I would argue that they probably should start doing that as well.

Nikita Popov  13:22  
	This this right, so this is both something easy to check for, and also easy to automatically fix.

Derick Rethans  13:28  
	Except that you need to choose which one is the correct name, of course. 

Nikita Popov  13:32  
	Yeah, that's right.

	But there is one more possibility, which is to allow the parameter names from both the parent method, and the child method. This will be like more or less a transparent way to fix that issue. The only problem you can run into this if both the parent method and the child method use the same parameter name but in a different position. If we would go with this option then we say that only in this particular case where parameter name is reused but different position that would become an inheritance error.

Derick Rethans  14:04  
	I quite like that actually, because that's a pragmatic approach isn't it?

Nikita Popov  14:07  
	I also quite like it, maybe it's just technically a bit problematic.

Derick Rethans  14:11  
	I can already imagine that if this gets accepted for PHP eight, which of course not sure at the moment, that Xdebug is going to have to show the variadics already with the names array elements which of course it doesn't do yet because it has no notion of. But that's good to know to have a heads up on these things. 

	PHP eight has already seen quite a lot of work for internal methods to get their names properly, recorded as well, so that types of stubs that you have already been working on. How does named arguments tie in with this?

Nikita Popov  14:38  
	The actual named arguments proposal is already pretty old. It dates back to PHP 5.6, I think, and one of the open questions since then was how we handle internal control functions, because they don't really have a notion of default values. We have optional parameters, but the default value is not known to the engine, it's only known to the implementation. There are kind of ways to work around that. They are not really safe, so they will work for most functions, but for some which who like argument context, we might end up just crashing if this function is used with named parameters and particularly weird way. One of the nice things in PHP eight is that thanks to the stub effort we actually have default values for functions available as collectible meta data so it's available for reflection, and we will would also be able to use this for named parameters. If an internal function parameter has been skipped, we can essentially fetch it from reflection and fill in the value, the same way we would do for for normal user functions. The issue there is that this only works if there are stubs available. This works for all of our internal functions. I mean, not internal but bundled functions for PHP, but it will not work out of the box with old extensions. So it will mostly work, just this kind of parameter skipping is not going to work. So it will give you an error like okay we don't have default information for this function so you can't call it like this.

Derick Rethans  16:17  
	There's this common myth saying that reflection is actually a very slow thing, you should never use this in your code. Is this going to be a concern for using reflection information this way for internal functions?

Nikita Popov  16:29  
	Well, I mean the self like you will be directly using reflection, but internal API's that do the same thing. There is a performance concern here because we store the default values, not as values but as strings. So, in the worst case we actually have to parse those strings, convert them into a syntax tree, validate the syntax tree. That's all. That's of course slow, but it's not like we can't add a bit of caching in there to make sure this only happens once, at which point the problem should be avoided. 

Derick Rethans  17:02  
	Especially when you use things like opcache. 

Nikita Popov  17:04  
	I should say that I do expect name parameter calls to be generally slower than positional calls, so maybe in super performance critical code you would stick with the positional arguments.

Derick Rethans  17:16  
	I mean it would work perfectly well so far object construction still right? 

Nikita Popov  17:19  
	For object construction the real cost is really in the object allocations so and so.

Derick Rethans  17:24  
	With the introduction of named arguments aren't going to be any BC breaks, potentially?

Nikita Popov  17:29  
	There are not going to be any direct BC breaks, but there are of course some concerns. The first one is the change I mentioned about the variadics. That variadics can now have string keys. But I should clarify what I mean by: no, no, BC breaks. If you don't use named arguments than nothing is going to break. But of course, if named arguments are used with code that did not expect them, then we can run into some issues. So that's one of the issues. And the other one is more of a like long term maintenance concern that if we introduce named parameters, then those parameters become significant to the API, which means you cannot rename parameter names in minor versions of a library if you're semver compatible. Because, you might be breaking some codes on using those parameter names. And I think one of the biggest concerns that has come up in the discussion is that this is a significant increase in the API burden for open source libraries.

Derick Rethans  18:34  
	Because now suddenly, they have to think about the names of the arguments to all their methods as well, right.

Nikita Popov  18:39  
	So I think, like, the merits of this proposal, mostly comes down to how much additional burden does this impose on people maintaining libraries versus how much like ergonomics improvements that we get out of the feature for everyone else. One more thing to consider is that named parameters really change how you design APIs or what APIs you can reasonably design. So right now if you have a method with, for example, three boolean arguments, that would be like a really horrible method, because you call it like, true, true, false, like what does this mean? If you have name parameters, and you have the same three boolean arguments, then it's not really a problem any more. So you can, of course, you say, what the argument means and you can leave out arguments that are that you don't want to modify.

Derick Rethans  19:30  
	You mentioned that this RFC is quite old already. Do you think this will make it into PHP eight, as we're getting closer and closer to feature freeze, we're not quite there yet we have another month or so to go. Do you think it's ready enough to throw to the lions, so to speak?

Nikita Popov  19:46  
	So I think I will at least give it a try, because I do think that PHP eight is a good target for such a change. Even though it nominally does not break backwards compatibility, it does have a very significant impact in practice, so it wouldn't be good to put this on a major version. And additionally, we also did all this work on stubs in PHP eight with this it'll also fits in very well. Oh, and finally, one thing I didn't mention before is that we get attributes in PHP eight. And attributes, firstly, replace the existing Doctrine annocation system, which already supports named parameter.

	For all the code that is now going to migrate from Doctrine Annotations to PHP Attributes, it would be helpful if we had named parameters, because it would make the migration a lot more straightforward, because you don't also have to change the meaning of the arguments at the same time. 

Derick Rethans  20:51  
	I'm curious to see what the reception of this will be, especially when it is going to be voted for. 

Nikita Popov  20:57  
	Yeah me as well. I never did get this to voting, the last time around, but we should at least get a vote this time and well if it doesn't go through then there is always next time. 

Derick Rethans  21:10  
	there's always next time yes. Okay Nikita Thank you for taking the time this morning to talk to me about named arguments. 

Nikita Popov  21:17  
	Thanks for having me Derick. 

Derick Rethans  21:20  
	Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.



Show Notes
----------

- RFC: `Named Parameters <https://wiki.php.net/rfc/named_params>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
