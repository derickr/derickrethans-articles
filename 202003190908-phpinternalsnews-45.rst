PHP Internals News: Episode 45: Language Evolution Overview Proposal
====================================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-03-19 09:08 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin045
   :Enclosure: media/pin-045.mp3

In this episode of "PHP Internals News" I chat with Nikita Popov (`Twitter
<https://twitter.com/nikita_ppv>`_, `GitHub <https://github.com/nikic/>`_,
`Website <https://nikic.github.io/>`_)
about the Language Evolution Overview Proposal RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-045.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:16  
	Hi, I'm Derick. And this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 45. Today I'm talking with Nikita Popov yet again about a non technical RFC that he's produced titled language evolution overview. Somewhere last year, there was a big discussion about P++, an alternative ID of how to deal with improving PHP as a language but also still think about how some other people already use PHP and I don't really want to change how they currently use PHP. Like then I didn't really have an episode about that because I'd like to keep politics out of this podcast, or definitely PHP's internals politics. I do think that we realised at that moment that something did have to happen, because there's not really policy about when we can add things, when we can remove things, and so on. So I was quite pleased to see that you have come up with a quite wordy RFC, not talking about anything technical, but more looking forward of were will see PHP in the near or medium future, I would say. What are your thoughts about making this RFC to start with?

Nikita Popov  1:29  
	As you mentioned we had some pretty, let's say heated discussions last year, concerning especially backwards incompatible changes. So there were a number of very, very contentious RFCs. One of them was the short opentags removal, and another one was the classification of undefined variable warnings. So whether those should throw or not throw, and well basic contention is this that PHP is a by now pretty old language, 25 years old. And we can all admit that it's not the language with the best design. So it has evolved relatively organically with quite a few words, and the famous inconsistencies. And now we have this problem where we would like to resolve some of these long standing issues. Many of them are genuine problems that are introducing bugs in code, that reduce developer productivity. But at the same time, we have a huge amount of legacy code. So there are probably many hundreds of millions of lines of PHP code. And every time we do a backwards compatibility break, that code has to be updated, or more realistically, that code does not get updated and keeps hitting on old PHP version that, at some point also drops out of security support. And now the question is how can we fix the problems that PHP has, while still allowing this legacy code to update their PHP version. The general idea of how to fix this is to make certain backwards compatibility breaks opt in. By default, you just get the old behaviour, but you can specify in some way, exactly how it's done doesn't really matter at this point, that you want to opt into some kind of change or improvement.

Derick Rethans  3:34  
	As one example being the strict types that have been introduced in PHP that you need to turn on with a switch with a declare switch. 

Nikita Popov  3:42  
	Strict types is really a great example because it has the important characteristic that has done per file. So you can turn on the strict types in one file and not affect any other code, at least in theory. So there are some edge cases, but I think like mostly you can just enable strict types in your library and you don't affect any other library that the project uses. We would like to extend this concept. It should be possible that libraries can update to your language, well, it's called language dialect without forcing other libraries or without forcing the using codes to update as well. Because this is what we have to do right now, though, before you can update your project to PHP eight, let's say, you first have to wait that all the libraries you're using update to PHP eight. And maybe there are libraries that are going to update but also say that: Okay, now actually PHP eight is required. And then you kind of get these complex dependencies with libraries supporting these versions and not supporting those versions, and doing updates becomes pretty hard. As I said, the idea is to make the these backwards incompatible changes opt in some way, and there are multiple general models. So as you mentioned, P++ is the most radical approach. It's more or less a separate language but sharing the same implementation. And as the name suggests that this is inspired by C and C++. So those are usually implemented in the same compiler. And they can be interoperable in a limited way, mostly in that you can use C code inside C++ easily. Using C++ code inside C code tends to be much harder. Yeah, P++ is, I think the option we are pretty unlikely to take for a couple of reasons, because it's this kind of one time huge break which first means that we only have one chance to get it right, and given all the track record, we should maybe not rely on that. Also means that the upgrade becomes especially hard because you have to do everything at once. It's not spread out over a longer time.

