PHP Internals News: Episode 75: Globals, and Phasing Out Serializable
=====================================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-02-11 09:03 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin075
   :Enclosure: media/pin-075.mp3

In this episode of "PHP Internals News" I chat with Nikita Popov (`Twitter
<https://twitter.com/nikita_ppv>`_, `GitHub <https://github.com/nikic/>`_,
`Website <https://nikic.github.io/>`_) about two RFCs: Restrict Globals Usage,
and Phase Out Serializable.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-075.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:14  
	Hi I'm Derick. Welcome to PHP internals news, a podcast dedicated to explain the latest developments in the PHP language. This is Episode 75. In this episode, I'm talking with Nikita Popov about a few RFCs that he has been working on over the past few months. Nikita, would you please introduce yourself?

Nikita Popov  0:34  
	Hi, I'm Nikita, I work at JetBrains on PHP core development and as such I get to occasionally, write PHP proposals RFCs and then talk with Derick about them.

Derick Rethans  0:47  
	The main idea behind you working on RFCs is that PHP gets new features not, you end up talking to me.

Nikita Popov  0:53  
	I mean that's a side benefit,

Derick Rethans  0:55  
	In any case we have a few to go this time. The first RFC is titled phasing out Serializable, it's a fairly small RFC. What is it about?

Nikita Popov  1:04  
	That finishes up a bit of work from PHP 7.4, where we introduced a new serialization mechanism, actually the third one, we have. So we have a bit too many of them, and this removes the most problematic one.

Derick Rethans  1:19  
	Which three Serializable methods or ways of doing things currently exist?

Nikita Popov  1:24  
	The first one, which doesn't really count is just what you get if you don't do anything, so just all the Object Properties get serialized, and also unserialized, and then we have a number of hooks, you can use to modify that. The first pair is sleep and wake up. Sleep specifies which properties you want to serialize so you can filter out some of them, and wake up allows you to run some code, after unserialization, so you can do some kind of fix up afterwards.

Derick Rethans  1:52  
	From what I remember, if you use unserialize, where does the wake up the constructor doesn't get called?

Nikita Popov  1:59  
	During unserialization the constructor, never gets called.

Derick Rethans  2:03  
	So wake up a sort of the static factory methods to re rehydrate the objects. 

Nikita Popov  2:08  
	Exactly. 

Derick Rethans  2:08  
	So that's number one,

Nikita Popov  2:10  
	Then number two is the Serializable interface, which gives you more control. Namely, you have to actually like return the serialized representation of your object. How it looks like is completely unspecified, you could return whatever you want, though, in practice, what people actually do is to recursively call serialize. And then on the other side when unserializing you usually do the same so you call unserialize on the stream you receive, and then populate your properties based on that. The problem with this mechanism is exactly this recursive serialization call, because it has to share state, with the main serialization. And the reason for that is that, well PHP has objects, or object identity. So if you use the same object in two places you really want it to be the same object and not two objects with the same content. Serializable has to be able to preserve that, and that requires that it runs in the middle of the unserialization.

Derick Rethans  3:14  
	Not sure if I follow that bit.

Nikita Popov  3:16  
	Well maybe it's not a hard requirement more like an issue with our serialization format that comes into play here. Way PHP implements this, is using back references. So at first unserializes an object and then later you can have like a pointer back to it, that says like, I want to use the same object as at position number, 10, or so. For these back references to work, we have to actually execute the serialization handler while unserializing because otherwise the offsets will no longer match. So we can just run this at the end of unserialization for example because then our offsets would be incorrect. And this is a big problem because it's not really safe to run code, during unserialization because things are partially initialized. To make these back references work, PHP has to actually store pointers to these objects. And if you somehow modify things in specific ways, then these pointers become invalid. They point to a memory that no longer exists, and a possibly exploitable crash. This is why we would like to get rid of this mechanism.

Derick Rethans  4:25  
	But of course, in order to get rid of things, we had to have a better way of doing things in place first, right, which came with PHP seven four. 

Nikita Popov  4:32  
	That's right. 

Derick Rethans  4:32  
	So that's number three. 

