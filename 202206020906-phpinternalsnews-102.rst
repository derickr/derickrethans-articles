PHP Internals News: Episode 102: Add True Type
==============================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2022-06-02 09:06 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin102
   :Enclosure: media/pin-102.mp3

In this episode of "PHP Internals News" I talk with George Peter Banyard
(`Website
<https://gpb.moe>`_, `Twitter
<https://twitter.com/Girgias>`_, `GitHub <https://github.com/Girgias>`_,
`GitLab <https://gitlab.com/Girgias>`_)
about the "Add True Type" RFC that he has proposed.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-102.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:00  
	Hi I'm Derick. Welcome to PHP internals news, the podcast dedicated to explaining the latest developments in the PHP language. This is episode 102. Today I'm talking with George Peter Banyard about the Add True Type RFC that he's proposing. Hello George Peter, would you please introduce yourself?

George Peter Banyard  0:33  
	Hello, my name is George Peter Banyard, I work part time for the PHP Foundation. And I work on the documentation.

Derick Rethans  0:40  
	Very well. We're co workers really aren't we?

George Peter Banyard  0:43  
	Yes, indeed, we all co workers.

Derick Rethans  0:45  
	Excellent. We spoke in the past about related RFCs. I remember, which one was that again?

George Peter Banyard  0:51  
	Making null and false stand alone types

Derick Rethans  0:53  
	That's the one I was thinking of him. But what is this RFC about?

George Peter Banyard  0:56  
	So this RFC is about adding true as a single type. So we have false, which is one part of the Boolean type, but we don't have true. Now the reasons for that are a bit like historical in some sense, although it's only from PHP 8.0. So talking about something historical. When it's only a year ago, it's a bit weird. The main reason was that like PHP has many internal functions, which return false on failure. So that was a reason to include it in the Union types RFC, so that we could probably document these types because I know it would be like, string and Boolean when it could only return false and never true. So which is a bit pointless and misleading, so that was the point of adding false. And this statement didn't apply to true for the most part. With PHP 8, we did a lot of warning to value error promotions, or type error promotions, and a lot of cases where a lot of functions which used to return false, stopped returning false, and they would throw an exception instead. These functions now always return true, but we can't type them as true because we don't have it, and have so they are typed as bool, which is kind of also misleading in the same sense, with the union type is like, well, it only returns false. So no point using the boolean, but these functions always return true. But if you look at the type signature, you can see like, well, I need to cater to the case where the returns true and when returns false.

Derick Rethans  2:19  
	Do they return true or throw an exception?

George Peter Banyard  2:22  
	Yeah, so they either return true, or they either throw an exception. If you would design these functions from scratch, you would make them void, but legacy... and we did, I know it was like PHP 8.0, we did change a couple of functions from true to void. But then you get into these weird shenanigans where like, if you use the return value of the function in a in an if statement, null gets because in PHP, any function does return a value, even a void function, which returns null. Null gets coerced to false. So you now get like, basically a BC break, which you can't really? Yeah, we did a bit and then probably we sort of, it's probably a bad idea. That's also the point of like, making choices, things that are static analysers can be like, more informants being like, Okay, your if statement is kind of pointless here.

Derick Rethans  3:06  
	Yeah, you don't want to end up breaking BC. Now, we already had false and bool, you're adding true to this. How does that work with Union types? Can you make a union type of true or false?

George Peter Banyard  3:18  
	No. So there are two reasons mainly. A. true and false is the same as like boolean, which is like just use Boolean in this case. But you can say, well, it's more specific, so just allow it. So that's would be reasonable. But the problem is, false has different semantics than boolean. False does not coerce values. So it only accepts false as a literal value. Whereas boolean, if you're not in strict type, which is a lot of code, it will cause values like zero to false one, or any other integers to true. It will coerce every other integer to true, like the true type follows the behaviour of false of being a value type. So it only accepts true, you would get into this weird distinction of does true or false, mean exactly true or false? Or do you get the same behaviour as using the boolean type?

Derick Rethans  4:07  
	So I would say that true or false would than be more restrictive than bool.

George Peter Banyard  4:12  
	Exactly, which is a bit of a problem, because PHP internally has true and false and separate types, which also makes the implementation of this RFC extremely easy, because PHP already makes the distinction of them. But at the same time, the boolean type is just a union of the bitmask of true and false. You can't really distinguish between the types, true or false, or the boolean type within the type system. Currently just does it by checking if it only has one then it can do like two checks. Specifically, you would need to add like an extra flag. I mean, it's doable, but it's just like, Well, who knows which semantics we want? Therefore, just leave it for future discussion because I'm not very keen on it to be fair.

