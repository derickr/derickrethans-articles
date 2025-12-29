PHP Internals News: Episode 58: Non-Capturing Catches
=====================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-06-18 09:21 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin058
   :Enclosure: media/pin-058.mp3

In this episode of "PHP Internals News" I chat with Max Semenik (`GitHub
<https://github.com/MaxSem>`_)
about the Non-Capturing Catches RFC that he's worked on, and that's been
accepted for PHP 8, as well as about bundling, or not, of extensions.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-058.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:18  
	Hi, I'm Derick, and this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 58. Today I'm talking with Max Semenik about an RFC that is proposed called non capturing catches. Hello Max, would you please introduce yourself.

Max Semenik  0:38  
	Hi Derick. I'm an open source developer, working mostly on MediaWiki. So that's how I came to be interested in contributing to PHP.

Derick Rethans  0:50  
	Have you been working with MediaWiki for a long time?

Max Semenik  0:53  
	Something like 11 years, I guess. 

Derick Rethans  0:56  
	That sounds like a long time to me. The RFC that you've made. What is the problem that is trying to address?

Max Semenik  1:03  
	In current PHP, you have to specify a variable for exceptions you catch, even if I you don't need to use this variable in your code, and I'm proposing to change it to allow people to just specify an exception type.

Derick Rethans  1:20  
	At the moment, the way how you catch an exception is by using catch, opening parenthesis, exception class, variable, and you're saying that you don't have to do the name of the variable any more. I get that right?

Max Semenik  1:33  
	Yes.

Derick Rethans  1:34  
	Is that pretty much the only change that this is making?

Max Semenik  1:38  
	Yes, it's a very small, and well defined RFC. I just wanted to do something small, as my start to contributing to PHP.

Derick Rethans  1:51  
	I'm reading the RFC, it states also that the what used to be an earlier RFC. How does that differ from the one that you've proposed?

Max Semenik  2:00  
	The previous RFC wanted to also permit a blanket catching of exceptions, as in anything. And that's all, which, understandably, has caused some objections from the PHP community. While most people commented positively on the part that I'm proposing now. Or should I say really propose because the RFC, passed and was merged yesterday.

Derick Rethans  2:35  
	I had forgotten about it actually, it's good that you reminded me. So yeah, it got merged and ready for PHP eight. Basically what you say you picked the non controversial parts of an early RFC?

Max Semenik  2:47  
	I actually chose something to contribute and then looked for an RFC, to see if it was discussed previously.

Derick Rethans  2:55  
	Oh, I see. So, your primary idea of wanting to contribute to PHP, instead of you having an itch that you wanted to scratch, it's like you're saying?

Max Semenik  3:04  
	I have way larger itches that I will scratch later when I will learn how to work with PHP's code base which, which is really huge.

Derick Rethans  3:16  
	That makes some sense I suppose. When looking at the vote for the RFC I actually couldn't see that you had voted it for yourself. I missed something?

Max Semenik  3:25  
	I don't have a php.net account so I can't vote for myself, obviously.

Derick Rethans  3:31  
	I actually think you can because you have written an RFC. 

Max Semenik  3:35  
	I haven't seen any interface to vote. 

Derick Rethans  3:38  
	Interesting. It's actually something to catch up on because I pretty much sure that you can. Should investigate that for some other RFCs that are still open because I think you should be able to. 

Max Semenik  3:49  
	Would benice. I mean, this wouldn't change anything but..

Derick Rethans  3:54  
	That's true but I mean you've started contributing. If you be able to vote right that's the fair thing to do, I suppose. So as you said, this is your first contribution to PHP itself. How did you find the whole process of getting this going and getting started with it?

Max Semenik  4:10  
	As far running an RFC, it was fairly straightforward to me. Maybe because I was looking at PHP RFCs in the past, so I knew how the process worked and it was really something that I already knew how to navigate. It's not the first open source community I'm contributing to, so I kind of know what to do in general.

Derick Rethans  4:40  
	How large is the MediaWiki community?

Max Semenik  4:43  
	It's probably larger than PHP community in terms of actively contributing people, as in which the Wikimedia Foundation has lots of paid programmers that work on the ecosystem. Obviously the outreach of your community is larger than MediaWiki's.

Derick Rethans  5:08  
	You're saying that there's more people working on, on it. But there's more people using PHP?

Max Semenik  5:15  
	And more people actively interested in development.

Derick Rethans  5:21  
	Do you think that's because it's easier to contribute to something that's written in PHP, than PHP itself?

Max Semenik  5:28  
	Not a lot of people know how to program in C these days. And while I used to be paid for writing C, my C's currently extremely rusty. Unlike PHP, for example.

