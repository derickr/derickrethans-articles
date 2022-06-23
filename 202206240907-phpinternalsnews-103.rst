PHP Internals News: Episode 103: Disjunctive Normal Form (DNF) Types
====================================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2022-06-24 09:07 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin103
   :Enclosure: media/pin-103.mp3

In this episode of "PHP Internals News" I talk with George Peter Banyard
(`Website
<https://gpb.moe>`_, `Twitter
<https://twitter.com/Girgias>`_, `GitHub <https://github.com/Girgias>`_,
`GitLab <https://gitlab.com/Girgias>`_)
about the "Disjunctive Normal Form Types" RFC that he has proposed with Larry Garfield.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-102.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:15
	Hi, I'm Derick. Welcome to PHP internals news, a podcast dedicated to explaining the latest developments in the PHP language. This is episode 103. Today I'm talking with George Peter Banyard again, this time about a disjunctive normal form types RFC, or DNF, for short, which he's proposing together with Larry Garfield. George Peter, would you please introduce yourself?

George Peter Banyard  0:39
	Hello, my name is George Peter Banyard, I work on PHP paid part time, by the PHP foundation.

Derick Rethans  0:44
	Just like last time, we are still got colleagues.

George Peter Banyard  0:46
	Yes, we are indeed still call it.

Derick Rethans  0:48
	What is this RFC about? What is it trying to solve?

George Peter Banyard  0:52
	The problems of this RFC is to be able to mix intersection and union types together. Last year, when intersection types were added to PHP, they were explicitly disallowed to be used with Union types. Because: a) mental framework, b) implementation complexity, because intersection types were already complicated on their own, to try to get them to work with Union types was kind of a big step. So it was done in chunks. And this is the second part of the chunk, being able to use it with Union types in a specific way.

Derick Rethans  1:25
	What is the specific way?

George Peter Banyard  1:27
	The specific way is where the disjoint normal form thing comes into play. So the joint normal form just means it's a normalized form of the type, where it's unions of intersections. The reason for that it helps the engine be able to like handle all of the various parts it needs to do, because at one point, it would need to normalize the type anyway. And we currently is just forced on to the developer because it makes the implementation easier. And probably also the source code, it's easier to read.

Derick Rethans  1:54
	When you say, forcing it up on a developer to check out you basically mean that PHP won't try to normalize any types, but instead throws a compilation error?

George Peter Banyard  2:05
	Exactly. It's, it's the job of the developer to do the normalization step. The normalization step is pretty easy, because I don't expect people to do too many stuff as intersection types. But as can always be done as a future scope of like adding a normalization step, then you get into the issues of like, maybe not having deterministic code, because normalization steps can take very, very long, and you can't necessarily prove that it will terminate, which is not a great situation to be in. Imagine just having PHP not running at all, because it's stuck in an infinite loop trying to normalize the format. It's just like, oh, I can't compile

Derick Rethans  2:39
	Would a potential type alias kind of syntax help with that?

George Peter Banyard  2:44
	Maybe, I'm not really sure. Actually reading like research about it from computer scientists, in functional programming languages, which is everything is compiled on my head. And they have the whole thing was like, well, they need to type type normalize, and especially with type aliases, they haven't really figured out a way yet. So I'm not sure how we are going to figure out a way if experts and PhD students and researchers haven't really figured out a way.

Derick Rethans  3:08
	And is the reason for that mostly, because PHP, resolves types while it is running code sometimes because it has to overload classes, and then it might find out it is an inherited class, for example?

George Peter Banyard  3:19
	Yes, I think it's like this weird thing where might maybe PHP has like kind of an advantage, because it doesn't need to, like resolve all of the types at once. And if you have a type alias, it's just oh, if it's used, and you just need to resolve it, and then try to figure it out. There's also the added complexity of like, variance checks, because most functional programming languages, they have variance to some degree, but they don't have the whole inheritance of like typical OOP languages have. It's kind of a very strange field, the fact that yeah, PHP is just like, well, we kind of do stuff at runtime, and you don't necessarily need everything. And it just works is like, well, we'll do. That's mainly the reason why the dev needs to do the normalization step, the form is done. It's also I think, the most easiest to understand, it's just like, Oh, you have this and this, or this group, or stuff, or this group of stuff, or this thing, simple type. The other form would be another normalized form would be conjunctive normal form, which is a list of ANDs of ORs to just have this thing, or X, like (A or B or C) and X and (Y or Z), which I think is harder to understand.

Derick Rethans  4:26
	What is the exact syntax then?

