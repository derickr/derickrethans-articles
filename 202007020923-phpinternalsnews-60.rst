PHP Internals News: Episode 60: OpenSSL CMS Support
===================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-07-02 09:23 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin060
   :Enclosure: media/pin-060.mp3

In this episode of "PHP Internals News" I chat with Eliot Lear (`Twitter
<https://twitter.com/eliotlear>`_, `GitHub <https://github.com/elear>`_,
`Website <https://www.ofcourseimright.com/>`_) about OpenSSL CMS support,
which he has contributed to PHP.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-060.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:16  
	Hi, I'm Derick, and this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 60. Today I'm talking with Eliot Lear about adding OpenSSL CMS supports to PHP. Hello Eliot, would you please introduce yourself.

Eliot Lear  0:34  
	Hi Derick, it's great to be here. My name is Eliot Lear, I'm a principal engineer for Cisco Systems working on IoT security.

Derick Rethans  0:41  
	I saw somewhere on the internet, Wikipedia I believe that he also did some RFCs, not PHP RFC, but internet RFCs.

Eliot Lear  0:49  
	That's correct. I have a few out there I'm a jack of all trades But Master of None.

Derick Rethans  0:53  
	The one that piqued my interest was the one for the timezone database, because I added timezone support to PHP a long long time ago.

Eliot Lear  1:01  
	That's right, there's a whole funny story about that RFC, we will have to save it for another time but there are a lot of heroes out there in the volunteer world, who keep that database up to date, and currently the they're corralled and coordinated by a lovely gentleman by the name of Paul Eggert and if you're not a member of that community it's really a wonderful contribution to make, and they need people all around the world to send an information but I guess that's not why we're here today.

Derick Rethans  1:29  
	But I'm happy to chat about that at some other point in the future. Now today we're talking about CMS support in OpenSSL and the first time I saw CMS. I don't think that means content management system here.

Eliot Lear  1:41  
	No, it stands for cryptographic message syntax, and it is the follow on to earlier work which people will know as PKCS#7. So it's a way in which one can transmit and receive encrypted information or just signed information.

Derick Rethans  1:58  
	How does CMS, and PKCS#7 differ from each other.

Eliot Lear  2:03  
	Actually not too many differences, the externally the envelope or the structure of the message is slightly better formed, and the people who worked on that at the Internet Engineering Task Force were essentially just making incremental improvements to make sure that there was good interoperability, good for email support and encrypted email, and signed email, and for other purposes as well. So it's very relatively modest but important improvements, from PKCS#7.

Derick Rethans  2:39  
	How old are these two standards? 

Eliot Lear  2:42  
	Goodness. PKCS#7, I'm not sure actually of how old the PKCS#7 is, but CMS dates back. Gosh, probably a decade or so I'd have to go look. I'm sorry if I don't have the answer to that one,

Derick Rethans  2:56  
	A ballpark figure works fine for me. Why would you want to use CMS over the older PKCS#7?

Eliot Lear  3:02  
	You know, truthfully, I'm not, I'm not a cryptographer, so the reason I used it was because it was the latest and greatest thing and when you're doing this sort of work. I'm an, I'm an interdisciplinary person so what I do is I go find the experts and they tell me what to use. And believe it or not, I went and found the person who's the expert on cryptographic signatures, which is what I need. I said: What should I use? He said: You should use CMS and so that's what I did. What I ran into some troubles though, which is that some of the tooling, doesn't support CMS. So, in particular PHP didn't support CMS. So that's why I got involved in the PHP project.

Derick Rethans  3:40  
	You are a new contributor to the PHP project. What did you think of its interactions?

Eliot Lear  3:45  
	I had a wonderful time doing the development. There was a fair amount of coding involved, and one has to understand that the underlying code here is OpenSSL and OpenSSL's documentation for some of its interfaces could stand a little bit of improvement. I needed to do a fair amount of work and I needed a fair amount of review so I got a lot of support from Jakub particular, who looks after the OpenSSL code base, as one of the maintainers, and I really enjoyed the CI/CD integration, which allowed me to check the numerous environments that PHP runs on. I really enjoyed the community review, and I really enjoyed it even though I didn't have to really do one in my case, I did do an RFC, as part of the PHP development process, which essentially forced me to write really good documentation or at least I hope it's really good. Before all of the caller interfaces that I defined, so it was a really enjoyable experience. I really liked working with the team.

