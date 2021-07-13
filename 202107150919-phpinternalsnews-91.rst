PHP Internals News: Episode 91: is_literal
==========================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-07-15 09:19 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin091
   :Enclosure: media/pin-091.mp3

In this episode of "PHP Internals News" I chat with Craig Francis (`Twitter
<https://twitter.com/craigfrancis>`_, `GitHub <https://github.com/craigfrancis>`_,
`Website <https://www.craigfrancis.co.uk/>`_), and Joe Watkins (`Twitter
<https://twitter.com/krakjoe>`_, `GitHub <https://github.com/krakjoe>`_,
`Website <https://blog.krakjoe.ninja>`_) about the "is_literal" RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-091.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:14
	Hi, I'm Derick. Welcome to PHP internals news, a podcast dedicated to explaining the latest developments in the PHP language. This is Episode 91. Today I'm talking with Craig Francis and Joe Watkins, talking about the is_literal RFC that they have been proposing. Craig, would you please introduce yourself?

Craig Francis  0:34
	Hi, I'm Craig Francis. I've been a PHP developer for about 20 years, doing code auditing, pentesting, training. And I'm also the co-lead for the Bristol chapter of OWASP, which is the open web application security project.

Derick Rethans  0:48
	Very well. And Joe, will you introduce yourself as well, please?

Joe Watkins  0:51
	Hi, everyone. I'm Joe, the same Joe from last time.

Derick Rethans  0:56
	Well, it's good to have you back, Joe, and welcome to the podcast Craig. Let's dive straight in. What is the problem that this proposal's trying to resolve?

Craig Francis  1:05
	So we try to address the problem where injection vulnerabilities are being introduced by developers. When they use libraries incorrectly, we will have people using the libraries, but they still introduce injection vulnerabilities because they use it incorrectly.

Derick Rethans  1:17
	What is this RFC proposing?

Craig Francis  1:19
	We're providing a function for libraries to easily check that certain strings have been written by the developer. It's an idea developed by Christoph Kern in 2016. There is a link in the video, and the Google using this to prevent injection vulnerabilities in their Java and Go libraries. It works because libraries know how to handle these data safely, typically using parameterised queries, or escaping where appropriate, but they still require certain values to be written by the developer. So for example, when using a query a database, the developer might need to write a complex WHERE clause or maybe they're using functions like datediff, round, if null, although obviously, this function could be used by developers themselves if they want to, but the primary purpose is for the library to check these values.

Derick Rethans  2:05
	That is a method of doing it. What is this RFC adding to PHP itself?

Craig Francis  2:09
	It just simply provides a function which just returns true or false if the variable is a literal, and that's basically a string that was written by the developer. It's a bit like if you did is_int or is_string, it's just a different way of just sort of saying, has this variable been written by the developer?

Derick Rethans  2:28
	Is that basically it?

Craig Francis  2:30
	That's it? Yeah.

Joe Watkins  2:32
	It would also return true for variables that are the result of concatenation of other variables that would pass the is literal check. Now, this differs from Google, because they introduced that at the language level, but not only at the language level, at the idiom level. So that when you open a file that's got queries in PHP, commonly, if they're long, basic concatenation is used to build the query and format it in the file so that it's readable. So that it wouldn't really be very useful if those queries that you see everywhere in stuff like PHPMyAdmin, and WordPress, and Drupal and just normal code weren't considered literal, just because they're spread over several lines with the concatenation operator. It's strictly not just stuff that's written by the programmer, but also stuff that was written by the programmer or concatenated, with other stuff that was written by the programmer.

Derick Rethans  3:33
	Now in the past, we have seen something about adding taint supports to PHP, right? How is this different, or perhaps similar, to taint checking?

