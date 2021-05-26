PHP Internals News: Episode 86: Property Accessors
==================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-05-27 09:14 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin086
   :Enclosure: media/pin-086.mp3

In this episode of "PHP Internals News" I chat with Nikita Popov (`Twitter
<https://twitter.com/nikita_ppv>`_, `GitHub <https://github.com/nikic/>`_,
`Website <https://nikic.github.io/>`_) about the "Property Accessors" RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-086.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:14  
	Hi I'm Derick. Welcome to PHP internals news, a podcast dedicated to explain the latest developments in the PHP language. This is episode 86. Today I'm talking with Nikita Popov about his massive property excesses RFC. Nikita, would you please introduce yourself?

Nikita Popov  0:32  
	Hi Derick, I'm Nikita, and I do work on PHP core development, on behalf of JetBrains.

Derick Rethans  0:39  
	This is probably the largest RFC I've seen in a while. What in one sentence, are you proposing to add to PHP here?

Nikita Popov  0:46  
	I would say it's an alternative to magic get and set, just for one specific property instead of all of them. That's the technical side. Maybe I should say something about the like motivation behind it, which is that since PHP seven four, we have type properties, that at least for me personally with that feature, the need to have this typical pattern of private property for storage, plus a public getter and setter methods, the main motivation for that has kind of gone away, because we can now use types to enforce any contracts on value. And now these getter and setter methods most if you like boilerplate. So the idea with accessors, at least my idea with accessors is that you really shouldn't use them. You should just have them as a backup option. You declare a public property in your class, and then maybe later, years later, it turns out that okay, that property actually requires additional validation. And right now if you have a public property, then you don't really have a good way of introducing that. Only way is to either break the API contract by converting the property into getter/setter methods where you can introduce arbitrary code, or by using magic get/set, which is definitely possible and persist the API contract, it's just fairly ugly.

Derick Rethans  2:09  
	You changes the public property that people could read into a private one. And because it's private, the set and get metric methods are being called. 

Nikita Popov  2:18  
	Exactly. 

Derick Rethans  2:19  
	This RFC is titled Property accesses, how do these improve on the situation?

Nikita Popov  2:24  
	So I think there are really two fairly orthogonal parts to this RFC. The first part is implicit accesses that don't have any custom behaviour, and just allow controlling the behaviour of properties a bit more precisely. In particular, the most important part is probably the asymmetric visibility, where you have a property that's publicly readable, but can only be set from within the class. So public read/ private write. I think that's a, maybe the most common requirement. The second part is where you can actually introduce some custom behaviour. So where you can say that okay, the get behaviour for this property looks like this, and the set behaviour, it looks like this. Which is essentially exactly the same as what magic get/set does, just for a single property.

Derick Rethans  3:10  
	For example, when you then do set, or you can add additional validation to it.

Nikita Popov  3:14  
	Exactly. Originally, you had a simple public property, then you can add a setter that checks okay this string cannot be empty.

Derick Rethans  3:23  
	Okay, what it's the syntax that you're proposing?

Nikita Popov  3:26  
	I went with these essentially the same syntax that's being used in C#. Looks like you write public foobar, and then you have this sort of semi colon you have a code block. And this code block contains two accessors, so then you have something like get, and another code block that specifies the get behaviour, and set, and the code block that specifies the set behaviour and so on.

Derick Rethans  3:52  
	The RFC talks about implicit and explicit implementations of these getter and setter accessors. What is the difference between them and how does it look different in syntax?

Nikita Popov  4:03  
	Yeah so the difference is, either you can write just get semi colon, set semicolon, that's an implicit implementation, or you actually specify a code block with real custom behaviour. To do the implicit implementation, you're saying that this is really a normal property, and PHP automatically manages the storage for you, is that you have this more fine grained control over how it works. Namely what you can do is you can say that you have get and private set. But that's a property that's publicly read only and internally writeable. You can write just get without set, in which case it's a real read only property both publicly and privately, or to be more precise, it's an init once property so you can assign to it once.

Derick Rethans  4:52  
	How do you keep track of the init once?

Nikita Popov  4:53  
	It's same mechanism as for Type Properties, where we distinguish between an initialized and an uninitialized property. You can assign to an uninitialized property, but you can't assign to an initialized one, if it's read only. The only maybe problem there is that this mechanism, requires that the property actually is uninitialized to start with, which means that for accessors you don't have any default values. To say there is no implicit default value, no implicit null value. If you want to have a default value the same as with type properties you have to specify it explicitly. Specifying a default value really only makes sense if the property is both readable and writable. For Read Only properties, if you specify the default then you will you can change that.

