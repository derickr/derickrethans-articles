PHP Internals News: Episode 73: Enumerations
============================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2021-01-28 09:01 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin073
   :Enclosure: media/pin-073.mp3

In this episode of "PHP Internals News" I talk with Larry Garfield (`Twitter
<https://twitter.com/crell>`_, `Website <http://www.garfieldtech.com/>`_,
`GitHub <https://github.com/Crell>`_) about a new RFC that he is proposing
together with Ilija Tovilo: Enumerations.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-073.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:14  
	Hi I'm Derick and welcome to PHP internals news that podcast dedicated to explain the latest developments in the PHP language.

Derick Rethans  0:22  
	This is Episode 73. Today I'm talking with Larry Garfield, who you might recognize from hits such as object ergonomics and short functions. Larry has worked together with Ilija Tovilo on an RFC titled enumerations, and I hope that Larry will explain to me what this is all about. Larry, would you please introduce yourself?

Larry Garfield  0:43  
	Hello World, I'm Larry Garfield, I am director of developer experience at platform.sh. We're a continuous deployment cloud hosting company. I've been in and around PHP for 20, some odd years now. And mostly as an annoying gadfly and pedant.

Derick Rethans  1:00  
	Well you say that but in the last few years you've been working together with other people on several RFCs right, so you're not really sitting as a fly on the wall any more, you're being actively participating now, which is why I end up talking to you now which is quite good isn't it.

Larry Garfield  1:15  
	I'm not sure if the causal relationship is in that direction. 

Derick Rethans  1:18  
	In any case we are talking about enumerations or enums today. What are enumerations or enums?

Larry Garfield  1:26  
	Enumerations or enums are a feature of a lot of programming languages, what they look like varies a lot depending on the language, but the basic concept is creating a type that has a fixed finite set of possible values. The classic example is Boolean; a Boolean is a type that has two and only two possible values: true and false. Enumerations are way to let you define your own types like that to say this type has two values, sort ascending or descending. This type has four values, for the four different card suits in a standard card deck, or a user can be in one of four states: pending, approved, cancelled, or active. And so those are the four possible values that this variable type can have. And what that looks like varies widely depending on the language. In a language like C or c++, it's just a thin layer on top of integer constants, which means they get compiled away to integers at compile time and they don't actually do all that much, they're a little bit to help for reading. At the other end of the spectrum, you have languages like rust or Swift, where enumerations are a robust Advanced Data Type, and data construct of their own. That also supports algebraic data types, we'll get into that a bit more later. And is a core part of how a lot of the system actually works in practice, and a lot of other languages are somewhere in the middle. Our goal with this RFC, is to give PHP more towards the advanced end of enumerations, because there are perfectly good use cases for it so let's not cheap out on it. 

Derick Rethans  3:14  
	What is the syntax? 

Larry Garfield  3:15  
	Syntax we're proposing is tied into the fact that enumerations as we're implementing them, are a layer on top of objects, they are internally objects with some limitations on them to make them enumeration like. The syntax is if I can do this verbally: enum, curly brace, a list of case foo statements, so case ascending, case descending, case hearts, case spades, case clubs, space diamonds, and so on. And then possibly a list of methods, and then the closed curly brace. So it looks an awful lot like a class because internally, the way that we're implementing them, the enum type itself is a class, it compiles down to a class inside the engine, but instead of being able to instantiate your own instances of it. There are a fixed number of instances that are pre created. And those are the only instances that are ever allowed to exist. And those are assigned to constants on that class, and you have the user status, then you have a user status enum, which has a case approved, or active, and therefore there is in the engine, a class user status which has a constant on it, public constant named approved, whose value is an instance of that object. And then you can use that like a constant, pretty much anywhere and it will just work. The advantage of that over just using an object like we're used to, is you can now type a parameter, or a property or return type for: This is going to return a user status, which means you're guaranteed by the syntax that what you get back from that method is going to be one of those four value objects, each of which are Singleton's so they will always identity compare against each other. 

Larry Garfield  5:07  
	So just like if you type something as Boolean, you know, the only cases you ever have to consider are true and false. If you type something as user status, you know, there are only four possible cases you ever have to think about, and you know exactly what they mean and you can document them, and you don't need to worry about what if someone passes in, you know, a string that says cancelled instead of rejected or something like that. That's syntactically impossible. 

Derick Rethans  5:32  
	What happens if you do that? 

Larry Garfield  5:33  
	You get a syntax error. If for example, a method is typed as having a parameter that takes a user status and you pass in the string rejected. Well, it's a string but what's expected is a user status object. So it's going to type error. 

Derick Rethans  5:47  
	You get a type error yes. 