Craig Francis  3:44
	At the moment today, there is a taint extension, which is something you need to go out your way to install, and actually learn about and how to use. But the main difference is that taint checking goes on the basis of say, this variable is safe or unsafe. And the problem is that it considers anything that had been through an escaping function like html_entities as safe. But of course, the problem is that escaping is difficult. And it's very easy to make mistakes with that. A classic example is if you take a value from a user, an SSH SSH, their homepage URL, if you use HTML encoding, and then put it into the href attribute of a link, that can also result in HTML injection vulnerability, because the escaping is not aware of the context which is used. Because if the evil user put in a JavaScript URL, that is in inline JavaScript, that has created a problem because taint checking would assume that because you use HTML encoding it is safe, and all I'm saying is that is it creates a false sense of security. And by stripping out all that support for escaping, it means that you can focus on libraries doing that work because they know the context, they understand the domain, and we can just keep it a much simpler, and much safer approach.

Derick Rethans  5:02
	Would you say that the is_literal feature is mostly aimed at library authors and not individual developers?

Craig Francis  5:09
	Yeah, exactly. Because the library authors know what they're doing. They're using well tested code, many eyes over it. The problem libraries have at the moment is that they trust the developer to write things themselves. And unfortunately, developers introduce a lot of injection vulnerabilities with those strings before they even get into the library.

Derick Rethans  5:30
	How would a library deal with with strings that aren't literal then?

Craig Francis  5:35
	So it really depends on each individual example. And the RFC does include quite a lot of examples of how each one will be dealt with. The classic one is, let's say you're sorting by a column in a database, because if we're dealing with SQL, the field name might come from the user. But that is also quite a risky thing to do if you start including whatever field name the user wrote. So in the RFC, I've created a very simple example where the developer would create an array of fields that you can sort by, and then whatever the user provides, you search through that array, and you pull out the one that you that matches and is fine. And therefore you are pulling out a literal and including into the SQL. To be fair, these ones are quite unique. And each one needs to be dealt with in its own way. But I've yet to find an example where you can't do it with a literal. Having said that, I think Larry Garfield actually gave an example where a content management system changed its database structure. And the way that would work is the library would have to deal with it, they would receive the value for a field, and then that field would be escaped and treated as a field, it understands it as a field, and it will process it as such, then it can include into the SQL, knowing full well that everything else in that SQL is a literal, and then it can just build up SQL in its own way internally.

Derick Rethans  6:58
	Okay, talking a little bit about the implementation here. Since PHP seven, we have this concept of interned strings, or maybe even before that actually, I don't quite remember. Which is pretty much a flag on each string and PHP that says, this's been created by the engine, or by coconut. Why would strings have to have an extra flag here to remember that it is created by the programmer?

Joe Watkins  7:21
	Well, interned does not mean literal. It's an optimization in the engine, should we use strings. We're free to do whatever we want with that. At the moment, it by happenstance, most interned strings are those written by the programmer. If you think about the sort of strings that are written by the programmer, like a class name, when those things are declared internally, by an extension, or by core code, those things are interned as if they were written by the programmer. They don't mean literal, we're free to use interned strings for whatever we want. For example, a while ago, someone suggested that we should intern keys while JSON decoding or unserializing. It didn't happen, but it could happen. And then we'd have the problem of, well, how do we separate out all this other input. There is another optimization attached to interned strings, which is one character strings, where if you type only one character, or you call a Class A or B, or whatever, the permanent interned string will be used. That results in when the chr function is called, that results in the return of that function always being marked as interned. So it would show as literal, which is not a very nice side effect. And that's just a side effect that we can see today. We don't want to reuse the string really, it does need to be distinct. Also, if you're going to concatenate, whether you do it with the VM or a specific function, obviously, you need to be able to distinguish between an interned string and a literal string, which interned means it has a specific life cycle and specific value. And we can't break that.

Derick Rethans  9:00
	So there are really two different concepts, is what you're saying, and hence, they need to have a special flag for that?

Joe Watkins  9:06
	Yeah, they're very, two very separate concepts. And we don't we don't want to restrict the future of what interned strings may be used for. We don't want to muddy the concept of a literal.