Derick Rethans  5:44  
	For me it's sort of the other way around, because I haven't been writing PHP code for quite some time now, except for some test cases, so I know nothing about frameworks whatsoever. I know C pretty well. In any case, we now have one more active contributor, that is you, that is you. You've things merged that makes you a contributor, in my eyes. As this is a pretty small RFC. And I think during the course of the last few months we have I've discussed with several other contributors that small RFCs are a good thing, because it makes it much harder to find problems with. There are a few other RFCs as well that are also so small and for which the authors declined to talk to me about that for various different reasons. And two of those are actually really really simple things, and they are both having to do with the bundling of extensions in PHP. Now, just thinking about this question. How does MediaWiki, for example, think about which extensions, it can use in its source code?

Max Semenik  6:45  
	For MediaWiki. First of all, on start-up MediaWiki quickly checks if all the hard required extensions are available, and they just bails out if they aren't available. I need to look, whether it checks for JSON or as soon as it's way too obvious to even consider whether it's present or not.

Derick Rethans  7:10  
	So you just mentioned the JSON extension. That makes sense because that's one of my notes. One of the RFCs as you just alluded to is to JSON extension, and PHP eight will have this always available now without you having to enable this in configure flags, which is pretty good way of making sure that extension is always available to everybody using PHP. Do you agree that having a JSON extension always available is a good idea for PHP?

Max Semenik  7:37  
	Yes absolutely. One of the aspects of writing software that's available for everyone to use, as opposed to some internal company software that's running on a few servers and that it, is that the you need to support a wide variety of systems. And if it's possible to compile PHP without JSON, it means that someone will compile without it. It also means that some Linux distribution developers will package it as a separate package, and then someone will not install it, and you will get people to complain that MediaWiki doesn't work on their system. For more, very popular extensions are available. If I will know that many popular extensions that I need, are always available, it makes my job easier and it also allows me to write better software, without having to resort to hacks and decrease the functionality.

Derick Rethans  8:52  
	An what some other framework to do this they start making polyfills for them.

Max Semenik  8:56  
	And these polyfills might have vital like orders of magnitude worse performance. If I can have guarantees that a system has JSON, as well as other extensions like mbstring, intl, and so on, it would be really awesome.

Derick Rethans  9:16  
	The argument always between, do we always want to have everything inside PHP or not, and at some point you need to start making a distinction about is this useful enough for everybody or just for a smaller group of people, and mbstring is probably an example where this is sort of, sort of on the line right. I mean it's useful enough, but is it useful enough to have it always enabled instead of having it easily installed as a package.

Max Semenik  9:42  
	Well you know lots of people are running software, whether it's MediaWiki whether it's some WordPress or something else on crappy shared hosting, which is the bane of every programmer's existence but they still have to support it. The question is really something can be messed up. Some people will have to run a node on systems that have messed up. And if we can avoid it. Why not?

Derick Rethans  10:11  
	Another RFC that's just gone through its unbundling extension. Some versions of PHP will have extensions, being brought into core and being always made available like we did with the hash extension in PHP seven four. But of course we also removing extensions from PHP to live somewhere else. Not even having them always enabled but not even having them distributed with a PHP source code. In PHP seven four we had for example the Firebase extension, I believe, because there wasn't a lot of people using this. In this case we having the XMLRPC extension. Have you ever heard of this XMLRPC extension, because you said you've been programming PHP for a while? 

Max Semenik  10:51  
	I've heard about the protocol itself and I might have heard about PHP having this extension, but I've never used it, and honestly I don't know why anyone using it. 

Derick Rethans  11:04  
	It's sort of being used a little bit when people really didn't want to use SOAP, because it was too complicated. But before we had invented JSON pretty much. That's a long long time ago.

Max Semenik  11:18  
	These days. XMLRPC is sounds like a legacy corporate system. That's why probably, it's no use having it in PHP proper.

Derick Rethans  11:32  
	I think I very much agree there. In any case, non capturing caches are in PHP eight. You said that the RFC was saccepted, has the patch being merged as well. 

Max Semenik  11:41  
	Yep. 

Derick Rethans  11:42  
	Great. I'm going to have to have a flavour that I'm going to give a talk next month for the Dutch PHP conference, where I'm talking about a new additions in seven four, but also what's coming up in eight dot zero, I might be able to have a slide about it in there.

Max Semenik  11:57  
	Awesome.

Derick Rethans  11:58  
	Thank you, Max for taking the time today to talk to me about non caption captures and bundling of extensions.

Max Semenik  12:05  
	Thank you, Derick for giving me this tribune. It was a nice talk.

Derick Rethans  12:09  
	Excellent. Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.


Show Notes
----------

- RFC: `Non-Capturing Catches <https://wiki.php.net/rfc/non-capturing_catches>`_
- RFC: `Always available JSON extension <https://wiki.php.net/rfc/always_enable_json>`_
- RFC: `Unbundle ext/xmlrpc <https://wiki.php.net/rfc/unbundle_xmlprc>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
