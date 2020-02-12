PHP Internals News: Episode 38: Preloading and WeakMaps
=======================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-01-30 09:01 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin038
   :Enclosure: media/pin-038.mp3

In this episode of "PHP Internals News" I chat with Nikita Popov (`Twitter
<https://twitter.com/nikita_ppv>`_, `GitHub <https://github.com/nikic/>`_,
`Website <https://nikic.github.io/>`_)
about PHP 7.4 preloading mishaps, and his WeakMaps RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-038.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Show Notes
----------

- RFC: `WeakMaps <https://wiki.php.net/rfc/weak_maps>`_

Transcript
----------

Derick Rethans  0:16
	Hi, I'm Derick. And this is PHP internals news, a weeklish podcast dedicated to demystifying the development of the PHP language. This is Episode 38. I'm talking with Nikita Popov about a few things that have happened over the holidays. Nikita, How were your holidays?

Nikita Popov  0:34
	My holidays days were great.

Derick Rethans  0:36
	I thought I'd start with something else then I did last year. In any case, and wanting to talk to you this morning about something that happens to PHP seven four over the holidays. And that is issues with preloading on Windows with PHP seven four. I have no idea what the problem is here. Would you try to explain this to me?

Nikita Popov  0:56
	So there were actually quite a few issues with preloading in early PHP 7.4 releases. The feature definitely did not get enough testing. Most of the issues have been fixed in 7.4.2. But if you're using preload-user, what you have to use if you're running on the root, then you will probably still see crashes and that's going to be fixed in the next release.

Derick Rethans  1:20
	In 7.4.3.

Nikita Popov  1:22
	Right. But to get back to Windows, Windows has a well very different process architecture than Linux. In particular, on Linux, or BSD we have fork. Which basically just takes a process and copies its entire memory state to create a new process. This is a lot cheaper than it sounds because it's all like reuses memory until it's actually changed.

Derick Rethans  1:48
	Its copy on write.

Nikita Popov  1:49
	Copy on write exactly. The same functionality does not exist on Windows, or at least it's not publicly exposed. So on Windows, you can only create new processes from scratch, that look, we use our memory from the previous one. And for OPcache, this is a problem because OPcache would really like to reference internal classes as defined by PHP. But because we store things in shared memory, which is shared between multiple processes, we now have the problem that these internal classes can reside at different addresses, in these different processes. On Linux, it's always going to be the same address because we are forking and that keeps the address. On Windows each process could have a different address. And especially because Windows since I think Windows Vista, uses address space layout randomization. This is actually pretty much always going to be a different address.

Derick Rethans  2:51
	Because that's a security feature?

Nikita Popov  2:52
	Exactly. It's a security feature.

Derick Rethans  2:54
	Would it also be a problem on Linux if you'd start a process instead of forking it?

Nikita Popov  2:59
	Yes, it would be a problem. The difference's just that on on Unix, we don't do that. OPcache has quite a different architecture on Windows. On Linux, we do not allow to attach to an existing OPcache from a separate process. So the only way to share an OPcache is to use fork. On Windows because of this restriction that we don't have fork, we do though this kind of attachments and that's where we have we have to deal with these kind of issues. So that's actually a general problem, not just for preloading on differences, just that normally, we can just: Hey, okay, we do not allow any references to internal classes from shared memory on Windows. It's like a slight hit to optimization, but it's not super important. While we're preloading, we have to link the entire class graph during preloading. And if you have any classes that for example, extend from an internal class, like extend from Exception. Or in some cases, you can just use an internal class as a type hint, then we would not be able to store these kinds of references in shared memory on Windows. And because for preloading, it's pretty much inevitable that you run into the situation you just can't realistically do preloading on Windows,

Derick Rethans  4:18
	Hence, the decision being made just turning it off, instead of trying to end and always failing pretty much.

Nikita Popov  4:24
	Yeah, I mean, it kind of did work before, it just got a bunch of warnings that these classes haven't been preloaded. And if people try that, oh, it's like with a simple example there, we'll see you great, preloading is working. But once they move to their actual complex application that uses internal classes at various points, it turns out that: Actually, no, it doesn't really work in practice. And so the way that we just disabled entirely

