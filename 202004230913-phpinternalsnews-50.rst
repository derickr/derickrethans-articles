PHP Internals News: Episode 50: The RFC Process
===============================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-04-23 09:13 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin050
   :Enclosure: media/pin-050.mp3

In this episode of "PHP Internals News", Henrik Gemal (`LinkedIn
<https://dk.linkedin.com/in/gemal>`_, `Website <gemal.dk>`_) asks **me** about
how PHP's RFC process works, and I try to answer all of his questions.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-050.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:16
	Hi, I'm Derick. And this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 50. Today I'm talking with Henrik come out after he reached out with a question. You might know that at the end of every podcast, I ask: if you have any questions, feel free to email me. And Henrik was the first person to actually do so within a year and a half's time. For the fun, I'm thinking that instead of I'm asking the questions, I'm letting Henrik ask the questions today, because he suggested that we should do a podcast about how the RFC process actually works. Henrik, would you please introduce yourself?

Henrik Gemal  0:52
	Yeah, my name is Henrik Gemal. I live in Denmark. The CTO of dinner booking which does reservation systems for restaurants. I've been doing a PHP development for more than 10 years. But I'm not coding so much now. Now I'm managing a big team of PHP developers. And I also been involved in the the open source development of Mozilla Firefox.

Derick Rethans  1:19
	So usually I prepare the questions, but in this case, Henrik has prepared the questions. So I'll hand over to him to get started with them. And I'll try to do my best to answer the questions.

Henrik Gemal  1:27
	I heard a lot about these RFCs. And I was interested in the process of it. So I'm just starting right off here, who can actually do an RFC? Is it anybody on the internet?

Derick Rethans  1:38
	Yeah, pretty much. In order to be able to do an RFC, what you would need is you need to have an idea. And then you need access to our wiki system to be able to actually start writing that, well not to write them, to publish it. The RFC process is open for everybody. In the last year and a half or so, some of the podcasts that I've done have been with people that have been contributing to PHP for a long time. But in other cases, it's people like yourself that have an idea, come up, work together with somebody to work on a patch, and then create an RFC out of that. And that's then goes through the whole process. And sometimes they get accepted, and sometimes they don't.

Henrik Gemal  2:16
	How technical are the RFCs? Is it like coding? Or is it more like the idea in general?

Derick Rethans  2:23
	The idea needs to be there, it needs to be thought out. It needs to have a good reason for why we want to add or change something in PHP. The motivation is almost as important as what the change or addition actually is about. Now, that doesn't always get us here at variable. In my opinion, but that is an important thing. Now with the idea we need to talk about what changes it has on the rest of the ecosystem, whether they are backward compatible breaks in there, how it effects extensions, or sometimes how it effects OPCache. Sometimes considerations have to be taken for that because it's, it's something quite important in the PHP ecosystem. And it is recommended that it comes with a patch, because it's often a lot easier to talk about an implementation than to talk about the idea. But that is not a necessity. There have been quite some RFCs where the idea was there. But it wasn't a patch right away yet. It is less likely that these RFCs will get accepted, because in order to get something into PHP not only needs to be there a good idea, that also needs to be there a good implementation of it. If you have been a long term contributor to PHP, then you should know how to write a patch yourself. In other cases, you'll see people that have an idea try to find somebody else to do and work on the implementation together. But all RFCs, if they get accepted. It's always pending a good implementation.

Henrik Gemal  3:52
	How is an RFC actually done? Is that like a template you fill out or is it like a website or how does it work?

Derick Rethans  3:59
	Our Wiki, I will add a link to that in the show notes, has a template of how to create an RFC. It has a set set of sections. There's always an introduction that basically lays out what it is about or why this change is being made. Then there is often a proposal of what the change actually is. And then there's a few sections that are sometimes empty or sometimes are filled in such as, at least backwards incompatible changes, for which PHP version is been targeted, what the impact is to all the parts of the PHP ecosystem. But these things are not always necessary, because they don't always make sense to do right? If you want to add a new syntax to PHP, then that almost never influences existing extensions, but it will influence OPCache, for example. And then there's also often things like open issues, things we haven't quite thought through yet. A bit of a discussion, discussion bits will get filled in after people in the PHP internals list, which I'm sure we'll get to in a moment, come up with better ideas or alternatives sometimes, and then things like future scope will also be part of the template. We don't really require a very rigid approach to this, but we do appreciate if all the sections are filled in, or at least thought about in such a way that there's either information or not information. And then at the end, there's often a proposed voting choice. Everything at the moment needs to pass by two thirds majority before it gets accepted. So yeah, those are the things in the template itself. But the template is important. And you do need to fill it in, if you want to propose an RFC.

