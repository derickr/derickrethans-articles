PHP Internals News: Episode 72: PHP 8.0 Celebrations!
=====================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-11-26 09:35 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin072
   :Enclosure: media/pin-072.mp3

In this episode of "PHP Internals News" we're looking back at all the RFCs
that we discussed on this podcast for PHP 8.0. In their own words, the RFC
authors explain what these features are, with your host interjecting his own
comments on the state of affairs.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-072.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:23  
	Hi, I'm Derick, and this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. 

Derick Rethans  0:32  
	This is Episode 72. PHP eight is going to be released today, November 26. In this episode, we look back across the season to find out which new features are in PHP eight dot zero. If I have spoken with the instigator of each of these features, I'm letting them explain what this new feature is. in the first episode of this current year, I spoke with Nikita Popov about weak maps, a feature that builds on top of the weak references that were introduced in PHP seven four. I asked: What's wrong with the weak references and why do we now need to weak maps. 

Nikita Popov  1:10  
	There's nothing wrong with references. This is a reminder, what weak references are about, they allow you to reference, an object, without preventing it from being garbage collected. So if the object is unset, then you're just left with a dangling reference, and if you try to access it you will get acknowledged sort of the object. Now the probably most common use case for any kind of weak data structure is a map or an associative array, where you have objects and want to associate some kind of data with some typical use cases are caches or other memoize data structures. And the reason why it's important for this to be weak, is that you do not. Well, if you want to cache some data with the object and then nobody else is using that object, you don't really want to keep around that cache data, because no one is ever going to use it again, and it's just going to take up memory usage. And this is what the weak map does. So you use objects as keys, use some kind of data as the value. And if the object is no longer used outside this map, then is also removed from the map as well. 

Derick Rethans  2:29  
	A main use case for weak maps will likely be ORMs and related tools. In the next, episode 38, I discussed this trainable interface with Nicolas Grekas from Symfony fame. I asked: Nicolas, could you explain what stringable is.

Nicolas Grekas  2:45  
	Hello, and stringable is an interface that people could use to declare that they implement some the magic to string method.

Derick Rethans  2:53  
	That was a short and sweet answer, but a reason for wanting to introduce this new interface was much more complicated, and are also potential issues with breaking backwards compatibility. Nikolas replied to my questioning about that with:

Nicolas Grekas  3:06  
	That's another goal of the RFC; the way I've designed it is that I think the actual current code should be able to express the type right now using annotations, of course. So, what I mean is that the interface, the proposal, the stringable is very easily polyfilled, so we just create this interface in the global namespace that declare the method and done, so we can do that now, we can improve the typing's now. And then in the future, we'll be able to turn that into an actual union type.

Derick Rethans  3:39  
	I had a chat with Nikita Popov about union types as part of the last season in Episode 33 before PHP seven four was released, but after the feature freeze for it happened. I happen to speak with Nikita quite a lot because he does so much work improving PHP. In episodes 40 and 43, we discussed a bunch of smaller features and tweaks to the language. First up was the static return type, Nikita explains:

Nikita Popov  4:05  
	So PHP has a three magic special class names that's self for into the current class parent, refering to the parent class, and static, which is the late static binding class name. And that's very similar to self. If no inheritance is involved than static is the same as self, introducing refers to the current class. However, if the method is inherited. And you call this method on the child class. Then, self is still going to refer to the original class, the parent. Well static is going to refer to the class on which the method was actually called.

Derick Rethans  4:50  
	Next, most the class name literal on objects, which adds the following:

Nikita Popov  4:55  
	And that just returns to the fully qualified class name, for example, have a use statement for that class, you can get back the full name instead of the short name. I think we've had this since PHP five five. That's a great feature because it's like makes it clear whether you're referencing the class and not just some random string, and that means, for example, that the IDE refactorings can work better and so on. 

