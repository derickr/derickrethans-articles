Indexing Freeform-Tagged Data
=============================

.. articleMetaData::
   :Where: Dieren, Netherlands
   :Date: 2012-06-26 09:11 Europe/Amsterdam
   :Tags: blog, php, mongodb, openstreetmap
   :Short: freetagidx

At last week's MongoDB UK I presented on `"Geolocation, Maps and MongoDB"`_.
During this talk I presented a few methods on how to store `OpenStreetMap`_'s
tags on geographical objects. OpenStreetMap's tags are free form and for
example for a road can look like::

	alt_name:    The Strand
	highway:     primary
	name:        Strand
	oneway:      yes
	postal_code: WC2

For this article, I am using a data extra from OpenStreetMap for just London.
There are about 2.2 million items in my database after importing taking up
413MB of storage (without indexes).

**Tags as a normal object**

The first method for storing into MongoDB basically uses a ``tags`` field
with each of the OpenStreetMap tag taking up a property in the array::

	"_id" : "w1572026939",
	"type" : NumberLong(2),
	"tags" : {
		"alt_name" : "The Strand",
		"highway" : "primary",
		"name" : "Strand",
		"oneway" : "yes",
		"postal_code" : "WC2"
	},
	"loc" : [ [ -0.125262, 51.5086046 ], …, [ -0.1270609, 51.5076398 ] ]

In order to make looking up tags and values faster, we will need to create some
indexes. We would like to look for streets by name::

	db.poi.ensureIndex( { 'tags.name': 1 } );
	db.poi.find( { 'tags.name': 'Strand' } );

And also find roads and paths::

	db.poi.ensureIndex( { 'tags.highway': 1 } );
	db.poi.find( { 'tags.highway' : { $exists: 1 } } );

As you can see, for every tag that we want to search on we would have to 
create an index. More indexes make inserts, updates and deletes slower and
MongoDB is also limited to 64 indexes per collection. In order to have an index
on **every**
tag we need to find another solution. Also note that the second query
doesn't even use an index when you check it with calling ``explain()``.

**Tags as key/value pairs**

The second method I introduced was the one where instead of storing the tags as
properties of the ``tags`` field, I stored objects containing a
tag/tagvalue pair each::

	"tags_indexed" : [
		{ "k" : "alt_name", "v" : "The Strand" },
		{ "k" : "highway", "v" : "primary" },
		{ "k" : "name", "v" : "Strand" },
		{ "k" : "oneway", "v" : "yes" },
		{ "k" : "postal_code", "v" : "WC2" }
	],

The index we then can create is::

	db.poi.ensureIndex( { 'tags_indexed.k' : 1, 'tags_indexed.v' : 1 } );

To answer the same questions as above we have to rewrite the queries. To
find all documents where the ``name`` is ``Strand`` we can not simply do::

	db.poi.find( { 'tags_indexed.k' : 'name', 'tags_indexed.v' : 'Strand' } );

This query will find all documents where **either** the key is ``name`` or the
value is ``Strand``. This includes documents where there is an array element
like ``{ "k" : "name", "v" : "Strand" }`` in the ``tags_indexed`` array, but
also every **other** document where there is one ``"k" : "name"`` and a
``"v" : "Strand"`` element, such as in::

	{
		"_id" : ObjectId("4fe4c4b044670a41222088b5"),
		"tags_indexed" : [
			{ "k" : "addr:housenumber", "v" : "79" },
			{ "k" : "addr:postcode", "v" : "WC2R 0DW" },
			{ "k" : "addr:street", "v" : "Strand" },
			{ "k" : "building", "v" : "yes" },
			{ "k" : "name", "v" : "Stamp Centre" }
		]
	}

In order to match only documents where there is **one** array element with both
the ``"k" : "name"`` and ``"v" : "Strand"`` properties, you need to use
`$elemMatch`_::

	db.poi.find( {
		'tags_indexed' : { $elemMatch: { 'k' : 'name', 'v' : 'Strand' } }
	} );

Let's look at the explain output of both variants. In the first case (with
``'name': 'Strand'``), the explain output looks like::

	> db.poi.find( { 'tags.name': 'Strand' } ).explain();
	{
		"cursor" : "BtreeCursor tags.name_1",
		"nscanned" : 12,
		"nscannedObjects" : 12,
		"n" : 12,
		"millis" : 1,
		"nYields" : 0,
		"nChunkSkips" : 0,
		"isMultiKey" : false,
		"indexOnly" : false,
		"indexBounds" : {
			"tags.name" : [ [ "Strand", "Strand" ] ]
		}
	}

And in the second case, the explain output looks like::

	> db.poi.find( {
	... 'tags_indexed' : { $elemMatch: { 'k' : 'name', 'v' : 'Strand' } }
	... } ).explain();
	{
		"cursor" : "BtreeCursor tags_indexed.k_1_tags_indexed.v_1",
		"nscanned" : 172145,
		"nscannedObjects" : 172145,
		"n" : 12,
		"millis" : 429,
		"nYields" : 0,
		"nChunkSkips" : 0,
		"isMultiKey" : true,
		"indexOnly" : false,
		"indexBounds" : {
			"tags_indexed.k" : [ [ "name", "name" ] ],
			"tags_indexed.v" : [ [ { "$minElement" : 1 }, { "$maxElement" : 1 } ] ]
		}
	}


.. image:: /images/content/mongodb-logo.png
   :align: right

In the first case, 12 documents are scanned and found. But in the second case,
the same 12 documents are returned, but 172145 documents are scanned.  This is
because at the moment, ``$elemMatch`` does not use the index fully and will
only use the first part of the compound index[1]_ — ``tags_indexed.k`` in our
case.

