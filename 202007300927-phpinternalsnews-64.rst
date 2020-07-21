PHP Internals News: Episode 64: More About Attributes
=====================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-07-30 09:27 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin064
   :Enclosure: media/pin-064.mp3

In this episode of "PHP Internals News" I chat with Benjamin Eberlei (`Twitter
<https://twitter.com/beberlei>`_, `GitHub <https://github.com/beberlei>`_,
`Website <https://beberlei.de>`_)
about a few RFCs related to Attributes.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-064.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:17
	Hi, I'm Derick, and this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 64. Today I'm talking with Benjamin Eberlei, about a bunch of RFCs related to Attributes. Hello Benjamin, how are you this morning?

Benjamin Eberlei  0:36
	I'm fine. I'm very well actually yeah. The weather's great.

Derick Rethans  0:39
	I can see that behind you. Of course, if you listen to this podcast, you can't see the bright sun behind Benjamin, or behind me, by the way. In any case, we don't want to talk about the weather today, we want to talk about a bunch of RFCs related to attributes. We'll start with one of the RFCs that Benjamin proposed or actually, one of them that he has proposed and one of them that he's put into discussion. The first one is called attribute amendments.

Benjamin Eberlei  1:05
	Yes.

Derick Rethans  1:06
	What is attribute amendments about?

Benjamin Eberlei  1:08
	So the initial attributes RFC, and we talked about this a few months ago was accepted, and there were a few things that we didn't add because it was already a very large RFC, and the feature itself was already quite big. Seems to be that there's more sort of an appetite to go step by step and put additional things, in additional RFCs. So we had for, for I think about three or four different topics that we wanted to talk about, and maybe amend to the original RFC. And this was sort of a grouping of all those four things that I wanted to propose and change and Martin my co author of the RFC and I worked on them and proposed them.

Derick Rethans  1:55
	What are the four things that your new RFC was proposing?

Benjamin Eberlei  1:59
	Yes, so the first one was renaming Attribute class. So, the class that is used to mark an attribute from PHP attributes to just attribute. I guess we go into detail in a few seconds but I just list them. The second one is an alternative syntax to group attributes and you're safe a little bit on the characters to type and allowed to group them. And the third was a way to validate which declarations, an attribute is allowed to be set on, and the force was a way to configure if an attribute is allowed to be declared once or multiple times on one declaration.

Derick Rethans  2:45
	Okay, so let's start with the first one which is renaming the class to Attribute.

Benjamin Eberlei  2:51
	Yeah, so in the initial RFC, there was already a lot of discussion about how should this attribute class be caught. Around that time that PhpToken RFC was also accepted, and so I sort of chose this out of a compromise to use PhpAttribute, because it is guaranteed to be a unique name that nobody would ever have used before. There's also, there was also a lot of talk about putting an RFC, to vote, about namespacing in PHP, so namespacing internal classes and everything. So I wanted to keep this open at the at the time and not make it a contentious decision. After the Attributes RFC was accepted, then namespace policy RFC, I think it was called something like that, was rejected, so it was clear that the core contributors didn't want a PHP namespace and internal classes to be namespaced, which means that the global namespace is the PHP namespace. And then there was an immediate discussion if we should rename at PhpAttribute to Attribute, because the prefix actually doesn't make any sense to have there. So the question was, should we introduce this BC break potentially, because it's much more likely that somewhere out there has a class Attribute in the global namespace. And I think the discussion, also one, one person mentioned that they have it in their open source project. Suppose sort of the the yeah should we rename it and introduce this potential BC break to, to users that they can't use the class Attribute and the global namespace anymore.

Derick Rethans  4:25
	So that bit passed?

Benjamin Eberlei  4:26
	That bit passed with a large margin so everybody agreed that we should rename it to Attribute. I guess consistent with the decision that the PHP namespace is the global namespace. It's sort of documented in a way. Everybody knows that we have the namespace feature for over 10 years now in PHP or oma, yeah exactly 10 years I guess. So by now, userland code is mostly namespaced I guess, and we can be a bit more aggressive about claiming symbols in the global namespace.

Derick Rethans  4:54
	Second one is grouping, or group statements in attributes, what's that?

Benjamin Eberlei  4:59
	The original syntax for the RFC, for attributes uses ... the ... we had this before. Remember? I can't spell, I don't know the HTML opening and closing signs.

Derick Rethans  5:15
	The lesser than and greater than signs.