Derick Rethans  5:20  
	In PHP seven, we touched up inconsistencies in PHP is compound variable syntax, but we missed a few cases, Nikita explains:

Nikita Popov  5:28  
	All of these remaining consistencies are like really really minor things and edge cases. But weirdly, all or at least most of them are something that someone that at some point ran into and either open the bug or wrote me an email, or on Twitter. So people somehow managed to still run into these things.

Derick Rethans  5:56  
	But syntax changes are not the only thing that needs fixing; sometimes we also get the theory slightly wrong. In this case related to inconsistencies with traits, Nikita explains again:

Nikita Popov  6:07  
	The problem is that traits are sometimes not self contained. So to give a specific example we have in the logger PSR. We have a trait called logger treat, which has a bunch of methods like: warning, error, info, notice, and so on. So we just simple helper methods which all call the log method with a specific log level, and this trait only specified these helper methods, but still requires the actual class to implement the log method that we usually indicate that is by adding an abstract method to the trait. You have all the methods you actually want to provide by the trait and to have a number of abstract methods that the trait itself requires to work. This already works fine. The problem is just that these methods are not actually validated, or they are all inconsistently validated. Even though the trait specifies this abstract method, you could implement it in the class with a completely different signature.

Derick Rethans  7:09  
	Probably one of the big new features and PHP 8.0 are attributes, certainly the most discussed new feature with various RFCs to introduce a feature, and then change the syntax a few times. It all started with Benjamin Eberlei introducing a feature in April, ran Episode 47 I asked Benjamin, what attributes are. He replied:

Benjamin Eberlei  7:30  
	They are a way to declare structured metadata on declarations of the language. So in PHP, or in my RFC, this would be classes, class properties, class constants, and regular functions. You could declare additional metadata there that sort of tags, those declarations with specific additional machine readable information.

Derick Rethans  7:56  
	At the moment, many tools are already used up block comments to do a similar thing. So I asked how attributes are different. Benjamin answered with his first proposed syntax:

Benjamin Eberlei  8:06  
	The idea is that we introduce a new syntax that is independent of the docblock comments. Essentially, before each declaration, you can use the lesser-than symbol twice, then the attribute declaration, and then the greater-than sign twice. This is the syntax, I've used from the previous attributes RFC. Dmitri at that point, use the syntax from Hack. And it makes sense to reuse this not because Hack and PHP are going in the same direction anymore, but because Hack at that point they introduced that they had the same problems with which symbols are actually still easy to use. And we do have a problem in PHP a little bit with that kind of sort of free symbols. We can still use at certain places, and lesser-than and greater-than at this point are easy to parse. There are a bunch of alternatives, and one thing that I would probably proposes an alternative syntax, where we start with a percentage sign, then a square bracket open, and then a square bracket close. It is more in line with our Rust declares attributes, by Rust uses the sort of the hash symbol which we can't use because it's a comment in PHP. 

Derick Rethans  9:23  
	He already hinted at alternatives for syntax and we'll get back to that in a moment. However, the main important thing is what was inside the surrounding attribute notation, and Benjamin explains again: 

Benjamin Eberlei  9:35  
	If you declare an attribute name, and then you sort of have a parenthesis, open parenthesis close to pass optional arguments. You don't have to use them so you can only use the attribute name. If you sort of want to tag something just, this is a validator, this is an event listener, or whatever you come up with to use attributes for. But if you need to configure something in addition, then you can use the syntax sort of looks like if you would construct a new class, except that you don't have to put the new keyword in front of it.

Derick Rethans  10:10  
	In the rest of that episode we spoke about how to use attributes and what the general ideas behind them were. Soon after the attributes RFC was accepted, Benjamin proposed a second related RFC to tweak some of the working that came up throughout the discussion phase. At the time of the original RFC he did not want to make any changes in order to keep the discussion focused, but there were some tweaks necessary. In Episode 64 Benjamin explains what these changes were:

Benjamin Eberlei  10:38  
	There was renaming the attribute class. So, the class that is used to mark an attribute from PHPAttribute to just Attribute. I guess we go into detail in a few seconds but I just list them. The second one is an alternative syntax to group attributes and yeah save a little bit on the characters to type and allow to group them. The third was a way to validate which declarations an attribute is allowed to be set on, and the fourth was a way to configure if an attribute is allowed to be declared once or multiple times, on one declaration.

Derick Rethans  11:23  
	All the suggested tweaks passed with ease. A contentious issue, however, was the syntax to enclose the attributes. Two RFCs later, the PHP development team finally settled on the syntax which has attributes enclosed in hash, square bracket open, and close with the square bracket.

Derick Rethans  11:44  
	George Peter Banyard likes tidying up things in the language. I spoke with him on several occasions this season, where he was suggesting to just do that. In the first instance he's suggesting to change PHP to use locale independent floating point numbers to string conversions. He explains the problem:

George Peter Banyard  12:02  
	Currently, when you do a float to string conversion. So or casting or displaying a float, the conversion will depend on like the current locale. So instead of always using like the decimal dot separator. For example, if you have like a German or the French locale enabled, it will use like a comma to separate like the decimals.

Derick Rethans  12:23  
	He explained what he suggested to change:

George Peter Banyard  12:26  
	Change more or less to always make the conversion from float to string, the same so locale independent, so it always uses the dot decimal separator. With te exception of printf was like the F modifier, because that one is, as previously said locale aware, and it's explicitly said so.

Derick Rethans  12:46  
	The second RFC was candidly titled saner numeric strings. I asked George, what the scope of the problem he wanted to address is.

George Peter Banyard  12:55  
	PHP has the concept of numeric strings, which are strings which have like integers or floats encoded as a string. Mostly that would arrive when you have like a GET request or a POST request and you take like the value of the form, which would be in a string. And the issue is that PHP makes some kind of weird distinctions, and classifies numeric strings in three different categories mainly. So there are purely numeric strings, which are pure integers or pure float, which can have an optional leading whitespace and no trailing whitespace. However trailing white spaces are not part of the numeric string specification in the PHP language. To deal with that PHP has a concept of leading numeric strings, which are strings which are numeric but ...

Derick Rethans  13:42  
	As you can hear the way how PHP handles these numbers in strings is extremely complicated. The fix is just as complicated, but it pretty much boils down to stop treating strings like "5elephant" as a number. In the last episode, I briefly discuss Larry Garfield's object ergonomics article where he sets out a more coherent way forwards into thinking on how to solve some more of the bigger pain points of PHP. Most related to value objects. Although he did not end up proposing any RFCs himself, Nikita Popov did take some inspiration from it. And he proposed two features for inclusion into PHP eight: constructor property promotion, and named arguments. In Episode 53 Nikita explains with constructor property promotion intends to solve.

Nikita Popov  14:29  
	Right now, if we take a simple example from the RFC, we have a class Point, which has three properties, x y and z. And each of those has a float type. And that's really all the class is ideally, this is all we would have to write. But of course, to make this object actually usable we also have to provide a constructor. And the constructor is going to repeat that, yes, we want to accept three floating point numbers, x y and z as parameters. And then in the body we have to again repeat that. Okay, each of those parameters needs to be assigned to a property. So we have to write this x equals x, this y equals y, this z equals z. I think for the Point class. This is still not a particularly large burden. Because we have like only three properties. The names are nice and short, the types are really short, and we don't have to write a lot of code. But if you have larger classes with more properties, with more constructor arguments, with larger and more descriptive names, and also larger and more descriptive type names. And this makes up for quite a bit of boilerplate code.

Derick Rethans  15:52  
	I asked: What is the syntax that you're proposing to improve this?

