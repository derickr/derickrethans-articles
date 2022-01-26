PHP Internals News: Episode 97: Redacting Parameters
====================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2022-01-27 09:09 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin097
   :Enclosure: media/pin-097.mp3

In this episode of "PHP Internals News" I chat with Tim Düsterhus
(`GitHub <https://github.com/TimWolla>`_) about the "Redacting Parameters in
Back Traces" RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-097.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:00  
	Before we start with this episode, I want to apologize for the bad audio quality. Instead of using my nice mic I managed to use to one built into my computer. I hope you'll still enjoy the episode.

Derick Rethans  0:30  
	Hi, I'm Derick. Welcome to PHP internals news, a podcast dedicated to explaining the latest developments in the PHP language. This is episode 97. Today I'm talking with Tim Düsterhus about Redacting Parameters in Backtraces RFC that he's proposing. Tim, would you please introduce yourself?

Tim Düsterhus  0:50  
	Hi, Derick, thank you for inviting me. I am Tim Düsterhus, and I'm a developer at WoltLab. We are building a web application suite for you to build online communities.

Derick Rethans  0:59  
	Thanks for coming on this morning. What is the problem that you're trying to solve with this RFC?

Tim Düsterhus  1:05  
	If everything is going well, we don't need this RFC. But errors can and will happen and our application might encounter some exceptional situation, maybe some request to an external service fails. And so the application throws an error, this exception will bubble up a stack trace and either be caught, or go into a global exception handler. And then basically, in both cases, the exception will be logged into the error log. If it can be handled, we want to make the admin side aware of the issues so they can maybe fix their networking. If it is unable to be handled because of a programming error, we need to log it as well to fix the bug. In our case, we have the exception in the error log. And what happens next? In our case, we have many, many lay person administrators that run a community for their hobby, they're not really programmers with no technical expertise. And we also have a strong customers help customers environment. What do those customers do? They grab their error log and post it within our forums in public. Now in our forum, we have the error log with the full stack trace, including all sensitive values, maybe user passwords, if the Authentication Service failed, or something else, that should not really happen. In our case, it's lay person administrators. But I'm also seeing that experienced developers can make this mistake. I am triaging issues with an open source software written in C. And I've sometimes seeing system administrators posting their full core dump, including their TLS certificates there, and they don't really realize what they have just done. That's really an issue that affects laypersons, and professional administrators the same. In our case, our application attempts to strip those sensitive information from this backtrace. We have a custom exception handler that scans the full stack face, tries to match up class names and method names e.g. the PDO constructor to scrub the database password. And now recently, we have extended this stripping to also strip anything from parameters that are called password, secret, or something like that. That mostly works well. But in any case, this exception handler will miss sensitive information because it needs to basically guess what parameters are sensitive values and which don't. And also our exception handler grew very complex because to match up those parameters, it needs to use reflection. And any failures within the exception handler cannot really be recovered from, if the exception handler fails, you're out of luck.

Derick Rethans  3:51  
	Quite a few things to think of to make sure that you're not sharing any secrets. And I certainly have seen almost doing this myself. We now know what the problem is. How is this RFC proposing to fix this?

Tim Düsterhus  4:03  
	Primarily, we want to propose a standardized way for applications or libraries to indicate which parameters hold sensitive values. Our custom exception handler uses reflection as we said before, and it only matches up the parameter's names, but we also have this attribute I am proposing, SensitiveParameter within our application itself. Any parameter names that are not definitely sensitive can be attributed with this attribute. But this only works within our software, but not with any third party libraries we are using, e.g. for encryption or whatever there is. Primarily we want to propose a standardized way an attribute that is in PHP core, anyone can use that and everyone knows what this attribute means. Secondarily, the RFC is proposing a default implementation to keep the exception handler simple. As I said before, we are using reflection. This is very complex, it does not work with the require_once or include_once family, because that are not functions. We need to handle this case to not try to attempt to reflect on those non functions when redacting any parameters. This is complex. And we want to simplify that. 

Derick Rethans  5:20  
	From what I understand this is then a way to make sure that there's a standardized method for marking arguments as being sensitive. And because this is that now standardized, only one solution to the problem has to be found right?

Tim Düsterhus  5:34  
	Basically, not every library is using their own attributes, possibly, or we can match parameter names that are not like password, secret, but it can be documented: hey, if you are using sensitive parameters, you should put this attribute and then those exception handlers will be aware that this attribute is sensitive and can strip it, or in case of the RFC PHP itself, will already strip those parameters from the stack trace.

