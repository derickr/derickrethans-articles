Crafty Code Coverage
====================

.. articleMetaData::
   :Where: London, UK
   :Date: 2019-12-09 13:30 Europe/London
   :Tags: xdebug
   :Short: ccc

Xdebug's code coverage functionality has had dead code analyses for years. It
is used to be able to mark lines of as having executable code on it, as well
as lines which can not be reached. In order to provide this functionality it
runs an algorithm code after each file has been compiled. For each class and
function it checks whether the algorithm to analyse executable lines and dead
code has already been run, as it makes no sense to check it for the same
class, method, or function twice.

Until PHP 7.4, Xdebug stored this information on whether it has seen a class
with a `special flag`_ on the class entry of a class, PHP's internal structure
that contains all the information the engine needs to be able to instantiate
objects and run methods.

.. _`special flag`: https://github.com/xdebug/xdebug/commit/9e76e8080423649bf1ad6d4837592d1336250e0b#diff-c23a49b975bde4591084a08c43ee6c45R292

PHP 7.4 changes the `values of these flags`_, which prompted me to ask Nikita_
whether it was actually safe to use a flag like this as a marker of whether
Xdebug has already analysed that class (and its methods). He said **no** and
suggested that instead I should use a hash table to store this information
instead. I implemented_ that for Xdebug 2.8, that was released just before PHP
7.4 came out.

.. _Nikita: https://nikic.github.io/aboutMe.html
.. _`values of these flags`: https://github.com/php/php-src/commit/0fbd2e6a168a5cfacec6c44f4c179879a52428f3
.. _implemented: https://github.com/xdebug/xdebug/commit/ddecc4143c419bb7ba9cef285cb732dbc6d49bdd#diff-c23a49b975bde4591084a08c43ee6c45R1050

Soon after PHP 7.4 came out, I received a `bug report`_ that code coverage was
now now significantly slower. A specific run went from 8 minutes to more than
**3.5 hours**. I tried this out for myself with the test suite of the
`Document package`_ of Zeta Components, and indeed, with PHP 7.3 and Xdebug
2.7.2 it took about 2.83 minutes, and with PHP 7.3 and Xdebug 2.8.0 **46**
minutes — a slow down of about 16 times.

.. _`bug report`: https://bugs.xdebug.org/view.php?id=1717
.. _`Document package`: https://packagist.org/packages/zetacomponents/document

I quickly discovered that I had forgotten a `!` in the code, which meant that
the analyses would run for every class after a new file was loaded/compiled. I
fixed_ that in Xdebug `2.8.1`_. This did improve the code coverage timings, but
it still took **22.26** minutes, instead of the original 8 minutes.

.. _fixed: https://github.com/xdebug/xdebug/commit/abd48292a0cfb8b8d60dfe39f23f594790d24c93
.. _`2.8.1`: https://xdebug.org/announcements/2019-12-02

I started talking to twitter user Anthony_ to try out a few more speed
improvements, and although we were making improvements, I was not getting
anywhere near the original timings of Xdebug 2.7.
At this point I referred back to Nikita to ask whether he had a good idea to
improve on this. He mentioned that classes and functions are always added to
the end of the class/function tables, and that they are never removed either.
He also hinted at a method to only loop over the newly added classes and
functions after each PHP file was compiled: loop over the table backwards up
to the point where the size of the list was the previous time that we looped.
This resulted in a patch_ that did exactly that. Xdebug now longer needs the
hash table mechanism to check whether we have analysed classes with their
methods, and functions already, and also no longer loops over the whole list
of classes after each file. As most people use one class per file, the
algorithm went from `O(n²)` to `O(n)` approximately.

.. _Anthony: https://twitter.com/ofcAnthony
.. _patch: https://github.com/xdebug/xdebug/commit/31aefbc4a793c5a680111aa6d0eea01d6dcf2144

This cut down the time to run the test suite with code coverage for the
`Document package`_ to **1.21** minutes. About 2½ times faster as Xdebug 2.7 and
earlier. Anthony was also pleased and surprised:

.. image:: /images/content/anthony-code-coverage.png

Although I tend to make an Xdebug release only once a month, in this case I
thought it warranted to expedite this. So here is your end-of-year present:
`Xdebug 2.9`_.

.. _`Xdebug 2.9`: https://xdebug.org/announcements/2019-12-09

🔔 📣 ⛄ ❄️ 📣 🎄 🎇 🎆 🎃 🎄 ✨ 🎉 🎊