Derick Rethans  5:54  
	You say that we need to get it right in one go, but that is hard to say because you don't know, in the future what else we want to add? Like the RFC mentions a few few other cases, like, for example, things like forbidding dynamic Object Properties, we'd have to do right away now as well, if he'd go with the two languages one implementation phase, right? I mean, if we hadn't thought about it, nobody would have thought about it after the split as we made, we'd still not be able to do it. 

Nikita Popov  6:20  
	That's true. So P++ is, one time, one time solution. It doesn't really scale over time. I mean, there are also other concerns. And I think like in the end, one of the big ones is just that we don't have the resources for it anyway. So we have only maybe three full time developers on PHP. And I don't think we want to start focusing on this huge separate language more or less. Now we're just going to take a couple of years. Next to having this entirely separate language, there are two other ways to approach the problem. One is editions, which is a concept used by the rust programming language. The idea there is that next to the version, which is more or less than implementation version, you also have this edition, which is a completely orthogonal concept. Basically, we will say: okay right now we are for example at edition zero. And then in addition one you opt into some kind of set of backwards incompatible changes. Then in addition two, there are more backwards incompatible changes, and so on. Each edition is essentially a superset of the previous one. 

Derick Rethans  7:32  
	Would it also mean you couldn't get new features in a new edition or is it purely about making backwards incompatible changes? 

Nikita Popov  7:40  
	So, this is purely about backwards compatibility. So, if a new feature can be added without breakage then should always be available. The editions switch would only control the backwards incompatible parts. This is to contrast with the second approach, which is to have fine grained declare statements. As you already mentioned, we have the existing strict types directive and we could continue down the same path. So, we could add new declare for no dynamic Object Properties equals one, and then for a strict operators equals one, and for whatever else equals one. And then you would have this long list of possible declares, with which you could enable or disable some particular bit of language behaviour. 

Derick Rethans  8:26  
	Then I can imagine that in another five years, that list might be 20 options long. 

Nikita Popov  8:31  
	Right. So, the concern there is of course, one part is maintenance, because we have to support basically an exponential combination of different options. And the other is from the programmer perspective, that the like mental model becomes more complicated because you have to keep in mind like which exact set of declares am I using right now? I should say, though, that this model is actually used by Python. Because Python has this import or use from future feature. So there is basically this magic module __future from which you can import language features that will become the default in newer Python versions. For example, you can import the new integer division behaviour inside an older version. This is more or less the same as doing the declares, the fine grained declares, just with a different syntax and with the I think, stronger focus that the behaviour is going to become the default in the future version.

Derick Rethans  9:38  
	So basically, you're opting into experimental functions really?

Nikita Popov  9:41  
	Could be either experimental functions, or it could be really functions from newer versions. In particular Python, also for a while had parallel development of Python 2 and Python 3, in which context this probably makes more sense.

Derick Rethans  9:56  
	There's pretty much three options that the RFC mentions: a new language common implementation or the PHP / P++ option, the editions, and the fine grained declares. These are all still going to be based per file?

Nikita Popov  10:12  
	So that's the second large question, what is the general model? And the second one is where we declare it. The approach I was initially pursuing was to have this declare it at the package level. So for a whole library or for for a whole project. 

Derick Rethans  10:32  
	How would you define what a package is?

Nikita Popov  10:33  
	We have namespaces. And there is a somewhat loose coupling between namespaces and packages. So I have an old RFC for a namespace scope declares, where you could, for example, specify strict types for whole namespace, which is, I think, maybe the most natural way to treat packages right now, because this is the closest thing to a package we have. Fortunately, it does have a few issues. One of them is that this namespace package mapping is not always there. So there are packages that have some somewhat odd nesting of name spaces. And I've also heard that some people, for example, define their models inside the Doctrine name space, because they're, you know, extend their classes. So they also put them the namespace. Of course, you shouldn't do that. But it's things that could happen, because we don't really have this enforcement that the namespace really is a package. And then there are also technical concerns, because right now, namespaces are really just a compile time thing to handle name resolution, and now they kind of turn into a feature that also has some kind of runtime impact. And you have to consider things like what happens if you have multiple namespaces in the same file, and also other considerations, like what happens if the names namespace is first used, and you issue some namespace scope declares afterwards. All that can be resolved, but it makes the model somewhat more complicated.

Derick Rethans  11:53  
	And I guess you end up having to declare these namespace scope declares maybe in a separate file or something like that? 