Benjamin Eberlei  5:17
	Two lesser than signs and then two greater than signs, for the opening and closing of attributes. And this is quite verbose. So if you have multiple attributes on one declaration, it could be nice to allow to use the opening and closing brackets only once, and then have multiple attributes declared; so that was about the grouping. There was a sentence in there that should there be an RFC that sort of changes the syntax and that would become obsolete. It was also the most contentious vote of all the four because it almost did not pass but it closely passed.

Derick Rethans  5:56
	32-14 Yeah, yeah. So the group statements also made it in. You can now separate the arguments by comma. The third one is validate attribute target declarations.

Benjamin Eberlei  6:07
	That one makes a lot of sense to have from the beginning. What is does is, whenever you create a new attribute, you can configure that this attribute can only be used for example on a class declaration, or only on a method and function declaration, only on a property declaration. And so there are different targets, what we call them, where an attribute can be put on. This is something that will be required from the beginning. So, if you start using attributes, they will only work on certain declarations. Let's say we are configuring routes. So, the mapping of URL to controllers using attributes in a controller can only ever be a function, or a method. So it makes sense to require that the route attribute is only allowed on the function and the method. This should be a good improvement. Another example would be if you use attributes for ORM. So database configuration, then you would declare properties of classes to be columns in a database table, but it would never makes sense to declare the column attribute on a method, or on a class, or something like this. So it would be nice if the validation fails, or the usage of the attribute fails on these other declarations.

Derick Rethans  7:25
	When is this target checked?

Benjamin Eberlei  7:27
	The target checks are only done, I started referring to this as deferred validation, so that the validation is deferred to the last possible point in the usage of attributes. Which is, you call the method newInstance(), so give me an instance of this attribute on the ReflectionAttribute class. This has been a point of discussion because it's quite late, it will not fail at compile time, for example, even though it potentially could. The idea to do this at this late point is: we cannot always guarantee that an attribute is actually available when the code is compiled. For example if you use attributes of PHP extensions, the extension might be disabled. And also, you might have different attributes of different libraries on the same declaration. If we would validate them early it could potentially lead to problems. Well, sort of one usage of one attribute of one library breaks others. We decided to make this validation as late as possible from the language perspective. I already see that potentially static analysis tools like Psalm, PHP Stan and IDEs will probably put it in your face very early that you're using the attribute on the wrong declaration. And I guess I don't know static analysis tools in PHP have gotten extremely good and more and more people are starting to use them. And I really like the sort of approach that the language at some point can't be too strict. And then the static analysis tools, essentially, introduce an optional build step that people can decide to use where the validation is happening earlier.

Derick Rethans  9:06
	Where do you define what the targets for an attribute are?

Benjamin Eberlei  9:10
	This was also a lot of discussion. We picked the most simple approach; the Attribute class gets one new argument: flags, and it is a bit mask, and by default, an attribute is allowed to be declared on all declarations; classes, methods, properties, and so on. You can restrict the flags, the bitmask, to only those targets that you want to allow. So you would, let's say you have the route attribute we talked about before, you specify this as an attribute by putting an attribute at the class declaration. And then you pass it the additional flags and say only allowed using the bitmask for function method. I think it's just one constant, you use a constant to declare it's only allowed on a method or a function declaration.

Derick Rethans  9:59
	Can you for example set it a target is both valid for a method and a property?

Benjamin Eberlei  10:02
	You can, because it's a bit mask you can just use the OR operator to combine them, and allow them to be declared on both.

Derick Rethans  10:10
	It's also passed, so this is also part of PHP eight. And then the last one is the validate attribute repeatability

Benjamin Eberlei  10:20
	Repeatability essentially means you're allowed to use an attribute once, or multiple times on the same declaration. Coming back to the routing example, if you're having route attribute, then you could say this is allowed multiple times on the same function, because maybe different URLs lead to the same controller. But if you have a column property for example, you could argue, maybe it doesn't make sense that you define a column twice on the same property so it's mapped to two columns or something. So you could restrict that it's only allowed to be used once. This is also validated again deferred, so only when newInstance() called, it would throw an exception if it's sort of the second or third usage of an attribute but it's only allowed to be there once.

Derick Rethans  11:06
	What's the default?

Benjamin Eberlei  11:07
	The default is that the attributes are not allowed to be repeatable and you would, again using the flag arguments to the attribute class, put the bit mask in there that it's allowed to be repeatable.

Derick Rethans  11:21
	Is there anything else in that RFC.

Benjamin Eberlei  11:23
	No.

