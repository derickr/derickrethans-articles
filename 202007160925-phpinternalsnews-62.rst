PHP Internals News: Episode 62: Saner Numeric Strings
=====================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-07-16 09:25 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin062
   :Enclosure: media/pin-062.mp3

In this episode of "PHP Internals News" I talk with George Peter Banyard
(`Website
<https://gpb.moe>`_, `Twitter
<https://twitter.com/Girgias>`_, `GitHub <https://github.com/Girgias>`_,
`GitLab <https://gitlab.com/Girgias>`_)
about an RFC that he has proposed to make PHP's numeric string handling less
complicated.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-062.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:17  
Hi, I'm Derick, and this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 62. Today I'm talking with George Peter Banyard about an RFC that he's proposing called saner numeric strings. Hello George, how are you this morning?

George Peter Banyard  0:36  
	How are you I'm doing fine. I'm George Peter Banyard. I work on PHP, and I'm currently employed by The Coding Machine to work on PHP.

Derick Rethans  0:46  
	I actually think I have a bug swatter from The Coding Machine, which is hilarious. Huh, I can't show you that okay of course in a podcast and not on TV. But yes, I think I got it in Paris at some point at a conference there, and it's been happily getting rid of flies in my living room. Anyway, that's not what we want to talk about here today, we want to talk about the RFC that is made, what is the problem that is RFC is hoping to address?

George Peter Banyard  1:09  
	PHP has the concept of numeric strings, which are strings which have like integers or floats encoded as a string. Mostly that would arrive when you have like a get request or post request and you take like the value of a form, which would be in a string. Issue is that PHP makes some kind of weird distinctions, and classifies numeric strings in three different categories mainly. So there are purely numeric strings, which are pure integers or pure floats, which can have an optional leading whitespace and no trailing whitespace. 

Derick Rethans  1:44  
	Does that also include like exponential numbers in there?

George Peter Banyard  1:48  
	Yes. However trailing white spaces are not part of the numeric string specification in the PHP language. To deal with that PHP has a concept of leading numeric strings, which are strings which are numeric but like in the first few bytes, so it can be leading whitespace, integer or float, and then it can be whatever else afterwards, so it can be characters, it can be any white spaces, that will consider as a leading numeric string. The distinction is important because PHP will sometimes only accept pure numeric strings. But in some other place where we'll accept leading numeric strings. Of casts will accept whatever string possible and will try to coerce it into an integer. In weak mode, if you have a type hint. It will accept leading numeric strings, and it will emit an e_notice that a non well formed string has been encountered. When you use like a purely string string, you'll get a non numeric string encountered warning. So the main issue with that is that like strings which have a leading whitespace are considered more numeric by PHP than strings with trailing whitespaces. It is a pretty odd distinction to make.

Derick Rethans  3:01  
	For me to get this right, the numeric string in PHP can have whitespace at the start, and then have numbers. There's a leading numeric string that can have optional whitespace in front, numbers and digits, and then rubbish. Then there's a non numeric string which never has any numbers in it.

George Peter Banyard  3:22  
	No numbers in the beginning. "HelloWorld5" will be considered non numerical. 

Derick Rethans  3:26  
	So it's a string that doesn't start with digits.

George Peter Banyard  3:29  
	Yes, or optional whitespace.

Derick Rethans  3:31  
	So there are three different numeric strings, sort of. There're two, and then one that is a string that doesn't have numbers. And you mentioned that some places. These are accepted and in other places they're not. So typecast will accept both numeric strings and leading numeric strings. Where is the leading numeric string, not accepted?

George Peter Banyard  3:53  
	If you use is_numeric call, it'll only return true on pure numeric strings.

Derick Rethans  4:00  
	And they have whitespace ain the end?

George Peter Banyard  4:02  
	They can only have leading white spaces. Explicit typecasting will work regardless, so even on non numeric strings, an int cast that will convert it to to zero, because that's how tight juggling works in PHP, and it will do. American leading numeric strings, it will take us to the initial leading numeric. 

Derick Rethans  4:27  
	And stripping out leading whitespace if there's any?

George Peter Banyard  4:30  
	Strip stripping leading white spaces and stripping garbage out of the end if it's a just a leading numeric string. String to string comparison with the double equal comparison operator will perform a numeric compare comparison, only if both strings are numeric, purely numeric. Whenever you do a string to int, or float comparison, the string will get type juggled to an int or to a float, regardless of its numericness. So, we'll get non numeric string for get typecast into zero implicitly, and you'll get warnings, but it has some odd behaviour. In weak typing mode, so strict types disabled, an int typecast where an int type declaration for an argument. When you pass it an numeric string to it. If it's a leading numeric string, it will convert it was an E notice, and it will do a type error if it's a non numeric string. This can be a slight issue, if you for example you pass in a hash, it should be a string. As always, but it starts with like a digit, then it will get type juggled to an int. And it will pass the type declaration check and just like work with.

