PHP Internals News: Episode 77: fsync: Buffers All The Way Down
===============================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-02-25 09:05 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin077
   :Enclosure: media/pin-077.mp3

In this episode of "PHP Internals News" I chat with David Gebler (`GitHub
<https://github.com/dwgebler>`_) about his suggestion to add the `fsync()`
function to PHP, as well as file and output buffers.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-077.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:13
	Hi, I'm Derick. Welcome to PHP internals news, a podcast dedicated to explaining the latest developments in the PHP language. This is Episode 77. In this episode I'm talking with David Gebler about an RFC that he's written to add a new function to PHP called fsync. David, would you please introduce yourself?

David Gebler  0:35
	Hi, I'm David. I've worked with PHP professionally among other languages as a developer of websites and back end services. I've been doing that for about 15 years now. I'm a new contributor to PHP core, fsync is my first RFC.

Derick Rethans  0:48
	What is the reason why you want to introduce fsync into the PHP language?

David Gebler  0:52
	It's an interesting question. I suppose in one sense, I've always felt that the absence of fsync and some interface to fsync is provided by most other high level languages, has always been something of an oversight in PHP. But the other reason was that it was an exercise for me in familiarizing myself with PHP's core getting to learn the source code, and it's a very small contribution, but it's one that I feel is potentially useful, and it was easy for me to do as a learning exercise.

Derick Rethans  1:16
	How did you find learning about PHP's internals?

David Gebler  1:19
	Quite the roller coaster. The PHP internals are very arcane I suppose I would say, it's it's something that's not particularly well documented. It's quite an interesting challenge to get into it. I think a lot of it you have to pick up from digging through the source code, looking at what's already been done, putting together the pieces, but there is a really great community on the internals list, and indeed elsewhere online, and I found a lot of people very helpful in answering questions and again giving feedback when I first opened my initial proof of concept PR

Derick Rethans  1:48
	Did you manage to find room 11 on Stack Overflow chat as well?

David Gebler  1:52
	I did not, no.

Derick Rethans  1:53
	I'll make sure to add a link in the show notes and it's where many of the PHP core contributors hang out quite a bit.

David Gebler  2:00
	Sounds good to know for the future.

Derick Rethans  2:02
	I read the RFC earlier today. And it talks about fsync, but it also talks about flush, or f-flush. What is the difference between them and what does fsync actually do?

David Gebler  2:14
	That's the question that will be on everyone's lips when they hear about this feature being introduced into the language, hopefully. What does fsync do and what does fflush do? To understand that we have to understand the concept of the different types of buffering, an application runs on a system. So we have the application or sometimes called the user space buffer, and we have the operating system kernel space buffer,

Derick Rethans  2:36
	And we're talking about writing to file here right?

David Gebler  2:39
	We're talking about writing to files with fflush and fsync yep. So fflush and fsync are both about getting data out to a file, and using it to push something out of the buffers. But there are two different types of buffers; we have the application buffers; we have the operating system buffers. What fflush does is it instructs PHP to flush its own internal buffers of data that's waiting to be written into a file out to the operating system. What it doesn't do is give us any guarantee that the operating system will actually write that data to disk, so the operating system has its own buffer as well. Computers like to be efficient, operating systems like to be efficient, they will save up disk writes and do them in the order they feel like at the time they feel like. What fsync does is it also instructs the operating system to flush its buffers out to disk, thus giving us some kind of better assurance that our data has actually reached disk by the point that function returns.

Derick Rethans  3:29
	I really only know about Linux here, but I know on Linux that there's a journal in the file system or, in most of the file systems that it uses. Would have seen make sure the data ends up in a journal, are also committed as how the file system does it itself?

David Gebler  3:45
	The manner in which fsync synchronizes is indeed dependent on the particular POSIX type operating system, and file system that it's running on. I think you're right in respect of modern Linux and ext4, that would ensure the journal was also updated and that the data was persisted to disk. Older versions may behave a little bit differently. And one thing to note with fsync is that you can't take it as a cast iron guarantee that the data has either basically been persisted to disk, or that the corresponding file system entries have been updated. It is the best assurance you can get that those things have happened, and by the point that function returns, again, in PHP implementation, you have a solid an assurance as you're going to get. Because that manner of synchronization is dependent on the underlying system. It can vary. Yeah Linux ext4 is probably your best bet with fsync, but one interesting thing to know if it does go into PHP and people start using it, is that when you call fsync on a file, if you want to ensure that the file system entries are properly updated as well, you should also call fsync or a handle to that file's containing directory. And of course on a Linux or POSIX type system you can do that you can you can fopen a directory, and you can then call a sync on that handle.

