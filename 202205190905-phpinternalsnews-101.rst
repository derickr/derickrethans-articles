PHP Internals News: Episode 101: More Partially Supported Callable Deprecations
===============================================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2022-05-19 09:05 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin101
   :Enclosure: media/pin-101.mp3

In this episode of "PHP Internals News" I talk with Juliette Reinders Folmer
(`Website <http://adviesenzo.nl/>`_, `Twitter <https://twitter.com/jrf_nl>`_,
`GitHub <https://github.com/jrfnl>`_) about the "More Partially Supported
Callable Deprecations" RFC that she has proposed.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-101.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:14  
	Hi, I'm Derick. Welcome to PHP internals news, the podcast dedicated to explaining the latest developments in the PHP language. This is episode 101. Today I'm talking with Juliette Reinders Folmer, about the Expand Deprecation Notice Scope for Partially supported Callables RFC that she's proposing. That's quite a mouthful. I think you should shorten the title. Juliette, would you please introduce yourself?

Juliette Reinders Folmer  0:37  
	You're starting with the hardest questions, because introducing myself is something I never know how to do. So let's just say I'm a PHP developer and I work in open source, nearly all the time.

Derick Rethans  0:50  
	Mostly related to WordPress as far as I understand?

Juliette Reinders Folmer  0:52  
	Nope, mostly related to actually CLI tools. Things like PHP Unit polyfills. Things like PHP Code Sniffer, PHP parallel Lint. I spend the majority of my time on CLI tools, and only a small portion of my time consulting on the things for WordPress, like keeping that cross version compatible.

Derick Rethans  1:12  
	All right, very well. I actually did not know that. So I learned something new already.

Juliette Reinders Folmer  1:16  
	Yeah, but it's nice. You give me the chance now to correct that image. Because I notice a lot of people see me in within the PHP world as the voice of WordPress and vice versa, by the way in WordPress world to see me as far as PHP. And in reality, I do completely different things. There is a perception bias there somewhere and which has slipped in.

Derick Rethans  1:38  
	It's good to clear that up then. 

Juliette Reinders Folmer  1:39  
	Yeah, thank you. 

Derick Rethans  1:40  
	Let's have a chat about the RFC itself then. What is the problem that is RFC is trying to solve?

Juliette Reinders Folmer  1:46  
	There was an RFC or 8.2 which has already been approved in October, which deprecates partially supported callables. Now for those people listening who do not know enough about that RFC, partially supported callables are callables which you can call via a function like call_user_func that which you can't assign to variable and then call as a variable. Sometimes you can call them just by using the syntax which you used for defining the callable, so not as variable but as the actual literal.

Derick Rethans  2:20  
	And as an example here, that is, for example, static colon colon function name, for example. 

Juliette Reinders Folmer  2:26  
	Absolutely, yeah. 

Derick Rethans  2:27  
	Which you can use with call_user_func by having two array elements. You can call it with literal syntax, but you can't assign it to a variable and then call it. Do I get that, right?

Juliette Reinders Folmer  2:36  
	Absolutely. That's it. There's eight of those. And basically, the original RFC from Nikita proposed to deprecate support for them in 8.2,  add deprecation notices and remove support for them altogether in PHP nine. And the original RFC explicitly excluded two particular things from those deprecation notices. That's the callable type and using the syntaxes in combination with the is_callable function, where you're checking if the syntax is callable. The argument used in the original RFC was to keep those side effect free. The problem with this is that with the callable type, this means you go from absolutely no notice or nothing, to a fatal error in PHP 9. Everything works, and you're not getting any notification. But in PHP 9, its fatal error at the moment that callable is being passed to a function.

Derick Rethans  3:31  
	This is the callable type in function declarations.

Juliette Reinders Folmer  3:33  
	Yeah, absolutely. And with is_callable, I discovered a pattern in my wanderings across the world where people use the syntax in is_callable, but then use it in a literal call. So not using call_user_func, not using a variable to call it, but it's callable static double colon method name, and then called static double colon method name as literal. And that pattern basically, for valid calls would mean that that function would no longer be called in PHP 9 without any notification whatsoever.

Derick Rethans  4:13  
	So it's a silent change that you can't detect at all.