Nikita Popov  12:14  
	At least what I have in mind that is that you would declare them in composer.json, and Composer would then take care of registering them with PHP itself. Of course, you could also do that manually, which are not using Composer but that at least was the 95% use case.

Derick Rethans  12:31  
	In applications that make use of Composer, it is very likely that Composer knows about all the libraries that a specific application uses, and hence will be able to construct an array, where it can tell PHP by calling a function declaring all the different options or editions of whatever that end's up being. 

Nikita Popov  12:49  
	So that's one of the approaches. There are also some alternatives. One is to instead introduce an actual package concept. One of the possibilities is to basically: add an extra line to each file, which says package and the package name. So that really removes any and all ambiguities. But you do have to add that extra line, which serves some very limited purpose. And basically only for these package scope declares, could maybe also be used for some extra features, like, package private symbols.

Derick Rethans  13:23  
	But it would also instantly make that code base non-parsable with older PHP versions. 

Nikita Popov  13:28  
	That's also true, right. But that's a general problem that most approaches I think, would have. So namespace scope declares is one that doesn't have it, but even the per file approach would have this problem because if you write for example, declare edition, then you would right now on PHP seven get the warning that the edition declare is not known. Yeah, last variant that I'm discussing here is to make packages based on the file system, which is something many other languages do. So you have some kind of magic file somewhere that says okay, this directory and all the sub directories are part of the package. In PHP, this kind of file system based approach is somewhat problematic, because our include mechanism is not really based on the file system but on fairly general stream abstraction. You can include from the file system, you can include, if you're really crazy from HTTP, but you can also include from Phar files, from an input stream, or from some kind of custom defined stream. These file system based packages require some additional operations to be well defined. So they have to have a notion of path canonicalization so you can determine whether a file is inside the directory, even if there are things like symlinks or the file system is case insensitive. Which does exist for the file system. So we have the real path syscall, but doesn't exist for streams right now. And a similar problem is that we need to be able to walk up from a path to the directories. And that's also something that doesn't exist for streams. And like more generally, not all streams really have a well defined concept of a directory. For example, if you are reading a file from stdin, so the stdin or the input stream, then there is no directory and like, which package is that going to be in?

Derick Rethans  15:31  
	I think it would be hard to end up debugging at some point. So why some things don't actually end up being in a package where you expect them to be, for example. And then on top of that, you also need to define: Well, how do I call this file and things like that, right? I mean, a PHP script wouldn't be just a single file, for example, would be a single file and this extra definition file. And that's the concept of course that we don't have in PHP at all. Everything is on profile pretty much.

Nikita Popov  15:56  
	Which is why at least to right now. I think, like the immediate way forward, is to use per file declares. So if we don't use the fine grained declare approach, and instead have a single edition, then it's not really a problem to put the declare edition inside every file, because this is already what we do for strict types. It's like not super ergonomic. But I think it's also not a huge problem. And it does have the one very big advantage that files are and remain self contained. So you don't have to consult an external definition that may be hard to locate to figure out how to process. 

Derick Rethans  16:36  
	And every IDE or tool would have to implement that same logic and make sure that it's all consistent with each other as well.

Nikita Popov  16:43  
	I wouldn't say it's really hard, but it might be somewhat fragile, especially when it comes to convention. I said if we put things in composer.json, there's probably something tooling can easily deal with. But if you then encounter a project that doesn't use Composer and uses as some other way to register the package declares, then you might run into problems.

Derick Rethans  17:09  
	Lots of things to talk about and discuss at some point. As you submitted this RFC to the mailing list some time ago now, what is sort of the feedback that you're getting on this?

Nikita Popov  17:19  
	So I think the general direction, at least this pretty clear. Most of the discussion is focused on the addition concept, not the finger in declaratives, or the P++. I think for now, we would also go with the per file approach. Now, the main two points that remain contentious is: first, how does the support timeline look like? So basically, the concept of editions just enables different libraries to upgrade independently. That's the core premise. But at least in Rust additionally editions of are also guaranteed to be supported forever. So you can leave your old code running on the old edition, and you do not have to ever update it.

Derick Rethans  18:10  
	How often do they make new editions? Every three years? 

