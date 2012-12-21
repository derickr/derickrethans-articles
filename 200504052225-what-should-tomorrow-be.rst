What should 'tomorrow' be?
==========================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20050405 2225 CEST
   :Tags: php

As you might know, I'm reimplementing PHP's date/time routines. At the moment PHP doesn't handle this `well`_ I think. But what should PHP's strtotime() do with
"tomorrow". Always 24 hours which will make "2005-10-29 02:00am tomorrow" show "2005-10-30 01:00am" because
time changes back to non-DST that day... but when it should always exactly a day "2005-04-02 02:30am
tomorrow" does not exist although "2005-10-29 02:00am tomorrow" will then correctly show "2005-10-30
02:00am"... what is your opinion on this?


.. _`well`: http://bugs.php.net/32555

