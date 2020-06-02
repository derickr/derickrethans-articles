PHP Internals News: Episode 57: Conditional Codeflow Statements
===============================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-06-11 09:20 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin057
   :Enclosure: media/pin-057.mp3

In this episode of "PHP Internals News" I chat with Ralph Schindler (`Twitter
<https://twitter.com/ralphschindler>`_, `GitHub
<https://github.com/ralphschindler>`_, `Blog <https://ralphschindler.dev/>`_)
about the Conditional Return, Break, and Continue Statements RFC that he's
proposed.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-057.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:17  
Hi, I'm Derick, and this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 57. Today I'm talking with Raluphl Schindler about an RFC that he's proposing titled "Conditional return break and continue statements". Hi Ralph, would you please introduce yourself.

Ralph Schindler  0:37  
Hey, thanks for having me Derick. I am Ralph Schindler, just to give you a guess the 50,000 foot view of who I am. I've been doing PHP for 22 years now. Ever since the PHP three days, I worked in a number of companies in the industry. Before I broke out into the sort of knowing other PHP developers I was a solo practitioner. After that I went worked for three Comm. And that was kind of a big corporation after that I moved to Zend. I worked in the framework team at Zend and then after that, I worked for another company based out of Austin for friend of mine Josh Butts. That offers.com, we've been purchased since then by Ziff media. I'm still kind of in the corporate world. Ziff media owns some things you might have heard of, PC Magazine, Mashable, offers.com. The company that owns us owns is called j two they are j facts. They keep buying companies, so it's interesting I get to see a lot of different products and companies they get bought and they kind of get folded into the umbrella, and it's, it's an interesting place to work. I really enjoy it.

Derick Rethans  1:39  
Very different from my non enterprise gigs

Ralph Schindler  1:43  
Enterprise is such an abstract word, and, you know, it's kind of everybody's got different experiences with it.

Derick Rethans  1:49  
Let's dive straight into this RFC that you're proposing. What is the problem that this RFC is trying to solve? 

Ralph Schindler  1:54  
This is actually kind of the bulk of what I want to talk about, because the actual implementation of it all is is extremely small. As it turns out it's kind of a heated and divided topic, My Twitter blew up last weekend after I tweeted it out, and some other people retweeted it so it's probably interesting. I really had to sit down and think about this one question you've got is what is it trying to solve. First and foremost, it's something I've wanted for a really long time, a couple years. 

Two weekends ago I sat down and it was a Saturday and I'm like, you know what I haven't haven't hacked on the PHP source in such a long time. The last thing I did was the colon colon class thing, and I was like seven or eight years ago. And again, I got into that because I really wanted the challenge of like digging into the lexer and all that stuff and, incidentally, you know, I load PHP source in Xcode, and my workflow is: I like to set breakpoints in things, and I like to run something, and I look in the memory and I see what's going on and that's how I learned about things. And so I wanted to do that again. And this seemed like a small enough project where I could say, you know this is something I want to see in language, let me see if I can hack it out. First and foremost, I want this. And, you know, that's, it's a simple thing. 

So what is it exactly is, it's basically at the statement level of PHP, it is a what they like to call a compound syntactic unit. Something that changes the statement in a way that I think probably facilitates more meaning and intent, and sometimes, not always, it'll do that and fewer lines of code. To kind of expand on that, this is a bit of a joke but a couple years ago there was that whole argument online about visual debt. I don't know if you remember hearing that, that terminology. 

Derick Rethans  3:34  
Yep.

Ralph Schindler  4:47  
Foo.

Derick Rethans  4:23  
Up to now we haven't spoken about but the RFC is proposing so maybe we should talk about it first and then get back to other things that he said have you spoken a little bit about the reasons why you want to change something. But what would you like to add to PHP or, or what would you like to modify in PHP?

It's, you know, it's, it's very closely related to what in computer science is called a guard clause, and I used that phrase lightly when I originally brought it up on the mailing list but it's very closely aligned to that, it's not necessarily exactly that, in terms of the syntax. In terms of like when you speak about it in the PHP code sense, it really is sort of a change in the statement; so putting the return before the if. That's really what it is. So guard clause, it's important to know what that is, is it's a way to interrupt the flow of control, you know, over the history of programming languages. 

Ralph Schindler  7:19  
Let's just go back to Pascal. Pascal like 50 years ago, there was no opportunity in Pascal code to exit early from either a loop, or a method, so you had to wait until you got to the very final sort of statement, and there was a single exit from a function. Guard clauses allow you to effectively, if you're inside of a block of code, or a loop, or some kind of flow of control. It gives you an opportunity to say I want to exit here instead of continuing on. They did a whole bunch of studies on Pascal and they found out that students were like, they couldn't come up with the right solution when let's say if you had a loop statement, it had to execute 100 times there was no opportunity to get out early. When you gave them the opportunity to interrupt the flow control the correctness of their solutions, ultimately got better. Almost 100% of the time they were able to, you know what this is an exceptional piece of code, I want to exit here. 