Nikita Popov  18:13  
	Yeah, it's not quite clear yet, but probably it's going to be every three years. And now for us, the question is, well, do we want to support old editions forever? Or do we want to give them a finite lifetime? Say we introduced a new edition in PHP eight, and then we supported until PHP nine. That means code can take its time to do the necessary updates, but it does have to do the updates at some point.

Derick Rethans  18:37  
	But you'd have five years?

Nikita Popov  18:39  
	It's more of the general question of if it's forever or if it's limited. So I think based on the discussion, there is a pretty strong preference to not support them forever.

Derick Rethans  18:51  
	But for how long then? I mean, it must be longer than what we support a normal PHP version for, right?

Nikita Popov  18:56  
	Yeah, would expect it to be something like a major version cycle. The second question is related to the strict types, as you said, strict types is like an existing example of a mechanism that works like this. And now we're introducing a second mechanism with the same basic characteristics. Are we going to merge them or not? Would we say that, in the new edition that strict types is enabled by default, or even always enabled? If we do that, and we say that additions have limited support life, that means that strict types is going to become the only option in the future at some point, at least. You can imagine that this is somewhat contentious because there are quite a lot of people who consider weak types to still be the superior option.

Derick Rethans  19:49  
	Whenever I go speak at conferences or user groups, that's not the case. One question is, which keeps recurring always is: Why isn't this the default in PHP eight? I think there's an expectation that strict title at some point is going to be turned on by default.

Nikita Popov  20:04  
	Yeah, and the thing, this is where people disagree whether this expectation is this or not. So there are plenty of people in the discussion thread, well, by plenty I mean, at least two, who strongly think that strict types should remain an option. I mean, PHP of deals with often deals with input coming from HTTP or from a database which is usually coming in as a string. And they think that the typecast you have to do to make that work with strict types actually kind of weaken the type safety guarantees, because if you perform an explicit cast, then that cast is performed basically without any checks. So you can like take a completely non numeric string cast it to integer and you will get zero without any warning or whatever. While even in weak typing mode, that would still result in an error. 

Derick Rethans  20:58  
	It's a curious thing actually when you mention databases because, of course databases, you've defined very strict types for your data in them. It's just that it's interesting that PHP's interface to most of these old SQL databases, just decided to always turn into a string.

Nikita Popov  21:14  
	It's it does actually support returning things in they're like native type. 

Derick Rethans  21:20  
	With PDO, yes. 

Nikita Popov  21:21  
	But under options, and I think it's also like dependent on whether you do emulation or not, and stuff like that. And you have all these different drivers that have differing support for that. But yeah, to get back to strict types, but one of the options is to really keep editions and strict types separate, and also evolve the strict and the non strict mode independently. So you could say that in the new edition, the strict typing mode becomes stricter, for example, by also extending to operators, arithmetic operators, not just to function arguments, but that of course doesn't mean that: Yeah, we saying strict types of states exist forever as a separate track of language.

Derick Rethans  22:06  
	Yeah, that's an interesting one. I'm not sure how to get to a conclusion there actually. Because there's always going to be people on each side side. 

Nikita Popov  22:13  
	Yeah. 

Derick Rethans  22:13  
	Would you think that this language evolution overview proposal would have been decided on which way to go by the time feature freeze for PHP eight comes around?

Nikita Popov  22:23  
	I think it would be pretty good to have this for PHP eight, because well, it's new major version and the time to introduce this kind of concept. I should say, though, that we already have quite a few backwards incompatible changes in PHP eight, and at least some of them are, like, we are definitely not going to retrofit them into the editions concept. So there are already certainly going to be breaking changes there.

Derick Rethans  22:52  
	Why wouldn't you retrofit them? I mean, if we end up deciding a PHP eight will have these editions, would they not be part of that or would they always end up breaking anyway? Because it seems like a sort of an ideal place to then do it. 

Nikita Popov  23:05  
	And yeah, problem is just that the there are some quite extensive changes, especially when it comes to warnings versus exceptions, and will just be like a lot of efforts to get this under an edition flag and to support both behaviours there. Maybe some of the existing changes could be moved into there, with not a huge amount of effort. But I think there are definitely going to be some like hard edition independent breaking changes.

Derick Rethans  23:37  
	New major PHP versions still might have some backward breaking changes independently from when we do the editions or not, or more declares or not?