Derick Rethans  11:25
	The second one was actually something that came out of the first one that she did.  The original RFC already mentioned that she could for example have attributes called deprecated, or JIT, or no JIT. I suppose this RFC fleshes out the deprecated attribute a little bit more?

Benjamin Eberlei  11:40
	Idea was to introduce an attribute "deprecated", which you can put on different kinds of declarations, which would essentially, replace the usage of trigger_error(), where you would say, e_user_deprecated and then say for example: this method is deprecated use another one instead. Benefit of having an attribute versus using trigger_error() is that static analysis tools have an easier way of finding them. It's both a way to make it more human readable because attributes are declarative, it's easy to see them and spot them. And also machine readable. Both developer and machine could easily see that this method, or this class is deprecated, and stop using it, or using the alternative instead. The way it was done at the moment, or until now, mostly you're using a PHP doc block @deprecated in there. Please understand that. You can also read it as human, of course, the problem is that it has no really no runtime effect. That means during development for example, you couldn't see that you're accidentally using deprecated methods and collect this information, or even on production for example, because it's only in a doc block. And with an attribute, it would be possible to have this information at runtime to collect it, aggregate this information so that you see, it's been deprecated code is being used, and also make it human readable and for the IDEs and everything.

Derick Rethans  13:12
	When I looked at this RFC, it was under discussion, are you intending to bring this up for a vote, before 8.0 because you don't have a lot of time now?

Benjamin Eberlei  13:19
	I'm not. The reason is that first, you say, I'm running out of time, and I'm a bit stressed at the moment. Anyways, and even then, there are two points that I wanted to tackle it first. I mentioned that you can use the deprecated attribute on methods and functions which is something people are using now. One thing that I saw would be a huge benefit which is not possible at the moment is to be deprecating the use of classes, vacating the use of properties and constants. This is something you cannot hook into at runtime at the moment. So you cannot really trigger it. Essentially has led to a lot of hacks specifically from the Symfony community because they are very very good about deprecations. So they are, they have a very very thorough deprecation process, and they have come up with a few workarounds to make it possible to trigger this kind of things that would like to make this a much better supported case from the language. The problem there is that it touches the error handling of PHP and the usage of the error handler for deprecation triggers is a contentious problem, because it triggers global error handlers as by this side effects, sometimes effects on the performance because it logs to a file. There was a lot of questions if we can improve this in some way or another, and I haven't found a good way for this yet and I hope to work on this for a month.

Derick Rethans  14:47
	Maybe 8.1 then. The third RFC that goes into attributes, is the shorter attribute syntax RFC. What's wrong with the original syntax, where you have lesser than, lesser than, attribute greater than, greater than?

Benjamin Eberlei  15:03
	The name implies, the verbosity is a problem, I would agree. So writing, writing the symbols lesser than or greater than twice, yeah, takes up at least four symbols. There's one problem with shorter attribute syntax would be nice. Problem I guess is that this syntax is sort of a unicorn. We do refer to HHVM as one language or Hack, in that case one language that uses it. Whatever the Hack has essentially no adoption and is not very widespread in usage, and since they have parted ways with PHP. They also have a pass to removing the syntax and they, I think plan to change the syntax to use the @ symbol instead of the lesser and greater. So one problem would be that we were like, language wise, we would be a unicorn by using a syntax that nobody else is using this also I guess a problem. And then there are some, some problems that focusing on future RFCs. Something that I don't see myself really advocating or wanting, is allowing to nest attributes, so one attribute can reference another attribute, so that you have a sort of an object graph of attributes stuck together. This is necessary if you want to configure, if you want to use attributes to use very detailed configuration. Not sure if it makes sense at some point. There's this trade off where you should go to XML, YAML, or JSON configuration instead off attribute based configuration. To confusion with potential future usages of PHP and existing usages, so: generics. PHP would ever get generics, in some way, it would use the lesser and greater than symbols, but only once. I cleared up with Nikita before that there's no conflict between so that that would not be a problem. However, if we had the symbol used, they would be near each other. So attributes would come first and then there would be the function declaration using generics and types. These symbols would be very near each other so it might introduce some confusion when reading the code. Then there's obviously the existing usage of this shift up operators so if you're doing shift operations with bits, then they are using exactly the same symbol, the lesser than, greater than symbol twice, to initiate this or declare this operation.

Derick Rethans  16:18
	I realized there's a few issues with the pointy pointy attribute pointy pointy, to give it a funnier name.