Derick Rethans  4:51
	That seems like a reasonable solution to this, do you think at some point this can be fixable in another clever way?

Nikita Popov  4:58
	Well, main way in which can be fixed is to avoid this kind of multi process attachments on Windows. The alternative to having multiple processes is to have multiple threads, which do share an address space. Basically same as fork just with threads then. But that, of course, depends on what kind of web server you're using and what kind of SAPI you're using. And I think nowadays, on Windows on threaded web servers are somewhat more popular than on Linux, it's still not the majority deployment strategy.

Derick Rethans  5:34
	I think it used to be that threaded process models on Windows were a lot more common when PHP just came out for Windows, because it was an ISAPI module which was always threaded. From what I remember the original reason why we had ZTS, in the first place. Yeah, at some point that started moving to PHP FPM kind of models because it didn't use threading and it was, tended to be a lot safer to use it that way.

Nikita Popov  5:57
	Right. I mean, threading has issues in particular because things like locales are per process, not per thread. So processes are usually safer to use

Derick Rethans  6:08
	Anything else interesting that happened that went wrong with a preloading, or do you not want to mention?

Nikita Popov  6:12
	The rest is mostly just that we have two different ways of doing preloading. One is using OPcache compile file, and others using require or include, and the difference between them is that OPcache compile file combines the file but does not executed. In that case, the way we perform preloading is that we first collect all classes and then we, like gradually, link them, actually register them, always making sure that all the dependencies have already be linked. And this is the mode that that I think mostly work well at the release of PHP seven point four. And the other one, they require approach is where we, well require directly executes the code and registers the classes. And in that case, basically, if it turns out that some kind of dependency cannot be preloaded for some reason, we simply have to abort preloading, because we cannot recover from that. This abortion was missing. And it that turns out that, in the end, the way people actually use preloading is using the require approach, not using the OPcache compile file approach.

Derick Rethans  7:26
	Although that's the one you see most of the examples that I've seen, and in the documentation.

Nikita Popov  7:30
	Right, it has some advantages you some require.

Derick Rethans  7:34
	Something else that happened over the holidays is that you've worked on several RFCs there're too many to talk about at all in this episode. But one of the earlier ones, was a WeakMap, or WeakMaps RFC, which sort of builds on top of the weak references that we already got in PHP seven four. What's wrong with the weak references, and why do we now need weak maps?

Nikita Popov  7:58
	There's nothing wrong with weak references. As a reminder what weak references are both, they allow you to reference an object without preventing it from being garbage collected. So if the object is unset, then you're just left with a dangling reference. And if you try to access it, you get back knowledge of the object. Now, the probably most common use case for any kind of weak data structure is a map or an associative array, where you have objects and want to associate some kind of data with them. Typical use cases are caches or other memoise data structures. And the reason why it's important for this to be weak is that you do not well, if you want to cache some data with the object, and then nobody else is using that object. You don't really want to keep around that cache data because no one has ever going to use it again. And it's just going to take up memory usage. And this is what the weak map does. So you use objects as keys, use some kind of data as the value. And if the object is no longer used outside this map, then is also removed from the map as well.

Derick Rethans  9:16
	So you mentioned objects as keys. Is that something new? Because I don't think currently PHP supports that.

Nikita Popov  9:22
	I mean, you can't use objects as keys in normal arrays. That doesn't work. For example, the array access interface and the traversable interface, they don't really care what your types are. So you can use anything.

Derick Rethans  9:37
	I glanced over that that point, yes. But weak map is something that then implements array access.

Nikita Popov  9:44
	That's right

Derick Rethans  9:45
	How does the interface of a weak map look like? How would you interact with it?

Nikita Popov  9:49
	Yeah, actually, it just implements all the magic interfaces in PHP. So ArrayAccess, you can access the roadmap by key, where the key's object. Traversable, that is you can iterate over the weak map and get both the keys and values, and of course Countable, so you can count how many elements there are in there. And that's it.