Fast forward guard clauses, they're kind of, if you've kind of followed the Kent Becks and the Martin Fowlers they would argue for guard clauses. Y'know over the line that's gotten more popular as an argument over the past, let's just say 15 years in our industry

Derick Rethans  8:23  
Would another term for this be like an early return?

Early returns are one of them, early breaks, and early continues, so getting to a place in code where you just say you know what this, there's a particular condition, in this normal flow of execution, I want to stop that normal flow and I want to break out of it. Goto is another tool that allows you to do this. I don't know if you can do it inside of loops, maybe you can. There's like some exceptions in PHP where you can jump to and from, 

You can jump out of loop, but you can't jump into one.

To some degree, these tools do sort of exist, goto, another heated topic in the PHP world. So getting back to what the guard clause is. More specifically, it's, it is very closely, and semantically aligned with a Boolean expression. You will generally say, I want to either return, break, or continue, based off of this Boolean. PHP itself does not have first class support for guards. The way we achieve it currently is, we will put the Boolean expression first, and as part of a block of code associated with that, so: if curly brace block of code, that might terminate in a early return. Inside of switch statements or loops, you'll see that if something something something continue one continue two, or break one break two. Return expression, break continue, along with a return or break expression, is the way we achieve it in PHP. This is kind of giving first class support to a guard clause. It would spell it out in the manual and it would be a tool that since it has a name, and it isn't the language, programmers could reach out and say, I know what that is, or: Here's what it is in the manual, how do I use that? That's kind of, you know what a guard clause is.

At the moment, if you mentioned the guard clause you can sort of implement by doing: if, your condition and then a curly braces return, or break, or continue, whatever you set. What is the syntax that you want to replace this with?

I don't want to replace syntax. PHP is a flexible language. We have multiple ways of doing lots of things. We have multiple ways of crafting closures and anonymous functions. We have two different ways that have existed since the beginning of PHP's time for doing if statements, one can be broken up by the, the semicolon, with the block the endif, or you can do with curly braces. You've noticed that with various PSRs and whatnot that people have gravitated towards a particular coding standard. And that, for all intents and purposes for the global community of programmers to have the shared diction, that's a good thing. 

Ralph Schindler  10:50  
With regards to PHP. So the most important characteristic of this RFC is that it is now, PHP is a left to right language, you know like much of the 90-95% of the speaking world left to right. They tend to put the emphasis, especially encoding of precedence on the left side. So this moves the return keyword to the left side of a statement or syntactic unit. So when you look at this code. The first thing you see is: return. In the variation one, which is the one I proposed of this, this feature, "return" is followed by "if", what you notice is that when you look at code you'll see "return if", and almost looks like its own key word. Those two individual, you know tokens, those key words must align themselves closely together exactly. You know, maybe there's like two spaces between them but return if are right next to each other, they can be treated almost as a new keyword and of itself. So as you're reading code top down, left aligned, you'll see return if, return if, finally at the bottom method, you'll see return. So that's variation one and what it does is it creates sort of this precedence that the keywords you know the static constant keywords return an effort first. Your expression is third. Your optional return value is fourth. In most of the cases where you're writing this, it does become a one liner. That's not to say we can't do one liners today, because you can do: if, if-expression, something, return. But what happens when you look at that code is that the return value is off to the right. Optionally if you don't, if you want to break outside of the PSR coding standards, or with the PSR coding standards. You can do curly braces and then put the return on the next line, now you got three lines of code, you've returned is indented. As you're visually approaching this code. See, you know what's most important to you is that there's a if statement there, but then you have to kind of scan the body of that to see if there's an early return. The fact that it's an early return in variation one becomes abundantly clear at the leftmost rail of the code, at the leftmost side of the statement, assuming you're not putting all of your code on one line.

Derick Rethans  12:59  
You talk about variation one, I guess there's a variation two as well. What is the difference between them?

