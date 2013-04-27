Introduction to Document Databases with MongoDB
===============================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2013-08-06 09:11 Europe/London
   :Tags: blog, mongodb, php
   :Short: docdb

This article originally appeared in Web & PHP issue XXX_.

By now most you will probably have heard of the term *NoSQL*. It's a vague
term that covers a lot of different types of database engines. The main
classes of *NoSQL* databases are *key/value stores*, *column databases*,
*graph databases* and *document databases*. Examples of a key/value stores
are memcache_ or Redis_, where data can only be stored and retrieved through a
specific key. Column databases, such as Cassandra_ and Hadoop_, are great
for processing large amounts of data. Graph databases such as Neo4j_ and
OrientDB_ model the *relations* between entities. `Apache CouchDB`_ and
MongoDB_ belong to the last category, Document databases. We will be
looking extensively at MongoDB in this article.

In a document database such as MongoDB the smallest unit is a *document*.  In
MongoDB, documents are stored in a *collection*, which in turn make up a
*database*. Document are analogous to rows in a SQL table, but there is one big
difference: not every document needs to have the same structure—each of them
*can* have different fields and that is a very useful feature in many
situations. Another feature of MongoDB is that fields in a document can
contain arrays and or sub-documents (sometimes called nested or embedded
documents).

**MongoDB's Strengths**

Supporting a **different set of fields for each document** in a collection is
one of MongoDB's features. It allows you to store similar data, but with
different properties in the same collection. A good example of this is storing
real (not MongoDB) documents in a way that is beneficial for a Content
Management System (CMS). The CMS might want to store articles, which have
certain properties (e.g. author, tags, and body), but also related books,
which have additional properties such as their ISBN number, but no *body*
field. An article may need to store the periodical's ISSN number in lieu of an
ISBN number. In a relational
database there are various ways to solve this. Most frequently it is either
solved by having a table per object "class" (article or book) or coming up
with a scheme that stores object's properties in linked tables (for example
through the `EAV pattern`_). In MongoDB you would simply store the article and
book with the fields they need::

	{
		_id: ObjectId("51156a1e056d6f966f268f81"),
		type: "Article",
		author: "Derick Rethans",
		title: "Introduction to Document Databases with MongoDB",
		date: ISODate("2013-04-24T16:26:31.911Z"),
		body: "This arti…"
	},
	{
		_id: ObjectId("51156a1e056d6f966f268f82"),
		type: "Book",
		author: "Derick Rethans",
		title: "php|architect's Guide to Date and Time Programming with PHP",
		isbn: "978-0-9738621-5-7"
	}

Even though the two documents represent different classes of objects, you can
still construct a query that looks for all the items by an author, or for all
the items with a specific title.

**Data Model**

Each document in a collection in MongoDB can look totally different, and how
you structure your documents is up to you. MongoDB doesn't enforce a schema,
but your application still should. Although MongoDB is generally very fast,
the way how you structure and index your documents and collections has a big
influence on the performance of your application. While designing your schema
you should focus more on how the data is inserted, updated and queried and
less on how the data is structured. If sometimes you need to denormalise your
data, then that is a totally normal thing to do, even though it might look
dirty at first.

**Interactions Between Collections**

MongoDB makes different choices regarding functionality and scaling than
relational databases. MongoDB is very easy to scale through replication and
sharding, but it misses out on features like joins and transactions because of
this. Operations in MongoDB are only atomic per single document, and only
operate on one collection. Not allowing operations between collections (joins)
sounds like a real issue, but with the **support of arrays and sub-documents**
this is actually in most cases not a problem. Let's have a look at the
following example:

Take an application where we store image (meta) data and tags that go with
those images. In a relational database you would store that in three different
tables:

**Images**

===== =========== =========== ======
*id*  filename    mimetype    size
===== =========== =========== ======
1     cow.jpg     image/jpg   9123
2     bunny.png   image/png   8192
===== =========== =========== ======

**Tags**

===== ==========
*id*  value
===== ==========
1     animal
2     cute
3     tasty
===== ==========

**ImageTags**

=========== ============
*image_id*  *tag_id*
=========== ============
1           1
1           3
2           1
2           2
=========== ============

And queries for both meta-data and the tags for the bunny (``id = 2``) are as
follows::

	SELECT *
		FROM Images
		WHERE id = 2

	SELECT value
		FROM ImageTags LEFT JOIN Tags ON (Tags.id = ImageTags.tag_id)
		WHERE ImageTags.image_id = 2

This is quite complex as you can see. There are three tables, and two queries
involved. In MongoDB, you might store the same data as:

**Images**

::

	{
		_id: 1,
		filename: 'cow.jpg',
		mimetype: 'image/jpg',
		size: 9123,
		tags: [ 'animal', 'tasty' ]
	},
	{
		_id: 2,
		filename: 'bunny.png',
		mimetype: 'image/png',
		size: 8192,
		tags: [ 'animal', 'cute' ]
	}

To provide the same results as with the two SQL queries above, you would run
in the *MongoDB shell*::

	db.Images.find( { _id: 2 } );

And on top of that, you have all the data right in one place ready for display.

Most examples for MongoDB will show your documents as *JSON* documents. This
is not how MongoDB stores it internally, but it is a good representation of
how MongoDB deals with documents. For use within PHP, you would convert
**both** objects and arrays to PHP arrays. The above can be translated to PHP
like so::

	$doc1 = array(
		'_id' => 1,
		'filename' => 'cow.jpg',
		'mimetype' => 'image/jpg',
		'size' => 9123,
		'tags' => array( 'animal', 'tasty' )
	},

Or if you use PHP 5.4 you can use the following::

	$doc1 = [
		'_id' => 1,
		'filename' => 'cow.jpg',
		'mimetype' => 'image/jpg',
		'size' => 9123,
		'tags' => [ 'animal', 'tasty' ]
	],

PHP 5.4's short array syntax can come in quite handy when dealing with MongoDB
documents with nested arrays and objects.

**Closing Words**

MongoDB is not a straight replacement for your relational database. Questions
such as *"How do I convert my relational database to MongoDB?"* make little
sense as such a different approach is required to write applications with
MongoDB. That doesn't mean that MongoDB is not a general purpose database—it
can replace a relational database in almost every situation. You just need to
approach it differently, and when you do so you should find working with MongoDB
a breeze.


.. _XXX: http://webandphp.com/April2013
.. _memcache: http://en.wikipedia.org/wiki/Memcache
.. _Redis: http://en.wikipedia.org/wiki/Redis
.. _`Apache CouchDB`: http://en.wikipedia.org/wiki/CouchDB
.. _Cassandra: http://en.wikipedia.org/wiki/Cassandra_%28database%29
.. _MongoDB: http://mongodb.org
.. _Hadoop: http://en.wikipedia.org/wiki/Hadoop
.. _Neo4j: http://en.wikipedia.org/wiki/Neo4J
.. _OrientDB: http://en.wikipedia.org/wiki/OrientDB
.. _`EAV pattern`: http://en.wikipedia.org/wiki/Entity%E2%80%93attribute%E2%80%93value_model