George Peter Banyard  4:28
	So the exact syntax is, if you want to have an intersection type was in a union type, you need to like bracket it by parentheses. And then you have like the normal pipe union operator and you can mix it with single types, you can mix it with true, you can mix it with false, which are literal types, which now exist, or just normal, bool types.

Derick Rethans  4:48
	The parenthesis is actually required. You don't rely on operator precedence to make things work?

George Peter Banyard  4:53
	Yes. Relying on operator precedence is terrible.

Derick Rethans  4:57
	Yep, I agree.

George Peter Banyard  4:58
	I'd say Oh, yeah, but I think I've heard this argument on the list like a couple of times, it's just, oh, yeah, but maths, like, has like, and as priority over like, or, I mean, I did three years of a maths degree and not gonna lie. Maths notation is terrible for most of us. People don't even agree on terminology. I'm just gonna say, let's, let's just do better.

Derick Rethans  5:19
	I agree. I mean, most coding standards for any sort of variable for like conditions, will already require parenthesis around multiple complex clauses anyway, right? I mean, it's a sensible thing to do, just for readability, in my opinion. So the RFC also talks about a few syntax that you aren't allowed to do, and that you have to normalize or deconstruct yourself, what kinds of things are these?

George Peter Banyard  5:41
	if you would want to have a type which has an intersection of a class A with at least one other class, so let's say X or Y, but you can always convert it into DNF form, how this type would be, it would be (A and X) or (A and Y). This seems to be the more unusual case, I would imagine. One of the motivating cases of DNF types is to do something like Array or (Traversable and Countable). I don't really see mixing and matching various different object interfaces in differencing, the most useful user land cases to be able to do Array or (Traversable and Countable) so that you can use just count or seeing something as an array, or you have like Traversable and Countable and ArrayAccess. And it's just like, Oh, here's an object, which kind of behaves like an array.

Derick Rethans  6:32
	I think there's currently another RFC just being proposed, that extends iterator_to_array to multiple types as well to accept more things. So that sort of fits into this category of things to do with iterables and traversals then I suppose.

George Peter Banyard  6:49
	yeah

Derick Rethans  6:50
	I'm hoping to talk to the author of that RFC as well. At the moment where two and a half weeks or so before a feature freeze, you now see a whole flurry of RFCs while it was a bit quiet in the last few months. So because you're adding to the type system, that's also usually has consequences for variance rules, or rather, how inheriting works with return types and argument types, as well as property types. What do DNF types mean for these variance checks?

George Peter Banyard  7:19
	The variance is checks, kind of follow the similar rules as before. So property types are easy. They are invariant, so you can't change them. You can reorder types, like was in your union if you want to. But that was already the case with Union types previously, because PHP will just check that, well, the types match. So contravariant, you can always restrict types, meaning you can either add intersections, or you can remove unions, broadly speaking. What you could do, for example, if you have like A or B or C, you could do A and X as a subtype, because you're restricting A to be of an extra, like an extra interface.

Derick Rethans  8:06
	So then you will have (A and X) or B or C.

George Peter Banyard  8:10
	Yes. So that's one restriction. You can add how many interfaces you want and do an intersection type, you can add them on every type you can. On the other side, you can just add like unions. So if for contravariance, or like an an argument type, it's like, well, I just want to return something new, well, then you can add unions, but you can't add an intersection to a type, you can only widen types of arguments. So if your type is A or B or C, you can't do A and B, and you can't do (A and X) or B or C, because you're restricting the type. If your type would be (A and X) or (B and Y) or (C and Z), then you could lift the restriction to A or B or (C and Z) because you loosening the requirements on on the type that you're accepting.

Derick Rethans  8:55
	To summarize this: argument types, you can always widen; return types you can only restrict, and, and property types you can't change at all. I specifically wanted to summarize that because I always find contravariance and covariance. These names confuse me. So that's why I prefer to talk about widening types and restricting types instead. Because there are so close together for me. We spoke a little bit about redundant types. What is this new functionality do if you specify redundant types?

George Peter Banyard  9:30
	Redundant types how they currently work in PHP are done at compile time. And they do exact class matches or constant class aliasing matches.

Derick Rethans  9:41
	That will need an explanation.

George Peter Banyard  9:44
	Class names and interface names in PHP are case insensitive. So you can write a lower-case a or upper-case A and it means the same class. If you provide let's say lower-case a or upper-case A, the engine realize this, this is the same class, so we'll serve it on the type error. So PHP has use statements, or use as. So these are compile time aliases. If you define a class A, and then you say use A as B. So B is a compile time alias of A. And then you do a type which has A or B, PHP already knows these things refer to the same class. So it will raise a compile time error.