Derick Rethans  5:37  
	You have basically have created a constant.

Nikita Popov  5:39  
	Yes, it is essentially a constant.

Derick Rethans  5:41  
	You mentioned already, PHP seven four introduced type properties. How do these types interact with the setter and getter accessors?

Nikita Popov  5:50  
	I would say in the obvious way. The getter is required to return type of property, modulo the usual weak typing conversions, and the setter also checks before it's called whether the passed value matches the type or not. But enforces that matches the type.

Derick Rethans  6:08  
	This does mean that if you provide an explicit implementation for the set accessor, you also need to specify the parameter name?

Nikita Popov  6:15  
	No, or you can specify the parameter name, and if you don't then that's just passed in as the value variable. It's also inspired by how C# and Swift do it. I mean there are some possible variations here we could always require an explicit name, some people for that, or I also heard that some people would like to have the name of this implicit variable match the name of the property, instead of always being just value.

Derick Rethans  6:41  
	Would you have to specify the type though?

Nikita Popov  6:43  
	You wouldn't have to and you're actually not allowed to. So the accessor implementation is somewhat strict about not allowing you to do anything that would be redundant because otherwise, you know, there are quite a lot of extra things you could be adding everywhere.

Derick Rethans  6:56  
	That's the same way as marking a property as private. And then the accessors as private as well. Right?

Nikita Popov  7:03  
	Yeah exactly. So, then that will also say: if the property is already private you can't, again say that the accessors also private.

Derick Rethans  7:11  
	I think that's the wise thing, otherwise people go overboard with adding private and final and whatever everywhere anyway right.

Nikita Popov  7:18  
	One could argue that it's really not our business and this is a coding style question, but you know it's better to not leave people, with the option of doing stupid things.

Derick Rethans  7:28  
	I saw in the RFC that it is also possible to use references with the get accessor.  Does this complicated implementation and the idea of this RFC, a lot, or just a little?

Nikita Popov  7:39  
	I think the important context to keep in mind here is that we already have magic get set, and the accessors are, like, largely based on their semantics. Magic getters already have this distinction between returning by value and returning by reference. The by reference return value is primarily useful for two cases. One and this is really the important one, is if you're working with arrays, any write operation on an array like setting an element or appending an element, those require that the getter returns by reference, because PHP will actually do the modification on the reference. Because some people asked about that. Why can't we just like get the array using the getter, then make the change and then assign back using the setter. That would theoretically work, but it would be extremely inefficient, and the reason is that this breaks PHP's copy on write mechanism. If the error is returned from the getter, then we have one array inside the property. And we have one copy of the array inside the property, and as the return value. Then we change the return value and the resource is now shared, we actually have to copy the whole array, and then we assign it back. So effectively what we do is we copy the array, we do single element change, and then we copy the array, we do a single element change and then we destroy the old array. That works in theory, but it's so inefficient that we would not want to promote this kind of usage.

Derick Rethans  8:42  
	The way around is of course, is having an implicit methods on the class to make this change to the array itself right?

Nikita Popov  9:10  
	That would be another option. Problem is that you will need a lot of methods, I mean it's not just a matter of setting a single element or unsetting an element, but you can also set like a deep element where you're not modifying the outermost array but, like, a multi dimensional array. You would actually have to pass through that information somehow as well. I don't think there is a simple solution to that problem beyond the reference based solution that we currently use.

Derick Rethans  9:34  
	I saw people arguing about not bothering with references in this new implementation at all, but I think you've now made a good case for keeping them.

Nikita Popov  9:42  
	Effectively not bothering with references just means not supporting that array use case. Which might be, maybe a reasonable limitation, especially if we like make a distinction and supported for the implicit accessor case where we can, you know, do internal magic to support that and not support it in the explicit accessors case. I mean, people were arguing that this reduces the complexity of the proposal, but it kind of also increases the complexity because now we are doing something else for the accessors and we're doing for the magic get/set, where we already have this established mechanism. I'm not really convinced by that.

Derick Rethans  10:20  
	And I also think it creates inconsistencies in the language itself because it does something different with an implicit or explicit accessor, as well as it being different between the original underscore underscore get magic method as well.