Ralph Schindler  13:05  
As with RFCs, people have preferences and they have. Just with politics in general, if you're in a political position, which this is a political changes to PHP, you have to know where your constellations are. You have to know, basically, if I want to appease the most amount of people like what will I have to give up in order to get something that is still beneficial to me. For me right now, it is the compromised position. That's not to say I won't like it more, maybe a month from now on, but effectively the variation two is moving the optional return value after the Return. Return, optional return value, then the if, i f, and then the optional, not the non optional if expression, followed by the semicolon. So basically it would read more like English, so to speak. Return this, if this. What I understand it is that way in Perl. I know it's that way in Ruby. So Ruby follows the same thing because the way they've implemented it is not necessarily in a single statement they've, they've implemented what they call a statement modifiers, which is any statement can be modified with this conditional at the end of it. That's the alternative syntax. If I were to use this, I get value out of it because maybe I don't return an optional expression and then I'm still left with return if this. I still have my escape hatch for methods that have an optional return, the ability to return void.

Derick Rethans  14:26  
In variation one, how do you separate out the condition with the optional return value?

Ralph Schindler  14:32  
Another reason why I thought variation one was good for PHP specifically. Let's just do like two seconds of history. If you go back 20 years, C++, the way you write a method signature in C++ is: you'll do public, int, method name, typed arguments, so the return, we call them, hints, the hint for the method in C++ precedes the method.

Derick Rethans  14:55  
I've just been talking to Dan Ackroyd for the podcast episode that came out last week, where he is saying that we should stop calling it hints, because they're no longer hints, they're not proper type names. Maybe we should pick that up here as well than?

Ralph Schindler  15:10  
We've had that discussion for 10 years now. But people know them as hints. We've such loaded phrasing and PHP like type coercion. Whatever we call them, I'll just continue with hints for the time being, because that's the audience at this particular podcast knows them as hints. The hint in C++ would have been all the way to the left of the line, whereas in PHP when we chose to implement typing of the return values, we did it in a way where it was the method signature had the semi colon and the return type at the end of the method signature. This particular variation one, this follows that same pattern, where your semi colon return value looks exactly how the layout of the method signature is where it's semi colon, what you see up top. There's a big parallel there between an early return with an optional return value. Also, I like optional things to be at the end. And when you look at this whole statement that's the optional part, whereas when variation two the optional part being in the middle means return optional part if, or return if are both valid things. So parallel is the method signature. That was kind of why I personally like the first one. They're both my children at this point I love them both. 

Derick Rethans  16:20  
As you said, introducing syntax is always a bit tricky and it's a political choice. What has been sort of the feedback and, and or the criticisms, to your suggested that additional language constructs?

Ralph Schindler  16:33  
Smallest changes always get the most feedback, because there's such a wide audience for a change like this, like they can immediately see the benefits or negative value of it in their own code, all the way from the junior programmer, all the way up to the senior programmer, I can't quantify who's Junior new senior, I can't also quantify who has been programming a long time and it was, for lack of a better term set in their ways and likes their style versus those who have adopted a certain flexibility in the way that they develop and like the size of the team they're on and how much of a leniency they put on someone else to write code that they will just you know code review and accept. So the interesting thing is that you have to kind of understand Junior programmers, or senior programmers. When the junior programmer gets in there, and they start programming, they tend to write code that is very brute force, they just write a lot of code because in order to get better at writing code you just keep writing code. To them, their perspective is from the code writing standpoint, they're not looking at this from a code reading standpoint, they're looking at it from a writing standpoint. So when you see a junior programmer they rely on ifs and loops and like the rudimentary techniques, less abstraction, fewer methods, more lines of code. They tend to not break things out into well equipped to well named methods. Whereas as they grow as programmers they start reading other people's code more and then they do start appreciating abstraction like this 50 line thing needs to be a five line thing. It needs to have its own name as a method over here, I need to reduce the number of inputs, have a very specific outputs, so on and so forth. So it's more highly structured code. Putting a feature out, you know like this, you get a range of perspectives from people. It goes without saying. I mean, Taylor retweeted it, I know he has a preference for this style of programming. I know exactly where it came from. He appreciates certain things in like the Ruby world, the return if statements in Ruby is a clear, concise, and very impactful statement, and too much of a degree he's, he's implemented that same thing in Laravel. So if you look at the helper methods in Laravel someone that writes Laravel applications is used to using something like abort if, or throw if. Interesting side note here, PHP is going to have a feature where you can put a throw expression, following a ternary. That in and of itself, allows exceptions to have a much more concise syntax. It allows you to use PHP exceptions for flow control. So you still can't do that with a return value for example, you can't have it a ternary with a return value. And I guess that is another way of being able to do achieve the same thing. This idiom, of being able to going back to guard clauses, and going back to thinking about early exits of methods, this was prevalent in Laravel where you could say in a controller method, and this is specific to an HTTP context, because you're inside of a controller, abort if, abort is highly specific to HTTP, where are you going to return a 404 or 500, it's going to throw an exception, an HTTP exception, which the framework knows to convert these kinds of exceptions into error paths in an application. So again we're still talking about application code, not necessarily library code. So abort if and abort unless is an idiom that I've seen is a fantastic idiom for controllers. I mean you can when you're thinking about a request which PHP is highly request driven, you can see when I start this method with the request object, you know, these are all my early outs, you know, this is where I'm going to return, and then at the very final spot I might be returning a view, which is a successful page for this MVC application. I feel like it was a successful idiom there and that was also part of the reason that drove me say, you know, it would be neat. If I could just say, return response, if this condition and have that early out.