Derick Rethans  5:02
	You mentioned that you can call fsync on a file handle. How's it different than calling just fsync without an argument? Is fsync without an argument just telling the operating system to flush its current buffers, and fsync on a file handle specifically to flush that specific file, and its metadata?

David Gebler  5:22
	So in the PHP implementation of fsync, you must supply it with a stream resource that is plain File Stream. You can obviously at the PHP level give it some other kind of resource but it will return a warning if it's not able to convert that into a plain File Stream. Under the hood, when you're talking about C, underlying operating system level, fsync operates on a file descriptor so it doesn't even operate on a stream handle; it operates on the actual underlying number that represents the file at the operating system level.

Derick Rethans  5:51
	So that is different from the Unix shell command called fsync.

David Gebler  5:55
	It is different. Yep.

Derick Rethans  5:57
	The RFC also talks about another function called fdatasync. How's that different from fsync itself?

David Gebler  6:03
	I think that was a really good question because it's got two answers. It's got the theoretical answer and the practical answer. The practical answer is that these days, it most likely makes no difference at all. The theoretical answer is the fdatasync only pushed out the actual file data to a disk write, but it wouldn't necessarily update metadata about the file, such as the time it was last modified the data that might be stored in the file system about the file. The reality now is that most operating systems, we'll update both of those things at once. The reason they were separated out in the POSIX specification is because of course updating the metadata is another write, so a call to fsync could require two rights were call to fdatasync would only require one. I'm not aware of any operating system and file system now that I'm going to use that actually still treats them differently

Derick Rethans  6:57
	In PHP, we try to make sure that functions that are implemented work exactly the same on operating systems, and you've already explained that fsync depends a little bit on how the file system handles the specific requests. Would fsync also work in a similar way on on Windows or OSX, or is it specifically meant for just Linux?

David Gebler  7:20
	It's not specifically meant for just Linux. fsync itself is part of the POSIX specification. Strictly speaking, fsync as an operating system level API does not exist on Windows. Windows does have a similar API mechanism called flush file buffers, and in the RFC, and in the implementation attached to that RFC, on Windows, that's what fsync does, it's a wrapped call to flush file buffers. It has, in practice the same effect. OSX is a bit of a trickier one. fsync does exist on OSX I know. I'm not a user of Apple products myself but I can tell you what I know about OSX. fsync on OSX, it will sort of attempt to flush your file buffers out, but OSX itself will not guarantee that the disk buffers are cached. So we have another layer of buffering there. You have the application space buffering, we have the kernel space buffering, and we have the hardware disk buffering itself. Physical drives will sometimes lie about having written data to disk. I mean USB drives are a notorious example of that. A USB drive will tell your operating system it's finished persisting data when it hasn't, and you can even see that on the little flashing LED on the drive, and if you pull it out your data will be corrupted. OSX is not so reliable. The interface exists, but whether it's actually worked or not is open to question because it may not have flushed the disk cache.

Derick Rethans  8:38
	Are there any backward compatibility issues with this RFC or the implementation of this?

David Gebler  8:43
	I'm pleased say there are no backwards compatibility issues at all. It's straightforwardly a new function that operates on plain file streams. It triggers a warning if you give it some kind of resource that can't be converted to a plain file descriptor - no consequence to not using it. You just get the same behaviour you've had in previous PHP versions. It's just a new optional function.

Derick Rethans  9:06
	What has the feedback to introducing fsync and fdatasync been so far?

