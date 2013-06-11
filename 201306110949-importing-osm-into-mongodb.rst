Importing OpenStreetMap data into MongoDB
=========================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2013-06-11 09:49 Europe/London
   :Tags: blog, mongodb, php, openstreetmap
   :Short: mongosm

In many recent MongoDB_ related presentations_ I have used OpenStreetMap_ data
as basis for most of my examples. I wrote a script that imports OpenStreetMap
(OSM) nodes, ways and to a lesser extend relations into MongoDB with a
specific and optimal schema. I have written about this briefly before in
`Indexing Freeform-Tagged Data`_, but now MongoDB received numerous updates to
geospatial indexes I find it warrants a new article.

In MongoDB 2.2 and before, the index type that MongoDB used was the *2d* type,
which was basically a two dimensional flat earth coordinate system index—with
some spherical features built on top. MongoDB 2.4 has a new index type:
*2sphere*. Instead of just being able to index points described by x/y
coordinates (longitude/latitude) it now has support for indexing points, *line
strings* and *polygons* as defined by GeoJSON_. 

I have modified my import script to use those new types, and also added
support for very simple multi-polygons that OpenStreetMap records through its
*relation* tag. The script also creates an indexes on *{ l: '2dsphere' }* (the
GeoJson object), *{ ts: 1 }* (the tags), and *{ ty: 1 }* (the type).

The structure it converts a node_ from OpenStreetMap to looks like::

	{
		"_id" : "n26486695",
		"ty" : NumberLong(1),
		"l" : {
			"type" : "Point",
			"coordinates" : [
				-0.1580359,
				51.4500055
			]
		},
		"ts" : [
			"addr:housenumber=97",
			"addr:postcode=SW12 8NX",
			"addr:street=Nightingale Lane",
			"amenity=pub",
			"name=The Nightingale",
			"operator=Youngs",
			"source:name=photograph",
			"toilets=yes",
			"toilets:access=customers",
			"website=http://www.youngs.co.uk/pub-detail.asp?PubID=430"
		],
		"m" : {
			"v" : NumberLong(5),
			"cs" : NumberLong(11229430),
			"uid" : NumberLong(652021),
			"ts" : NumberLong(1333911628)
		}
	}

There are several sections that make up the document:

 - *_id*: Is a combination of *n* and the OSM node id.
 - *ty*: Is the type. For nodes, this is always *1*.
 - *l*: The point's location in GeoJson format. An OSM point is translated to a
   GeoJson *Point* feature with an array describing the longitude and latitude.
 - *ts*: Are the tags that describe the node. Each tag is stored as a
   concatenation of its *key* and its *value*. This creates both a smaller
   index and it still allows for exact tag/value matches as well as matching
   against specfic keys through a regular expression match. For example, we
   could find the above document with: 

   ``db.poiConcat.find( { ts: 'name=The Nightingale' } );``

   And the index would also be used when we look for all amenities:

   ``db.poiConcat.find( { ts: /^amenity=/ } );``

 - *m*: Contains meta information that describes the node. The following
   fields are currently present:

    - *v*: The object's version. This is the version number of the version that
      was found in the imported file. There is always just one version per
      object.
    - *cs*: The changeset ID in which this object was last updated.
    - *uid*: The user ID of the OpenStreetMap contributor who uploaded the
      latest version.
    - *ts*: The Unix timestamp of when this object was last updated.

OpenStreetMap ways are stored in two different types. `Unclosed ways`_ are stored
as GeoJSON_ LineStrings (think roads)::

	{
		"_id" : "w2423886",
		"ty" : NumberLong(2),
		"l" : {
			"type" : "LineString",
			"coordinates" : [
				[ -0.1044769, 51.508462 ],
				[ -0.1044093, 51.5106306 ],
				[ -0.1044139, 51.5107814 ],
				[ -0.104427, 51.5108453 ],
				[ -0.1044459, 51.5109208 ],
				[ -0.1045131, 51.5110686 ]
			]
		},
        …

And `closed ways`_ are stored as GeoJSON_ Polygons (think buildings and parks)::

    {
        "_id" : "w24257746",
        "ty" : NumberLong(2),
        "l" : {
            "type" : "Polygon",
            "coordinates" : [
                [
                    [ -0.0745133, 51.560977 ],
                    [ -0.0742252, 51.5609742 ],
                    [ -0.0742308, 51.5606721 ],
                    [ -0.0745217, 51.5606721 ],
                    [ -0.0745133, 51.560977 ]
                ]
            ]
        },
        "ts" : [
            "amenity=park",
            "leisure=park",
            "name=Kynaston Gardens"
        ],
        "m" : {
            "v" : NumberLong(1),
            "cs" : NumberLong(357805),
            "uid" : NumberLong(5139),
            "ts" : NumberLong(1210169336)
        }
    }

Both ways and areas (closed ways) will have a *ty* value of 2, as they both
come from a *way* primitive as stored in OpenStreetMap.

The script is available on GitHub as part of the 3angle_ repository. The
latest version is at
https://raw.github.com/derickr/3angle/master/import-data.php and it also
requires https://raw.github.com/derickr/3angle/master/classes.php for some
GeoJSON helper classes and
https://raw.github.com/derickr/3angle/master/config.php where you can set the
database name and collection name (in my case, *demo* and *poiConcat*).

*Map data © OpenStreetMap contributors (terms_).*

.. _MongoDB: http://mongodb.org
.. _presentations: /talks.html
.. _OpenStreetMap: http://openstreetmap.org
.. _`Indexing Freeform-Tagged Data`: /indexing-free-tags.html
.. _GeoJSON: http://www.geojson.org/
.. _3angle: https://github.com/derickr/3angle
.. _node: http://www.openstreetmap.org/browse/node/26486695
.. _`Unclosed ways`: http://www.openstreetmap.org/browse/way/2423886
.. _`closed ways`: http://www.openstreetmap.org/browse/way/24257746
.. _terms: http://www.openstreetmap.org/copyright