Henrik Gemal  5:33
	Are all RFCs public or do you have like private RFCs?

Derick Rethans  5:38
	All RFCs have to be public, otherwise they can't be voted on. But some RFCs start out of just a conversation with a few developers coming up with an idea. In the last few months, some more complicated RFC start out on a GIT repository. As a pull request, they never get merged anywhere. Because on GitHub, it makes it much easier to comment on specific sections for adopting feedback. Instead of having large discussions on the PHP internals mailing list, where sometimes comments might just get lost because there's too much text in there. Even though these RFC start out, while they're still sort of public, but nobody knows about them. In the end, they will always have to be public otherwise there won't be any voting, done on it, and it won't get accepted.

Henrik Gemal  6:27
	Where's the RFC sent to and who's kind of in charge of the RFC? Is the one that makes the RFC or is it like a RFC commander?

Derick Rethans  6:37
	The person that makes the RFC is responsible for guiding it through the whole process that we have. Once they are finished, there is a requirement for you emailing the PHP internals list with a specific prefix, which I think is RFC in square brackets. And then that starts a minimum discussion periods of two weeks. That discussion period might end up longer, in cases, lots of things to talk about or discuss or lots of disagreements, but the discussion period has to be a minimum of two weeks on the PHP internals mailing list.

Henrik Gemal  7:09
	I was wondering a little bit about the priority RFCs because I see RFCs as like, a little bit like feature requests. So wondering who actually decides on the priority of an RFC?

Derick Rethans  7:23
	Nobody really decides on the priority. Multiple RFCs can go through the process at the same time, you don't really have a priority of which one is more important than others. So yeah, there's nothing really there for it.

Henrik Gemal  7:35
	I was just wondering if it's done like a normal project, you know, there might be many RFCs at the same time. I'm wondering how many kind of RFCs are there at the moment, are we talking 10 or are talking thousands?

Derick Rethans  7:50
	This depends a bit on where in PHP's release cycle we are. PHP should get released at the end of November or the start of December. In all PHP seven releases that actually has happens. Usually the period between December and March, there will be like maybe one or two a week, which is great because that makes it possible for me to pick the right one to make an episode out for the podcast. At the moment, there are 10 outstanding RFCs. That means there are so many that I don't actually have enough time to talk about all of these on the podcast. However, they are often more just before we go to feature freeze, which happens at the end of June. So there's still two months to go. But you also see that over the last two years, there's a lot more smaller RFCs than there are big RFCs. So big RFCs like union types. They tend to be early in a release cycle, where smaller RFCs, as an example here, there's currently an RFC that there is no episode about, that suggests to do a stricter type checks for arithmetic or bitwise operators. Those are tiny, tiny changes. And in the last two years, there have been more and more smaller RFC than bigger RFCs because they tend to limit the amount of contention that people can disagree with and hence, often makes it easier to then get accepted. That is a change that I've been seeing over the years. But no, there are no thousands for each PHP version, I would say on average, there's about one a week, so about 50.

Henrik Gemal  9:19
	I want to get a little bit into the voting part, because that sounds kind of interesting, who can actually vote?

Derick Rethans  9:28
	After the two week minimum discussion period is over on the PHP internals mailing list, an RFC author can decide to put up the RFC for a vote. And that also requires you then to send an email to the PHP internals mailing list prefixing your subject with the word vote in capital letters. Now at this moment, you unfortunately see that people start paying attention to the RFC. Instead of doing that during the discussion period. At a moment of vote gets called you shouldn't really change RFC unless it's for like typos or like minor clarifications to things, you can't really change syntax in it for example. People can vote our people with a PHP commit access. And that includes internals developers, documentation contributors, and people that do things in the infrastructure. Everybody that has a PHP VCS account and VCS, version control system, that used to be CVS and now then SVN, and now GIT, as well as people that have proposed RFCs. So the group that technically could vote is over 1000 big, but the amount of people that vote is very much under 50 most of the time. We don't really have any criteria beyond you have to have an account to be able to vote in PHP RFCs.