Larry Garfield  5:49  
	If you try and pass in user status, double colon rejected, when that's not a type, you get an error because there is no constant on that class named rejected therefore, you'll get that error instead. Building on top of objects means an awful lot of functionality, kind of happens automatically. And the error checking you want to kind of happens automatically. And it's in ways that you're already used to working with variables in PHP. 

Derick Rethans  6:14  
	For example, it also means that enums will be able to be auto loaded because they're pretty much a class?

Larry Garfield  6:19  
	Exactly. They autoload exactly the same way as classes, you don't you didn't even do anything different with them just follow the standard PSR four you're already following, and they'll auto load like everything else. 

Derick Rethans  6:28  
	That also means that class names and enum names, share the same namespace then? You can't have an enum with the same name as a class.

Larry Garfield  6:36  
	Right. Again, making it built on classes means all of these. So it's kind of like a class, the answer is yes, almost always, and it makes it easier to work with is more convenient, it's like you're used to. 

Derick Rethans  6:47  
	I suppose it's also makes it easier to implement? 

Larry Garfield  6:50  
	Substantially. 

Derick Rethans  6:51  
	I'm familiar with enums in C, or c++ where enums is basically just a way of creating a list of integers, that's auto increase right, then I've been reading the RFC, it doesn't seem to do anything like that. How does this work, are the constants on the enums classes, are they sort of backed by static values as well?

Larry Garfield  7:12  
	They can be. There's some subtlety here that's one of the last little bits we're still ironing out as we record this, the user status active constant is backed by an object that is an instance of user status and has a hard coded name property on it, called active. That is public but read only. That's how internally in the engine, it keeps track of which one is which. And there is only ever that one instance of user status with the name active on it. There is another instance that has name cancelled or name pending or whatever. 

Derick Rethans  7:53  
	There's no reason to be more than one instance of these at all. 

Larry Garfield  7:58  
	Then you can look at that name property but that's really just for debugging purposes, most of the time you'll just use user status double colon active user colon user status colon approved, whatever as a constant and, roll, roll with that. The values themselves don't have any intrinsic primitive backing, as we've been calling it. You can optionally specify that enumeration values of this type, are going to have an integer equivalent, or a string equivalent. And those have to be unique and you have to specify them explicitly. The syntax for that would be enum user status colon string, curly brace, blah blah blah. And then each case is case active equals.

Derick Rethans  8:46  
	Is it required that the colon string part is part of your enum declaration?

Larry Garfield  8:51  
	If you wanted to have a primitive backing, yes, a scalar equivalent. We're actually calling its scalar enums at the moment because that's an easier word to say than primitive. The idea here is enumerations themselves conceptually should not just be an alias for a scalar, they have a meaning conceptually unto themselves rather than just being an alias to an integer. That said, there are plenty of cases where you want to be able to convert enumeration case/enumeration value, into a primitive to store it in a database, to show it on a web page, where I can't store the object in a database that doesn't really work well, MySQL doesn't like that, I added this ability to define a scalar equivalent of a primitive. And then, if you do that, you get two extra pieces that help you. One is a value property, that is again a public read only property, which will be that value, so if approved, or active is a, then user status double colon active arrow value will give you the value A. Which means if you have an enum and you want to save it to a database table, you just grab the value property and that's what you write to the database, and then it's just a string or integer or whatever it is you decide it to make that. When you then load it back up, there is a from method. So you can say user status double colon from. If it's some primitive, and it will return the appropriate object Singleton object for that type so user status double colon active. 

Derick Rethans  10:28  
	You mentioned that enums are quite like classes, does that also mean, you can define methods on the enums?

Larry Garfield  10:35  
	Enums can have methods on them, both normal objects methods and static methods. They can also take interfaces, and then they have to conform to that interface like any other class. They can also use traits, as long as the traits do not have any properties defined, as long as they are method only traits. We are explicitly disallowing properties, because properties are a way to track and maintain and change state. The whole point of an enumeration is that the enumeration value with the case is a completely defined instance of into itself and user status active is always equal to user status active.

Derick Rethans  11:16  
	And you can't do that, if there are properties.

Larry Garfield  11:19  
	Right. You don't want the properties of user status to change dynamically throughout the course of the script that's not a thing.

Derick Rethans  11:25  
	Yeah, what would you use methods for on enums?

Larry Garfield  11:29  
	There's a couple of use cases for dynamic methods or object methods, the most common would be a label method. As an example, if you have a user status enumeration, the example we keep going back to. You can have a method for label. Give each of them a make it a scalar enum, so they all have a string or an integer equivalent that you can save for a database, then you give them all a label method. A which in this case would be a single label method that has a match statement inside it and just switches based on the, the enumeration, expect matching enumeration fit together perfectly they're, they're made for each other,