Derick Rethans  5:54  
	 And you're get a notice?

George Peter Banyard  5:56  
	So you get a notice. Whereas like if it's, if it would be an a hash was just purely which starts with a with a character, you would get an e_warning, as in like a non well formed string like numeric string has been encountered.

Derick Rethans  6:10  
	That sounds quite complicated. You mentioned that there's one other place where you can use numeric strings, which is in array keys.

George Peter Banyard  6:21  
	Yes, array keys and string offsets. So array keys have a special semantic, which are like integer strings, which are separate concept and kind of same; as in, it needs to start with a nonzero digit, or be zero. For the zero index. It needs to be only digits, and that will be interpreted as an integer key. Otherwise, anything else will be interpreted as a string key, "5.5", which is a float like a numeric float string, will stay as "5.5" as the array key. This behaviour is different to string offsets.

Derick Rethans  7:07  
	So you're saying that a string with "5.5" in it, in array key stays "5.5"?

George Peter Banyard  7:15  
	Yes, and the same if you have a string key which is "03", you'll get a string key which is "03", it won't get evaluated as three. You can try it yourself, because it is the most weirdest behaviour, ever. I got what's quite surprised about that.

Derick Rethans  7:32  
	You are correct, but if it's a float it gets truncated. 

George Peter Banyard  7:36  
	Yes, to five. 

Derick Rethans  7:38  
	Hey, I've learned something new here, I thought that would also truncate.

George Peter Banyard  7:41  
	That would be kind of logical, in some sense, but it doesn't.

Derick Rethans  7:46  
	Continuing

George Peter Banyard  7:47  
	Array offsets have this behaviour, string keys have the more usual behaviour of using numerical, like numeric strings, as there can't be a string offset first, like it can only be like an integer. So that's why it's more lax, in some sense, it will use the usual semantics. However, if the numeric string is a float, or if it's a leading integer string, it'll emit the illegal string offset warning, but still used explicit int cast to cast it to an integer. "2str" would be cast to two, like a string index "foo" would be casted to zero, and "5.5" would be cast it to 5. It's all kind of confusing I wish doesn't follow other illegal offset behaviour for some sentence. If you try to pass an array as a as an offset you'll get a type error in PHP 8.

Derick Rethans  8:55  
	I have to admit, I am totally getting lost here. This sounds also complicated, and that something needs to be done about this. Am I correctly understanding that this is exactly what your RFC is trying to do?

George Peter Banyard  9:08  
	Yes, this is an attempt to bring back sanity into this whole mess.

Derick Rethans  9:13  
	So what are you proposing here?

George Peter Banyard  9:14  
	The proposal is to get rid of the concept of leading numeric strings, because it's mostly weird, and it's more confusing than it needs to be. To do that, numerical strings, will accept trailing white spaces. So numeric string which has leading whitespace won't be more numeric than a string with trailing white spaces. On top of that, all current, e_notices a non well formed numeric value encountered, will be changed to emit a non numeric value encountered e_warning. There's a promotion and severity in some sense as well. Should only affect purely non numeric strings, or leading numeric strings with have jibberish after the digit. For string offsets, numeric strings which correspond to well formed floating point numbers will emit the more usual string offset cast occurred warning, instead of the illegal string offset. Leading numeric strings which currently emit a non well formed numeric value and countered notice will emit the illegal string offset, and still continue to evaluate the previous value to ease the migration to PHP eight and for backwards compatibility. However, non numeric strings, which don't represent a number at all. Now throw in an illegal offset type error. This would affect our estimates operation on strings, so plus minus, multiplication, etc. Then float type declarations. So, in turn, float type declaration for internal and user land functions. Comparisons operator which considered that numeric strings with trailing white spaces weren't numeric, and so would produce false, say for example, the string "123 ", equal, equal to string " 123" will now produce true instead of false. The built in is_numeric function would return true for numeric strings which have trailing white spaces, where before it would emit false. And the plus plus, minus minus, increment, decrement operators would convert numeric strings with trailing white spaces to integers or floats and use the numerical increment instead of the alphanumeric would increment rules. 

Derick Rethans  11:35  
	You say whitespace, do you just mean the space characters or does it include like tabs and returns as well? 

George Peter Banyard  11:43  
	Tabs, new lines vertical ,spaces. Mostly what would consider white spaces.

Derick Rethans  11:48  
	I guess there's a horizontal tab and a vertical tab and stuff like that. What's the potential for for breaking changes here because messing around with PHP's type juggling rules is always a bit tricky. What are the BC implications here?