Derick Rethans  4:47  
	That's good to hear. I think sometimes although an RFC wasn't particularly necessary here, as an RFC one particularly necessary I always find writing down the requirements that I have for my own software, first, even though this doesn't get publicized or nobody's going to review that always very useful to just clear my head and see what's going on there. 

Eliot Lear  5:06  
	Yeah, I think that's a good approach. 

Derick Rethans  5:07  
	During the review, was there a lot of feedback where you weren't quite sure, or what was the best feedback that you got during this process?

Eliot Lear  5:15  
	Biggest issue that we had was, how to handle streaming, and we have some code in there now for streaming, but it's it's unlikely to get really heavily exercised in the way that the interfaces are defined right now. It's essentially files in/files out interface which mirrors the PKCS#7 interface. One of the future activities that I would like to take on if I can find a little bit more time, is to move away from the files in/files out interface, but rather use an in memory structure or in memory interface. So that can actually take advantage of streaming and can be more memory efficient, over time. 

Derick Rethans  5:56  
	When you say file now you actually provide a file name to the functions? 

Eliot Lear  6:00  
	That's right, you know, depending on which of the interfaces you're using, there's an encrypt, there's an encrypt call there's a decrypt call. There's a sign and a validate call, and or a verify call, and each of them has a slightly different interface, but you know if you're encrypting you need to have the destination that you're encrypting through these are all public key, you know PKI based approaches so you have to have the destination certificates, that you're sending. If you're verifying you need to have the private key to do or you need, I'm sorry you need to have the public key chain and if you're decrypting to have the private key to do all this. So, but they're all filenames that are passed and it's a bit of a limitation of the original interface in that you probably don't really want to be passing file names from most of your functions you'd rather be passing objects that are a bit better structure than that. 

Derick Rethans  6:53  
	Is the underlying OpenSSL interface similar or does that allow for streaming in general?

Eliot Lear  6:59  
	The C API allows for streaming in such. The command line interface, it doesn't seem to me that they do any particular things with with streaming. If you look at the cryptographic interface that we that we did for CMS, mostly it is an attempt to provide the capability that you would otherwise have on the open using the OpenSSL command line interface and I think the nice thing here is that we can evolve from that point.

Derick Rethans  7:26  
	And the progress wouldn't only be done implemented for the CMS mechanism, but also for PKCS#7, as well as others that are also available.

Eliot Lear  7:35  
	Yes. Another area that I would like to look at, I'm not sure how easy it will be, we didn't try it this time was to try and combine the code bases because they are so close, and be a little bit more code efficient, but there are just slight enough differences in the caller interfaces between PKCS#7 and CMS that, I'm not sure I could get away with using void functions for everything I have. I might have to have a lot of switches, or conditionals in the code. But what I am interested in doing for both sets of code is, again, providing new interfaces, where instead of passing file names, you're passing memory structures of some form that can be used to stream. That's the future. 

Derick Rethans  8:22  
	I've been writing quite a bit of GO code in the last couple of months. And that interface is exactly the same, you provide file names to it, which I find kind of annoying because I'm going to have to distribute these binaries at some point. And I don't really want any other dependencies in the form of files, so I need to figure out a way how to do that without also provide those key files at some point. 

Eliot Lear  8:43  
	Indeed, that's, that's an issue, and for us right well who are web developers I did this because I was doing some web development. A lot of the stuff that I want to do. I just want to do in memory and then pass right back to the client and I don't really want to have to go to the File System. And right now, I'll have to take an extra step to go to the File System and that's alright, it's not a big deal, but it'll be a little bit more elegant when I get away from that. We'll do that you know at an appropriate time.

Derick Rethans  9:11  
	Yes, that sounds lovely. I'm not an expert in cryptography either. I saw that the RFC mentions the X 509. How does it tie in with CMS and PKCS #7?