Nikita Popov  23:46  
	Yeah, that's like one more question, what exactly is the scope of editions? What goes into the edition, what doesn't go into there? I mean, there is always a cost to ending something with this mechanism. One is just maintenance for us. And of course that like user has to consider more different versions of the language. And I think one particularly large aspect that would likely never fall under edition concept is changes to the standard library. So additions work well for language changes, but I don't think they really make sense for a standard library changes. So everything that involves depreciations, or functions with eventual removal would not be covered for that.

Derick Rethans  24:31  
	Do you have an example of such a change in the standard library that PHP eight might have? 

Nikita Popov  24:36  
	What I just said might as the general that, usually in every PHP version, we deprecate a bunch of functions and are going to remove them at some point. And these deprecations are like going to apply independently of what edition you set. Actual changes in terms of like real behaviour changes of the standard library I think that's something we quite rarely do. Actual changes to the standard library where the behaviour of a function is changed. That's something we generally try to avoid. Specifically because this causes relatively subtle backwards compatibility breaks. So usually we will either do changes by introducing a new flag or a new function, or by deprecating the functionality entirely. Even when it comes to language changes, there is like I know one example. And the discussion was, well, if we had the edition concept, and we wanted to introduce something like traits, the trait functionality in general is not backwards compatibility breaking. But the trait feature does introduce two new reserved keywords, which is trait and insteadof. So there is technically a backwards compatibility break even though it's finer. And now you have the trade off. Do you introduce traits in the new edition and only reserve the keywords there, thus removing any backwards compatibility break. Or do you you introduce it always, which means that everyone can benefit from it, even if they haven't updated the code to the new edition yet. But it does introduce the small backwards compatibility break. And then you get this trade off and the discussion what you should be doing about that. 

Derick Rethans  26:17  
	I think making that kind of decisions will have to be done based on evidence. And I think in the past you've used the top thousand projects on GitHub and see whether things break or not to make a decision. For example, having the nested, or the triple, quadruple nested ternary. Anytime people use it, it's pretty much a bug in the code.

Nikita Popov  26:36  
	Yeah, so to give one example, in PHP 7.4, we introduced the short closure syntax with the fn keyword, and they're the source code analysis showed that basically, fn is not used outside of tests, apart from one library, which is my own. Which does have quite a few dependencies. And that library was indeed broken essentially completely by that change. So in that case, I think there might have been an argument that this feature should be introduced under an edition, because there is like evidence of actual breakage in the wild.

Derick Rethans  27:14  
	This is one of us trying to get it right. We now have evidence for it.

Nikita Popov  27:18  
	And probably like the insteadof keyword for traits, that there's much less problematic.

Derick Rethans  27:24  
	Again, as I say, it's the data that speaks that there right? That was quite a bit to go through. I'm curious to see where those discussions ends up going. Hopefully, we get to a conclusion somewhere in the next few months and ready for PHP 8.0. Who knows? Maybe we have another podcast episode where we introduce a new editions concept. 

Nikita Popov  27:43  
	So this is probably my most vague RFC, with a somewhat unclear goal and the somewhat unclear discussion outcome.

Derick Rethans  27:53  
	Do you have anything else to add to this discussion that we've missed?

Nikita Popov  27:55  
	I think there is just one thing maybe worth mentioning, which Rust uses pretty extensively, which has automatic upgrades. So they have some tooling to do that, which is mostly reliable. And I think it would be pretty nice if in PHP, we had something similar. In PHP, we can't really make this reliable because language is just way too dynamic. And we actually do have some tooling in the form of the rector library. But we might want to think about providing something under the PHP project umbrella that is more geared towards like doing updates that are as safe as possible. So you can run them without thinking but still reduce your loads some what.

Derick Rethans  28:40  
	And that is something that is definitely for the future. Thanks for talking to me about the language evolution overview proposal.

Nikita Popov  28:46  
	Thanks for having me, Derick.

Derick Rethans  28:53  
	Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP line. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.

Show Notes
----------

- RFC: `Language Evolution Overview Proposal <https://github.com/nikic/php-rfcs/blob/language-evolution/rfcs/0000-language-evolution.md>`_
- `Rector PHP Library <https://getrector.org/>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
