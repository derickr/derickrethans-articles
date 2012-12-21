PHP Vikinger 2008 Wrap-up
=========================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20080623 1045 CEST
   :Tags: blog, cms, conference, php, work

PHP Vikinger is over again. With about 35 attendees, I would think it
was a great success. After opening the event, we figured out which
topics people were interested in. After voting for the topics, we came
up with a nice couple of topics. First we had a little discussion on QA
and Testing, which Thomas Nuninnger moderated. I include my (raw) notes
from this:

`Selenium`_ takes a long time
to run tests, so Thomas only uses it for front end only, whereas `phpunit`_ is used for as much as possible
backend code. Selenium apparently has some functionality for parallizing
( `selenium grid`_ ), but
there are some issues as well as you can not tell it to run specific
tests parallel. `WebDriver`_ does exactly
the same, except for running it in the same browser. But it's going away
from actually using a real browser... which sorta defeats the point of
Selenium. Thomas also mentions that sometimes a fixed value as recorded
by the IDE is not good... but you can "fix" that yourself in
the PHPUnit tests that Selenium can export. The IDE is getting better
BTW. `CruiseControl`_ , `phpUnderControl`_ , `phing`_ (phing is not gnu make) - a port
of `Ant`_ .

After this, we had a discussion about the deployment of web
applications, where we discussed some different approaches such as
"svn check-out", but also tools for doing so such as `Capistrano`_ .

Sebastian then explained a little bit about the `PHP Object Model`_ . We also tried to
figure out a strange profiling issue in one of Zoë Slattery's
applications with `Xdebug`_ , but we
could not manage to figure out what it was just yet. The last talk
before lunch was from Kore Nordmann about `CouchDb`_ .

After the pizza lunch provided by Klosterøya, we continued with
presentations on `Project Zero`_ , by Ant Philips of IBM; the new lexer in PHP by Scott
MacVicar and a talk by Sebastian Bergmann on the `eZ Components' Workflow`_ component. The last talk of the day was by Tobias Schlitt on `database abstraction with eZ Components`_ .

I will put the slides online at the `PHP Vikinger`_ website once I receive
all of them.


.. _`Selenium`: http://selenium.openqa.org/
.. _`phpunit`: http://phpunit.de
.. _`selenium grid`: http://selenium-grid.openqa.org/
.. _`WebDriver`: http://code.google.com/p/webdriver/
.. _`CruiseControl`: http://cruisecontrol.sourceforge.net/
.. _`phpUnderControl`: http://www.phpundercontrol.org/
.. _`phing`: http://phing.info/
.. _`Ant`: http://ant.apache.org/
.. _`Capistrano`: http://www.capify.org/
.. _`PHP Object Model`: http://php.net/class
.. _`Xdebug`: http://xdebug.org
.. _`CouchDb`: http://incubator.apache.org/couchdb/
.. _`Project Zero`: http://www.projectzero.org/
.. _`eZ Components' Workflow`: http://ezcomponents.org/s/Workflow
.. _`database abstraction with eZ Components`: http://ezcomponents.org/s/Database
.. _`PHP Vikinger`: http://phpvikinger.org