George Peter Banyard  12:05  
	I would expect most reasonable code to not be affected. It changes, one which is relatively minor, which is, if you, for some reason, your code needs the string to be numeric and only have leading white spaces, but no trailing white spaces, which is a pretty specific requirement. Then accepting trailing white spaces would break that code, because that would be considered a valid numeric string, whereas the code assumes that would be non non well formed, which is an odd requirement to have. That's why I don't expect it to be that big. Second one, more problematic one, is code which has liberal use of leading numeric strict. If for example you pass the DOM, an XML or a CSS file or something, and you get 2px, for example, for 2 pixel. And you just take that string, and dump it into various things and expect it to get two out of it. Sometimes you will need to now use an explicit cast to get the previous behaviour. That would be notified by you or by the by an e_notice in PHP 7.4, and it would it would inform you with a e_warning in PHP 8.

Derick Rethans  13:28  
	Considering you get a warning ish thing in both cases it's not really a BC break, I mean it's not suddenly going to start throwing an exception, which could break your code flow for example.

George Peter Banyard  13:39  
	Yes, and also all behaviour should be identical to PHP 7.4 and PHP 8. If there wasn't a warning before, if it was a notice, and it's been moved to a warning, the behaviour should be the same, except for like non numeric strings which sometimes will emit a type error, that's most likely a bug, were you expecting something to be an integer like and it's just pure or strict.

Derick Rethans  14:07  
	Oh, of course for user input, we know we shouldn't casting anyway, we should use the filter extension to get to this data, does this impact the filter extension at all?

George Peter Banyard  14:19  
	No, I don't think so. I don't think the filter extension uses the C is_numeric, is_numeric_string function. And it uses its own parsing of strings. 

Derick Rethans  14:30  
	Have you gotten any feedback about this so far? 

George Peter Banyard  14:33  
	Some feedback was to clarify some of the changes if it would affect code. Also, I had some doubt about how to handle the string offset case, which initially one of the proposals was to promote the leading number of strings to emit the warning, but also returned zero instead of returning the previous value, which would be pretty hard to detect, although they emitted a notice previously. So I've changed that again to like more in line with the behaviour, it has in PHP seven, where it just truncates the gibberish and cast it to an integer. So at least that BC concern should be removed.

Derick Rethans  15:24  
	As I mentioned, this is all pretty hard to wrap my head around, not because you don't explain this correctly, but mostly because it's so complicated to begin with. I would probably recommend that people that listen to this podcast episode would also have a look at the RFC, because it will come with examples in the cases as well, and sometimes just looking at the examples is a lot easier than listening to the exact descriptions of strengths as parsed by the PHP engine.

George Peter Banyard  15:53  
	Yes, which, at time can be mostly weird and nonsensical, but mostly based on Perl semantics.

Derick Rethans  16:02  
	Sometimes we steal from Java, sometimes we steal from Rust, and sometimes some Perl it seems them. And there's nothing wrong with that.

George Peter Banyard  16:10  
	There's nothing wrong, and in some sense, if you steal all the good things you get a better language, and sometimes you make some slight mistakes along the way.

Derick Rethans  16:19  
	let me not start about the @@ operator. We'll keep that for another episode, maybe. 

George Peter Banyard  16:25  
	Yes. 

Derick Rethans  16:26  
	When do you think you're going to put this up for a vote?

George Peter Banyard  16:29  
	So I started the discussion early this week. So on the 29th of June. I would expect the two weeks discussion period, because feature freezes coming up pretty soon. It needs to be voted on before and implemented into core before that. Voting should start on the 13th of July for two weeks until the 27th, which would give like another week to land stuff; to land it into core and tweak the implementation details.

Derick Rethans  16:59  
	I'm expecting a lot more RFCs just wanting to get in, just before the deadline.

George Peter Banyard  17:05  
	I suppose so, it's also kind of difficult because getting really tight. 

Derick Rethans  17:09  
	Okay, George. Thanks for this. Would you have anything else to add?

George Peter Banyard  17:13  
	No, thanks for having me on the show again Derick, and I hope you have a nice evening.

Derick Rethans  17:17  
	Thanks very much. 

	Thanks for listening to this installment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.



Show Notes
----------

- RFC: `Saner numeric strings <https://wiki.php.net/rfc/saner-numeric-strings>`_
- Related `Saner string to number comparisons <https://wiki.php.net/rfc/string_to_number_comparison>`_ RFC
- Related `Permit trailing whitespace in numeric strings <https://wiki.php.net/rfc/trailing_whitespace_numerics>`_ RFC

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