Derick Rethans  4:55  
	True or false are really only useful for return values and not so much for arguments types, because if you have an argument that that always must be true, then it's kind of pointless to have of course.

George Peter Banyard  5:05  
	Same as like it was with the null type RFC. Although there might be one case where PHP internal functions might change the value to true for an argument, I can maybe two types, would be like with the define function, this thing being like case insensitive or case sensitive, I don't remember what the parameter actually; could actually either be false or true, because at the moment, I think emits a notice, things do like the this thing is not supported, therefore the values what was ignored. But we could conceivably see that in PHP 9, we would actually implement this as a proper like: Okay, this only accepts true, yes, this argument is pointless, but it's in the middle of the function signature, so you can't really move it. The spl_register_overload function has like as its second argument, the throw on error or not, which since PHP 8 only accepts true, but it's in the middle of the function. The last argument is still very useful. It's prepend, instead of append the autoloader, I think, or might be the other way around, check the docs. Since PHP 8, this only accepts true. So if you pass in false, it will emit a notice and saying you'd like this argument has just been ignored. So whatever. But we can't really remove the argument. Because well, it's, if you use the third argument, as with positional arguments, then you would change like the signature and you would break it. Now, we don't have a way to enforce in PHP to use named arguments, because that would be a solution. It's just like, well, if you want to set this argument, you need to use named arguments, but we can't do that. Otherwise, then creating a new function, which has an alias, which is also kind of terrible. That would be one of the maybe only cases where you would actually get like true as a as an argument

Derick Rethans  6:39  
	is that now currently bool? And there's a specific check for it?

George Peter Banyard  6:42  
	It's currently bool, and if you pass in false enrolment, like a warning, or notice.

Derick Rethans  6:47  
	How would inheritance work? As return types, you can always make them smaller, right? More restrictive.

George Peter Banyard  6:53  
	Yes, that's also the thing. But that already exists in some sense a problem of. Like if you go from boolean to false, you're already restricting the type. And that problem existed, even before the restricting, well allowing false as a stand-alone type if you had like, as a union, because you could always say like, I don't know. That problem already existed with Union types. Because you could have something like overturn an array or bool and then you change it to either an array or false. And then if you try to return like zero, then you will get like a coercion problem. So the same problem applies with true, because it only affects return values. And like you control the code within a function compared to like how you pass it, that's less of an issue. It applies also, with argument types where you can go from true to like boolean, or true and like a union type.

Derick Rethans  7:44  
	So there's nothing surprising here. I see that the RFC also talks a little bit about future scope. Can you tell a bit more about that?

George Peter Banyard  7:53  
	True and false are part of what are called value types, they are a specific value within the type. One possible future scope would be to expand value types to all possible types. So that you could say, oh, this function returns one, two or three.

Derick Rethans  8:08  
	Would you not rather use an enum for that?

George Peter Banyard  8:09  
	Exactly. That's the point I was going to make is that enums serve this purpose, in my opinion. And as a type purist, ideally, I would have preferred that we didn't have to enforce because the code, it kind of goes against the grain in this sense.

Derick Rethans  8:23  
	We've had it for 25 years, booleans.

George Peter Banyard  8:26  
	Yes, right. But boolean is its own type, in some sense, which you could say is a special enum. Enums are types. But we have false, and not having true is just so weird to me. It's like, oh, you've got this thing, but you don't have this other thing. And there are loads of cases where functions return true, or due to legacy reasons and to preserve BC, and PHP 8 promoted a bunch of warnings to to error. So now you've got functions which used to return false, don't return false any more. And they only return true. Now, some of the famous examples are probably like array_sort of, like actually, the sorting array functions, return true for basically all of PHP 7. I think there was something changed in PHP 7, probably was the engine or something like that, that they stopped returning false, which is strange. And I've made the discovery somewhat recently, I'm like, this is so pointless, because you see loads of loads of code checking like that the return value of the sort function is correct.

Derick Rethans  9:20  
	It's also that most of the sort functions actually sort by reference instead of returning the sorted array, which I can understand as a performance reason to do but...

George Peter Banyard  9:29  
	it's not very functional. You modify stuff in place and like passing it around. And because yeah, I think the initial thing was that like, well do it would return a false or true because sometimes it could, the sort could fail.

Derick Rethans  9:42  
	I don't understand how a sort could failure, but there we go.

George Peter Banyard  9:46  
	I mean, I suppose if you have like incomparable values within the array like that somewhat logical, I suppose.

Derick Rethans  9:53  
	Was there anything else in future scope?

George Peter Banyard  9:56  
	One of the future scope, I feel was everything type related. It's like type aliases, because when you start making more complicated types, having a way to type alias, it is probably nice. Don't think we'll get this for PHP 8.2. I don't think we any of us had the time to work on it.