Nikita Popov  4:34  
	That's number three. Number three is actually very similar to number one: two new magic methods, double underscore serialize and double underscore unserialize. Serialize returns an array, usually like an array of properties for example, and then unserialize populates the object from that array. In practice, this works very similar to the Serializable interface, just that you don't manually call serialize and unserialize, but PHP will do so on your behalf. So you just return an array or get an array, and PHP will integrate that into the like main serialization, and because it's left to PHP, PHP can control where these calls occur.

Derick Rethans  5:19  
	With sleep originally you only return the name of the properties. Whereas with this new interface you return the names of the properties but also their values.

Nikita Popov  5:30  
	That's right. The new mechanism, this, like, in practice, it serves as a replacement for the Serializable interface. But from a technical side it's really close to sleep and wake up, um, just that, as you said, instead of returning property names you return both names and values.

Derick Rethans  5:51  
	And this is now the recommended way of doing serialization.

Nikita Popov  5:54  
	Like the motivation is one problem was, what I mentioned the security problem. Maybe the thing that impacts users more commonly is that things like calling parent::serialize and parent::unserialize with the Serializable interface, usually doesn't do what you want. Again, due to these back references because, like, the calls get out of order, we should do the same thing with the magic methods, with the underscore underscore serialize and unserialize and you can safely call parent methods and compose serialization in that way.

Derick Rethans  6:29  
	That's our state of serialization right now. We haven't spoken about RFC, what are you proposing to do here?

Nikita Popov  6:34  
	The RFC proposes to get rid of the Serializable interface. And, like in a way that is a bit more graceful than just deprecating it outright. And the idea is that if you have code that is still compatible with PHP 7.3, where the new mechanism doesn't exist, you probably still want to use Serializable. So if we just deprecated out right that would be fairly annoying to have code that's compatible with PHP 7.3, and 8.1. So instead what we do is we only deprecate the case where you implement Serializable without implementing the new mechanism. If you implement both of them, then you're fine for now.

Derick Rethans  7:15  
	The new mechanism, the one we're introducing PHP 7.4, would overrides the PHP 7.3 one already anyway.

Nikita Popov  7:22  
	Exactly. So on PHP 7.3 you would end up using Serializable and PHP seven four and higher, you would be using the new mechanism. And then, at a later point in time we would actually also deprecate Serializable itself and then remove it, though, like based on mailing list response, some people at least didn't like the long timeline. I'm not exactly sure what the alternative is, so either to deprecate Serializable right away, or to later remove it without deprecation of the interface itself.

Derick Rethans  7:57  
	Yeah, from what I saw the, the long-term-ness of phasing it out. I think had mentioned that it finally got removed in PHP 10, which is potentially 10 years away right. If we following every five years with a new major release. But then in the end, it does have some merit making sure that people can move on without being left in the dark at some point right. What is your own preference?

Nikita Popov  8:22  
	My own preference is what I proposed. I would also be fine with, like say in PHP 8.1, we call the proposal so you only get a warning if you only implement Serializable without the new mechanism, and the PHP nine we could just drop Serializable entirely. I think that would not be, because then the only problem then would be if you have code that is competitive with PHP 7.3 and PHP 9.0. I am sure that code will exist ... pretty normal version range to have. 

Derick Rethans  9:08  
	Yeah, I probably would agree with you there. When I read the RFC it also mentioned PDO. Why would it mention PDO?

Nikita Popov  9:15  
	This all is something I only found out while writing it's on there is a PDO fetch serialize flag, which automatically calls unserialize when fetching values. So I will not comment on the really dubious idea of storing serialized data in the database.

Derick Rethans  9:35  
	I mean, people would currently said that the alternative is to store JSON, in these columns as values. 

Nikita Popov  9:40  
	That would still be better. 

Derick Rethans  9:42  
	But it's still a serialized format?

Nikita Popov  9:44  
	But at least the way this flag is implemented is effectively broken, because it doesn't just call unserialize, the function; it calls unserialize on the Serializable interface. I have no idea how this was intended to be used in practice, because it's not compatible with, like the normal serialization of the class. In practise like everything I have found about this online is basically just that okay if this functionality is broken, you shouldn't use it.