Juliette Reinders Folmer  4:17  
	Yeah, which to me sounded dangerous. I started asking some questions about that. But six weeks ago, the conclusion was, well, maybe this should be changed. But as this was explicit in the original RFC, we can't just change it. We need to have a new RFC to basically amend the original RFC and remove the exception for these two situations and allow them to throw deprecation notices.

Derick Rethans  4:44  
	What are you proposing to change with this RFC than?

Juliette Reinders Folmer  4:47  
	What this RFC is proposing is simply to remove the exception that the callable type and is_callable are not throwing a deprecation notice. This RFC is proposing that they should throw a deprecation notice, so that more of these type situations can be discovered in time for PHP 9 to prevent users getting fatal errors.

Derick Rethans  5:08  
	Now, of course, we have no idea when PHP nine is actually showing up, but I don't think it will be this year. Well, I know it won't be this year, and it certainly won't be be next year neither, I think.

Juliette Reinders Folmer  5:17  
	That's all the same. I mean, it makes there'll be two, three years ahead, but it doesn't really make sense to have the main deprecation in 8.2 and then have the additional deprecation in 8.4 or something.

Derick Rethans  5:29  
	Absolutely.

Juliette Reinders Folmer  5:30  
	It's a lot more logical to have it all in in the same version. Because it's all related. It's basically the same thing without the exception for callable type. And is_callable.

Derick Rethans  5:42  
	Although there is no current application, would this be able to be found if you had like a comprehensive test suite?

Juliette Reinders Folmer  5:48  
	Yes and no. Yes, you can find this with a test suite. But one, you're presuming that there are tests. Two, that the tests covered the effected code with enough path coverage. Three, imagine a test you've written yourself at some point in the past where which affected callables, you might have, you know, a data provider where you say: Okay, valid callable function, which you've mocked or, you know, closure, which you've put in and second, this function does not exist. Okay, so now you're testing this function, which at some point in its logic has a callable, and expects that type to receive that type. But are you actually testing with the specific deprecated partially supported callables? Even if you have a test, and the test covers the affected code, if you do not test with one of these eight syntaxes, which has been deprecated, you still cannot detect it. And then, four, you still need to make sure that the tests are routinely run, and in open source, that's generally not a problem. Most open source projects, use GitHub actions by now to run the tests automatically on every pull request, etc. But, have the tests been turned on to actually run against PHP 8.2. Are the tests run against pull requests? I mean, there are still plenty of projects, which don't do that kind of thing. Yes, you can detect it with a good test suite. But there's a lot of caveats when you will not detect it. And more importantly, you will not be able to detect it until PHP 9.

Derick Rethans  7:23  
	Yes, when your code and stops behaving as you were expecting it to be.

Juliette Reinders Folmer  7:28  
	Yeah, because in 8.2, you're gonna get deprecation notices for everything else, but these two situations. But not in 8.2, not in 8.3, not in 8.4, and then whatever eights we're gonna get until nine, you will not be able to detect without deprecation notices, until PHP 9 actually removes support for these partials deprecated callables. Yes, but no.

Derick Rethans  7:53  
	We already touched a little bit on how you found out for the need for this RFC or for changing behaviours. But as people have stated in the past, adding deprecation notices is a BC break. That's a subject that we will leave for some other time because I don't necessarily believe that. But would, the changes in your RFC not add more backwards compatibility issues?

Juliette Reinders Folmer  8:14  
	The plain and simple the backward compatibility break is in the original RFC. That's where the deprecation is happening. This RFC just makes it clearer where the BC break is going to be in PHP 9. It's not PHP 8.2, which has a backward compatibility break. It's PHP 9 which will have to backward compatibility break. Yes, I've heard all those arguments, people saying deprecation notes are BC break, no they're not. But they are annoying. And Action List, to for everything that needs to be fixed before 9. Given big enough projects, you cannot say: Okay, I'm gonna do this at the last moment, just before 9 comes out. It literally means 10 months of the year I for one am working on getting rid of deprecation notices in project to prepare them all to be ready for PHP 9 when PHP 9 comes round.

Derick Rethans  9:06  
	But it's still better to have them than to not,.and then you code starts breaking right? Because that is exactly why you're proposing this RFC as far as I understand.