Nikita Popov  15:57  
	The syntax is to merge the constructor and the property declarations, so you declare the constructor, and you add an extra visibility keyword in front of the normal parameter name. So instead of accepting float x, in the constructor. You accept public float x. And what this shorthand syntax does is to also generate the corresponding property. So you're declaring a property, public float x, and to also implicitly perform this assignment in the constructor body so to assign this x equals x. This is really all it does so it's just syntactic sugar. It's a simple syntactic transformation that we're doing, but that reduces the amount of boilerplate code you have to write for value objects in particular, because for those commonly, you don't really need much more than the properties and the constructor.

Derick Rethans  16:58  
	Tying in the constructor property promotion was a slightly more controversial RFC, named arguments, which I discussed with Nikita in Episode 59. I asked him what named arguments are:

Nikita Popov  17:09  
	Currently if you're calling a function or a method you have to pass the arguments in a certain order. So in the same order in which they were declared in the function or method declaration. And what named arguments are, and parameters allow us to do, is to instead specify the argument names, when doing the call. Just taking the first example from the RFC, we have the array_fill function, and the array_fill function accepts three arguments. So you can call like array_fill(0, 100, 50). Now, like what what does that actually mean. This function signature is not really great because you can't really tell what the meaning of this parameter is and, in which order you should be passing them. So with named parameters, the same code would be something like array_fill( start: 0, number: 100, value: 50). And that should immediately make this call, much more understandable, because you know what the arguments mean. And this is really one of the main like motivations or benefits of having named parameters.

Derick Rethans  18:21  
	We also briefly touched on the main issues where the introduction of named arguments could introduce backward compatibility issues.

Nikita Popov  18:28  
	If you don't use named arguments that nothing is going to break. But of course, if named arguments are used with codes that did not expect them, then we can run into some issues. So that's one of the issues. And the other one is more of a like long term maintenance concern, that if we introduce named parameters, then those parameters become significant to the API. Which means you cannot rename parameter names in minor versions of libraries if you're semver compatible. Of course, you might be breaking some codes, using those parameter names. And I think one of the biggest concerns that has come up in the discussion is that this is a significant increase in the API burden for open source libraries.

Derick Rethans  19:15  
	Beyond the main features that we've discussed so far. PHP eight also outs a few smaller ones. For example, the non capturing catches, which I discussed with Max Semenik in Episode 58. He explains his short proposal:

Max Semenik  19:29  
	In current PHP, you have to specify a variable for exceptions you catch, even if you don't need to use this variable in your code. And I'm proposing to change it to allow people to just specify an exception type.

Derick Rethans  19:48  
	This proposal password 48 votes for, and one against. The last few major PHP releases a lot of focus was put into strengthening PHP's type system. You see that and PHP seven four with additions to OO variance rules, and in PHP eight already with union types, the stringable interface and a static return type. Dan Ackroyd explains in episode 56, why he was suggesting to ask the predefined union type "mixed".

Dan Ackroyd  20:14  
	I have a library for validating parameters, and due to how that library needs to work the code passes user data around a lot. Internally, and then back out to whether libraries return the validator's result. So I was upgrading that library to PHP 7.4, and that version introduced property types, which are very useful things. What I was finding was that I was going through the code, trying to add types everywhere occurred. And as a significant number of places where I just couldn't add a type, because my code was holding user data. It could be any other type. The mixed type had been discussed before, an idea that people kind of had been kicking around but it just never been really worked on. So that was the motivation for me, I was having this problem where I couldn't upgrade my library, as I wanted to, I kept forgetting: has this bit of code here, been upgraded and I just can't add a type, or is it the case that I haven't touched this bit of code yet.

Derick Rethans  21:16  
	When I spoke with Dan, he also mentioned that sometimes he assists with writing RFCs in case some person would benefit from some technical editing, for example, due to language barriers. In the same way I ended up speaking to Dan again in Episode 65 about a null safe operator, on which he was working with Ilija Tovilo. That explains what a feature is about.