Derick Rethans  10:25
	These use aliases are per file only, right?

George Peter Banyard  10:28
	Yes, that's usually to do with if you import traits or like a namespaces. And you get conflicting class names. That's how you handle it about. PHP has also this feature, which you can do this at runtime, using the function called class_alias. Now, obviously, compile time checks are done at compile time. So it doesn't know at runtime that you aliasing these classes or using this name as an alias. So then PHP won't complain.

Derick Rethans  10:53
	But will don't complain during runtime.

George Peter Banyard  10:56
	No.

Derick Rethans  10:56
	You really just wanted to shoot yourself in the foot, we'll let you do this.

George Peter Banyard  11:00
	Yet, during this at runtime, just as like a whole layer of time, because it's not it's not really useful. Basically, what it means that PHP won't guarantee you the type is minimal. I.e. you might have redundant types, but it will just try to tell you, it's like oh, the- these are exactly the same types. And I know these are the same types, you probably do get mistake. So if it can determine this at compile time, it will tell you.

Derick Rethans  11:23
	The variance is still checked when you're passing in things.

George Peter Banyard  11:26
	Yes, so variance is checked on inheritance. When the class is inherited and compiled, because it needs to load the parent class, it will then check that it's built properly, and otherwise it will raise an error, that's fine. But just checking that the types is minimal is not possible. A) because inheritance, you don't know how it works, because it will only do the checks on basically on the name of the strings, it will do like compare strings of class names. And if it doesn't know the class name, or if it or if it needs to do some inheritance, it just won't do an instance of check. They just ignore that. It's just like, well, maybe it is maybe it's not I don't know. And that's fine.

Derick Rethans  12:08
	Of course, if you pass in a wrong type at runtime, then it will still get rejected during runtime anyway.

George Peter Banyard  12:14
	Yes, that hasn't changed.

Derick Rethans  12:16
	The only thing that you might end up in a situation where you don't get warned during compile time whether type is redundant.

George Peter Banyard  12:23
	Yes. So that's the behaviour we currently are the behaviour is added. So, it will check that two intersection types within the union are identical using the same class stuff. So for example, if you have class A, and you say use a as B, and then you have a type which is (A and X) or (B and X), it will tell you: Okay, these classes are the same. The check it adds now also it will check that you don't have a more restrictive type with a wider type. So if your type is T or (T and X), because T is wider than T and X, it will error at compile time, it'll tell you well, T is less restrictive than T and X. So the T and X type is redundant.

Derick Rethans  13:11
	Okay, so nothing strange. Basically, what you expect to happen will happen. And PHP does its best telling you at compile time whether you've done something wrong or not.

George Peter Banyard  13:22
	Yes.

Derick Rethans  13:24
	I think we've spoken mostly about the functionality itself and types. I'm a little bit interested in whether you encountered some interesting things while implementing this feature.

George Peter Banyard  13:33
	This feature basically, was a bit in limbo for the implementation, because I was waiting on a change to make Iterable, a compile time alias of Array or Traversable, which shouldn't affect userland. Because previously, all of the checks needed to cater to if you get Iterable, then you need to check for the variance. Has it Array , has it a Traversable type, does this accept? Is it why the is it more restrictive, it's identical. It's just this weird edge case, which makes the variance code harder. Moving this to a compile time alias, where now it just uses the standard, a standard union type in some sense, just makes a lot of the variance checks already streamlined and simpler. And because this is simpler, in some sense, was DNF types. When you hit the intersection, you need to recurse one step to check the variance. This helps. This is also kind of why DNF types are enforced like as like the structure on the dev because otherwise, you could potentially get into the whole like, oh, infinite recursion if you do like very nested types, because it's just like, oh, you hit one nested type and so, oh okay, now I'm again in unnecessary time and then you recurse again and then you recurse again, and so that's all you get into the thing: Oh you need to normalize the type. The variance check is: Can you see if it's a union type is the first type a sub list So a list of intersection types, okay, is it balanced? And then just recall the same function in some sense, like, check the types for variance, is this correct? Okay, move to the next type back into the Union and everything. So the implementation is conceptually simple, because all of the implementation details already exist. And all the everything hard has already been done. Now, it's just like, in some sense, it was extracting it into its own function, and then like recurse into it, and not forget to update opcache properly.

Derick Rethans  15:31
	You mentioned that in order to make the DNF types work, you were waiting on this Array or Iterable or Traversable kind of type. Is this also type people can use it and userland? Or is it internal only?

