Importing OpenStreetMap data into MongoDB
=========================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2013-05-07 09:49 Europe/London
   :Tags: blog, mongodb, php, openstreetmap
   :Short: mongosm

In many recent MongoDB_ related presentations_ I have used OpenStreetMap_ data
as basis for most of my examples. I wrote a script that imports OpenStreetMap
nodes and ways into MongoDB with a specific and high performant schema. I
have written about this briefly before in `Indexing Freeform-Tagged Data`_, but
now MongoDB received numerous updates to geo indexes I find it warrants a new
article.

In MongoDB 2.2 and before, the index type that MongoDB used was the *2d* type,
which was basically a two dimensional flat earth coordinate system index—with
some spherical features built on top. MongoDB 2.4 has a new index type:
*2sphere*. Instead of just being able to index points described by x/y
coordinates (longitude/latitude) it now has support for indexing points, *line
strings* and *polygons* as defined by GeoJSON_. 

I have modified my import script to use those new types, and also added
support for very simple multi-polygons that OpenStreetMap records through its
*relation* tag. 

.. _MongoDB: http://mongodb.org
.. _presentations: http://derickrethans.nl/talks.html
.. _OpenStreetMap: 
.. _`Indexing Freeform-Tagged Data`: /indexing-free-tags.html
.. _GeoJSON: http://www.geojson.org/