Derick Rethans  12:07  
	And also match will also tell you if you have forgotten about one of the cases.

Larry Garfield  12:12  
	Ilija started working on match in PHP 8.0, he knew he wanted to do enumerations, I came in later. This is a long plan that he's been working on I've joined him. You have a label method that matches on the enumeration value, which is dollar this inside the method will refer to the enumeration object you're on, just like any other, and then return a human friendly label for each of those. You can then call a static method called cases, which will return an array of the enumeration cases you have, you'll loop over that and read the value property, and you read the label, and you stick that into a select box or checkboxes or whatever, you know template system you have. There you have a way to attach additional static data to a enumeration without throwing on more stateful properties. That's the most common case, there are others. You can have a enumeration method that returns a specific other enumeration case. Like if you're on enumeration you say what's next, you're creating a series of steps like a junior very basic state machine. And you call next and it'll return, another enumeration object on the same type for whatever the next one is, or whatever else you can think of them. I'm sure there are other use cases I'm not thinking of. For static methods, the main use case there is as an alternate constructor. So if you want to say you have a size enumeration small, medium, large, you have a from length static method, you pass it an integer, and it will map that integer to the small, medium or large enumeration case based on whatever set of rules how you want to define which size is which. Probably the most common case for our static method is that kind of named constructor, essentially, maybe others, cool, come up with some.

Derick Rethans  14:19  
	During our compensation you mentioned two functions: cases and from, where do these methods come from?

Larry Garfield  14:26  
	cases is defined on all enumerations period, it just comes with it. You are not allowed to define your own, and it returns. It's a static method, and it returns a list of the instances for that that enum type from the static method that is generated automatically on scalar enum so if there's a scalar backing, then this from method gets automatically generated, you cannot define it yourself it's an error to do so.

Derick Rethans  14:55  
	Is it an error, even if it isn't a scalar enum, to define the from method?

Larry Garfield  15:01  
	At the moment I don't believe so you can put a from on a non scalar enum, like unit enum. We recommend against it. Maybe we should block that just for completeness sake, I'll have to talk to Ilia about that. There's a few edge cases like that we're still dealing with. Nikita has a suggestion for around error handling with from, for what happens if you pass an invalid scalar to from, we're not quite sure what the error handling of that is. It may involve having an extra method as well. In that case, details like that we're still working on as we speak, but we're at that level of shaving off the sharp corners, the bulk of the spec is pretty well set at this point.

Derick Rethans  15:45  
	When I read the RFC it mentioned somewhere that adding enumerations to PHP is part of a larger effort to introduce something called algebraic data types. Can you explain what these are, and, and which future steps and proposals would additionally be needed to support these there?

Larry Garfield  16:05  
	We're deliberately approaching this as a multi part RFC. The larger goal here is to improve PHP's ability to make invalid states, unrepresentable, which is not a term that I have coined I've read it elsewhere. I think I've read it multiple elsewhere, but its basic idea is you use the type system to make it impossible to describe states that are allowed by your business rules, and therefore, you don't need to write error handling for them. You don't need to write tests for them. Enumerations are a form of that, where you don't need to write a test for what happens when you pass user status joking, to a method, because that's a type error, that's already covered, you don't need to think about it any more.

Derick Rethans  16:53  
	Would that be a type error or would you get an error for accessing a constant, that doesn't exist on the enum if you say user status joking?

Larry Garfield  17:02  
	I think you'd actually get an error on a constant that doesn't exist in that case. If you just pass a string, an invalid string to the method and you would get a type error on string doesn't match user status, and the general idea being you structure your code, such that certain error conditions, physically can't happen. Which means you don't need to worry about those error conditions, it's especially useful where you have two things that can occur together but only in certain combinations, my favourite example here, if you're defining an airlock. You have an inner door and an outer door. The case of the indoor and outdoor both being open at the same time is bad, you don't want that. So you define your possible states of the airlock using an enumeration to say: inner door closed out other door closed, is one state is one enumeration case; inner open outer closed as one state, and inner closed outer open is one state. Those are the only three possible values that you can define so you cannot even describe the situation of both doors are open and everyone dies. Therefore, you cannot accidentally end up in that state. That's the idea behind making invalid states unrepresentable. Algebraic data types are kind of an extension of the idea of enumerations further by adding the ability for the cases, to have data associated with them. So they're no longer singleton's. There's a number of different ways that you can implement that different languages do it in different ways. Most of them, bake them into enumerations somehow for one reason or another, a few use the idea of sealed classes, which are a class, which is allowed to be extended by only these three or four or five other classes, and those are the only subclasses you're allowed to have. They're conceptually very similar, we're probably going to go with enumeration based, but there's still debate about that, there's no implementation yet. The idea with ADTs as we're envisioning them is, you can define, like, like you can have an enum case that is just its own thing. You can have an enum case that's backed by a scalar, or you can have an enum case, that has data values associated with it. And then it's not a singleton, can spawn new instances of it. But then those instances are always immutable. The most common example of that that people talk about would be: You can now, implement, monads very easily. I'm not going to go into the whole, what is a monad here.