Derick Rethans  6:04  
	You're suggesting that PHP standard way of showing stack traces also takes care of the sensitive parameter here? 

Tim Düsterhus  6:11  
	Yes, exactly.

Derick Rethans  6:13  
	Which internal PHP functions are likely to get this attribute?

Tim Düsterhus  6:16  
	Basically anything with a parameter called password or secret, as I said before, examples include PDO's constructor, the database password will be in there and possibly also the user name or host name, which might be considered sensitive. But the password is the most important thing I have on my list. ldap_bind, which possibly includes user passwords; the password_hash function; possibly various OpenSSL functions. One will need to look and this list can be extended in the future as well, if someone realizes we missed anything.

Derick Rethans  6:55  
	Now, I know sometimes that there's a problem where an application connects to the wrong server with PDO. And as you say, the host name was also in this PDO constructor, would it not then make debugging that specific case harder because the hostname would also be redacted from the stack traces?

Tim Düsterhus  7:14  
	The attribute I am proposing as the parameter attribute, each parameter can be sensitive or non sensitive. We would need to decide whether we consider the hostname sensitive or not. It usually is not. So I would not put the attribute on the host name, or on the DSN string in the first parameter. The password definitely is sensitive. And the username possibly is a grey area. By default, I probably would not put the attribute there. But this is something that needs to be discussed in the greater community possibly.

Derick Rethans  7:47  
	I saw in the RFC that when you request a stack trace in PHP with get back trace or whatever the name of this function is, is that the sensitive parameters are being replaced by an object of the class SensitiveParameter. Why did you pick that instead of just a string, saying something like "redacted".

Tim Düsterhus  8:06  
	We cannot force users to put the attribute only on parameters that take strings. If we use a redacted string we might violate the type hint. If a function takes some key pair class, or an option of a key pair class, this usually is a sensitive attribute, we cannot simply put a string there. We can but then we would violate the typing. And as we violate the typing in at least some of the cases, we can also violate it in all of the cases and then make it very clear that this parameter was redacted and not a real value that just looks like a string "redacted". Exception handlers would be able to use an instanceof SensitiveParameter check to possibly make it more user friendly when they render the stack trace. When you using an GUI to handle your exceptions as such a Sentry can show some placeholder instead of pretending it's a real string in there. 

Derick Rethans  8:07  
	And of course, the string "redacted" can already exist as an argument value yet anyway, right?

Tim Düsterhus  9:12  
	Yeah.

Derick Rethans  9:13  
	Where would attribute be checked?

Tim Düsterhus  9:16  
	My proposal would extend PHP to check this attribute within the function that generates the stack trace, because as I said, I want to keep my exception handler simple, so they won't need to use reflection to check this attribute. PHP itself will check this attribute when the stack trace is generated. So no exception handler can miss to check this attribute.

Derick Rethans  9:39  
	Would it be possible for code that checks for SensitiveParameter to see what the original value was? I can imagine that in some cases, an exception handler as part of a debugging toolbar, whatever does want to show this extra information, although there's going to be hidden by default.

Tim Düsterhus  9:58  
	Not with the current version of my RFC, but I can imagine that this sensitive parameter replacement value gets an attribute where the original value can be stored. Care would need to be taken, so exception handlers don't simply serialize that value and ship it to a third party service, basically negating the benefit. But a future extension, or maybe the further discussion of my RFC can extend this replacement value. So you can use sensitive parameter, arrow, original value, or whatever.

Derick Rethans  10:34  
	In PHP attributes are basically markers on parameters or arguments. But they don't necessarily have to have an object implementation. Is your RFC also including the SensitiveParameter class that PHP core implements?

Tim Düsterhus  10:51  
	Yes, in my current RFC, and my current proof of concept implementation, I'm just reusing that attribute class as the replacement value within the stack trace. So we can kill two birds with one stone by doing that, by including proper class, also, any IDE will be able to see that class and know where that attribute can be applied. Because attributes have a property where they say where they can be applied in this case parameters only. And by putting it on the method by accident, you will possibly get an error or the IDE can warn you that you're doing this not correctly,

Derick Rethans  11:32  
	You might be aware that I work on Xdebug, a debugger for PHP. And in many cases, some of the users have already previously said that Xdebug should, for example, follow the debug_info() magic method on objects to show redacted information. Now, would you think that when people debug PHP with a debugger such as Xdebug, should they see the contents of the arguments that are set with SensitiveParameter, or should it stack traces show the real value?