Eliot Lear  9:21  
	X 509 is essentially a certificate standard. In fact, that's what really what it is. A certificate essentially has a bunch of attributes, along with a subject being one of those attributes and a signature on top of the whole structure. And the signature comes from a signer, and the signer is essentially asserting all of these attributes on behalf of whoever sent the request. X 509 certificates are, for example the core of our web authentication infrastructure. When you go to the bank online, it uses an X 509 certificate to prove to you that it is the bank that you intended to visit, that's the basis of this and CMS and PKCS#7 are structures that allow the X 509 standard to be serialized, so there's the distinguishing coding rules that are used underneath PKCS#7 and CMS, and then what you have, CMS essentially was designed as at least in part for mail transmission. So how is it that you indicate the certificate, the subject name, the content of the message. All of this information had to be formally described, and it had to be done in a way that is scalable. And the nice thing about X 509, as compared to say just using naked public keys, is with naked public keys, the verifier or the recipient has to have each individual public key, whereas with X 509, it uses the certificate hierarchy such that you only need to have the top of the chain, if you will, in order to validate a certificate. So X 509 scales, amazingly well, we see that success, all throughout the web. And so that's what CMS and PKCS#7 help support.

Derick Rethans  11:24  
	Like I said, I've never really done enough research into this but I think it is something that many web developers should really know how that works because this comes back, not only with mail, but also with HTTPS.

Eliot Lear  11:35  
	It's another part of the code right. So CMS isn't directly used for supporting TLS connections, there's a whole a whole set of code inside of PHP for that.

Derick Rethans  11:44  
	Would you have anything else to add?

Eliot Lear  11:46  
	I would say a couple of things. The basis of this work was that I was attempting to create signatures for something called manufacturer usage descriptions. The reason I got involved with PHP is that I'm doing tooling that supports an IoT protection project. And this this manufacturer usage descriptions essentially describes what the device, what an IoT device needs in terms of network access. And the purpose of using PHP and adding the code that I added was so that those descriptions could be signed, and that's why Cisco, my employer, supported my activity. Now Cisco loves giving back to the community. This was one way we could do so it's something I'm very proud of when it comes to our company. And so we're very happy to participate with the PHP project. I really enjoyed working with

Derick Rethans  12:33  
	That's glad to hear. I'm looking forward to some other API improvements because I agree that the interfaces that the OpenSSL extension has aren't always the easiest to use and I think it's important that encryption is easy to use, because more people will use it right.

Eliot Lear  12:49  
	I have to say, in my opinion, the encryption interfaces that we have today are still relatively immature. And not just CMS, the code that I wrote, which is really you know fresh it just got committed, but the whole category of interfaces, is something that will evolve over time and it's important that it do so because the threats are evolving over time and people need to be able to use these interfaces, and we can't all be cryptographic experts, I'm not. I just use the code but I needed to write some in order to use it in my case, but as we go on I think will enjoy richer and easier to use interfaces that normal developers can use without being experts.

Derick Rethans  13:38  
	PHP has been going that way already a little bit because we started having a simple random interface, and in a simple way of doing hashes and verifying hashes, to make these things a lot easier because we saw that lots of people are implementing their own ways in PHP code, and pretty much messing it up because, as you say not everybody's a cryptographer.

Eliot Lear  13:56  
	That's right. And so that's a really good thing that PHP did, because as you pointed out, it eliminates all the people who are going onto the net looking for the little snippet of code that they're going to include in PHP, whether that snippet is correct or not that's a big issue. 

Derick Rethans  14:11  
	Absolutely. And cryptography is not something that you want to get wrong.

Eliot Lear  14:15  
	That's right, because for every line of code that you've written in this space, there's going to be somebody who's going to want to attack it, maybe several.

Derick Rethans  14:23  
	Absolutely. Thank you, Eliot, for taking the time this morning to talk to me about CMS support.

Eliot Lear  14:28  
	It's been my pleasure Derick, and thanks for having me on. And again, it was really enjoyable to work with the PHP team and I'm looking forward to doing more.

Derick Rethans  14:38  
	Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool, you can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.



Show Notes
----------

- RFC: `Add CMS Support <https://wiki.php.net/rfc/add-cms-support>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