Nikita Popov  10:34  
	It's not a secret that I'm not a big fan of references, and I would certainly love to get rid of them, but it's a hard problem, and this array modification behaviour for magic get or for get accessors is certainly a large part of that problem, and I just don't have a good solution for it.

Derick Rethans  10:52  
	I don't either. The RFC also goes into great detail about inheritance and variance. Would you have a few words on that?

Nikita Popov  11:00  
	I think mostly inheritance works like inheritance does for methods, at least that's how it's supposed to work. Of course there are some interactions, because you can for example mix real properties and accessor properties. In which case, if you have parent accessor property, you can always replace it with a normal simple property, because normal properties they support all operations that accessor properties do. What you can't do is the other way around. If you have a parent normal property, then you can't replace that with an accessor property. And reason is that it does have some limitations. Not a lot, but there are some limitations. One of them is related to references, I mean, we're already talking about this topic. What the by reference get allows is taking a reference to the property, so you can do something like a reference equals the property. What you can't do is the other way around the property reference equals something else. So you can't assign a new reference into the property, that just doesn't work on a pretty fundamental level, because it would require an additional set handler for set by reference. As we don't particularly love references, adding a new mechanism to support that is not a very popular choice.

Derick Rethans  12:20  
	Variance wise, I guess, the same rules apply as for normal properties and property types?

Nikita Popov  12:27  
	Approximately. Properties are apparently invariant, so you can't change the type or I mean you can change it but it has to be an equivalent type. If you have a read only property, with only a getter, then the implementation makes the type covariant, which means you can use a smaller type in the child class. This is similar to how if you have a getter method, you could also give it a smaller type in the child class. The converse case, if you have a property that can only be set, then the type is contravariant, you can have larger type in the child class, though I should say that properties that can only be set are somewhat odd and really only supported for the sake of completeness, so maybe it might be worthwhile to drop the type specific behaviour there, because a set only property should already be really rare, and then set property with a contravariant inheritance that's like a edge case of an edge case.

Derick Rethans  13:24  
	Would it even make sense to support set only properties?

Nikita Popov  13:27  
	Not sure. So for the C#, implementation, I think they don't support this and there is a StackOverflow question about that, and people try to convince their, that they should support this, that the are really use cases. Currently the imagined use cases are along the lines of injecting values into a class, so using setter injection, just that now it's property based setter injection. Okay, I'll be honest I think it doesn't make sense.

Derick Rethans  13:55  
	To be fair, I don't think either. It would reduce the length of the RFC a little bit. 

Nikita Popov  14:00  
	A little bit, yes. 

Derick Rethans  14:01  
	Can you say a few words about abstracts, traits, private accessors shadowing and things like that. So a lot of complicated words, maybe you, you can distil that into something slightly simpler.

Nikita Popov  14:12  
	Well I think actually abstract properties are worth mentioning. In particular, the fact that you can now specify properties inside interfaces. If you have public properties, then it makes sense to have them really on the same level as public methods, so they are part of the API contract, and as such should also be supported in interfaces. Typically what the RFC allows is, you can't specify a simple property in the interface, but you can specify an accessor property, which tells you which operations have to be supported. So you can't have a property declaration that says, it just has a get accessor, or it has get and set. The implementation of course can always implement more, so if the interface requires get, then you can implement both get and set, but it has to implement at least get, either through an accessor offer another property. I think in most cases implementation will just be a normal property.

Derick Rethans  15:03  
	Because a normal property would implement an implicit get already anyway?

Nikita Popov  15:07  
	Yeah. 

Derick Rethans  15:08  
	How do property accessors tie in, or integrate with constructor property promotion? 

Nikita Popov  15:13  
	They are supported and promotion with the limitation that it's only implicit accessors. If you use constructor promotion, then you can specify your read only property in there, or property that is Public Read/ private write. You cannot specify a property with complex behaviour in there. This is mainly because it would mean that you embed large code blocks into the constructor signature, which is I think, pushing the limits of shorthand syntax, a bit. Like there is nothing fundamental that will prevent it, it's more a question of style. 

Derick Rethans  15:50  
	The RFC talks a little bit about how, or rather what happens if you use foreach, var_dump, or an array cast on properties with explicit accessor. What are the restrictions here? Is something chasing from normal standard properties like we currently have.