Derick Rethans  20:12  
What's been the biggest criticism so far?

Ralph Schindler  20:15  
Biggest criticism is we can already do this. See, I hear that all the time, with all sorts of other features to varying levels varying degrees. I can do this with if something return something early. I said earlier that the proposed syntax might not be shorter and that's true. It is just changing the order of the operators, or the order of the keywords but, you know, that's an important distinction, like I want the precedence of the return to be earlier in the line. I think that's the important distinction. And I feel like maybe people that are saying it doesn't reduce the amount of code need to take that into account. And it's hard to see it really take that into account, unless you see variations of this sort of mental model of code. That's on me. I've been taking all the sort of like criticism, I'm kind of in a cooldown phase right now. I've been looking, I've been watching Twitter, I've been watching the Reddit. It's generally cooled down on internals mailing list, and I'm just kind of thinking about it because going back to likening this to a political sort of thing is that I have to rephrase my argument so that people that have a very firm stance on: I don't like this because I don't like it, or I don't like this because it doesn't shorten my code. I have to find an argument that gets them to start thinking about why this might be a good thing. I understand like this might get shot down in PHP. Right now, if I was a betting man, we were in Vegas, and someone asked me: Do you think this is going to go through, I probably would have to bet against myself I think 40-60. The temperature that I've taken on internals and everywhere else seems to indicate that it wouldn't be successful, but I'm collecting my evidence right now and putting out a blog post that kind of explains why it is, what it is, and putting a better argument forward. If that can't push it over the threshold, you know, I'll accept the defeat, so to speak, look at the history of PHP: annotations, and whatever they were called attributes, eight years ago were shot down. And, interestingly, I use the annotations back in the day with doctrine, I'd no longer use doctrine. So I voted to accept them. I might have voted to not accept them eight years ago, and I voted to accept them now, even though I don't use a variation of that any more.

Derick Rethans  22:15  
There's a few things that keep changing over time, right, first of all people turn from junior programmers into senior programmers, so they think about how to structure code more and more. And at the same time they also start seeing the value of some things that PHP never had right and. A good example is the scalar typing, that's been spoken about for maybe 15 years even, and it took so many different approaches, and as you say attributes, although attribute is a little bit different because this RFC is absolutely not the same as the earlier ones where the implementation is quite different from the version one then end up solving lots of problems that people found with the original RFC.

Ralph Schindler  22:53  
I have not been part of sort of the global PHP community. I started in the mid, 2000s. And having worked with PHP since 1998. I remember the early days where PHP was not fast at all. It was as fast as other things, but I gravitated towards it because I liked the syntax. Back in that day, I would have had more of an emphasis on things that would run faster, regardless of how they look because, I had projects for example in college I wrote a program where kids would go up and like on Valentine's Day, put all their preferences in. That was a week leading into Valentine's Day, and then on Valentine's Day they could come back to the University Center, and get a printout of all the other people that have fill out the questionnaire, and matched. When you have 1000 people fill out a questionnaire, this was PHP in 2000, 99 on 2000. And when I tell you, it took hours for the script to run and calculate all of the matches for a person, changing just the way an if statement would run, or changing the way you early exited an if statement when you know that you had to filter out a person. It changed the output by hours. The code was very, very closely aligned to like the performance, whereas now, PHP eight: I don't think that we have so many more affordances. You don't have to think about: Should I interpolate strings inside of a single quote or double quote, like none of that matters any more. We've solved all those problems. You can call sprint off just as quickly as you can do an echo, echo out and no one really cares, it's gonna perform the same. Wasn't the case 20 years ago, it is the case now, so now we have this affordance where we can look at the, you know, for lack of a better term, you know, is the code pretty, like is it easy to read.

Derick Rethans  24:32  
Thank you all for taking the time this afternoon, or in your case morning, I think, to talk to me about your RFC. I'm looking forward to seeing this coming to vote at some point.

Ralph Schindler  24:43  
I appreciate you having me on the, on your podcast. Thank you.

Derick Rethans  24:47  
Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.


Show Notes
----------
i
- RFC: `Conditional Return, Break, and Continue Statements <https://wiki.php.net/rfc/conditional_break_continue_return>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