Benjamin Eberlei  17:34
	Pointy pointy is good.

Derick Rethans  17:41
	I don't think I've come up with a name for a specific spaceship that looks like this yet, but there were a few other proposals. The original RFC already had the @:, which was rejected in that RFC. There was another proposal made that uses the @@ syntax to do attributes. It this something used by all the languages.

Benjamin Eberlei  18:01
	Syntax was the most discussed thing about attributes, so everybody sort of agreed that we need them or they we benefit. Nobody could agree on the syntax. So there were tons of discussion everywhere about how ugly the pointy pointy syntax is, and because we're using the @ symbol and the doc blocks already you by we couldn't use the @ symbol for attributes themselves, so there are lots of discussions running in circles. Just using the @ symbol once it's just not possible because we have the error suppression operator. Then a lot of people always suggest okay let's get rid of that, because it's anyways it's bad practice to use it

Derick Rethans  18:38
	It's a bad practice but sometimes it's necessary to use it.

Benjamin Eberlei  18:41
	So there are some APIs in the core that cannot be used without them. For example, unlink, mkdir, fopen. Because of race conditions you have to use the symbol there in many cases. Even if we would deprecate and remove that sort of syntax we would still somehow need it in a way for these functions. So I'm not seeing that we will ever remove them, and even then it will take two cycles of major versions, so it doesn't make sense. That one is out. The pointy syntax was something that Dmitri proposed many years before and people were sort of in favour of it, so I didn't change it for the initial RFC. We looked a lot at different syntaxes but none of them were nice and didn't break BC. And then when the vote started some core contributors proposed that maybe we can do slight BC breaks in the way we have existing symbols and one proposal was to use @@, which means we are using the @ symbol twice. This is actually allowed in PHP at the moment, we consider breaking this not a big problem because you can just use it once and it has the same effect. Problem would only be going through the code and finding all @@ usages which we assumed are zero anyways.

Derick Rethans  19:57
	Is there any other language that uses @@?

Benjamin Eberlei  19:59
	A few other languages use just to @, like many other, like many people propose PHP should also do, which I explained I guess is impossible. It's only proposed because it's looking close enough to @, and so it wouldn't be a huge difference and sort of familiar in that way.

Derick Rethans  20:18
	There was another alternative syntax proposed which is #[ attribute ]. Where does this come from?

Benjamin Eberlei  20:25
	So the @@ syntax is different to the pointy the syntax that doesn't have a closing symbol, it only has the starting symbols. Nikita and Sara both said that would be fine as a BC break and Sara is the release master for PHP eight, so she has a lot of pull on these kinds of things. Nikita as well. So, they said it's okay to have small BC breaks, we did look at other syntaxes. I saw that, which I liked before already but couldn't use because would be BC break, was the yeah hash, and then square brackets, opening and there's a closing square bracket close. This is used in Rust. It is a small BC break in PHP because the hash is used as a one line comment. It's not used that much any more, by people to use comments because coding styles discourage the use of it. I use it myself sometimes, like, specifically in one time shell scripts and stuff it's nice to use it, to use the hash as a comment. And the change, would there be that if you have a hash, followed by any other symbol than an opening square bracket, it would still be a comment. If the first symbol is square bracket opening, then it would be parsed as an attribute instead.

Derick Rethans  21:42
	So this is something you say that rust uses?

Benjamin Eberlei  21:43
	Yes.

Derick Rethans  21:44
	Because they're pretty much three alternatives now, we have the original pointy pointy one, which I still think is the best name for it, @@ operator, and then the hash square brackets operator. With three alternatives it's difficult to do our normal voting which is, how would you two thirds majority, but with three variants that is a tricky thing. As a second time ever, I think we use STV to pick our preferred candidate. Well this is STV kind of thing.

Benjamin Eberlei  22:11
	You need to help me with the abbreviation any ways resolving it.

Derick Rethans  22:14
	Single transferable vote.

Benjamin Eberlei  22:16
	STV is a process where you can vote on multiple options, and still make sure that the majority wins, or that the most preferred one wins. Ensure that it has a majority across the voters. The way it works is that everybody puts up that their preferences. There are many different options, in our case three, everybody puts their preference, I want this one first, this one second, is one third. And then the votes are made in a way where we look at the primary votes, who gave whom the primary vote. And then the option that has the least votes gets discarded. All those people that pick the least one, they have a second preference, and we look at the second preference of these voters and transfer, sort of change, the votes, their originally, but discarded option to the options that are still available. Now, somebody has the majority, they win if they don't, we discard another option and it goes on and on. With just three options, it's a transfer of a vote would essentially only happen once, because once one is discarded, one of the two other ones, not must have because it can be a tie, but probably has a majority, yeah.