Derick Rethans  10:15  
	So you have less concerns just removing that straight away, I suppose. 

Nikita Popov  10:19  
	Yeah. 

Derick Rethans  10:20  
	Do you have anything else out about serialization.

Nikita Popov  10:22  
	I think this proposal is a very simple one and we have actually talked, way too much about this.

Derick Rethans  10:29  
	Let's move on to the next RFC, which is titled Restrict Globals Usage. This title almost sounds worse than it is as it might imply that you want to get rid of the globals array altogether. But I bet that's not the case. And I also suspect that restricting the globals array is a lot more technical as a subject as it might seem.

Nikita Popov  10:49  
	That's right. So this is really, mostly motivated by internal concerns, and has hopefully not a great deal of impact on like practical usage. There are a couple motivations, so some of them are about semantics, so globals is a very magic variable, that does not follow the usual semantics of PHP a number of ways. In particular array are typically by value. In all other cases, they are by value, which means that if you modify, like if you copy an array and modify one copy, then the other one doesn't get modified, I mean it's a copy so obviously it doesn't get modified. For globals if that's not the case. If you make a copy of globals and you modify the copy, then the original array also gets modified.

Derick Rethans  11:36  
	Which is not the case for other super globals such as underscore get and underscore post.

Nikita Popov  11:41  
	The other super globals are a bit magic but not that magic. There are a couple of other concerns with edge cases, but I think the real motivation here is the internal concern. And that's how globals is implemented. PHP, normally, manages variables in functions and scripts, using so called compiled variables. And this works by well when the script is compiled we actually see all the variables with the used, at least all the variables that don't go through something like variable variables or globals or something like that. And we reserve a slot for each of these variables, so we can directly access it. We don't have to look up, like the variable by name, we just say this is variable number seven and we can directly access it, which is much much more efficient. The problem is, then if you have something that globals you want to both have this access by index, and access by name, and they do that by storing a pointer inside the globals array to the actual location of the variable. Yeah, so this is a very special concept. So we call this an indirect, a variable of indirect type, and it essentially occurs only inside the globals array, and for object properties. For object properties it happens for the same reason, so object properties are normally accessed by index, but if you do something like variable object dynamic object access, then we also have to look it up by name. There we do the same thing, so we have a like map from property names to values, and if the value is really stored inside an object property slot then we just store a pointer there. The thing with the objects is that this is like really an internal concern that's well encapsulated and doesn't leak into normal PHP code. That's not the case with globals because globals is on the surface just a normal array. So you can do everything with it, you do with a normal array you can pass it to functions. Like in theory, all the functions, need to deal with this special value type that says: okay actually this is not the value itself is just a pointer to the value. The way you do it is every time you access a value you check okay is this an indirect value; if it is, follow the pointer.

Derick Rethans  14:01  
	I have plenty of code in Xdebug for this.

Nikita Popov  14:04  
	So it's really a super simple operation to do, but you actually have to do it. And you have to do it absolutely everywhere, if you're being pedantic. In practice that just doesn't happen. In PHP's own code, in the standard library, the array functions are those do consistently handle this edge case. But if you like go further, even most bundled extensions, and certainly most third party extensions, they are not going to do this and if they don't either they just get some, like you know benign misbehaviour where it looks like array elements are missing, or you get a crash, because the type is simply not handled. Yeah, well that's not a great state to be in, because like pushing passing the globals array into something like array pop or something, is very weird operation to do. I don't know if ever, anyone has done that for purposes outside testing PHP. But to support it, we have to like handle this special case everywhere, which is not robust and also has a certain performance impact when it comes to low level operations. So we also have to do this check every time you access an array for example from normal PHP code The idea is to remove the special case. That's the motivation here.

Derick Rethans  15:23  
	What are you proposing to change?

Nikita Popov  15:26  
	One is if you just access variable in globals. So you write $GLOBALS[], some variable name. Then we treat that especially and compile it down to an access to this global variable. So it could be a read access, could be a write access, or anything else,

Derick Rethans  15:44  
	But it is something that happens, when PHP compiles scripts.

