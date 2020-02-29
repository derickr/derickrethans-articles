PHP Internals News: Episode 43: Syntax Tweaks
=============================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-03-05 09:06 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin043
   :Enclosure: media/pin-043.mp3

In this episode of "PHP Internals News" I chat with Nikita Popov (`Twitter
<https://twitter.com/nikita_ppv>`_, `GitHub <https://github.com/nikic/>`_,
`Website <https://nikic.github.io/>`_)
about the RFCs. One on abstract methods in traits, and one about an
improvement to the tokenizer.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-043.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:16  
	Hi, I'm Derick. And this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 43. Today I'm talking with Nikita Popov yet again about a few RFCs that he's produced for PHP 8. Good morning, Nikita. How are you doing?

Nikita  0:34  
	Good morning, Derick. I'm doing great.

Derick Rethans  0:37  
	I've given up on introducing you because we've done this so many times. Now, you don't need an introduction any more. The first RFC I wanted to talk about a little bit this morning is the abstract trait methods validation RFC. What are traits?

Nikita  0:51  
	We usually talk about traits as compiler assisted copy and paste. Basically, we just take all the methods and properties from a trait and copy them into the class that's using the trait. That's a bit over simplified, in particular, you can use multiple traits in the single class. And those traits might be defining the same method, in which case you have to resolve the conflict in some way. So that's where you have these insteadof or use annotations to specify precedents and aliases. 

Derick Rethans  1:23  
	Traits has been in PHP for quite a long time. What is now the problem that you're trying to solve through this RFC? 

Nikita  1:29  
	The problem is that traits are sometimes not self contained. So to give a specific example, we have in the logger PSR, we have a trait called logger trait, which has a bunch of methods like warning, error, info, notice, and so on. So just simple helper methods, which all called the log method with a specific log level and this trait only specified these helper methods but still requires the actual class to implement the log method. The way you'll usually indicate that is by adding an abstract method to the trait. You have all the methods you actually want to provide by the trait. And you have a number of abstract methods that the trait itself requires to work. This already works fine, but the problem is just that these methods are not actually validated, or they are only inconsistently validated. Even though the trait specifies this abstract methods, you could implement it in the class with a completely different signature. 

Derick Rethans  2:30  
	Okay, just like any signature? 

Nikita  2:32  
	Just like any signature right. The method still has to be present in some way. But the signature can be completely different. Could also be like different method type, like a static method, or an instance method. 

Derick Rethans  2:43  
	Just basically checks for the name is what you're saying?

Nikita  2:46  
	Yeah, it only checks with the name.

Derick Rethans  2:49  
	Is this the only place, is this the only time where these abstract methods are not being validated. Or are there other situations where that could happen as well? 

Nikita  2:57  
	No, I think this is the only place. 

Derick Rethans  3:00  
	Are all the situations where these abstract methods in the trait will get validated. And also on signature?

Nikita  3:07  
	As I mentioned, it's not like the signatures are completely unvalidated. They are just inconsistently validated. It depends a lot on exactly how you use the trait. If you just use the trait and specify the methods of the same class, it doesn't get validated right now. If instead of the method is provided by the parent class, so it's inherited, then it does get validated. If you don't implement the method that makes the class abstract instead, then it's also going to get validated in the child class. It kind of already works halfway. And this RFC just tries to make it work always. 

Derick Rethans  3:44  
	Okay, that seems like a reasonably good addition to almost a no brainer. 

Nikita  3:48  
	I would say it's basically, a bug. Especially if you look at the implementation, there is clearly some validation code there. The conditions are just a little bit off, but so we do have to go through the proposal, because this is a backwards compatibility break. 

Derick Rethans  4:02  
	Yes, I was about to ask if it's a bug fix, why bother with an RFC? But if it's a BC break then yeah, we still need to do it of course. I doubt there be many controversies about is? 

Nikita  4:12  
	Actually there is one contentious point. Um, so something I didn't mention yet is that the RFC also allows you to define private abstract methods in traits. Normally private abstract is like a contradiction in terms because private means only visible in the same class. And abstract means it has to be implemented in the child class, you can't really have both. You can't have both with traits, because traits can see the private members in the class. I think that by itself is like not controversial. That's a reasonable thing to have a trait. The part that is controversial is what you do with existing visibility modifiers. This pattern already exists. So people already define abstract methods in traits but because right now private abstract is forbidden, the lowest they can use is actually protected abstract, even though they don't actually want that method to be publicly exposed, or even protectively exposed. So there is an argument there that we should maybe ignore the normal visibility validation that we do, and allow even implementing a protected abstract method from a trait with a private method inside the class, simply for backwards compatibility reasons. 