In order to make index usage better, it therefore makes more sense to have the
property with the highest cardinality_ to be the first key in the compound
index. In my data-set, the cardinality of ``tags_indexed.v`` is (a lot)
higher than ``tags_indexed.k``::

	> db.poi.distinct( 'tags_indexed.k' ).length;
	1281
	> db.poi.distinct( 'tags_indexed.v' ).length;
	151581

So let's change the keys in the compound index around, and run the explain for
the second query again::

	> db.poi.dropIndex( 'tags_indexed.k_1_tags_indexed.v_1' );
	> db.poi.ensureIndex( { 'tags_indexed.v' : 1, 'tags_indexed.k' : 1 } );

	> db.poi.find( {
	... 'tags_indexed' : { $elemMatch: { 'k' : 'name', 'v' : 'Strand' } }
	... } ).explain();
	{
		"cursor" : "BtreeCursor tags_indexed.v_1_tags_indexed.k_1",
		"nscanned" : 44,
		"nscannedObjects" : 44,
		"n" : 12,
		"millis" : 0,
		"nYields" : 0,
		"nChunkSkips" : 0,
		"isMultiKey" : true,
		"indexOnly" : false,
		"indexBounds" : {
			"tags_indexed.v" : [ [ "Strand", "Strand" ] ],
			"tags_indexed.k" : [ [ { "$minElement" : 1 }, { "$maxElement" : 1 } ] ]
		}
	}

Now with the higher cardinality on the first part of the compound key only
44 documents are scanned. And instead of 429 milliseconds it now took less
than 1 millisecond. Excellent news!

The second question from the first section ("find roads and paths") can be
rewritten as::

	db.poi.find( { 'tags_indexed.k': 'highway' } );

However, this query can not be serviced by our rewritten index
(``'tags_indexed.v' : 1, 'tags_indexed.k' : 1``) as MongoDB can only use
keys starting from the left side of a compound index for queries. We can now
either add another index that only encompasses the ``k`` part or perhaps find
a different solution.

**Tag keys and values combined**

There is one other alternative that I would like to explore, and that is
where I combine the key and value of each tag into one string, and store
those all in an array in the object, like::

	"tags_combined" : [
		"alt_name=The Strand",
		"highway=primary",
		"name=Strand",
		"oneway=yes",
		"postal_code=WC2"
	],

And then we index this as::

	db.poi.ensureIndex( { 'tags_combined': 1 } );

The first query we used earlier in this article ("all items where the name
is ``Strand``") now looks like::

	> db.poi.find( { 'tags_combined': "name=Strand" ).explain();
	{
		"cursor" : "BtreeCursor tags_combined_1",
		"nscanned" : 12,
		"nscannedObjects" : 12,
		"n" : 12,
		"millis" : 0,
		"nYields" : 0,
		"nChunkSkips" : 0,
		"isMultiKey" : true,
		"indexOnly" : false,
		"indexBounds" : {
			"tags_combined" : [ [ "name=Strand", "name=Strand" ] ]
		}
	}

As you can see the index is very good at resolving this case, as it only
scanned the 12 documents that the query also returned—just like where
we had a direct index on ``tags.name``.

The second query (all paths and roads) and its explain syntax are::

	> db.poi.find( { 'tags_combined': /^highway=/ } ).explain();
	{
		"cursor" : "BtreeCursor tags_combined_1 multi",
		"nscanned" : 210441,
		"nscannedObjects" : 210440,
		"n" : 210440,
		"millis" : 741,
		"nYields" : 0,
		"nChunkSkips" : 0,
		"isMultiKey" : true,
		"indexOnly" : false,
		"indexBounds" : {
			"tags_combined" : [
				[ "highway=", "highway>" ], [ /^highway=/, /^highway=/ ]
			]
		}
	}

Here we use a regular expression search (``/^highway=/``) anchored to the start
of the string (with the ``^``). This can use an index, and is almost as good
as the the case where ``tags_indexed.k`` was the first part of the compound
key, except that it is a bit slower (527 vs 741 milliseconds)::

	> db.poi.find( { 'tags_indexed.k': 'highway' } ).explain();
	{
		"cursor" : "BtreeCursor tags_indexed.k_1_tags_indexed.v_1",
		"nscanned" : 210440,
		"nscannedObjects" : 210440,
		"n" : 210440,
		"millis" : 527,
		"nYields" : 0,
		"nChunkSkips" : 0,
		"isMultiKey" : true,
		"indexOnly" : false,
		"indexBounds" : {
			"tags_indexed.k" : [ [ "highway", "highway" ] ],
			"tags_indexed.v" : [ [ { "$minElement" : 1 }, { "$maxElement" : 1 } ] ]
		}
	}

**Conclusion**

Right now, I think that having an indexed on ``tags_combined`` where each
tag is combined with key and value and an array element ``tags_combined`` is
the best solution. Its index scanning is better for the ``name=Strand`` case
than the other, and not significantly slower in the ``highway=*`` case. I
would still like to see how the speed differences progress when I add
more objects to the database.

.. _`OpenStreetMap`: http://openstreetmap.org
.. _elemMatch: http://www.mongodb.org/display/DOCS/Advanced+Queries#AdvancedQueries-%24elemMatch
.. _cardinality: http://en.wikipedia.org/wiki/Cardinality
.. _`"Geolocation, Maps and MongoDB"`: /talks/osm-mongouk12
.. [1]: https://jira.mongodb.org/browse/SERVER-4520