Tim Düsterhus  12:07  
	In case of debugging, you're not usually not in production. So within your debugging environment or development environment, you shouldn't really have any sensitive value such as passwords, or credit card numbers, or whatever there is. In that case, debugability and ease of development should be more important. Xdebug, or any other debugger should see through those sensitive attributes and show the real value, possibly with an indicator that this value would usually be sensitive. But you shouldn't need to work around PHP hiding something from you, because you really want or need to see what happens there.

Derick Rethans  12:48  
	Now Xdebug also override PHP's standard exception handler, and then creates a stack trace of its own. Do you think that should redact the SensitiveParameter arguments?

Tim Düsterhus  13:00  
	I'm not really sure if people run this in production. If this is something people usually do, then of course, Xdebug should make sure to redact those values, possibly with a special ini flag or something. If that's only used in development. In my case, I only use Xdebug in development and production servers don't have that; you don't really connect to your production server with your IDE and then step through the code. That does not happen. So we don't need Xdebug in production.

Derick Rethans  13:32  
	I know some people do run Xdebug in production. But I also don't think those are the people that care about leaking sensitive parameters. I think the RFC talks about a few existing features that PHP already has for redacting some values. What are these? And how are they not sufficient?

Tim Düsterhus  13:49  
	There are two php.ini values you can set. One of those is do not collect parameters in stack traces, I don't have the exact name. But basically, all functions will just show an empty parameter list within the stack trace. That makes debugging very hard, especially with PHP and the non-strict typing, it can happen that you pass some completely invalid value to a function, even in production after testing and such. And you really want to know about this value, because it makes debugging very hard. Not collecting the parameters makes the stack traces much, much less useful. So this targeted redaction, as I'm proposing, hides the sensitive values but the non sensitive values will still be visible. And the other one is that the length of collected strings within the stack place can be configured. By default. I think it's on 15, but 15 characters already include user passwords such as password, exclamation mark, or 12345. And also credit card numbers will be exposed to three fourths by then. And the last four digits are shown in clear text on many pages. So that doesn't really help with those type of user credentials. Of course, your database password might be 40 characters completely random. But that's not really the values you want, or need to protect, because the database server will not be exposed to the internet, in many cases.

Derick Rethans  15:33  
	What has the feedback been so far to this RFC?

Tim Düsterhus  15:36  
	Both positive, and "we don't need that nobody does that". It's a bit mixed. I've got some very good feedback. There's a Twitter account that tweets any new RFCs. And so the users on Twitter, the actual users, and not PHP internals list seem to be very happy with my proposal. On the list, many said, just don't log that values, or they don't really see the benefit yet, I think. Not really sure how the feedback is really.

Derick Rethans  16:07  
	That's always a tricky thing, isn't it? Because the people that think "Oh, this is all right", often bother responding, because they don't have anything to add or criticize.

Tim Düsterhus  16:17  
	Exactly. People that are happy won't write any reviews for whatever, just the people that complain are complaining.

Derick Rethans  16:24  
	Yeah, it's either the people that are complaining are the people that are really happy about something. Are you expecting there to be any backward compatibility breaks?

Tim Düsterhus  16:34  
	Yeah, obviously, when the attribute class name will be taken by default by PHP, userland code cannot use that any more. But I don't think that anyone is using a SensitiveParameter class in the global namespace. I used GitHub search and SensitiveParameter in PHP code only appears in some strings, in the AWS SDK or something like that. The replacement value will break any type signature. So if the exception handler checks, the original parameter types for whatever reason, that will, or might break, but I don't really think that's likely either. I don't expect any major backwards compatibility breaks.

Derick Rethans  17:17  
	That's good to hear. And also good to hear that you have done some research into this. Do you have any extra selling points to convince people?

Tim Düsterhus  17:26  
	My initial selling point was PDO's constructor. Or not really selling point, but example, because it's very obvious and it's in PHP core. I later expanded that with the credit card numbers and user passwords, and made, attempted to make this more clear that those sensitive values are not just values from your personal computing environment, but also something user input into your application. And that stack traces will be sent to third parties e.g. Sentry, which might even be run as a software as a service solution. And then your deep in GDPR territory. You don't want that.

Derick Rethans  18:03  
	No, absolutely not. Tim, thank you for taking the time this morning to talk to me about your RFC.

Tim Düsterhus  18:10  
	Thank you for having me.

Derick Rethans  18:15  
	Thank you for listening to this installment of PHP internals news, a podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening. I'll see you next time.


Show Notes
----------

- RFC: `Redacting parameters in back traces <https://wiki.php.net/rfc/redact_parameters_in_back_traces>`_
- `PHP RFC Bot on Twitter <https://twitter.com/PHPRFCBot/>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
