PHP Internals News: Episode 54: Magic Method Signatures
=======================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-05-21 09:17 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin054
   :Enclosure: media/pin-054.mp3

In this episode of "PHP Internals News" I chat with Gabriel Caruso (`Twitter
<https://twitter.com/carusogabriel>`_, `GitHub <https://github.com/carusogabriel>`_,
`LinkedIn <https://linkedin.com/in/carusogabriel>`_)
about the "Ensure correct signatures of magic methods" RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-054.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:16  
	Hi, I'm Derick, and this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 54. Today I'm talking with Gabriel Caruso about his ensure correct signatures of magic methods RFC. Hello Gabriel, would you please introduce yourself?

Gabriel Caruso  0:37  
	Hello Derick and hello to everyone as well. My name is Gabriel. I'm from Brazil, but I'm currently in the Netherlands. I'm working in a company called Usabila, which is basically a feedback company. Yeah, let's talk about this new RFC for PHP eight. 

Derick Rethans  0:52  
	Yes, well, starting off at PHP eight. Somebody told me that you also have some other roles to play with PHP eight. 

Gabriel Caruso  0:59  
	Yeah, I think last week I received the news that I'm going to be the new release manager together with Sara. We're going to basically take care of PHP eight, ensuring that we have new versions, every month that we have stable versions every month free of bugs, we know that it's not going to happen.

Derick Rethans  1:17  
	That's why there's a release cycle with alphas and betas. 

Gabriel Caruso  1:20  
	Yeah. 

Derick Rethans  1:21  
	I've been through this exactly a year early, of course, because I'm doing a seven four releases. 

Gabriel Caruso  1:25  
	Oh, nice. Yeah. So I'm gonna ask a lot of questions for you.

Derick Rethans  1:29  
	Oh, that's, that's fine. It's also the role of the current latest release manager to actually kickstart the process of getting the PHP, in this case, PHP eight release managers elected. Previously, there were only very few people that wanted to do it. So in for the seven four releases it was Peter and me. But in your case, there were four people that wanted to do it, which meant that for the first time I can ever remember we actually had to hold some form of election process for it. That didn't go as planned because we ended up having a tie twice, which was interesting. So we had to run a run off election for the second person between you and Ben Ramsey, that's going to go continuing for you for the next three and a half years likely. 

Gabriel Caruso  2:11  
	Yep. 

Derick Rethans  2:12  
	So good luck with that. 

Gabriel Caruso  2:13  
	Thank you. Thank you very much.

Derick Rethans  2:15  
	In any case, let's get back to the RFC that we actually wanted to talk about today, which is the ensure correct signatures of magic methods RFC. What are these magic methods?

Gabriel Caruso  2:24  
	So PHP, let's say out of the box, gives the user some magic methods that every single class have it. We can use that those methods for anything, but basically, what magic methods are are just methods that are called by PHP when a given action happens to the class. So for example, if a class is being constructed, then the construct magic method is going to be called. If I'm calling serialize function, then the magic method serialize as per PHP seven four or PHP eight. I don't remember, so this is basically what magic methods are, are methods that PHP hook into the classes and then once a certain action happened with the class, then PHP is going to call those magic methods in something magic, so to speak is going to happen.

Derick Rethans  3:13  
	And other options are like underscore underscore get, and underscore underscore set. 

Gabriel Caruso  3:17  
	We have, we have a lot.

Derick Rethans  3:19  
	Exactly, what do people tend to use these magic methods for?

Gabriel Caruso  3:22  
	So that's something interesting. As the magic method is called by a number of actions we can use, for example, for let's let's get the example of ORM for example, Doctrine or Eloquent or whatever one. Let's say I'm a maintainer of that library. I don't know what fields do you have in your database. So when I'm porting, when I'm doing the translation, what it can do is map in a property, all those columns and values that I have in the database. And then when you instantiate your entity and you try to access a variable that is does not exist, then we're going to go to a magic method in this case is get, as I said, and I'm going to say okay, is not set in the class, but is mapped in the entity that I have. So this is one case, we also have the case for testing your you have, for example, the famous PHP Unit test framework, every time that a test case is called with all those methods is starting in with test, the call magic method is invoked. And then you can perform whatever action you have. You also have middlewares and the examples go go even further

