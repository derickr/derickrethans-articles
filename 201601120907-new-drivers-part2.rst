New MongoDB Drivers for PHP and HHVM: Architecture
==================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2016-01-12 09:07 Europe/London
   :Tags: blog, php, mongodb
   :Short: newmongo2

We recently released a new version of the MongoDB driver for PHP. This
release is the result of nearly a year and a half work to re-engineer and
rewrite the MongoDB driver. In the `previous blog post`_, I covered 
the back story of the how and why we undertook this effort. In this new blog
post, I will talk about the architecture of the new driver.

.. _`previous blog post`: /new-drivers.html

Let's go over the goals that we came up with at the end of the previous post
(in a different order), and then explain what that means for the new
architecture.

Support for other PHP engines like HHVM
---------------------------------------

When we were starting the process of writing the drivers, we noticed a big
uptake in HHVM usage, and hence we thought about creating something that would
not only work for PHP, but also for HHVM.

PHP 5 has been around for ages, but we wanted to support a reasonably new
version only. Some of the older versions (such as PHP 5.3) added a significant
amount of work to maintain, and some features where not even available when
using the old Mongo driver with PHP 5.3, so we set a minimum version
requirement of PHP 5.4.

On the other side, HHVM_ is a new PHP engine, developed by Facebook as an
iteration on an earlier effort to *compile* PHP code to C++. HHVM contains a
JIT_ engine, and also supports a PHP language extension called Hack_.

HHVM is an engine that we did not have any experience with. But users were
asking about solutions on running HHVM with an official MongoDB driver. Its
extension API does not share anything with PHP's, and it's in C++ instead of
plain C.

.. _HHVM: http://hhvm.org
.. _JIT: https://en.wikipedia.org/wiki/Just-in-time_compilation
.. _Hack: http://hacklang.org

.. image:: images/elephpants-cake.jpg
   :align: right

And then, PHP 7 came along. It is very much different internally than PHP 5,
although you can't really see that from PHP userland. (Userland is PHP code
that people write, unlike internals, which are written in C). With PHP 5 and
PHP 7 being so different we were not quite sure whether they would qualify as
a different engines or not. (In the end, they didn't.)

Of course, to support three (two) engines means that we need to spend three
(two) times the amount of, and that is not great. Which brings us to the next
point.

The Driver Should be Bare Bones
-------------------------------

Because we did not want to do three times the amount of work, we decided that
the new driver should be "bare bones". This means that the new drivers would
only implement features that are **strictly** necessary to communicate with
MongoDB. That means no command helpers, and no fancy syntactic sugar. 

Concentrating on the basic features only would make it a lot faster to
maintain the drivers, and a lot less work to keep up to date.

Besides that, the old Mongo driver for PHP also implemented the MongoDB wire
protocol: the binary protocol that drivers use to communicate with the server.
Every other driver in the MongoDB ecosystem was reimplementing the exact same
thing. That is of course not very efficient.

No Reinvention of the Wheel
---------------------------

Luckily, when we were starting on the new PHP driver, a project to implement a
C driver for MongoDB, including handling BSON, our data format, was nearly
complete. This meant that we also did not have to implement (and maintain) the
wire protocol for all new drivers. Instead, we choose to use libbson_, a "new
shared library written in C for developers wanting to work with the BSON
serialization format", as well as mongo-c-driver_ (formerly ``libmongoc``), a
"client library written in C for MongoDB". We ended up having to use a few
"private" APIs in the C driver, as the C driver is mainly focussed on being a
driver, and not a client library. They are fixing that though.

.. _libbson: https://github.com/mongodb/libbson
.. _mongo-c-driver: https://github.com/mongodb/mongo-c-driver

Provide A Easy to Use API
-------------------------

A bare bones driver that implements the most basic things is great as a driver
author, but it does not necessarily suit our users well. They often prefer an
easy to use API to write their applications against. This is why in addition
to the drivers we are also developing a `PHP library`_ to write applications
with. 

.. _`PHP library`: https://packagist.org/packages/mongodb/mongodb

This PHP library is installable through composer, and implements a nicer and
easier to use API. In the future, this is where we well add user management,
GridFS_ support, and various other non-core extensions to the driver. Because
these additions are implemented in PHP, maintaining this library is a lot less
complex than maintaining them through C or C++. On top of that, there is no
need to reimplement this for both engines (PHP and HHVM), and hence this
approach will save us time.

.. _GridFS: https://github.com/mongodb/specifications/blob/master/source/gridfs/gridfs-spec.rst

Backwards Compatibility
-----------------------

Neither the new MongoDB driver or the PHP library implement the APIs from the
old Mongo driver. This means that effort is required to port existing
applications that make use of the old Mongo driver to either the APIs of the
new MongoDB driver, or the APIs from the PHP library. We strongly recommend
to use the APIs of the PHP library though, as it would be much easier to work
with. However, it is possible to install the old Mongo and the new MongoDB
drivers alongside each other (from new MongoDB driver version 1.1.2), but you
can not share resources (connections, query results) between them. A migration
guide for converting applications from the old Mongo driver, to the new
MongoDB driver will follow shortly.

Overview
--------

When we tie all the bits and pieces together, the new architecture diagram
looks like:

.. image:: images/mongo-new-driver-architecture.png

As you can see, in the bottom we have the system libraries ``libbson`` and
``mongo-c-driver``, then on top of that the drivers for PHP 5 and 7, and HHVM,
and on top of that again the userland PHP Library. As part of this userland
library, we are also working on support for GridFS_, helping out on the
effort to make Doctrine ODM to work with the new driver, and supporting an
effort to write a userland library to imitate the old Mongo driver's APIs.

What's Next?
------------

In the next instalment of this series on the new MongoDB driver, we will
focus on a new feature in the new MongoDB driver: ODS - the Object Data Store.
