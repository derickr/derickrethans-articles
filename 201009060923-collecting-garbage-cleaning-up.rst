Collecting Garbage: Cleaning Up
===============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2010-09-06 09:23 Europe/London
   :Tags: blog, php

This is the second part of three-parts column that was originally published
in the `May 2009`_ issues of `php|architect`_.

.. _`May 2009`: http://www.phparch.com/magazine/2009/may/
.. _`php|architect`: http://www.phparch.com/magazine

Part one is here__.

__ /collecting-garbage-phps-take-on-variables.html

-----

In this second part of the three part column on the new garbage collecting
mechanism in PHP 5.3, we'll dive into a solution to the problem with circular
references. If we look quickly back, we found that by using code like the
following, an in-request memory leak is created:

.. image:: images/gc-part2-figure1.png

Traditionally, reference counting memory mechanisms such as PHP uses, fail to
address those circular reference memory leaks. Back in 2007 while looking into
this issue, somebody pointed me to a paper by David F. Bacon and V.T. Rajan
titled `"Concurrent Cycle Collection in Reference Counted Systems"`__ .
Although the paper was written with Java in mind, I started to play
around with it to see if it was feasible to implement the synchronous
algorithm as outlined in the paper in PHP. At that
moment I didn't have a lot of time, but along came the `Google Summer of
Code`__ and we put up the implementation of this paper as one of our ideas.
Yiduo (David) Wang picked up this idea and started hacking on the first
version as part of the Summer of Code project.

__ https://pages.cs.wisc.edu/~cymen/misc/interests/Bacon01Concurrent.pdf
__ http://code.google.com/soc/2007/

Explaining how the full algorithm works goes slightly too far for this column,
but I will try to explain the basics. First of all we have to establish a few
ground rules.  If a refcount is increased, it's still in use and therefore not
garbage. If the refcount is decreased and hits zero, the zval can be freed.
This means that garbage cycles can only be created when a refcount argument is
decreased to a non-zero value. Secondly, in a garbage cycle it is possible to
discover which parts are garbage by checking whether it is possible to
decrease their refcount by one, and then check which of the zvals have a
refcount of zero. 

.. image:: images/gc-part2-figure2.png

To avoid having to call the checking of garbage cycles with every possible
decrease of a refcount the algorithm instead puts all possible roots (zvals) in the
"root buffer" (marking them "purple"). It also makes sure that each possible
garbage root ends up in the buffer only once. Only when the root buffer is
full, the collection mechanism starts for all the different zvals inside. This
diagram shows this in step A.

In step B runs the algorithm a depth-first search on all possible roots to
decrease the refcounts of each zval it finds by one, making sure not to
decrease a refcount on the same zval twice (marking them as "grey"). In step C
the algorithm runs again a depth-first search from each root node, to check
the refcount of each zval again. If it finds that the refcount is zero, the
zval is marked "white" (blue in the diagram). If it's larger than zero, it
reverts the decreasing of the refcount by one with a depth-first search from
that point on and they are marked "black" again. In the last step (D) the
algorithm walks over the root buffer removing the zval roots from there, and
in the mean while checks which zvals have been marked "white" in the previous
step. Every zval marked as "white" will be freed.

Now that you have a slight understanding of how the algorithm works, we will
look back on how this weaves in with PHP. By default, PHP's garbage collector
is turned on. There is however a php.ini setting that allows you to change
this: `zend.enable_gc`_.

.. _`zend.enable_gc`: http://php.net/zend.enable_gc

When the garbage collector is turned on, the cycle finding algorithm as
mentioned above is executed whenever the root buffer runs full. The root
buffer has a fixed size of 10.000 possible roots (although you can change this
easily by re-compiling PHP and changing a constant). When the garbage
collector is turned off, the cycle finding algorithm will never run. However,
possible roots will always be recorded in the root buffer, no matter whether
the garbage collection mechanism has been activated with this configuration
setting. If the root buffer becomes full with possible roots while the garbage
collection mechanism is turned off, further possible roots will simply not be
recorded. Those possible roots that are not recorded will never be analyzed by
the algorithm. If they were part of a circular reference cycle they would never be
cleaned up, and create a memory leak. The reason why possible roots are
recorded even if the mechanism has been disable is because it's faster to
record possible roots, then to have to check whether the mechanism is turned
on every time a possible root could be found. The garbage collection and
analysis mechanism itself however can take a considerable time.

Besides changing the configuration setting, it is also possible to turn the
garbage collecting mechanism on and off by either calling `gc_enable()`_ or
`gc_disable()`_. Calling those functions has the same effect as turning on or off
the mechanism with the configuration setting.  It is also possible to force
the collection of cycles even if the possible root buffer is no full yet. For
this you can use the `gc_collect_cycles()`_ function. This function will return
how many cycles were collected by the algorithm. 

.. _`gc_disable()`: http://php.net/gc_disable
.. _`gc_enable()`: http://php.net/gc_enable
.. _`gc_collect_cycles()`: http://php.net/gc_collect_cycles

The rationale behind the possible to turn the mechanism on and off, and to
initiate cycle collection yourself is that some parts of your application
could be sensitive for time. In those places you might not want the garbage
collection mechanism to kick in. Of course, by turning off the garbage
collection for certain parts of your application you might risk getting memory
leaks because some possible roots might not fit into the limited root buffer.
Therefore it is probably wise to call `gc_collect_cycles()`_ just before you call
`gc_disable()`_ to free up the memory that could be lost through possible roots
that are already recorded in the root buffer. This then leaves an empty buffer
so that there is more place for storing possible roots while the cycle
collecting mechanism is turned off.

In this installment we saw how the garbage collection mechanism works, and how
it is integrated into PHP. In the `next and last part`_ of the series, we will
look at performance considerations and benchmarks.

.. _`next and last part`: /collecting-garbage-performance-considerations.html
