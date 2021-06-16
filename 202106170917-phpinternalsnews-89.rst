PHP Internals News: Episode 89: Partial Function Applications
=============================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-06-17 09:17 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin089
   :Enclosure: media/pin-089.mp3

In this episode of "PHP Internals News" I chat with Larry Garfield (`Twitter
<https://twitter.com/Crell>`_) and Joe Watkins (`Twitter
<https://twitter.com/krakjoe>`_, `GitHub
<https://github.com/krakjoe>`_, `Blog <https://blog.krakjoe.ninja/>`_ about the
"Partial Function Applications" RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-089.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:14
	Hi, I'm Derick. Welcome to PHP internals news, a podcast dedicated to explaining the latest developments in the PHP language. This is Episode 89. Today I'm talking with Larry Garfield and Joe Watkins about a partial function application RFC that they're proposing with Paul Crevela and Levi Morrison. Larry, would you please introduce yourself?

Larry Garfield  0:36
	Hello World. I'm Larry Garfield or Crell on most social medias. I'm a staff engineer for Typo3 the CMS. And I've been getting more involved in internals these days, mostly as a general nudge and project manager.

Derick Rethans  0:52
	And hello, Joe, would you please introduce yourself as well?

Joe Watkins  0:55
	Hi, I'm Joe, or Krakjoe, I do various PHP stuff. That's all there is to say about that really.

Derick Rethans  1:02
	I think you do quite a bit more than just a little bit. In any case, I think for this RFC, you, you wrote the implementation of it, whereas Larry, as he said, did some of the project management, I'm sure there's more to it than I've just paraphrased in a single sentence. But can one of you explain in one sentence, or if you must, maybe two or three, what partial function applications, or I hope for short, partials are?

Larry Garfield  1:27
	Partial function application, in the broadest sense, is taking a function that has some number of parameters, and making a new function that pre fills some of those parameters. So if you have a function that takes four parameters, or four arguments, you can produce a new function that takes two arguments. And those other two you've already provided a value for in advance.

Derick Rethans  1:54
	Okay, I feel we'll get into the details in a moment. But what are its main benefits of doing this? What would you use this for?

Larry Garfield  2:01
	Oh, there's a couple of places that you can use partial application. It is what got me interested. It's very common in functional programming. But it's also really helpful when you want to, you have a function that like, let's say, string replace takes three arguments, two of which are instructions for what to replace, and one of which is the thing in which you want to replace. If you want to reuse that a bunch of times, you could build an object and pass in constructor values and save those and then call a function. Or you can just partially apply string replace with the things to search for, and the things to replace with and get back a function that takes one argument and will do that replacement on it. And you can then reuse that over and over again. There are a lot of cases like that, usually use in combination with functions that wants a callback. And that callback takes one argument. So array map or array filter are cases where very often you want to give it a function that takes one argument, you have a function that takes three arguments, you want to fill in those first ones first, and then pass the result that only takes one argument to array map or a filter, or whatever. So that's the one of the common use cases for it.

Derick Rethans  3:15
	That's the benefits and some of its background comes from functional programming, as you've just mentioned. What is the syntax that you're proposing and some of the semantics?

Larry Garfield  3:26
	The syntax that we've developed, are two placeholders that you can use in a function call. So if you're calling a function as you normally would, but for one of the arguments, you pass a question mark, or at the tail end, you have an ellipsis (dot dot dot), then that tells the engine: This is not a function call. This is a partial application. And what it will do is return not the result of the function but return a closure object that has the the arguments that correspond to those question marks. And then when called with those arguments, we'll pass those along with the original function. Probably easier to explain, if I use a concrete example, using the string replace example we talked about before, you would call it with str_replace, the example from the RFC, hello, hi, question mark. What that gives you is a callable, a closure that has one argument, which will take its type and name from str_replace. So the third argument to str_replace essentially gets copied into that closure. And what closure does internally when you call it with that one argument is it just calls string replace with hello, hi, and whatever argument you gave it and returns that value. It is conceptually very, very similar to just writing a short lambda or an arrow function that takes one arguments and calls string replace hello, hi, and that argument. In most cases, it ends up functioning almost exactly like that. There's a few subtle differences in a few places. But most of the time, you can think of it working essentially like that. The question mark means one required argument only. The dot dot dot means zero or more arguments, if you want to, say provide the first argument to a function, and then dot dot dot would mean: And then all of the other arguments, however many there are, even if it's that zero, those are what's left, which languages other languages that have partial application as a first class feature, usually end up doing it that way where you can only pre fill from the left. PHP, because the placeholder lets us do it in any order. So we can skip over arguments if we want to, which is quite nice. But it means that you can take a function and reduce it to, I want to prefill just these two arguments and leave these three arguments for the new function, or I want to prefill these arguments from the left, and then everything else, whatever it is, is left. It also lets you do cute things like if you provide all of the arguments to a function, and then just tack on a dot dot dot the end of it, then you get back a closure that takes essentially zero arguments. But when called, will call that other function. So it's lets lets you really easily build a delayed function as you need to.

Derick Rethans  6:15
	When do the arguments to the function get evaluated then?

Larry Garfield  6:18
	Arguments are evaluated in advance. So this is the subtle difference between partial application and the short lambda syntax. In a short lambda, what happens is, essentially, that entire expression on the right hand side gets wrapped up into a closure. And so any arguments that are compound like they have a function call that is inside one of the placeholders, or one of the arguments, that'll get evaluated later. With partial application, the function that is in a parameter position gets evaluated first and reduced to a value. And that value gets partially applied to the function. 90% of the time, that's not going to be an issue. There are a few cases where doing it one way or the other may be subtly different, but you'll spot those fairly easily.

Derick Rethans  7:02
	So the RFC talks about things that you can do, but also a few things that you cannot do or don't want to do yet. What are these things that partials won't support, or run support yet, at least?

Larry Garfield  7:13
	The main thing that it doesn't support is named placeholders. You can pre fill a value or an argument with a named named argument. But not a named placeholder. Those have to be positional. Named placeholders are complicated to implement, and run into a question of, if you provide those in a different order, does that also change the order of the arguments in the partially applied function that you get back in that closure? And there's a good argument to be made that either way is logical. And so we're like, no, does not deal with it, too complicated. We'll just positional only. And you cannot specify an optional arguments either. It's just again, too complicated. Things get too weird. If you have those advanced cases, use our short lambda, that works just fine. If you want to just make a new function that defers to a new function, and change its API in the process, short lambda works fine. And it's still quite short.

Derick Rethans  8:13
	I know the RFC talks a little bit about references, but I don't like talking about references. So let's skip that part. In my opinion, they should be removed from the language. But I know we can't.

Larry Garfield  8:22
	There's occasionally used for them. But very occasionally.

Derick Rethans  8:25
	There's a bunch of technical things that I also want to chat about. And hopefully, Joe, if you want to fill in, I'd be more than welcome to hear your opinions on these things. But the first one is that PHP has this thing called func_get_args. How does that work with these partials? How does that tie in together?

Joe Watkins  8:42
	It should mostly behave as if you've invoked the function directly. We don't want there to be a huge discrepancy between. The callee know whether they've been called through partial application or complete application. It should be the same.

Derick Rethans  8:58
	That is good to know. I mean, I always like it how things work as people expect them to work, right?

Joe Watkins  9:03
	Yeah.

Derick Rethans  9:04
	We already have used the dot dot dot operator for variadics. But you're reusing the dot dot dot, or ellipses, as you more eloquently call it earlier. Here again, as well, is that not going to cause issues? Or does that tie in well together?

Joe Watkins  9:18
	Well, there's quite a lot of debate about what's the right symbol to use. I think it's dot dot dot, and I think Larry agrees with me. But there's some people who want to stick an extra question mark on the end, which to me looks like it reads zero to one. And to Larry, it looks like an extra character that's just not needed. Other people say it makes sense for them. But if you can type three characters and not four, I mean, you need a really good argument. The arguments that have been put forward so far don't really make very much sense for me. Maybe we should ask that question and it doesn't really matter. In the end, what the syntax is, is if it's a difference between it getting in and not getting in, then we'll just put the extra question mark on there. I don't really have a really good argument to change it like to be like that.

Derick Rethans  10:05
	To be honest, to me, it looks like you then have two placeholders.

Joe Watkins  10:09
	Yeah.

Derick Rethans  10:10
	I don't feel the need for it.

Joe Watkins  10:11
	That's also another argument because we've introduced this one symbol, and then this other symbol, and then you put them together. And that's two things. I mean, you can't have one and one equals one.

Derick Rethans  10:20
	Fair enough. The RFC does touch on another quite interesting thing, I think, which is constructors, which it also be able to partially apply. But of course, you've mentioned that, that arguments get applied immediately when you do the substitution, when you do the partial application. But of course, the constructor is a bit weird because a constructor runs immediately after an object has been constructed. So how does that work together with partials?

Joe Watkins  10:47
	So at first, we made it so like if you invoke a constructor with reflection, and you just invoke it over and over again, it'll invoke it on the same object, you won't get back a new object. It's not the constructor that returns the object, it's the new operator. So first, we had a bit dumb. And we did just like what reflection does. And if you applied to a constructor, you'd get back a closure that just repeatedly invokes the constructor, which is, as Larry called it, quite naive. So we went back and revisited that. And so now it acts like a factory. Every time you invoke the closure return from an application, you get a brand new object, which is more in line with what people expect. And it's also quite cool. It's one of my favourite bits, actually as it turns out.

Derick Rethans  11:31
	In my opinion, it also makes more sense than then having an apply to the same object over and over again. Whether I'd like it or not, I don't know yet.

Joe Watkins  11:39
	Oh, the other option is traditional constructors to avoid the surprising behaviour. But that would be just a strange.

Larry Garfield  11:45
	There are a lot of use cases where you want to take a bunch of values, convert them to objects using an array map, supporting constructors for that makes total sense to me.

Derick Rethans  11:54
	And I would probably say, though, that I would prefer not allowing it over it applying over the same object over again. You've touched a little bit on some common cases where you want to use this, do you perhaps have some other ideas where this might be really useful?

Larry Garfield  12:10
	So there's three use cases that we think are probably going to be the lion's share. One is to just use the dot dot dot operator. So you have some function or method call, call it with dot dot dot, and that's it. You prefill nothing, which gives you back a closure that is identical in signature to the function or the method that you're applying it to. Everything we've said about functions applies the methods here as well. Which means we now effectively have a new way to refer to a function or a method and make a callable out of it, that doesn't involve just sticking it into a string. You just say, hey, function called dot dot dot, or an arrow bar, parentheses, dot dot dot, parentheses. And now you can turn any function or method into a callable and pass that around. And it's still, it's not wrapped up into the silly array format, it's still accessible to static analysers and refactoring tools. Hopefully, with this, you will never need to refer to a function name using a string ever again, never refer to a method call as an array of object and method. So that that just is not needed any more in the vast majority of cases.

Derick Rethans  13:20
	That alone is probably worth having them, maybe.

Larry Garfield  13:23
	And Nikita had an RFC that was doing just that, and nothing else. It's kind of a junior version of this. I don't think that's necessary, the full full scope here works, and gives us that. The second use case that I think is going to be common are unary functions. That's functions that take a single argument. More to the point, as I mentioned before, a lot of functions take a callback. And that callback needs a single argument, array map, array filter, some validation routines, a lot of other things like that. So it's now stupidly easy to take any arbitrary function or method and turn it into a single parameter function, which you can then pass as a callback to array map, array filter, all these other tools, and it just becomes really easy to pre fill things that way. The third is the other one I mentioned earlier, if you pre fill all the arguments, and then just put a dot dot dot at the very end, which means zero or more, you now have a function that takes no arguments, but calls the original function you specified with all the arguments you specified. This often the case for default values, where I want to have a default value available, but don't want to take the time to compute it in advance because it might be expensive. Whatever function it is that will determine that default value, I just partially apply that and give it all the arguments and I get back a callable. That creating a callable is dirt cheap, but when I actually need that value, I can then call it at that time, but it won't actually get called unless I need it. That's another use case that we expect to be common. There are no doubt others that we haven't thought of, or that will be less common, but still useful. I think this will probably replace a large chunk of the use cases for short lambdas. Not because short lambdas are bad, they're wonderful. But so many of them convert a function to a simpler function. And this gives us an even more compact, more readable syntax for that, with even less extra symbols and flotsam around it.

Derick Rethans  15:24
	I saw, hopefully as a joke, saying that, instead of using the question mark, we should use dollar sign dollar sign, and then we should call the token name T_BLING.

Larry Garfield  15:36
	This RFC actually has a storied history. Several years ago, Sara Golemon had proposed porting the pipe operator from Hack to PHP. The pipe operator is an operator available in a lot of different languages that lets you string together a series of functions. So you pass a function, pass an argument into one function, its results you pass to the next function, its results, you pass the next function and so on, which is a good case for unary functions. In Hack's syntax, they don't use a function on the right hand side, they use an arbitrary expression, and then dollar dollar as a placeholder for where to put the value from the left hand side from the previous step. It's the only language that does that.

Derick Rethans  16:20
	The other language that does it is bison.

Larry Garfield  16:23
	Or Bison also does that style of?

Derick Rethans  16:25
	It does something weird like that, yeah. Have a look at the grammar file.

Larry Garfield  16:29
	I've looked in there. It's scary. So at the time, she didn't actually put an implementation in for it. But there was some discussion about it. I joked that if she wanted to do that, she should call it T_BLING. And she thought it was hilarious, but never went anywhere. A year ago, I started working on a pipe operator RFC that did just the pipe part, but used a callable on the right hand side, instead of an expression, more like F#, and Haskell, and other languages that have a pipe operator. And their main response to that was, we'd like this, this is cool, except that just using short lambdas on the right all the time to make unaries is too ugly. We want partial application first. So I spent a while trying to bribe someone with more experience and knowledge than me to work on partial application. I tried bribing Ilya Tovolo, to do so by working with him on enumerations. And we got enumerations in, but he doesn't have the time to work on partial application. Levi and Paul had already written an RFC for partial application that had no implementation. It's just a skunkworks, essentially. Then a few weeks ago, Joe pops up and starts working on an implementation for partials. And I, to this day, don't know what interested him in it. But I'm very happy about this fact. So as we updated the RFC, I knew that people want a bike shed about syntax. So I threw that in as a joke. I don't think we're actually going to do that. It's just a little inside reference that is now no longer inside.

Derick Rethans  17:56
	Joe what made you work on partials, then?

Joe Watkins  17:58
	It's interesting to write. I've had my fun whether it gets in or not.

Derick Rethans  18:02
	Sometimes that's the case, right? So just working on this is all the fun.

Larry Garfield  18:06
	Sometimes it's fun to just run down rabbit holes for the heck of it. And sometimes really cool things can come out of that sometimes.

Derick Rethans  18:12
	At some point, I might have to implement support for partials like I have for closures in Xdebug as well. Because at some point, people might want to debug these things. So I'm a little bit interested in how do these the closures that it generates? Where does it store the already applied arguments?

Joe Watkins  18:29
	So partials have the same binary struct up to this point or of the closure, and then after that there's some extra fields.

Derick Rethans  18:36
	Would they still have the names?

Joe Watkins  18:39
	No, because named arguments aren't actually named, that information is lost. By the time we've got them, we don't have any name information. We've only got their correct position, according to the call that was made.

Derick Rethans  18:50
	And every argument that hasn't been filled and doesn't have a special placeholder in there, or does it keep track of which ones have been filled in?

Joe Watkins  18:56
	We've got two special placeholders internally, you won't see as undef or null or anything.

Derick Rethans  19:02
	Okay, that's good to know. What has the reaction been so far?

Larry Garfield  19:05
	Slightly positive. There were a lot of discussions early on about do we support argument reordering? And should it use a single placeholder or two separate placeholders? Originally, we had one and realized after a while, that doesn't actually work. There're use cases where that will be confusing. Overall, the feedback has been quite positive, and I fully expect that to pass. Really the only question people are still debating about at this point is ellipsis versus ellipses question mark.

Joe Watkins  19:34
	Yeah, I think the first version of the RFC was quite well received. Someone said we could document it as to make a partial sprinkle or question mark over it and hope for the best.

Derick Rethans  19:44
	Oh, that's good to hear. With feature freeze coming not pretty soon now. When do you think you're putting this up for a vote?

Larry Garfield  19:51
	Probably in the next couple of days. The only question I think is whether we include a second question for which variadic placeholder to use, which syntax/ Or if we just say it's dot dot dot, go away. Other than that it should go to a vote probably before this episode airs.

Derick Rethans  20:06
	Thank you very much, both of you for taking the time to me today to talk about partials.

Larry Garfield  20:11
	Thank you again Derick, hopefully see you once more on this season.

Joe Watkins  20:15
	Thanks Derick, see you soon.

Derick Rethans  20:21
	Thank you for listening to this installment of PHP internals news, a podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next time.




Show Notes
----------

- RFC: `Partial Function Applications <https://wiki.php.net/rfc/partial_function_application>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