Derick Rethans  10:12
	All the methods, there's plenty of em then, there should be nine or 10 or so right?

Nikita Popov  10:17
	Five.

Derick Rethans  10:18
	No there's the six of iterator.

Nikita Popov  10:20
	Right, yeah, there is this little detail where when you implement Traversable, internal classes, you don't actually have to implement iterator methods. That's why there is a few, a few less.

Derick Rethans  10:33
	Who's going to benefit from this new feature?

Nikita Popov  10:35
	Like one of the users for weak maps are things like ORMs. Where, well, database records are represented as object, and there is data storage related to these objects. And I think it's a, well, well known issue that if you're using ORMs you can sometimes run into Memory Usage issues. And the absence of weak structures is one of the reasons why that can happen. So that they just keep holding onto information even though the application actually doesn't use it anymore.

Derick Rethans  11:12
	Did a specific ORM request this feature?

Nikita Popov  11:15
	I don't think so.

Derick Rethans  11:16
	Because weak maps are something done as an internal class in PHP, how are these things implemented? Is there something interesting because I remember talking to Joe about weak references last year, there is some functionality where it would automatically do something on the destructor or rather of the objects. Is this something that also happens with weak maps.

Nikita Popov  11:37
	So yeah, the mechanism how weak references and maps work is basically the same. So there is a flag on each object, that can be set to indicate that it has a weak reference or weak map. If the object is destroyed, and has this nice flag, then we execute a callbeck that is going to remove the object from the Weak Reference or from the weak map, or from multiple maps.

Derick Rethans  12:05
	Is it because there are some kind of registry that links an object?

Nikita Popov  12:08
	So when we store all the weak references, weak maps, and the object as part of, so we can efficiently remove it.

Derick Rethans  12:16
	When I was reading the RFC, I saw something like SPL object ID mentioned, which is a way how to basically identify a specific object. Is this something related to weak references or weak maps? Or is this something else no longer used, or people should no longer use pretty much, because I guess this was a way previously how to identify an object and then associated extra data with it. Like you mentioned that ORMs were due for cache.

Nikita Popov  12:44
	Right. So it's kind of related, but I'm also not. So one is not a replacement for the other, just different use cases. We used to have SPL object hash for a very long time. And I think, somebody went around PHP 7.0, or maybe later SPL object ID was introduced, which this the same just because an integer and because because of that is more efficient. But in the end, what these functions do is return a unique identifier for an object. But this identifier is only unique as long as the object is alive. So these object IDs are reused when objects are destroyed.

Derick Rethans  13:30
	And that makes them not usable for associating cache data with a specific object?

Nikita Popov  13:35
	That makes them usable for associating cache data. But you also have to store the object to make sure it does not get destroyed in the meantime. So that's how you get around the restriction that you cannot use objects as array keys. That's what you need the ID for. But you still have to store the like a strong reference to the object to make sure it's not garbage collected. And this ID starts referencing some kind of other objects.

Derick Rethans  14:04
	When you say Strong Reference, that is what PHP references are traditionally?

Nikita Popov  14:08
	That's the normal reference.

Derick Rethans  14:10
	Well, because it's been quite some time since it's got introduced from what I understood this has been accepted?

Nikita Popov  14:16
	It is accepted: 25, zero

Derick Rethans  14:18
	25, zero. That doesn't happen very often.

Nikita Popov  14:22
	Most RFCs are maybe not anonymous, but usually either they are 95% accepted, or they rejected really hard. There is not a lot of middle ground.

Derick Rethans  14:34
	That's pretty good, though. In any case, we will see this in PHP 8, I suppose, coming out later in the year.

Nikita Popov  14:39
	That's right. Yes.

Derick Rethans  14:41
	Well, thank you for taking the time today to talk to me about weak references and preloading especially on Windows. Thank you for taking the time.

Nikita Popov  14:50
	Thanks for having me Derick

Derick Rethans  14:52
	Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