Nikita Popov  15:48  
	That's right. The second part is you can also access the globals array in a read-only way, so you can take the whole array, and for example, do a for each loop over it. And that continues to work. The part that doesn't work is to take the whole globals array and modify it in some way, for example, passing globals to array pop, which requires passing it by reference is going to throw an error.

Derick Rethans  16:13  
	At which state. Is that going to throw an error?

Nikita Popov  16:15  
	That's usually during compilation, but specifically for the case of by-reference passing it can't be detected at runtime, because we don't always know if it's a by-reference or by-value pass. But for most of the cases it's a compile time error. Maybe one particular case that's worth mentioning is that you also can do a foreach by-reference over it. So if you like want to loop over globals and modify entries while doing so the way to do it now would be to do by-value loop and then just again access specific elements in it, like access globals key or something. And the reason why this helps us is that we can just return, like when you access globals, we can actually return a copy of the array. We don't have to maintain these like indirect pointers which are only necessary to support modifications, we can just return a copy. That means we no longer have to deal with this edge case in most places, in the engine and in third party extensions,

Derick Rethans  17:15  
	Talking about third party extensions, the code that implements this RFC has already been merged into PHP eight one, but the moment you did that, tests in Xdebug started failing, because I read the globals array, but it doesn't seem like it exists any more now.

Nikita Popov  17:31  
	That's actually a good point. Globals, I would know view it as a like, more like a syntax construct, similar to variable variables, or even the $this variable. So this is also not a real variable. Globals is no longer added as an actual variable in the symbol table, which is directly compiled down to either an access to the specific global or returns a copy of the table. So for Xdebug you, I probably filter you you have to access the EG symbol table.

Derick Rethans  18:02  
	Yes, but it wasn't as simple as it seemed because this is a hash table, and no longer is that a full array, which means that all my logic code doesn't work with that. So I've decided that globals just no longer exists and stuff, which is what it logically is in PHP eight one anyway.

Nikita Popov  18:22  
	So that might actually be nice. So I know that, like code that does work with globals, like as an array, usually also always skips skips globals itself when iterating over it, because otherwise you usually run into some kind of infinite recursion issue. That's actually another thing, so globals is the one way you can have a recursive array, without references being involved. So I know that the Symfony like variable/cloner dumper. That goes for a lot of effort to detect cycles, like has some extra fun hacks to detect globals correctly for that reason, because usually you just take references but for globals that doesn't work. 

Derick Rethans  19:09  
	Right, how much of an impact is this going to have to existing code?

Nikita Popov  19:12  
	So I like analysed the top composer packages and found, not a lot of usages. I don't remember the exact number, it was maybe five cases that break. That's not to say that it has no impact. I do know that PHPUnit eight point whatever, had such a globals use, which was fixed already because Sebastian Bergmann now, adds support for new PHP versions to PHPUnit eight and nine both. If you're using PHPUnit seven, then probably, it's no longer going to work for that reason. Of course, it also doesn't work for many other reasons, as well. Depending on which features to use, but I do know that you know sometimes if you're not using mocks, then you can often use old PHPUnit versions, but I think that's no longer going to work in this case.

Derick Rethans  20:04  
	It's something that users of PHP and PHPUnit, probably should start testing once the alpha and beta releases of PHP eight one start happening.

Nikita Popov  20:16  
	Right. I mean, I hope that it's not going to be a big issue. After all, this is minor PHP version. So we really shouldn't be introducing bad breaks, but at least the usage I've seen in open source project suggests that it should not be a big problem.

Derick Rethans  20:33  
	Excellent. As I've mentioned this RFC is already been merged. So I don't really have to ask about feedback, because it's irrelevant right now. It's already there. 

Nikita Popov  20:44  
	Well, you could still have feedback afterwards.

Derick Rethans  20:48  
	Thank you, Nikita for taking the time to explain these several RFCs to me today.

Nikita Popov  20:52  
	Thanks for having me Derick.

Derick Rethans  20:57  
	Thank you for listening to this instalment of PHP internals news, a podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next time.


Show Notes
----------

- RFC: `Restrict Globals Usage <https://wiki.php.net/rfc/restrict_globals_usage>`_
- RFC: `Phase Out Serializable <https://wiki.php.net/rfc/phase_out_serializable>`_


Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