Derick Rethans  5:21  
	Because if you wouldn't allow that then, how would it break things? 

Nikita  5:26  
	It would break things because there is existing code, using these abstract protected methods simply because we don't support abstract private yet. So those code would start throwing visibility error, and I mean, could be fixed by just dropping the abstract method, but there's also not ideal. 

Derick Rethans  5:45  
	Because people use it to make sure that, I mean it's there in the class that implements the trait pretty much. Do you have any idea when this is going to for vote? 

Nikita  5:53  
	I think it can already go up for vote? Mainly I need to resolve that question about the visibility first.

Derick Rethans  5:59  
	I'm looking forward to seeing that showing up sometime soon then. 

	How do you call your second RFC? 

Nikita  6:05  
	Object based token get alternative? 

Derick Rethans  6:07  
	I think that's a great title. There's a few words in there that we might have to explain first. What are these tokens you're talking about? 

Nikita  6:14  
	So the token_get_all function, which we already have, exposes a part of the PHP compiler infrastructure. PHP compilation generally has three steps. The first is the tokenization. The second part is the parser, and then the compiler. So the tokenizer converts the raw character stream into tokens, which encode higher level concepts, for example, that like the sequence of FUNC and so on is actually a function keyword, or that double code followed by characters is actually a string. So this part only recognises like not larger structures, like whole functions but at least the the atoms that make up language. 

Derick Rethans  7:00  
	Would you say these are the words that make up the sentences? 

Nikita  7:03  
	Yeah, that's that's the right analogy. 

Derick Rethans  7:06  
	Why would you want to have access to them? 

Nikita  7:08  
	For example, I have a PHP parser library, which converts these tokens into an actual syntax tree. And then on top of that, you can easily analyse PHP source code. So this is what all these static analyzers, like PHPStan or Psalm are based on.

Derick Rethans  7:27  
	Do they all use the tokens?

Nikita  7:29  
	Those two, in particular, use my PHP parser library, and that one uses the tokens internally. There is also other tooling that's more directly based on tokens, for example, code formatters or code style inspection tools like PHPCS. Those all directly operate on the tokens instead. 

Derick Rethans  7:47  
	But as you say, these tokens only are words and they don't really provide a structure. How would these tools then convert that into a structure? 

Nikita  7:54  
	If you're looking for, if you're looking just at formatting, then you may not really need a lot of structure. So you probably do need to write like that of extra code to recognise that, okay, the function token followed by white space, followed by an identifier, that's function declaration. For the more complicated tooling that builds a syntax tree, you need to implement a parser, either based in code generation, or based on recursive descent approach. 

Derick Rethans  8:26  
	Why would you not want to have direct access to PHP's AST instead because that already provides a structure for you?

Nikita  8:33  
	We do have direct access to the AST through the AST PECL extension, which is not part of core yet. I don't know if there are plans in that direction. 

Derick Rethans  8:43  
	Well you wrote it so you surely can make these plans. 

Nikita  8:46  
	Yes, I can make them but I don't know if I should make them. 

Derick Rethans  8:50  
	I think you should. 

Nikita  8:51  
	I mean, the nominal advantage of the AST extension is that it's always up to date with PHP. In practice that really isn't an issue, because some of the userland tooling is also pretty quickly updated. The more practical advantage is that the extension is a lot faster than implementing this in userland code. Well, I mean, this is really one of the areas where C code is faster than PHP code. The AST extension only exposes the structure that PHP itself needs. PHP is not interested in like precise formatting, and things like that at all. So it throws away quite a few things. You can, for example, get accurate on position information. Like, where, exactly not just which line but of which column, something is defined. And that's something you're quite often interested in. 

Derick Rethans  9:46  
	Also, from what I've known, it throws away all the comments unless they are doc bloc comments. How does the tokenizer currently return information about the tokens? I've played with this in the past and I didn't think it was the prettiest format to get back out of it. 

Nikita  10:02  
	token_get_all returns an array of tokens. And there are generally two types of tokens. One is single character tokens, like a semicolon, or a comma, or whatever, which are just returned as a string. So it's a single character string. And then there are complex tokens, like the function keyword, like white space, like strings, which are returned as an array where the first element is the token ID, which is an integer. And we have constants defined for these integers. The second element is the actual string content of the token. So for the function keyword, that's always going to be function, but it could be written in different ways because the keyword is case insensitive, so it could be all lowercase, or uppercase, hopefully it's all lowercase. 

Derick Rethans  10:52  
	You'll get the odd situation where the first letter is the capital, I suppose, but that's about it, hopefully. 