Derick Rethans  19:36  
	It's a word that I've heard many times but have no idea what it means.

Larry Garfield  19:40  
	You should read my book I explained it wonderfully. But it allows you to say, the return value of this method is either nothing, or there is something and that something is this thing. An enumeration, where the two possible types are nothing, and something. And the something has parametrised with the thing that it actually is carrying. That's one use case. Another use case Rust actually in their documentation is a great example of this. If you're building a game, then the various moves that a player can take: turn left, turn right, move forward, move backward, shoot, back up, those are a finite set of possible moves so you make that an enumeration. And then those are the only moves possible in the code. But some of them, like, move forward by how much. By how much is a parametrised value on the move forward enum case. And you can then write your code to say alright, these are the five possible actions a player can take; these three have no extra parameters, this one has an extra parameter. And I don't need to think about anything else because nothing else is can even be defined. So that's the idea behind ADTs, or algebraic data types. There are way to do the kind of things you can do now just in a much more compact and guaranteed fashion, you can do that same thing now with subclasses, but you can't guarantee that no one else is adding another subclass you have to think about. With an ADT you can guarantee that there's no other cases and there will be no other cases. 

Larry Garfield  21:17  
	Another, add on that we're looking at is pattern matching. The idea here is match right now, the match construct in PHP eight, do an identity match. So match variable against triple equals, various possibilities, and that covers a ton of use cases it's wonderful. I love match. However, it would be also very useful to say match this array, let's say an associative array, against a case where the Foo property is bar, I don't care about the rest. If the case where the foo property is beep. And I don't care about the rest I'm gonna do stuff with the rest on the right side of match. With enumerations, that comes in with ADTs, where I want to match against: Is it a move left match against move left and then I want to extract that how far is the right hand side and do something with that. If it's a move right on extract the how far is that on the right. If it's a turn left. Well that doesn't have an associated property so I can match that and do whatever with that. It's a way to deconstruct some value, partially, and then take an action based on just part of that object. That's a completely separate RFC, that, again, we think would make enumerations, even more powerful, and make it a lesson number of other things more powerful of make the match statement, a lot more robust, all the pieces of that we're looking at is looking at match against type, so you can do an instance of or an isint or whatever check as part of the match. Again this is all still future RFC nothing has actually been coded for this yet, it's in the, we would like to do this next if someone lets us

Derick Rethans  23:12  
	Coming back to this RFC. What has the feedback been to it so far?

Larry Garfield  23:17  
	Very positive. I don't think anyone has pushed back on the idea. Or earlier draft used a class per case, rather than object per case, which a couple of people push back on it being too complicated so we simplified that to the object per case. And at this point, I think we're just fighting over the little details like: Does that from method return null, or does it throw? What does it throw? Do we have a second method that does the opposite? Exactly what does reflection look like? We have an isenum function or do we have an enum exists function? We're at that level. I expect this RFC is probably going to pass with like 95%. At this point, something like that. It's not a controversial concept, it's just making sure all the little details are sorted out, the T's are crossed, and the i's are dotted and so on.

Derick Rethans  24:09  
	Did we miss anything in the discussion about enums, do you have anything to add?

Larry Garfield  24:15  
	I don't think so, at least as far as functionality is concerned. To two other things process wise: one, I encourage people to look at this multi stage approach for PHP. I know I've talked about this on, on this podcast before; having a larger plan that you can work on in pieces and then multiple features that fit together nicely to be more than the sum of their parts. It's a very good thing, we should be doing more of that, and collaborating on RFC so that small targeted functionality adds up to even more functionality when you combine them, which is what we're looking to do with enums, and ADTs, and pattern matching for example. Credit where it's due, Ilija has done all of the code for this patch. I'm doing design and project management on this, but actually making it work is 100% Ilija he deserves all the credit for that, not me.

Derick Rethans  25:06  
	Thank you, Larry for explaining enums to me today. I learned quite a lot.

Larry Garfield  25:11  
	Thank you. See you on a future episode, hopefully.

Derick Rethans  25:16  
	Thank you for listening to this instalment of PHP internals news, a podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool, You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next time.

Show Notes
----------

- RFC: `Enumerations <https://wiki.php.net/rfc/enumerations>`_
- Episode #51: `Object Ergonomics <https://phpinternals.news/51>`_
- Episode #69: `Short Functions <https://phpinternals.news/69>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