Derick Rethans  9:16
	Of course, any sort of mechanism that languages built into solve or prevent injections in any sort of form, there's always ways around it. Theoretically, how would you go around the is_literal checks to still get a user inputted value into something that passes the is_literal check?

Craig Francis  9:36
	Generally speaking, you would never need it because the library should know how to deal with every scenario anyway. And it's not that difficult. We're only talking about things like in the database world, you'll be taking value from field names and therefore it should receive field names or table names. And, you know, we are providing a guardrail as a safety net. And what should happen is that the default way in which programmers work should guide them, to do it the right way. We're not saying that you can't do weird things to intentionally work around this. A really ugly version, which you should never do, but use eval and var_export together, it's horrible. But if you are so desperate, you need to get around this. That's what we're doing it. But in reality, we can't find any examples where you'd actually need to do this.

Joe Watkins  10:22
	I would say that, hey, there's this idea that most people writing PHP are using libraries, and they're using frameworks. I don't actually find that to be true. I've been working in PHP for a long time. And most of the big projects I've worked on for a long time did not start out using frameworks. And they did not start out using libraries. They look a bit like that today, but their core, they are custom. There may be a framework buried in there. But there is so much code that the framework is a component and is not the main deal. Most code, we actually do write ourselves, because that's what we're paid to do. I think we don't decide how people are going to use it, and we don't decide where they're going to use it. The fact is, like Craig said, it's a guardrail that you can work around easily. And if you find a use case for doing that, then we shouldn't prejudge, and say, well, that's the wrong thing to do. It might not be the wrong thing to do. For example, an earlier version of the idea included support for integers. We considered integers safe, regardless of their source. If you wanted to do that, in your application, you could do that very easily and still retain the integrity of the guardrail is not compromised. I wouldn't focus on this is for libraries, and this is for frameworks, because these things become so small in the scheme of things that they're meaningless. I mean, most of the code we work on is code that we wrote, it is not frameworks.

Derick Rethans  11:48
	That also nicely answers my next question, which is what's happened to integers, which have now nicely covered. The RFC talks about that as hard to educate people to do the right thing. And that is_literal is more focused, so to say, on libraries, and perhaps query building frameworks as the RFC alludes to. But I would say that most of these query building tools or libraries already deal with escaping from input value. So why would it make sense for them to start using is_literal if you're handling most of these cases already anyway?

Craig Francis  12:24
	If you look at the intro of the RFC, there's a link to show examples of how libraries currently receive the strings. And you're right about the Query Builder approach is a risky thing, I would still argue it's an important part. That's why libraries still provide them. Doctrine has a nice example of DQL. The doctrine query language is an abstraction that they've created, which is also vulnerable to injection vulnerabilities. And it gives the developer a lot more control over a very basic API. I still think people should try and use the higher level API's because they do provide a nice safe default, but that depends on which library use, they're not always safe by default. So for example, when you're sort of saying: I want to find all records where field parameter one, is equal to value two, a lot of the libraries assumed that the first parameter there is safe and written by the developer. They can't just necessarily simply escape it as though it's a field because that value might be something like date, bracket, field, bracket, and it's sort of relying on the developer to write that correctly, and not make any mistakes. And that hasn't proven to be the case, you know, they do include user values in there.

Derick Rethans  13:43
	Just going back a little bit about some of the feedback, because feedback to the RFC has happened for quite some time now. And there were lots of different approaches first tried as well, and suggested to add additional functions and stuff like that. So what's been the major pushback to this latest iteration of the RFC?

Joe Watkins  14:01
	So I think the most pushback has come from an earlier suggestion that we could allow integers to be concatenated and considered literal. We experimented with that, and it is possible, but in order to make it possible, you have to disable an optimization in the engine, that would not be an acceptable implementation detail for Dmitri. It turns out we didn't actually, we don't need to track their source technically, but it made people extremely uncomfortable when we said that, and even when we got an independent security expert to comment on the RFC, and he tried to explain that it was no problem, but it was just not accepted by the general public. I'm not sure why.