Derick Rethans  4:32  
	In the title of RFC you have the word signature, what is the signature?

Gabriel Caruso  4:37  
	All the attributes that our method can have. So for example, the name of a method is its signature, what does it return? What parameters does it take? And also what modifiers so for example, is it static or not? Is it public, private or protected? So all this information together in usually is one line in PHP. So for example, private static MyMethod, that receives a string and returns a Boolean. There you go. This is the signature of my method

Derick Rethans  5:06  
	Because some of these magic methods have been in PHP for a long long time. Back in the time where we didn't have argument types or return types or perhaps not even static. All the way back from the past PHP hasn't really done anything with signatures because they've simply didn't exist. At the moment which signature checks this PHP already do?

Gabriel Caruso  5:26  
	I don't remember a by the RFC but I think was introduced together with the scalar type RFC. But only constructors and destructors until PHP seven four, those two only magic methods were being checked. If they have none return type, not even void, just no return type. But in PHP eight, we're gonna have the new stringable interface and then every single toString magic method. If it is typed, this is very important if it is typed it needs to be a string and these are the only from the 17 that we have only three in PHP 8 are being checked. 

Derick Rethans  6:01  
	PHP seven four.

Gabriel Caruso  6:02  
	Yeah, in PHP seven four only two and then PHP eight, we have the new toString.

Derick Rethans  6:07  
	But this RFC suggesting to change that of course. 

Gabriel Caruso  6:10  
	yeah. 

Derick Rethans  6:11  
	What's the reason why you want to extend these checks to the other magic methods?

Gabriel Caruso  6:14  
	That brings me back how I figured out that. I was looking at some bugs, because we have the https://bugs.php.net, where we centralized all the bugs of PHP. Then there is a bug report explaining in complaining exactly about that. Like, I can't hide my magic method. Back in the days I can say, for example, that my tostring method is going to return an integer or a Boolean. That makes no sense. And then I was like, yeah, makes makes no sense. We need to fix that out and then I start to search how do we type that? How what types do we have and then I was like, we can't in PHP eight, because this is going to be a new major version. So we are allowed to at least vote for do that. We can check if someone is using types, we can check those types. We are not going to force, we are not going to require, we're not going to evaluate even run static analysis. Nope, we're going to simply check. Okay. Are you saying that this get magic method is going to return anything? Okay, that's okay. Oh, but I want to my guess is that you specifically return a string. That's also okay. As to how to pronounce that liskov mistook principle, right? 

Derick Rethans  6:36  
	The liskov substitution principle.

Gabriel Caruso  7:26  
	Yeah. And so this is what we're going to basically do with this RFC, there's going to be voted. We're going to simply check if you're using the right types, because, in my opinion, magic methods are a foundation in PHP. As we have theses methods across different code bases across different projects from different behaviours, at least when I'm looking at that code. Okay, I'm looking at this magic method. I know what parameters does it take. I know what return does it have. This is worth less tab to the bug are trying to understand what is happening. Because today maybe I'm debugging a toString method there is return an integer. And I'm like, okay, this is the bug, it's supposed to return a string. But once you ensure those all those signatures, is one less bug that we're gonna have in production.

Derick Rethans  8:17  
	When are these signatures being ensured?

Gabriel Caruso  8:19  
	It's not at compile time because he does not have a compile time. But he's when the Zend machine is compiling the code, we have a very specific method that is checking all the modifiers. So for example, the signature that we mentioned before so all the magic methods needs to be public. This has been checked, for example, they callStatic magic method needs to be static. So this has also been checked. And then I'm extending how do we check for signatures for param types and also for return types. So during compilation of the Zend VM.

Derick Rethans  8:52  
	Taking as example callStatic in the RFC, I see that the name has to be a string and the arguments has to be an array. What happens if you use a different type there?