George Peter Banyard  15:44
	It is the standard Iterable type that you can already use. So currently, PHP considered Iterable, a full type in some sense. And what the this implementation change basically makes it Iterable into ... compile time alias of Array or Traversable. Iterable exists since, PHP, 7.1, I think. Can still use it, reflection should still be fine if you use it as a single type.

Derick Rethans  16:08
	So to change there is more, instead of: if you encounter Iterable, we check for both Array and Traversable. Then, instead of making the check every time you look at Iterable is already part of the type system, so you don't have to make the check every time.

George Peter Banyard  16:23
	Exactly, you basically move when it's being transformed in some sense. Now it has some repercussion on other parts, which needed to be taken care of, which is probably why it was in limbo for 10 months. I had already done the implementation of DNF types, basically, working on my local copy of that branch. It's just like: Okay, this got merged, nice, I can now open the PR onto PHP SRC. So I didn't wait for it to land until start working on it.

Derick Rethans  16:50
	Things like that also often affect reflection, because you're adding more complex types to the type system. So what kind of changes does that make to PHP's reflection system? And does this end up breaking backwards compatibility?

George Peter Banyard  17:04
	So in theory, no, it doesn't. How the reflection API works around the type system is that most method calls will turn a reflection type interface, ReflectionNameType, ReflectionUnionType, and ReflectionIntersectionType, are all instances of a ReflectionType. And methods if you would call on the list. So on a union type, the type it would return if you get like getTypes is a ReflectionType. The type system and how the reflection idea was designed, there is no BC break. How the standard was working, it's like, Oh, if you had like a union type, or an intersection type, if you call the getList or getListOfTypes, or getTypes, I don't remember exactly what the method name is actually called, you will always get an array of reflection name types, because you can only have like one level of list in some sense. However, now, if your top type is a union type, then if you get getTypes, you might get an array of ReflectionNameTypes with ReflectionIntersectionTypes. So that's the case that you now need to cater to. So if you get another ReflectionIntersectionType in between. There, you could only have ReflectionNameTypes, there was no nesting, whereas now if you have a union type, one of the types that you get back from the getTypes method in the array will be a ReflectionIntersectionType. Technically, all of the types of the part of the reflection type, so it's an array of reflection types that you get. How it worked before is that you didn't need to care about this distinction between: Oh, it returns a ReflectionType and a ReflectionNameType because well, it only return a ReflectionNameType. But now this is not the case. So you now need to cater to that that oh, you might have nesting. Which kind of boils down to like if in the future, we decide to like have oh, you can nest union types in an intersection type, then the getTypes method might return a union type with other name types.

Derick Rethans  19:03
	You just need to make sure that you check for more than just one thing that it previously would have done. You can't assume not everything is a ReflectionType any more. It could also be ReflecionIntersectionType.

George Peter Banyard  19:18
	Yes, exactly.

Derick Rethans  19:20
	I think that sort of what's in the RFC, is there any future scope?

George Peter Banyard  19:25
	I mean, the future scope is type alias. As usual. Everything I feel when you talk about the type system, it's like type aliases. At one point when your types gets very complicated. It would be nice to just be able to refer this as a as a named type in some sense, instead of needing to retype every time the whole union slash intersection of it. Hopefully we can get this running for 8.3. We are starting to get kind of complicated types. It would be nice being able to have this feature. The other obvious future scope in some sense, who knows if it's actually desirable is to allow either having conjunctive normal form so you can have like a list of ANDs or ORs

Derick Rethans  20:05
	You call these conjunctive normal forms?

George Peter Banyard  20:08
	Yes. Or just a type, which is not normalized. Not sure if it's really desirable to have this feature, because then you get into the whole thing of, if PHP doesn't, either PHP doesn't know how to like normalize it, or it's not in the best form, and then you get into like, very long compilation units or just checking. It's like, okay, does it respect the type? Does it do all of the instance of checks? And I'm not sure if it's super desirable.

Derick Rethans  20:38
	So it could be considered future scope. But from what I gather from you, you don't actually know what it is actually a desirable thing to add to the language?

George Peter Banyard  20:46
	Yes.

Derick Rethans  20:47
	Okay, George, thank you for taking the time this morning to talk about this new DNF types RFC.

George Peter Banyard  20:54
	Thank you for having me. As always.

Derick Rethans  20:59
	Thank you for listening to this installment of PHP internals news, a podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next time.




Show Notes
----------

- RFC: `Disjunctive Normal Form Types <https://wiki.php.net/rfc/dnf_types>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
