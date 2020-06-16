PHP Internals News: Episode 61: Stable Sorting
==============================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-07-02 09:24 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: pin061
   :Enclosure: media/pin-061.mp3

In this episode of "PHP Internals News" I chat with Nikita Popov (`Twitter
<https://twitter.com/nikita_ppv>`_, `GitHub <https://github.com/nikic/>`_,
`Website <https://nikic.github.io/>`_) about his Stable Sorting RFC.

The RSS feed for this podcast is
https://derickrethans.nl/feed-phpinternalsnews.xml, you can download_ this
episode's MP3 file, and it's available on Spotify_ and iTunes_.
There is a dedicated website: https://phpinternals.news

.. _download: /media/pin-061.mp3
.. _Spotify: https://open.spotify.com/show/1Qcd282SDWGF3FSVuG6kuB
.. _iTunes: https://itunes.apple.com/gb/podcast/php-internals-news/id1455782198?mt=2

Transcript
----------

Derick Rethans  0:18  
	Hi, I'm Derick, and this is PHP internals news, a weekly podcast dedicated to demystifying the development of the PHP language. This is Episode 61. Today I'm talking with Nikita Popov about a rather small RFC that he's proposing called stable sorting. Hello Nikita, how are you this morning? 

Nikita  0:36  
	Hey, Derick, I'm great. How are you? 

Derick Rethans  0:38  
	Not too bad myself. Let's jump straight in here. The title of the RFC is stable sorting, what does that mean, what is stable sorting, or what is sorting stability?

Nikita  0:48  
	Sorting stability refers to the behaviour of the sort when it comes to equal elements. And equal share means that we sort comparison function. For example, the one you pass to usort says the elements are equal, but there is still some way to distinguish them. For example, if you're sorting some objects, to take the example from the RFC, we have an array with users, and users have an age, and we use usort to only sort the users by age. Then according to the comparison callback all users with the same age are equal. But of course, the user also has other fields on which we can distinguish it. And the question is now in what order will equal elements appear. If we have a stable sort, then they will appear in the order they were originally in. So it's something not going to change. 

Derick Rethans  1:41  
	And that is not what PHP sorting mechanism currently does? 

Nikita  1:44  
	Right. PHP currently uses an unstable sort, which means that the order is simply unspecified. It will be deterministic. I mean if you take the same input array and sort it, then every time we will get the same result. But there is no well specified order or relative order of elements. There's just some order. The reason why we have this behaviour is that well there are, I would say, two, the only two sorting algorithms. There is merge sort. Which is a guaranteed n log n sort that the stable, but has the disadvantage that that requires additional memory to perform the merge step. The other side there is a quicksort, which is an average case n log n sorting algorithm and is unstable, but does not require any additional memory. And in practice, everyone uses one of these algorithms, usually with a couple of extensions on sort of merge sort. Nowadays we use timsort, but which is still based on the same underlying principle, and for quicksort, we have sort which is better than quicksort, which tries to avoid some of the bad worst case performance which quicksort can have. PHP currently uses us a quicksort, which means that our sorting results are unstable.

Derick Rethans  3:07  
	Okay, and this RFC suggesting to change that. How would you do that? How would you modify quicksort to make it stable? 

Nikita  3:15  
	Two ways. One is to just change the sorting algorithm. So as I mentioned, the really popular stable sorting is timsort, which is used by Python by Java and probably lots of other languages at this point. And the other possibility is to stick with an unstable source. So to stick with quicksort, but to artificially enforce that the comparison function does not have, does not report equal elements that are not really equal. And we can do that by introducing an extra artificial fallback comparison. We remember the order of the elements in the original array. And as the comparison function tells us that elements are equal. You will check against this original order, which means that, okay are sort of still unstable, but because the comparison, we'll never actually report that two elements are equal unless they really equal. It doesn't matter for the result.

Derick Rethans  4:16  
	So you're basically artificially changing the key to have the original index in the array. 

Nikita  4:24  
	That's pretty much exactly the implementation. And this is actually also how you would implement the stable sort if you'd do it in PHP code. So you would take your array and convert it into an array of pairs, where you have the original array value and the original position of the array element. Difference is just that if you do this in PHP code this is extremely extremely inefficient, in terms of memory and performance, while when we do it internally it's essentially free because we already have a little bit of unused space in each array element. We can easily store the current position.

Derick Rethans  5:02  
	Do you think there will be much of a performance hit here?

Nikita  5:04  
	So I expect that there is a bit of performance hit, but for typical usage, not much. For the good case where your array does not actually contain any equal elements, the overhead should be very small, something like maybe one or 2%,. If your array does contain a huge number of duplicates. Then there is more overhead, and the effect is basically that the sort performance, no longer depends on the number of duplicates you have. Previously if you had a lot of duplicates, then the sort became faster, the more duplicates you had. Well now, as you add more duplicates, the sorting performance will stay both stable. That's really the difference in performance. 

Derick Rethans  5:53  
	If you have the numbers in the RFC I'll make sure to link to them. There are possibility that is that this is going to break any code? 

Nikita  6:01  
	Yes, it could break tests. 

Derick Rethans  6:04  
	Tests, because the test's output can change because the sorting order of arrays might have changed.

Nikita  6:11  
	Exactly. So we already had such a change in PHP seven, where we switched from a pure quicksort, to a hybrid quicksort and insertion sort, which means that effectively we have a stable source for arrays smaller than 16 elements and an unstable source for larger arrays, which is weird, weird intermediate state. 

Derick Rethans  6:33  
	Yes. 

Nikita  6:35  
	I think that one already had quite a bit of fallout for testing purposes. Hopefully this one will be a little bit smaller because most tests will work on a few elements. Those would have already been stable previously. But there is definitely going to be a little bit of fallout for unit testing.

Derick Rethans  6:56  
	At the moment we're talking about this, the RFC's already up for voting. By the time this podcast has come out. It's pretty likely that it has been accepted for PHP eight, because I think the voting was 51 to zero or something like this.

Nikita  7:10  
	It's 36 to zero.

Derick Rethans  7:13  
	There you go. Thank you, Nikita for taking the time this morning to talk to me about stable sorting.

Nikita  7:19  
	Thanks for having me.

Derick Rethans  7:23  
	Thanks for listening to this instalment of PHP internals news, the weekly podcast dedicated to demystifying the development of the PHP language. I maintain a Patreon account for supporters of this podcast, as well as the Xdebug debugging tool. You can sign up for Patreon at https://drck.me/patreon. If you have comments or suggestions, feel free to email them to derick@phpinternals.news. Thank you for listening, and I'll see you next week.


Show Notes
----------

- RFC: `Stable Sorting <https://wiki.php.net/rfc/stable_sorting>`_

Credits
-------

.. credit::
   :Description: Music: Chipper Doodle v2
   :Type: Music
   :Author: Kevin MacLeod (incompetech.com) — Creative Commons: By Attribution 3.0
   :Link: https://incompetech.com/music/royalty-free/music.html