Derick Rethans  10:11  
	Well, we only have a month left anyway.

George Peter Banyard  10:13  
	Yeah. And I mean, I'll probably be back on here. I'm trying to get DNF types working, but...

Derick Rethans  10:19  
	Can you explain that these are?

George Peter Banyard  10:20  
	Disjoint normal form types?

Derick Rethans  10:22  
	That did not help.

George Peter Banyard  10:24  
	But it's the being able to combine union types with intersection types together,

Derick Rethans  10:28  
	I can understand that doing that is kind of complicated. You also need to sort of come up with a with a language to define them almost right? I mean, you then get the argument, are you going to require a parenthesis around things?

George Peter Banyard  10:38  
	I'm requiring parentheses. People have told me the argument of like: Yeah, but in maths like and takes priority, it's just like, have you seen mathematicians, mathematicians don't agree on notation, and it's terrible, or they call stuff and the different they call it something is like, oh, sometimes a ring is commutative, and sometimes it's not. Don't follow mathematicians, don't follow mathematician,

Derick Rethans  10:57  
	Type aliases is something that would only apply to single files. See, that's what you're suggesting. And then there's exported type definitions, which I guess could be autoloaded at some point; would be nice to have, I guess.

George Peter Banyard  11:09  
	I think that's the trouble just like defining the semantics. Type aliases within a file are nice, but they're not very useful. Most of the time, you would want to export the type. For example, if you say: Oh, I accept, I don't know, something which looks like an array, which is like an array and like Traversable, and ArrayAccess or something. I'm sure, it's nice to have it in your own file. But like, if you use it around a project, and you need to redefine the type, every single file kind of defeats the purpose.

Derick Rethans  11:35  
	It's kind of tricky to do with type definitions, because you sort of need to make sure that there are available and maybe can be autoloaded, just like classes can be right. And that makes things tricky. Because having a type definition and just three lines in a file, is kind of annoying, but I guess that is sort of necessary to do the kind of thing in a PHP ish way.

George Peter Banyard  11:55  
	Yeah, we talked about it with Ilija because he he was on about it. And I was like: Well, ideally, you would want the separate autoload of types. That's how I initially conceived it, it's like having a different autoloading for types. But then the problem is, is like if anytime you hit a class, like in an argument, if you autoload the type first, it will go through all of the type definitions. And if, okay, at the moment, that wouldn't be there wouldn't be much. But if you go into like importing 30 composer projects, or libraries, which are define their own types, it will go through all of those first, before going to the classes autoloaded, and trying to find it then, which is not ideal. Yeah, it's going to be a tricky problem. It's either you merge these symbols together. But then the class table is not always a class. And sometimes you can't do new type. Like I said, tricky problems.

Derick Rethans  12:43  
	Yeah, that's a tricky problem, but an interesting one.

George Peter Banyard  12:47  
	Yeah.

Derick Rethans  12:47  
	So that's future scope then.

George Peter Banyard  12:50  
	Exactly. That is future scope.

Derick Rethans  12:52  
	Do you have anything else to add?

George Peter Banyard  12:54  
	Um, no, not really. I think I've said all I have to say it's pretty straightforward. Should be uncontroversial, hopefully.

Derick Rethans  13:02  
	It currently looks like it's 20 for, and zero again. So I guess it will pass.

George Peter Banyard  13:07  
	Brilliant.

Derick Rethans  13:08  
	Who said that, that if your RFC ends up passing unanimously, it is too boring?

George Peter Banyard  13:13  
	Nikita.

Derick Rethans  13:14  
	Which is not incorrect.

George Peter Banyard  13:16  
	It is not incorrect. But I mean, at the beginning, because I was like: Well, this is pretty straightforward. So I wrote the RFC, it was tiny. And I put it on to the list and people was like: Yeah, but what's the motivation for? I understand for adding false, because they already exist. But what's the motivation for adding a new type, and I was like, I now need to go back to the drawing board and write more. To be fair, that was a smart, because I then discovered the whole issue about true and false. That false is just a value type and doesn't do coercions. And it's like, okay, how do you handle the semantics and everything?

Derick Rethans  13:46  
	I'm glad to hear it. Then all I have to say thank you for taking the time today to talk about this new RFC.

George Peter Banyard  13:53  
	Thank you for having me as usual.

Derick Rethans  13:59  
	Thank you for listening to this installment of PHP internals news, a podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@php internals.news. Thank you for listening, and I'll see you next time.




Show Notes
----------

- RFC: `Add True Type <https://wiki.php.net/rfc/true-type>`_
- RFC: `Allow Null and False as Standalone Types <https://wiki.php.net/rfc/null-false-standalone-types>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