Henrik Gemal  10:43
	How is the voting actual done?

Derick Rethans  10:47
	Since about last year, each RFC needs to be accepted, with a two thirds majority. On each RFC on the wiki, once a vote gets called you as an RC alter needs to include a small code snippets that then creates a poll. Very often do we want this? Or do we not want this? So it's a yes or no question. But sometimes there are optional votes, whether we want to do it a specific way, or another specific way. Sometimes that allows you to then select between different syntaxes. I don't think that is necessarily a good idea to have. I think the RFC author should be opinionated enough about picking a specific syntax. It is probably better to have a secondary vote as we call those. Those secondary votes don't to have two thirds majority is often which one of the options wins out of these. But the main RFC won't get accepted, unless there's a two thirds majority with a poll done on the wiki.

Henrik Gemal  11:46
	What happens after the vote? You know if it's both if it's Yes or no?

Derick Rethans  11:53
	I'll start with the easy case, the no case. If it's a no then the RFC gets rejected. That also means that sometimes an RFC fails for a very specific reason. Maybe some people didn't like the syntax, or it was like a one tweak where it would behave in a wrong way or something like that. But as a rule that says that you cannot put the same subject back up for discussion for six months, unless there are substantial changes. Now, this has happened with scalar type hints, for example, and a few other big ones. If an RFC gets accepted, then pending on whether there is an implementation, the implementation will get set up as a pull request to the PHP project on GitHub. And then the discussion about the implementation starts. If the implementation doesn't get to the point where it is actually good enough, or whether it can actually not be implemented in a way that it doesn't impact performance, it still might end up failing, or might not get merged. And in some cases, it means that a feature will get added at some point but it might not be necessarily in the PHP version that it got targeted for. I don't actually have an example for that now. If the implementation is already good and already discussed it can get merged pretty much instantly. And then it will be part of the next PHP version.

Henrik Gemal  13:08
	How many RFCs voted on every year? And what majority voted yes or no?

Derick Rethans  13:16
	I don't have the stats for that. But there is a website called RFC watch, where you can see which RFCs had been gone through and which one had been accepted or not, in a nice kind of graph way. I will add a link in the show notes for that. I would guess that during a year, about 50 RFCs are voted on. And I will think that about half of them are passing. But that's a guess I don't have the stats.

Henrik Gemal  13:42
	Thank you very much for the answers. It brought me closer to the whole process of the PHP development. You have any other things to add?

Derick Rethans  13:52
	I don't think so at the moment. I think what we she'd be a bit careful about is that although we're getting closer and closer to feature freeze at the end of June. We currently have just elected the new PHP eight zero release managers, but I keep the names secret, because this podcast is recorded in the past. They are going to be responsible now for doing all the organisatorical work for PHP eight zero. And that also means that feature freeze will happen at the end of June somewhere. And I expect to see a bunch of RFCs coming up with just enough time to make it into PHP eight zero, or not. So that's going to be interesting to see what comes up there. But other than that, I think we have explained most things in the RFC process now. And I thought it was a fun thing for once somebody else asking the questions and me giving the answers. And I think in the future, I think I would like to do like a Q&A session where I have multiple people asking questions about the PHP process. I also thought this was a good experiment and thanks for you taking the time to ask me all dthese questions today.

Henrik Gemal  15:00
	No problem. I love your podcast. I listen to it whenever I bike to work. It's nice to get some insights into the PHP development.

Derick Rethans  15:10
	Yeah, and that is exactly why I started it. Thank you Henrik for taking the time this morning to ask me the questions. And I hope you enjoyed it.

Henrik Gemal  15:18
	Thank you very much for having me on the show.

Derick Rethans  15:22
	Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.

Show Notes
----------

- `How to create an RFC <https://wiki.php.net/rfc/howto>`_
- `List of RFCs <https://wiki.php.net/rfc>`_
- `php RFC Watch <https://php-rfc-watch.beberlei.de/>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