Derick Rethans  23:34
	In this case, when we're looking at votes , when were counted them, it actually was obvious that no transferring of votes was actually necessary because the @@ won outright, and had quorum. In the end it up ended up being a pointless exercise, but it's a good way of picking among three different options. Which one won?

Benjamin Eberlei  23:53
	The @@ syntax won over the Rust Rusty attributes, and the pointy, pointy pointy attributes. One was clearly chosen as a winner, I think, but 54%, 56% in the first vote already. However, one thing that came up in the sort of aftermath of the discussion is that the grammar of this new syntax is actually not completely fine parse any language like PHP or any other programming languages to have defined the grammar and then the grammars inputs to a parser generator, as required some hacks to get around the possibility of using namespaces relative name step basis afterwards. This is something where Nikita satisfaction not allowed. In the way PHP builds its grammar, without any further changes @@ is actually going to be rejected as a syntax. However, Nikita independently worked on sort of change in the tokensets of PHP, that would provide a workaround, or not a workaround. It's actually a fix for the problem. If this new way of declaring tokens for namespaces would be used in PHP eight, then it's also possible to use the @@ syntax for attributes. If we don't change it, then the @@ syntax is actually not going to make it.

Derick Rethans  25:18
	In my own personal opinion, I think, @@ is a terrible outcome of voters is going to be so does that mean I should vote against Nikita's namespace names as token RFC or not? I mean it's a bit unfair.

Benjamin Eberlei  25:30
	I will vote for it. The premise of Nikita's RFC, and he worked on it before the @@ problem became apparent, so he did not only work on it to fix this problem. The problem he's trying to fix is that parts of namespace could use potentially keywords that we introduce as keywords in the future. An example is the word enum, and people are talking about adding enums to PHP for years. He had also one use case where one of his own PHP libraries broke by the introduction of the FN keyword for PHP 7.4, the short closure syntax. Essentially, it would help if some part of a namespace would contain a word that becomes a keyword in the future, that it wouldn't break the complete namespace. Because at the moment, the namespace. In this case is burned. You can't even use it anywhere in the namespace. The key word, which I guess it's a problem. But this reason I am actually for this proposal that Nikita has, even though I agree with you. I was actually a proponent of the Rusty attribute syntax, so I would have preferred to have that very much myself.

Derick Rethans  26:40
	I hope to talk to Nikita about what this namespace token thing is exactly about in a bit more detail in a future episode. Feature freeze is next week, August 4th. Are you excited about what PHP eight is going to be all about?

Benjamin Eberlei  26:54
	It's been a lot of last minute, things that have been added that to look really, really great. I can't remember PHP seven any more that closely, but I remember that it was the case with PHP seven as well. So anonymous classes was added quite late. Essentially PHP seven initially was just about the performance and then there was a lot of additional nice stuff added, very late, and made it a from a future perspective, very nice release, and it seems, it could be the same for PHP eight. Match syntax, attributes, then potentially the named parameters, which at the time of our speaking is still an open vote but looks very very promising.

Derick Rethans  27:36
	And then of course we have union types as well, and potentially benefits of the JIT right?

Benjamin Eberlei  27:40
	Union types this already accepted for so long I forgot about it.

Derick Rethans  27:45
	Alpha versions of PHP eight zero are already out, so if you want to play with these features you can already, and please do. And if you find bugs, don't assume it's you doing something wrong, instead file a bug report at https://bugs.php.net. Thank you, Benjamin, for talking to me about more attributes, this morning.

Benjamin Eberlei  28:03
	Thank you very much for having me.

Derick Rethans  28:06
	Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.



Show Notes
----------

- RFC: `Attribute Amendments <https://wiki.php.net/rfc/attribute_amendments>`_
- RFC: `Deprecated attribute <https://wiki.php.net/rfc/deprecated_attribute>`_
- RFC: `Shorter Attribute Syntax <https://wiki.php.net/rfc/shorter_attribute_syntax>`_
- RFC: `Treat namespaced names as single token <https://wiki.php.net/rfc/namespaced_names_as_token>`_
- `Psalm <https://psalm.dev/>`_
- `PHPStan <https://github.com/phpstan/phpstan>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