Dan Ackroyd  21:38  
	Imagine you've got a variable that's either going to be an object, or it could be null, so the variable is an object, you're going to want to call a method on it, which obviously if it's null, then you can't call method on it because it gives an error. Instead, what the null safe approach allows you to do is to handle those two different cases in a single line, rather than having to wrap everything with if statements to handle the possibility that it's null. The way it does this is through a thing called short circuiting, so instead of evaluating whole expression. As soon as use the null safe operator, I want the left hand side of the operator is null, and then get short circuited and it all just evalutates to null instead.

Derick Rethans  22:18  
	It also gets an additional benefit related to having shorter code.

Dan Ackroyd  22:24  
	And having the information about what the code's doing in the code, rather than in people's heads makes a lot easier for compilers and static analyzers to their jobs.

Derick Rethans  22:35  
	The last big nice syntax feature in PHP eight zero is the match expression, again by Ilija Tovilo. Instead of me interviewing Dan again, I decided as a joke to interview myself on the subject. It was a little bit surreal but I think it worked out well enough, as a one off event. I first explained to myself the problem with the existing switch language construct. 

Derick Rethans  22:56  
	So, before we talk about the match expression, we really need to talk about switch. Switch is a language construct in PHP that you probably know allows you to jump to different cases depending on the value. So you have to switch statement: switch, parentheses opening, variable name, parenthesis closes, and then for each of the things that you want to match against your use case condition, and that condition can be either static value or an expression. But switch has a bunch of different issues that are not always great. So the first thing is that it matches with the equals operator, or the equals, equals sign. And this operator as you probably know, will ignore types, causing interesting issue sometimes when you're doing matching with variables that contain strings with cases that contains numbers, or a combination of numbers and strings. So, if you do switch on the string foo, and one of the cases has case zero, then it will still be matched because it could type juggle the foo to zero, and that is of course not particularly useful. At the end of every case statement you need to use break, otherwise it falls down to the case that follows. Now sometimes that is something that you want to do, but in many other cases that is something that you don't want to do and you need to always use break. If you forget, then some weird things will happen sometimes. Anothercommon thing to use it switches that we sit on on a variable. And then, what you really want to do is the result of, depending on which case's being matched assign a value to a variable. And the current way how any student now is case, say case zero, $result equals string one, break, and you have case two where you don't set: return value equals string two and so on and so on. Which isn't always a very nice way of doing it because you keep repeating the assignment, all the time. And another but minor issue with switch is that it is okay not to cover every value with a condition. So, it's totally okay to have case statements, and then not have a condition for a specific type and switch doesn't require you to add default at the end either, so you can actually end up having a condition that would never match any case, and you have no idea that that would happen. 

Derick Rethans  25:16  
	Before I went into details, I also explained how the new match language construct could solve some of these criticisms. 

Derick Rethans  25:24  
	The match expression is a new language keyword, which also allows you to switch depending on a condition matching a variable. You're saying matching this variable against a set of expressions just like you would do with switch. But there's a few major differences with switch here. Unlike switch, match returns a value, meaning that you can do return value equals match, then your variable that you're matching on, and the value that gets assigned to this variable is the result of the expression on the right hand side of each condition. 

Derick Rethans  26:04  
	That's it for the new features in PHP eight, but I haven't spoken yet about a reason why the PHP team is releasing PHP eight and not PHP seven dot five. And of course, that is PHP's new JIT engine that is slated to improve performance, quite a lot. I have some concerns of my own. And in Episode 48 I spoke with Sara Goleman, which articulated my main concerns with it more eloquently.

