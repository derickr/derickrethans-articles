Stranded and Date
=================

.. articleMetaData::
   :Where: Amsterdam, The Netherlands
   :Date: 20050311 1332 CET
   :Tags: work, php

Bah, plane is atleast an hour late. This allows me to talk a bit on the date stuff that I'm still working on
for PHP 5.1. What is done: Routines to parse string containing date/time information (a la the current
strtotime()); routines to interpret timezone data; routines to convert between local time and GMT time.
Except for timezone information everything works with 64bits, giving more than enough room for all times and
dates that you ever want to use in PHP. Things that are missing: wrappers around the code so that PHP can
interact with all the new routines.