Nikita Popov  16:03  
	I don't think so. So here is once again the case where we have this distinction between the implicit accessors, which are really just normal properties with limitations. So those show up in var_dump and array cast, foreach, as usual. And we have explicit accessors, which are really virtual properties, so they don't have any storage themselves. Any storage to use, you have to manage separately somehow. So, these don't show up in var_dump, foreach, and so on. Both these actual computed properties, they don't show up because that would require us to actually call all the accessors if you do foreach and that seems rather dubious to me.

Derick Rethans  16:44  
	How this will work for internal API's that some extensions use to access, like a list of all the properties, for say, for a debugger.

Nikita Popov  16:51  
	It'll work the same way as var_dump. I mean, in the end it's all, well it's not quite based on the same API's, but still, the answer is the same. You only get those properties that have some kind of backing storage, and those are only the ones that are either normal properties, or the ones with implicit accessors.

Derick Rethans  17:09  
	That means I need to go find out a way how to be able to read the ones with explicit accessors.

Nikita Popov  17:14  
	Yeah, if you want to. I don't think that the debugger should read those by default, because that means that doing a dump, will have side effects, which is not ideal, but maybe you want to have an option to show them.

Derick Rethans  17:26  
	That's something for me to think about, because I'm pretty sure people are going to want to see the contents of these properties, even in a debugger, even though that could mean that are side effects, which I'm not keen on.

Nikita Popov  17:36  
	I guess that's one of the, I would say advantages of using this over just magic get/set, because actually know which properties you're supposed to look at, with for magic get/set you just don't know at all.

Derick Rethans  17:51  
	The RFC talks a little bit about the performance impacts and although I saw the numbers I didn't actually read them, when preparing for this recording. What are the performance impacts for implicit accessors as well as explicit ones?

Nikita Popov  18:02  
	Impact is basically if you use implicit accessors that has similar performance to plain properties, performance is a bit worse. The reason is essentially that we have some limitations on caching. So we can't just cache it as if it were a normal property, because it could have asymmetric visibility. And we reuse the same cache slots for reads and writes. I've been thinking about maybe splitting that up but at least for now there is a small additional performance impact of using implicit accessors, but it's not really significant. On the other side if you use explicit accessors. Those are expensive, they are not quite as expensive as using magic get/set, but they are more expensive than using normal method calls. Reason is basically their normal method calls, they are very optimized, and they do not have to re enter the virtual machine, so we just stay in the same virtual machine loop, and we just switch to different stack frames. For magic get/set we actually have to like recursively call the virtual machine, because we don't have a good point to re enter it, at least based on our current API's. And we also have to deal with some additional stuff, particularly the fact that magic get/set and property accessor as both, they have recursion guards. Normally if you recurse methods in PHP, we don't do any checks about that. Xdebug does, but PHP itself doesn't, so you can infinitely recurse and PHP is fine. The only thing that happens is that at some point you'll run out of memory.

Derick Rethans  19:37  
	Or when extensions are loaded such as Xdebug, you'll actually still get a stack overflow.

Nikita Popov  19:41  
	So that's something we should still be addressing, at least the baseline behaviour that you can get to that memory limit error. For properties will set have recursion guards, which say that if you recursively access a property in magic get/set, that it will not call magic get/set again and instead, access the property as if they didn't exist.

Derick Rethans  20:01  
	Instead of throwing in an error?

Nikita Popov  20:03  
	Yeah. For property accessors I'm actually throwing an error on recursion, and the reason for that is if we didn't throw an error, then this would end up accessing dynamic property of the same name as the accessor, which would technically work, but it's very likely not what the programmer actually intended. So it's going to be really inefficient because you actually have to allocate space for the dynamic properties and access for those. So if you wanted to have some kind of backing storage for the property, then you should just explicitly declare it and access that, rather than accessing something with the same name and implicitly creating a dynamic property.

Derick Rethans  20:41  
	Yeah, that sounds all very complicated. 

Nikita Popov  20:44  
	It's cleaner to just make it an error and let the programmer fix it, instead of PHP try to fix it for you.

Derick Rethans  20:51  
	Are there any BC considerations about the introduction of property accessors?