Derick Rethans  14:45
	All right. Do you have anything to add Craig?

Craig Francis  14:48
	The explanation given by people is they liked the simpler definition of what that was as if it's a string written by the developer. Once you start introducing integers from any source, while it is safe, it made people feel, yeah, what is this. And that's where we also had the slight issue because we had to find a new name for it. And I did the silly thing of sort of asking for suggestions, and then bringing up a vote. And then we had, I think it's 18 to three people saying that it should be called is_trusted, and you have that sinking moment of going, Oh, this is going to cause problems, but hey, democracy. It creates that illusion that it's something more. So that's why we sort of went actually, while I like Scott's idea of having the idea of maybe calling it is_noble. It is a vague concept, which people have to understand. And it's a bit strange. Whereas going back to the simpler, original example, they've all seem to grasp grasp of that one. And we could just keep with the original name of is_literal, which I've not heard any real complaints about.

Derick Rethans  15:53
	I think some people were equivalenting is_trusted with something that we've had before in PHP called Safe mode, which was anything but of course.

Craig Francis  16:02
	Yes, no, definitely.

Derick Rethans  16:03
	We're sort of coming to the end of what to chat about here. Does the introduction of is literal introduce any BC breaks?

Craig Francis  16:11
	Only if the user land version of is_literal, which I'm fairly sure is going to be unlikely. So on dividing their own function called that.

Derick Rethans  16:18
	Did you check for it?

Craig Francis  16:20
	Yes.

Derick Rethans  16:21
	So if you haven't found it, then it's unlikely to to exist.

Craig Francis  16:24
	There are still private repositories, we can't shop through all their show, check through all their code. But yeah.

Derick Rethans  16:29
	Did I miss anything?

Craig Francis  16:31
	We covered future scope, which is the potential for a first class type, which I think would be useful for IDs and static analysers. But this is very much a secondary discussion, because that could build on things like intersection types, but we still need to focus on what the flag does. And there's also possibility of using this with the native functions themselves, but we do have to be careful with that one, because, you know, we got things like PHPMyAdmin. We have to be able to make the output from libraries as trusted because they're unlikely to still be providing a literal string at the end of it. So that's a discussion for the future. And the only other thing is that, you know, the vote ends on the 19th of July.

Derick Rethans  17:08
	Which is the upcoming Monday. How is the vote going? Are you confident that it will pass?

Craig Francis  17:13
	Not at the moment, we're sort of trying to talk to the people who voted against it. And we've not actually had any complaints as such. The only person who sort of mentioned anything was saying that we should rely on documentation and the documentation is already there. And it's not working. I think a lot of people just voted no, because they just sort of going well, that's the safe default. I don't think it's necessary. Or, you know, I'd like the status quo. And we still are trying to sell the idea and say: Look, it's really simple. It's not really having a performance impact. And it can really help libraries solve a problem, which is actually happening.

Derick Rethans  17:46
	Is this something that came out of the people that write PHP libraries or something that you came up with?

Craig Francis  17:52
	So I've come gone to the library authors and suggested you know, this is how Google do it. Would you like something similar? And we've certainly had red bean and Propel ORM saw show positive support for that. And I've also talked to Matthew Brown, who works on the Psalm static checking analysis. He's very positive about it, so much so that Psalm now also includes this as well. Obviously, static analysis is not going to be used by everyone. So we would like to bring this back to PHP so that libraries can use it without relying on all developers using static analysis.

Derick Rethans  18:25
	Thank you very much. Glad that you were both here to explain what this is_literal RFC is about.

Craig Francis  18:31
	Thank you very much, Derick.

Joe Watkins  18:33
	Thanks for having us.

Derick Rethans  18:37
	Thank you for listening to this installment of PHP internals news, a podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening and I'll see you next time.


Show Notes
----------

- RFC: `is_literal <https://wiki.php.net/rfc/is_literal>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