David Gebler  9:11
	When I originally proposed the RFC, I had a bit of feedback on the internals list and around some of the other aspects of the PHP community that I reached out to in various places. Some people didn't see a need for it, which is fair enough, but my answer to that would be the when we look at the history of PHP as a web first language, you can see why people might not have had much of a use case for something like fsync. I mean it only fits particular situations, which is where you want some kind of transaction or guarantee. Quite often what people were doing with PHP were web applications where there was a degree of volatility in file rights, that wasn't important to them; they didn't particularly care about it. What I would say now is that when we look at PHP eight, and the evolution of the PHP ecosystem. You're seeing that it is, it is such a versatile and performance general purpose programming language these days, that people are using it for all manner of tasks. Using it in micro service architectures for back end services, they're using it even for things like data science and machine learning, emerging industries like that, so it's got so many more applications now. Broadly the feedback I had was, for the most part, probably wouldn't take much more than a pull request for it to be accepted. I think it's a very non controversial RFC. It's a small feature to add in the form of a new function.

Derick Rethans  10:33
	And that is very different than Introducing enumerations or Fibers, of course.

David Gebler  10:38
	Both of which I'm looking forward to.

Derick Rethans  10:40
	We spoke a little bit about file system buffers, once you do a write it up in the application buffers, with flush, you can flush that to the operating system buffers and fsync to the file system. But in PHP development, there's also other buffers, that if you output something to the screen on the command line that ends up directly on the terminal. When you do this when PHP runs in a web server environment it doesn't do that because there's more buffers in between right. There's PHP's own buffers, there's buffers you can configure, there is then web server buffers, and networking buffers. So it's buffers all the way down pretty much, isn't it? How does the buffering in PHP's output work, and what kind of things can you do with that?

David Gebler  11:21
	When we look at buffering in PHP, this is very easy to get confused, because he says buffers all the way down. On one hand we may be talking about buffers at the file system level and application space and kernel space, and on the other side we're gonna be talking about these kinds of buffers that you've just mentioned. Again, this is something that PHP does primarily for performance and perhaps for a few other purposes in how it manages the application that is running. Buffering is a way of storing output, somewhere before it gets sent on to the web server and ultimately from the web server to a user's browser. As you say a web server has its own buffering as well and PHP provides a couple of functions by which you can also attempt to force output to the browser. So again we have much as we have fflush for files. We have flush for regular output that you're trying to send to a web server. And that function will flush the internal output buffers of PHP, and it will then try and flush your web server buffers. There's an interesting parallel here because much like with fsync, flush versus fsync, you can't necessarily guarantee with fflush what the operating system will do. Your data wanted received it into its own buffers. With PHP you can't necessarily get a guarantee that a web server will flush its own buffers when you flush your output there. Perhaps we need to invent some kind of fsync for the web server as well. Output buffering is something that you can configure in PHP and you can stack and nest output buffers as well, so that means from an application developer point of view, you are using some other components, some other bit of code in your PHP system that produces its own output. When I say output, I mean via the normal mechanisms you would write something to a browser in PHP, so things like echo at the simplest level. What you can do is you can use PHP's output buffer functions, which are all the ones that start with ob then an underscore, to capture that output and control what you do with it, instead of it being output to the browser; you can capture it into a string, you can manipulate it, you can discard it, you can throw it up the chain to be output yourself. That's what we would primarily use those buffers for in PHP, but output buffering is also something you can configure in the php.ini file. You can turn it off, you can set the buffer size. Do you have a little bit of fine tuning there that you can do.

Derick Rethans  13:39
	Something of that just popped in my mind is that when you call the new fsync on a file resource handler, file resource in PHP are implemented with an underlying interface because streams. Is there another fall writing buffer in streams itself as well?

David Gebler  13:58
	There is a buffer in the actual C library level when it comes to streams. That's going into the detail a little bit of what I was talking about earlier where you have user space buffers and kernel space buffers. The reason those things exist is to do with the way an operating system manages a computer and keeps everything safe. User space isn't able to access kernel space. It's about range of memory addresses in the computer; we're getting quite low level now in terms of how all this works. In the underlying implementation in PHP source code, we're using the C File Stream functions to write the streams, and that means that the actual data gets copied into a buffer in userspace. What fsync does is it instructs the kernel to make a copy of that data in its own memory space, and then push that out to disk. It's getting quite low low level when we get into those details, it has to do with how file access is managed in PHP. There is an even lower level of access, which is more suitable for very high throughput intensive IO operations, which isn't available in PHP at all. I'm not so familiar with how this works on Linux because the file systems are different, but I can tell you in Windows, if you if you want really intensive throughput, you don't want to be calling fsync or equivalent flush file buffers, you know, hundreds of 1000s of times per second. You want to use what we call unbuffered output, where you write directly at the block level to disk. It wouldn't be suitable to try and do something like that in PHP, because it's to higher level language. It's a very very fine level of control that you need to be able to do that kind of thing. But with that level of control you also have a high level of consequence if something goes wrong.

