Random Bugs and Testing RCs
===========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2012-02-24 13:00 Europe/London
   :Tags: blog, php
   :Short: random-bug

At the `PHP UK Conference`_ Rasmus_ mentioned that he wants more people 
contributing to PHP. There are plenty of ways how you can do that. 

The first one is testing release candidates RCs of PHP releases. You can
do a very basic test by running "make test" after compiling PHP, but it's even
a lot more important to test your *own* code with the RCs. This often catches
more things as the PHP Development Team doesn't quite know how everybody uses
PHP. It's a little bit late in the game now for PHP 5.4, but head over to
http://qa.php.net/rc.php for a quick intro and a link to where to download the
RC8. PHP 5.4.0 is not released yet, so testing the RC is still very valuable.
Hurry up though, as this is most likely the last RC for 5.4.0! Also, you are
not allowed to complain about PHP 5.4 breaking stuff unless you've tested RCs
:-)

The second thing that Rasmus brought up is a new feature on http://bugs.php.net
It's modelled like the "random cute cat picture" features that sites like
http://cuteoverload.com use for giving you a random cat picture. Except that
we don't show cute cats, but a random PHP bug. If you have a few spare minutes
go hit http://bugs.php.net/random until there is either an unconfirmed bug
report that needs triage, or perhaps you can just fix it because you're a C
wizzard. This is hopefully a better time waster than random cute cats.

Oh, yeah, go test the PHP 5.4.0 RC! 🐱

.. _`PHP UK Conference`: http://phpconference.co.uk/
.. _Rasmus: http://twitter.com/rasmus