Sara Golemon  26:31  
	If you go and look at the engine, particularly the runtime pieces of the engine, although the compiler's complex as well. You have to do a lot of digging before you even get to a point that you can see how the pieces maybe start to fit together. You and I have spent enough time in the engine code that we know where to look for a particular thing like let's say that opcode you mentioned that implements strlen. We know that Zend VM def dot h has got the definition for that. We also know that that file is not real code, it's a pre processed version of code that gets built later on. Somebody coming to that blind is not going to see a lot of those pieces. So there's already this big ramp up just to get into the Zend engine, as it exists now in 7.4. Let's add JIT on top of that, you've got code that is doing call forward graphs and single static analysis and finding these tracelets, and making sense of the code at a higher level than a single instruction at a time, and then distilling that down to instructions that the CPU is going to recognize, and CPU instructions are these packed complex things that deal with immediates and indirects, and indirects of indirects, and registers, and the x86 call API is a ridiculous thing that nobody should ever have to look at. So you add all this complexity to it, that by the way, sits in ext/opcache, it's all isolated to this one extension, that reaches into the engine and fiddles around with things to make all this JIT magic happen and we're going to take your reduced set of developers who know how to work on Zend engine and you're going to reduce that further. I think at the moment it's still only about three or four people who actually understand how PHP's JIT is put together enough that they can do any effective work on it.

Derick Rethans  28:20  
	I am still a little apprehensive about whether the effort of introducing a JIT engine is going to pay off. I certainly hope that I'm going to be proven wrong and that the JIT engine is going to be a massive performance boost. But in the end, we do definitely need more people to understand and work on the PHP engine and a new JOT engine that is built into opcache. Perhaps that's something you yourself might want to have a look at in 2021. PHP 8 will be out later today and I hope that you're pleased with all the new features that a PHP development worked on hard throughout the year. With this I'm concluding this episode and also this year's season. I will be back in the new year with more episodes where I hope to demystify the development of the PHP engine some more. Enjoy the holidays and stay safe.

Derick Rethans  29:08  
	Thank you for listening to this installment of PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next year.


Show Notes
----------

- Episode `#33 <https://phpinternals.news/33>`_: `Union Types <https://wiki.php.net/rfc/union_types_v2 >`_
- Episode `#38 <https://phpinternals.news/38>`_: `WeakMaps <https://wiki.php.net/rfc/weak_maps>`_
- Episode `#39 <https://phpinternals.news/39>`_: `Stringable Interface <https://wiki.php.net/rfc/stringable>`_
- Episode `#40 <https://phpinternals.news/40>`_: `Static Return Type <https://wiki.php.net/rfc/static_return_type>`_, `Class Name Literal on Object <https://wiki.php.net/rfc/class_name_literal_on_object>`_, and `Variable Syntax Tweaks <https://wiki.php.net/rfc/variable_syntax_tweaks>`_
- Episode `#43 <https://phpinternals.news/43>`_: `Validation for abstract trait methods <https://wiki.php.net/rfc/abstract_trait_method_validation>`_
- Episode `#47 <https://phpinternals.news/47>`_: `Attributes v2 <https://wiki.php.net/rfc/attributes_v2>`_
- Episode `#48 <https://phpinternals.news/48>`_: PHP 8.0 and JIT
- Episode `#52 <https://phpinternals.news/52>`_: `Locale-independent float to string cast <https://wiki.php.net/rfc/locale_independent_float_to_string>`_
- Episode `#53 <https://phpinternals.news/53>`_: `Constructor Property Promotion  <https://wiki.php.net/rfc/constructor_promotion>`_
- Episode `#54 <https://phpinternals.news/54>`_: `Magic Method Signatures <https://wiki.php.net/rfc/magic-methods-signature>`_
- Episode `#59 <https://phpinternals.news/59>`_: `Named Arguments <https://wiki.php.net/rfc/named_params>`_
- Episode `#64 <https://phpinternals.news/64>`_: `Attribute Amendments <https://wiki.php.net/rfc/attribute_amendments>`_, `Shorter Attributes Syntax <https://wiki.php.net/rfc/shorter_attribute_syntax>`_, and `Shorter Attributes Syntax Change <https://wiki.php.net/rfc/shorter_attribute_syntax_change>`_


Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