Gabriel Caruso  9:01  
	So nowadays if you use a different type that's allowed. So if you say there, you're going to receive an integer, and you're going to receive a string. This is allowed today. And this is what I mentioned about when you are debugging or analyze different code bases, you're going to be like why in the documentation says that we need to receive a string and an array, and there's this specific code base is receiving a string and an integer. So this is what kinds of mismatch I want to avoid. Of course, when using types, because we also know that PHP in some projects does not use types. And that's perfectly fine. If you're not using types, I'm not going to ask you, hey, you need to type those magic methods. Well, what I'm going to do is okay, you're using types and I need to make sure they're using right otherwise this is going to be a mess.

Derick Rethans  9:47  
	If you type it; say use an integer for the name of underscore underscore get, will give you a warning or a compile error, or parse error? What what kind of feedback which you get back from that?

Gabriel Caruso  9:59  
	While you are running your code, as soon as that class get referenced, we're going to check. Is not when is initiated, when is not when is called, as soon as I think the autoload detects that class is gonna parse, is going to identify, and then is going to compile and during the compile time that we mentioned. We're going to identify that. So it's going to be early in the stages. Perhaps as soon as you run something or you would upset me, you're going to have that feedback saying: hey, this is not compatible with what we are expecting. 

Derick Rethans  10:32  
	Is that a warning or type error?

Gabriel Caruso  10:34  
	It's going to be a fatal error, because this is what we are constantly returning with the destructors and constructors.

Derick Rethans  10:41  
	Yeah, we alluded to mixed already a little bit and the RFC mentioned mixed a few times, of course mixes in the type and PHP yet. So what do you want to do about that?

Gabriel Caruso  10:51  
	Today we are 11th of May of 2020. Right now we have an RFC voting in PHP to introduce the mixed type. I'm not going to say if I agree or disagree, it's being voted. If that RFC gets accepted then I have already talked with the authors of the that RFC, I'm going to wait until they merge into master. I'm going to rebase and readapt to my RFC, to have those mixed types. And there we go PHP eight probably can have mixed, and probably can already have the usage of mixed in the magic methods. So either No, I'm gonna need to wait for the end of their RFC. If it's approved, there go I need to rebase my PR. In the other case, we are going to keep as comments because we can't ensure that in the compile time with the VM.

Derick Rethans  11:41  
	At the moment, it looks like that vote will and in May 21. The current votes are 35 to six for passing. So it looks like that will go through

Unknown Speaker  11:50  
	And then I need to rush because we have the upcoming feature freeze of PHP eight. So I need to make sure that I start to vote and implement my RFC before that time.

Derick Rethans  12:00  
	Feature freeze should be by the end of July. So I think you have plenty of ime pfor that. And of course you have a release manager, you can make an exception. That's how that works. Usually adding extra checks will have impact to existing code. Is there much impact to existing code here as well?

Gabriel Caruso  12:18  
	That was the interest question that I made myself. Okay, I'm going to touch the magic methods of PHP. I'm going to break some code in an issue identified those breaking changes in an each map in the RFC. How do I map across many projects, many libraries, many PHP codes out there? How do I do that? I remember that Nikita back in his RFC about the parenthesis origin, like how do we present this ordering and yada yada yada. He made a script, where he went through I think was the top thousand or top 10,000 packages. On packagist, that is the official composer package provider and he identified everything, and ask myself how he did that. And actually was very easy. He just cloned other repositories. He instantiate a new PHP parser instance that is his magic parser. That is behind PHP Stan, is behind psalm, is behind a lot of infection, a lot of big projects, where you analyze the code. So you have a code base where you can analyze and say: Do I have magic methods wrong? And then I run this script, identify, I think six or seven types that were not perfect. Three of them. I have already submitted a request because we're in PHP Unit and I said to Sebastian: hey, this actually is not right. Because I'm proposing this RFC, he was like: Okay, perfect, let's merge it. And the other cases are the cases that I mentioned. For example, with get. Get, you need to return mixed but by the LSP, you can nail down to an integer or a string. So there you go, at least in the top 10,000 packages of composer is not going to be a breaking change. But of course, it's going to be breaking change for people that I can't map. So this is why it's mentioned the RFC that if you're using types with magic methods wrong, we're going to warn you.