Juliette Reinders Folmer  9:16  
	Yes, absolutely. I mean, I'm always very grateful for deprecation notices, but it would be nice if we had fewer changes, which would cost them, for a year or two, so I can actually catch my breath again.

Derick Rethans  9:28  
	I think PHP 8.2 will have fewer of these changes in there. There will still be some of course.

Juliette Reinders Folmer  9:35  
	Well, I mean, this one is one deprecation. And then we have the deprecated Dynamic Properties and that one is already giving me headaches before I can actually start changing it in a lot of projects. I'm not joking, that one really is going to cause a shitload of problems.

Derick Rethans  9:51  
	It's definitely for products have been going on for so long, where dynamic properties are used all over the place. And I see that in my own code as well. I just noticed this morning does actually breaks Xdebug.

Juliette Reinders Folmer  10:03  
	I know it's currently breaking mockery, we're gonna have to have a discussion how to fix that or whether or not to fix it. If Mockery is broken, that means all your tests are broken. So the test tooling needs to be fixed first.

Derick Rethans  10:18  
	That's always the case, if you work with CLI tools that make people run code on newer PHP versions, that's always a group of tools that needs to be upgraded first, which is your sniffers, your static analysis, your debugger still will always need to go first.

Juliette Reinders Folmer  10:27  
	Which is why I look at things a lot earlier, probably then the majority of people. I mean, I see him huge difference between the open source and closed source community. For open source, I started looking at it well, I've been looking at 8.2 since the beginning. And I started running tests for all the CLI tools. As soon as 8.1 comes out, 8.2 gets added to the matrix for running in continuous integration. And then for applications, it gets added like in you know, once alpha 1-2-3 has come out. For the extensions, it gets added in September once the first RFC gets added. And all of them are trying to get ready before the release of 8.1 or 8.2 in this case, because you do not know as an open source maintainer, what version people are going to run your code on. And you can say IP, you can manage that via Composer, no you can't. Sorry, you can only do that if your users are actually installing via Composer. If your users are downloading a zip file, and uploading it to a web host via FTP, there's literally no way you can control whether they're running on 8.0, or 8.1, except for maybe during check: You cannot run on 8.1 yet.

Derick Rethans  11:52  
	Upgrading software with version support is an issue that's been going on for 40 years and will go on for at least another 40 more. This is not a problem that we can solve easily.

Juliette Reinders Folmer  12:03  
	But what I see there is like the closed source community is like, oh, yeah, but you know, by the time I want to upgrade my server to 8.1, or 8.2, I just run Rector and all will be fine. And I'm like, yeah, sorry, that does not work for open source. We need cross version compatible with multiple versions. And I try to keep that range of version small for the project, I initiate, I don't always have control over it. If for instance, one of the projects I maintain is Requests. And that's a project which does HTTP requests. It's used by WordPress, it cannot be let go of the minimum of 5, PHP 5.6, until WordPress, lets go of that.

Derick Rethans  12:48  
	Well, the alternative is that WordPress uses an older version until it can let go of it.

Juliette Reinders Folmer  12:54  
	Yeah, the only problem then is that we don't want to maintain multiple stable branches. For security fixes.

Derick Rethans  13:03  
	For Xdebug, what I do is I support what the PHP project support when a PHP release comes out, which is a bit longer than PHP itself usually, but not by much more than a year or two.

Juliette Reinders Folmer  13:15  
	I understand that. And I mean, I applaud Sebastian for at some point, having the guts to say to the community, I'm limiting the amount of versions I'm supporting. And I'm sticking to the officially supported PHP versions. That does not mean that that didn't give a large part of community which does need to support a wider range of PHP versions a problem. I fully support that people limit the amount of fish and stay support and like Sebastian, who I know got half the community up in arms against him when he said, I'm not going to support older PHP versions any more. It did create a problem and but the problem which I've tried to solve for instance with the PHP unit polyfills, which now is solvable by using the PHP Unit polyfills in quite a transparent way, which is helpful for everyone. It takes the complainers of Sebastian's back, and at the same time, it allows them to run the test.

Derick Rethans  14:10  
	I think another good thing that Sebastian recently has done is make sure that deprecation notices are no longer failing your tests.