Nikita  10:57  
	And finally, the last element is the line number. So the starting line number. 

Derick Rethans  11:02  
	So if you want to look at the position on the line, you'd have to calculate it yourself? 

Nikita  11:08  
	Right you would have to track that yourself. I mean, there are two problems. One is just that you have these single character tokens and the complex tokens using different structure. So all the codes using them as to always switch back between those; check if it's an array or a string, or a test to do some kind of normalisation itself. And the second problem is that arrays in PHP are fairly memory inefficient when it comes to storing a fixed amount of data. Storing three elements inside an array always means allocating an array for eight elements. Because its minimum array size, you have to use space to store the key, and so on. Generally, if you have a fixed structure, it's much much more efficient to store it inside an object. Using a class that has declared properties. So this makes a very large difference in some cases, especially if your array only has like two or three elements, you can save a lot of memory with it. 

Derick Rethans  12:12  
	Have you done any benchmarks to see how much memory you'd actually save some likes some some particular scripts that you've parsed with how to tokenizer doesn't matter and how you proposing to do it?

Nikita  12:22  
	Yeah, I have here in the RFC, some numbers for some particular script that goes down from 14 megabytes to eight megabytes. So that's nearly half the memory usage. Well, actually, maybe I should first actually say what the RFC proposes. The RFCe proposes to instead return objects, an array of objects. And these objects have four properties. So first is again, the ID of the token, then the textual content, the line number, and also the starting position of the token in the string. 

Derick Rethans  12:54  
	Is this something that the tokenizer extension and tracks for you? 

Nikita  12:58  
	I mean, that's something that can easily do, so we can just as will expose it. And these objects are always used. So we no longer make the distinction between single character tokens and complex tokens. So we always return the uniform array of tokens, of token objects. Despite doing that, removing this optimization for a single character tokens, the end result is still that we use half as much memory, simply because objects are that much more efficient than arrays. 

Derick Rethans  13:27  
	That's a clever trick. I'm sure people like that, that using less memory, at least I know I would. Is it also faster or doesn't particularly matter much? 

Nikita  13:35  
	It's also faster, like maybe 30% or something, because memory usage and performance tend to be pretty heavily correlated. So if you use less memory, you also are faster. 

Derick Rethans  13:46  
	That makes sense. Are you thinking of other things that you can add to the tokenizer extension to make working with them even easier?

Nikita  13:52  
	The way this new functionality is implemented is, we have a PHP token class and on it we have a static method getAll. So instead of calling the token_get_all function, you call PHPToken::getAll(). And one nice thing this allows you to do is to extend this token class. So you can say, MyPHPToken extends PHPToken, and then you call MyPHPToken::getAll() and then we will actually construct your extension class. That means that you can add whatever methods you like, in addition to what we provide by default. 

Derick Rethans  14:29  
	Is that a pattern that we have in other places in PHP as well? Because I don't usually think that even if you'd call an inherited static method, why wouldn't suddenly return the inherited classes object? wDo we did it in other places? 

Nikita  14:42  
	So this is somewhat uncommon in PHP internals. I think it's a pretty common pattern for userland where generally if you return new objects from static methods, you always use new static, not new self. This is essentially late static binding, which we did discuss quite recently. So, there is one limitation here namely that the constructor of the PHPToken class is final. So, you can extend the class and you can add extra methods, but you cannot modify the construction behaviour, because we would like to internally construct these tokens very efficiently by more or less directly writing the values into the right slot in memory and not doing slow constructor calls, becouse this functionality tends to be very performance sensitive. And the same trick where you can extend the class but not change the constructor is also used by the SimpleXML extension. Does exist but not very common in, generally where internal code is concerned, we usually do not really plan for extension. I think nowadays we mark nearly all internal all new internal classes as final simply because extension is such a pain to deal with. And for old classes who usually wish that we had marked them as final. I mean, this is also a general recommendation for userland that, like you should mark things final as much as you can get away with it. But it's much bigger concern for internals because dealing with userland extensions that do unexpected things is much harder for us.

Derick Rethans  16:23  
	You even need to make sure that your internal structures are properly constructed by the parent's constructor being called from inherited classes but in PHP, there's no such requirement that you do. Pretty sure I've had problems with that for the Date extension a long, long time ago, where people would extend from it, not call the constructor. And then because he didn't think of it, nothing is defined and everything just falls down.