Derick Rethans  15:39
	I think there's actually an extension in PECL, or there used to be one at least, I'm not sure but it's still maintained for newer PHP versions, called dio or d.i.o., standing for direct IO, which I think implements some of these features, but I don't quite remember but I thought that was file system related, or just only network related direct IO.

David Gebler  16:00
	I think direct IO extension did have lower level file system access off the top of my head. It's been a long time since I looked at it, and I think it actually had the option to open a file in synchronized mode, which means every write is essentially implicitly calling fsync. What it didn't have was the ability to open file writes in unbuffered mode, which is probably because that's lower level, still I mean that requires you to literally know the block size of the file system you're writing to and to write in that, in that size,

Derick Rethans  16:30
	Doing that from PHP goes a little bit too far, I suppose.

David Gebler  16:34
	It does go too far, I think.

Derick Rethans  16:36
	Then again, if you can open a file you can open a file device as long as you have permission. So, I guess, nothing stops you from at least on Linux opening /dev/sda whatever number it is, as long as you're running PHP as root and writes to it, but I don't think this is a wise thing to do.

David Gebler  16:52
	Definitely something I'd urge people to try with caution. I mean, obviously on Linux you you do have this kind of extraordinary power from the combination of the user you're running a process as, coupled with the fact that Linux treats everything as a file, and then actually has some interesting implications for fsync, you can try compiling the branch that I've submitted in the PR for the RFC; compile it on Linux, call fsync on some different handles to things which aren't actually files, but which to the underlying Linux operating system look like files. Hopefully the implementation I've provided is robust enough that it should just return false when you try and fsync things that are not actually files.

Derick Rethans  17:28
	What kind of things are you thinking of here, things like directories?

David Gebler  17:32
	Directories are fine for fsync and you should actually be able to get a successful fsync on directories because they are literally part of the file system. It's fine on POSIX type systems to do that. I'm not actually sure whether you can do that on Windows or not, but I don't think it provides any particular benefit to fsync a directory on Windows because the underlying flash file buffers API works on the file level only, but on Linux you could try opening file handles to something like a Unix socket and try and fsync that. You should just get false in PHP land from that function because it knows that it can't convert that file handle into an actual file stream, and thus can't get a regular fsync on it.

Derick Rethans  18:12
	Have you created a test case for that situation?

David Gebler  18:15
	I have created some test cases for opening streams to things that are not files from PHP. I can't whether I've covered that particular one or not, but I have got a couple in there for things which are not files at the PHP level.

Derick Rethans  18:29
	That's good to hear. When do you think he will be putting this up for a vote?

David Gebler  18:34
	I'm planning to put this up for a vote later this week actually. The RFC has been open for a little while, I think it's been about three weeks since I announced it on internals, and obviously there hasn't been a huge amount by way of feedback or discussion on it lately. That doesn't particularly surprise me, it is not a particularly controversial thing to add. I think a lot of people probably don't have much feeling about it one way or the other. But then, I'm hopeful, people who vote will see that as a reason to include it.

Derick Rethans  19:00
	Thank you David for explaining the fsync RFC and related topics to me today.

David Gebler  19:05
	Well, thanks for having me on. It's been great talking to you. And of course to anyone listening who's interested in the RFC, do check it out on the PHP wiki. Do have a look at the discussion on internals. And if you're a voting member, don't forget to vote when I open it up to vote later this week.

Derick Rethans  19:21
	Thank you for listening to this instalment of PHP internals news, a podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool, you can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next time.

Show Notes
----------

- RFC: `fsync function <https://wiki.php.net/rfc/fsync_function>`_
- `Room 11 <https://chat.stackoverflow.com/rooms/11/php>`_ on StackOverflow
- PECL extension `DIO <https://pecl.php.net/package/dio>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