Nikita Popov  20:55  
	Not strictly, but I'm sure that it's going to break, various assumptions for people, or at least in the sense that, right now, most assumptions should already be broken through magic get/set. I mean you can always have this kind of magic behaviour. If we have accessors this is probably going to be a lot more common, and people will have to deal with things like properties being publicly readable, but privately writable much as because someone very rarely manually implements that behaviour, but because the language though has native support for it and it's going to be common.

Derick Rethans  21:28  
	We spoke a little bit about all the different sticking points in his RFC, for example with references, but there's one other thing and I think it's an argument you make somewhere on the bottom of the RFC, that there is a separation between implicit and explicit property accessors. I'm wondering whether it would make sense to consider adding whether the implicit part of this RFC first and then maybe later look at adding explicit property accessors.

Nikita Popov  21:54  
	That's really the main sticking point, and also my own problem with the RFC. I mean, you mentioned at the very start, that this is a very long RFC and still a little bit incomplete so it's going to be longer. It's a fairly complex feature that has complex interactions with other features in PHP. The implementation is actually, maybe less complex, then you think, given the RFC length. The main concern I have is that, at least for me personally the most useful part of the RFC, are the read only properties. The read only properties and the like Public Read, Private Write properties. I think these two cover like 90% of the use cases, especially because if you have a property that is only publicly readable, then you don't really have to be concerned about this case where you have to, later on, add additional validation. I mean after all the property is read only, or you control all the sets because they're private. There is no danger of introducing an API break, because you have to add additional validation. I think like the largest part of the use case of the whole accessors proposal will be covered by these two things, Or maybe even just one of these two things, that's a bit of a philosophical question. There are some people who think we should have just public read / private write and no proper read only properties, because that like looks the same from the user perspective, but still gives you more flexibility. I think that's like the most important use case, and we could implement that part with a lot less language complexity. So the question is really does it make sense to have this full accessor proposal, if we could get the most useful part as a separate simpler feature, and, well, I heard differing opinions on that one. I was actually pretty surprised that their reception of the on like a full accessors proposal was fairly positive. I kind of expected more pushback, especially as, this is the second proposal on the topic, we had earlier one with, like, similar syntax even though different details, and that one did fail.

Derick Rethans  24:02  
	How long ago was that?

Nikita Popov  24:03  
	Oh that was quite a while actually, at least more than five years.

Derick Rethans  24:06  
	I think that the mindset of developers has changed in the last five to 10 years, like introducing this 10 years ago would never happened, or even typed properties, right. It would never have happened. 

Nikita Popov  24:17  
	That's true. 

Derick Rethans  24:19  
	Do you have any idea when you're going to put us up for a vote? Because, of course, PHP 8.1 feature freezes coming up in not too far away from now. 

Nikita Popov  24:28  
	Yeah, I'm not sure about that. I'm still considering if I want to explore the simpler alternatives, first. There was already a proposal. Another rejected proposal for Read Only properties, probably was called write once properties at the time. But yeah, I kind of do think that it might make sense to try something like that, again, before going to the full accessors proposal or instead. 

Derick Rethans  24:54  
	Do you have anything else to add?

Nikita Popov  24:56  
	What are your thoughts on this proposal, and the question at the end?

Derick Rethans  24:59  
	I quite like it, but I also think that it might make sense to split it up. I'm always quite a fan of splitting things up in smaller bits, if that's possible too, and still provide quite a lot of use out of it. And that's why I was asking whether it makes sense to split it up into the implicit part and the explicit part of it, especially because it makes the implementation and the logic around it quite a bit easier for people to understand as well.

Nikita Popov  25:24  
	It's maybe worth mentioning that Swift also has a similar accessor model but it is more like a composition of various different features like read only properties, properties with asymmetric visibility, and then finally properties with like fully controlled, get and set behaviour rather than this C# model where everything is modelled using accessors with appropriate modifiers. So there is certainly precedent in other languages of separating these things.

Derick Rethans  25:55  
	Something to ponder about, and I'm sure we'll get to a conclusion at some point. Hopefully some of it before PHP eight one goes and feature freeze, of course. We've been chatting for quite a while now, I think we should call it the end for this RFC. Thank you for taking the time today to talk about property accessors.

Nikita Popov  26:11  
	Thanks for having me, Derick.

Derick Rethans  26:12  
	Thank you for listening to this installment of PHP internals news, a podcast, dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening and I'll see you next time.


Show Notes
----------

- RFC: `Property Accessors <https://wiki.php.net/rfc/property_accessors>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