Nikita  16:44  
	Yeah, so this is one of the common problems. And the other one is that internal classes often define custom object handlers. So that's something only internal classes can do. Just to give one example, they can define debug info handler that modifies the output of var_dump, but nowadays we also have the user land magic methods on get you back into and I think pretty much all internal classes are just going to ignore that, and always return their own internal debug information even if this method has been overwritten, simply because no internal class actually checks for that. And this kind of problem also exists for a lot of other magic, and generally no one takes it into account, and things are just more or less softly broken. 

Derick Rethans  17:31  
	Very recently there was a pull request for Xdebug to change that as well because in Xdebug's debugging output get sent to IDEs. For internal classes always uses internal get debug handler, and for userland classes it uses whatever is userland defined; I mean if there's a magic method we'll use that. The pull request wanted to change Xdebug in such a way that it would also use the get debug info magic method for internal classes, whenever overridden. After some discussion about this, we figured out, this is probably a bad idea to do, and hence, we haven't merged that. Although we end up fixing some other things that the developer also found. That's a curious situation to be in. We would like us to be sort of work the same. But at the same time, you sometimes really want to see the internal information from the classes without developers having hidden the information behind it, right. 

Nikita  18:20  
	Yeah, that's true. 

Derick Rethans  18:21  
	And that is just from a from a debug perspective. And even from, let's make sure things don't crash perspective. I see that the RFC also rejected a few features that aren't part of the current iteration yet or might make sense to add it later. And one of them is about a lazy token stream. What would that be and what sort of different interface would it have? 

Nikita  18:43  
	The lazy token stream basically just means that instead of returning an array of tokens, we return an iterator of tokens, which means that we do not have to store the full array in memory, which, like for the example, I used. The memory usage for the whole token array was eight megabytes, even after these memory usage improvements, which wasn't a fairly large file, but definitely not the largest file. You can encounter especially when it comes to generated files. So there is an advantage of processing tokens one by one as a stream, because then your memory usage is going to be basically O(1), not O(n). The problem is, I mean, the PHP lexer does indeed work one token at a time, so it can support it. The problem is that it has a lot of internal state. And in order to implement this iterator, we would have to backup and restore the state on each produced token to make sure that it's still possible to for example, include and compile other files in the meantime. So this is something that can be improved;  we can make that cheaper, but that would be a larger effort. And I'm not really sure it's worthwhile because, while you can process one token at a time. And this is, for example, what the PHP parser does internally. Many practical applications in userland will generally want to have all tokens as an array. Because it makes it simply, makes things easier if you can always look ahead and look back. And I think it would be fairly hard to rewrite the existing libraries in terms of the latest tree. It may be a nice to have, but I'm not the most useful thing for it now.

Derick Rethans  20:32  
	What has been the feedback for this RFC?

Nikita  20:35  
	I think pretty good. This is something that we've already discussed years ago. Last time the discussion kind of got a bit got a bit sidetracked. Yeah, one of the dangerous when you start introducing object oriented interfaces. Well, actually, I just call this RFC object-based intentionally, because when you do object oriented then people would like to have their tokens, and their token streams, and their token stream factories, and the token stream managers. And this is basically held this the whole time. But generally everyone who is working on tokens, which is not a lot of people, but those who are working with them know that memory usage is a problem. And the current, current inconsistent structure is a problem, which is why most of them implement their own token objects, and basically do the same thing we propose here just themselves. 

Derick Rethans  21:30  
	When it's this one going up for a vote at the same time? 

Nikita  21:32  
	Soon.

Derick Rethans  21:33  
	Both of these RFCs that we spoken about today are both targeted to a PHP eight, I suppose?

Nikita  21:37  
	Yeah. So right now, I think all RFCs are targeted at PHP 8.

Derick Rethans  21:42  
	Thank you for taking the time with me today, Nikita to talk about a bunch of little RFCs that you've written. Perhaps by the time this podcast comes out, we've started voting on them and see what happens to them. 

Nikita  21:52  
	Thanks for having me once again.

Derick Rethans  21:56  
	Thanks for listening to this instalment of PHP internals news. The weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.


Show Notes
----------

- RFC: `Validation for abstract trait methods <https://wiki.php.net/rfc/abstract_trait_method_validation>`_
- RFC: `Object-based token_get_all() alternative <https://wiki.php.net/rfc/token_as_object>`_
- PSR-3 `Logger Interface <https://www.php-fig.org/psr/psr-3/>`_
- `PHPStan — PHP Static Analysis Tool <https://github.com/phpstan/phpstan>`_
- `Psalm <https://psalm.dev/>`_
- `PHP-AST <https://github.com/nikic/php-ast>`__
- `PHPCS <https://github.com/FriendsOfPHP/PHP-CS-Fixer>`__

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