Derick Rethans  14:13  
	But at least it's an easy thing to check for. Because even running all your files through PHP minus L should catch it.

Gabriel Caruso  14:20  
	Yeah, there you go.

Derick Rethans  14:22  
	So it's a very easy to check for something. You provided a link to Nikita's script where he checks for those ternairies, do you have a version of your own script available as well?

Gabriel Caruso  14:33  
	That's interesting. I thought the RFC was updated. So I'm going to update the RFC, because I do have the script locally. 

Derick Rethans  14:39  
	Then I can link to it for the podcast as well. 

Gabriel Caruso  14:41  
	Okay, perfect. 

Derick Rethans  14:42  
	In the future, are you thinking of extending checks to a few more things? 

Gabriel Caruso  14:46  
	So this is something that I fought about this RFC, like how much you want to break and explode people's code. And I think starting with checking types in the signature is the first step. The next step is to actually check the return type. We do that with toString. So for example, although you have type right for maybe, some logic or something is wrong, you're returning an integer. There is a check before the actual type saying you're supposed to return a string you're return an integer. And actually, there is a check in the magic method saying this magic method was supposed to return a string. I think is gonna break even more code because then it's something that I can't measure. So I was like: Okay, let's first start with types and then we can give it next step that is: okay, inside this method, what is being returned, okay, is something different from the signature: explode. You're returning something that I was not supposed to return. But this is not a fight that I'm going to pick. So I leave it up for the next major version of PHP or whatever.

Derick Rethans  15:49  
	Wouldn't PHP's strict versus weak type mechanism already catch these things. So from debugInfo, if you would type that as returning an array, and then you end up returning an object, which is not necessarily wrong, just not what you expected. PHP's return type checking mechanism should already catch that for you.

Gabriel Caruso  16:13  
	If you have a magic method typed. If it's not typed, so we can say that some efforts do have that check. And then we're going to expand when we don't have types in the signature.

Derick Rethans  16:24  
	That's clear now. Do you have anything else to add?

Gabriel Caruso  16:27  
	The only thing that I want to add that is, I have created another RFC, and this is something that I always tell everyone that is easy to do; is not impossible. Anyone can go there, identify a bug or catch a bug report and then try to fix it. And this is what I'm doing. Like I'll do them to release many of PHP eight. I'm also fixing bugs, improving documentation and everything else. This is something that I try to do and share with everyone. So everyone can also be the next one contributor to the to PHP and it's evolution.

Derick Rethans  16:57  
	This RFC isn't out for voting yet. You set you want to sort of wait until mixed gets passed or not. What's the reception been so far?

Gabriel Caruso  17:05  
	So I asked a couple of key members of the PHP community, both internal and external people. They agree, they said that the right approach is to first check for the signature, because if someone is already using types, that project is type friendly, so we can at least play with that. But if someone is not typing, then this is a bigger fight. And then we're going to talk about that in the future.

Derick Rethans  17:29  
	Thank you, Gabriel for taking the time this morning to talk to me. I've learned a few more things about this RFC, so that's always good to know. And again, congratulations of being the PHP eight release manager together with Sara.

Gabriel Caruso  17:41  
	Thank you very much. Also thank you for inviting me for this new podcast is amazing. Always listen to all these famous people of PHP that talked with you. And I'm like, Whoa, Derick has invited me this is going to be so much fun. Thank you very much.

Derick Rethans  17:55  
	Thanks for listening to this installment of PHP internals news, the weekly podcast dedicated to demystify the development of the PHP language, I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to Dderick@phpinternals.news. Thank you for listening, and I'll see you next week.


Show Notes
----------

- RFC`: `Ensure correct signatures of magic methods <https://wiki.php.net/rfc/magic-methods-signature>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