Juliette Reinders Folmer  14:17  
	I don't agree. The thing is, I do understand him making that change. But changing that default from not showing those deprecation notices or not not allowing deprecation notes to fail the test, or not in a patched version, I don't think was the right thing to do. That should have been in a minor, let alone or maybe even in a major not in a 9.5.18 patch version. Also with the whole idea, I mean, again, this is very much an open source versus closed source discussion for closed source I completely understands that people say I don't want to know until I actually am ready to upgrade to that version.

Derick Rethans  14:56  
	I understood it's more of a difference not necessarily between open and closed source, but rather between library maintainers and application maintainers. And the applications can then also be closed source.

Juliette Reinders Folmer  15:06  
	The open source work I work in, I mean, I do want to see them. And the problem with the deprecation notices anyhow, and I've seen various experiments via Twitter fly past for the past year. Say you build something on top of something else, you want to see the deprecation notices and the errors which apply to your code. We don't want to see the ones which come from the framework on which you build on top. The silencing deprecation notices or not, allow tests to error out on deprecation and just not solve that problem.

Derick Rethans  15:39  
	The only thing it does is make things a little bit less noisy so that fewer people complain to library authors isn't it? That's pretty much what it does.

Juliette Reinders Folmer  15:48  
	The thing would I see what it has done is that people think the tests are passing.

Derick Rethans  15:54  
	Well they are passing, but...

Juliette Reinders Folmer  15:56  
	Yeah, but most people don't read change logs of PHP unit, especially as releases don't get actually have to change log included. When PHP Unit releases its actual release, it doesn't actually post a release on GitHub. So people who watch the PHP unit repo for releasing doesn't, don't get notifications, let alone a changelog. So they actually have to go into the repo to find out what has changed. Most people don't do that. They just get you know depend-a-bot update, which won't say much, because again, it doesn't have release information.

Derick Rethans  16:28  
It'd be nice, maybe if Composer ,when you upgrade packages, that it can show like the high level changes when you do an upgrade. The Debian project does that if you upgrade packages that have like either critical or behavioural changes, you actually get a log when you run the update.

Juliette Reinders Folmer  16:43  
	And then the change should have been in major or minor, because in a patch release, you don't expect it kind of changes. I also know the struggle there. They've been going through to four PHP units and which is similar to what I'm struggling with with the amount of changes from PHP 8.0 and 8.1 which has to be deal dealt with. Projects are being delayed, we're having trouble keeping up as an open source community, we still need to look after our own mental health as well.

Derick Rethans  17:10  
	What has the feedback been to far on the RFC or non?

Juliette Reinders Folmer  17:13  
	The feedback on this particular RFC has been next to nothing. And that's not surprising. I mean, basically, the discussion has happened before. And I started the discussion six weeks ago, eight weeks ago, which led to this RFC. So far the responses, which I have seen, either on Twitter or in private or in our people will read through the RFC. They're like, yeah, it makes sense.

Derick Rethans  17:37  
	I think this is quite a nicer way of getting RFCs done, you discuss them first. And if there's then found a need actually spend a time on writing an RFC. In other cases, the other way around happens, right? People write a long, complicated RFC, and then complain that nobody wants to talk about it.

Juliette Reinders Folmer  17:53  
	When I started the previous discussion, it was I see this, I noticed this, was this discussed? And then I got back: yeah, nobody actually discussed the previous RFC and I'm like: Okay, so what's this whole point about under discussion if nobody's discussing?

Derick Rethans  18:10  
	Well, you can't force people to talk, of course.

Juliette Reinders Folmer  18:14  
	It does make me wonder, again, what we were talking about before, people who work in managed environments versus people who will have to support multiple PHP says, I sometimes wonder how many people who actually have voting rights work in those closed environments, and think, you know, upgrading is something you do with Rector. Now I have a feeling that often open source gets a little forgotten.

Derick Rethans  18:38  
	Yeah, that's perhaps true. Thank you for taking the time this morning to talk about this RFC then.

Juliette Reinders Folmer  18:44  
	Thank you Derick for having me. It was a pleasure to do you like always.

Derick Rethans  18:49  
	Thanks.

Derick Rethans  18:54  
	Thank you for listening to this installment of PHP internals news, a podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next time.


Show Notes
----------

- RFC: `Expand deprecation notice scope for partially supported callables <https://wiki.php.net/rfc/partially-supported-callables-expand-deprecation-notices>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
