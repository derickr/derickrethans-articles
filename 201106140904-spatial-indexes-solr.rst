Spatial Indexes: Solr
=====================

.. articleMetaData::
   :Where: London, UK
   :Date: 2011-06-14 09:04 Europe/London
   :Tags: blog, php, openstreetmap
   :Short: spat-solr

In two previous articles I introduced `the spherical Earth model`_, using
SQLite_ as a geographical data storage and using MySQL_ as a geographical data
storage.  In this article we're going to have a look at importing the data into
something else than a relational database: the search platform Solr_. (Yes, I
know I've skipped PostgreSQL, but I'll come back to that).

.. _`The spherical Earth model`: http://drck.me/spat-dist-8kf
.. _`SQLite`: http://drck.me/spat-osm-sqlite-8la
.. _`MySQL`: http://drck.me/spat-mysql-8ls

**Solr**

Solr_ is "the popular, blazing fast open source enterprise search platform from
the Apache Lucene project." Since version 3.1, Solr has support for `spatial
search`_, including geospatial search. It can store coordinates in two
different field types: ``solr.PointType`` for n-dimensional points, and
``solr.LatLonType`` for a two-dimensional point for geospatial search. The main
difference is that with ``solr.PointType``, calculations are done according to
the flat Earth model, and ``solr.LatLonType`` does calculations for a spherical
Earth model—which is just what we need.

In this example we will use the default Solr configuration, unless noted otherwise.
First we download Solr 3.1 and then untar it with ``tar -xvzf
apache-solr-3.1.0.tgz``.  Then we edit ``example/solr/conf/solrconfig.xml`` to
comment out the section on ``Query Elevation Component`` so that it looks
like::

	<!--
	<searchComponent name="elevator" class="solr.QueryElevationComponent" >
	  pick a fieldType to analyze queries
		<str name="queryFieldType">string</str>
		<str name="config-file">elevate.xml</str>
	  </searchComponent>
	-->

See here__ for the reason.

.. _Solr: http://lucene.apache.org/solr/
.. _`spatial search`: http://wiki.apache.org/solr/SpatialSearch
__ http://stackoverflow.com/questions/3631823/solr-queryelevationcomponent-requires-strfield-uniquekeyfield-error

**Setting Up Solr**

Before we can start using Solr, we need to define a schema. This schema is
quite analogous to a database schema, except that Solr only has one table.  By
default Solr comes with a schema defining lots of fields in
``example/solr/conf/schema.xml``.  I've changed the whole ``<fields>`` section
to look like::

	<fields>
		<field name="id" type="string" indexed="true" stored="true" />
		<field name="type" type="int" indexed="true" stored="true" />
		<field name="name" type="text" indexed="true" stored="true" />
		<field name="amenity" type="string" indexed="true" stored="true" />
		<field name="address" type="text" indexed="true" stored="true" />
		<field name="postcode" type="text" indexed="true" stored="true" />
		<field name="phone" type="text" indexed="true" stored="true" />
		<field name="cuisine" type="string" indexed="true" stored="true" />
		<field name="location" type="location" indexed="true" stored="true" />
		<field name="location_0_coordinate" type="double" indexed="true" stored="true" />
		<field name="location_1_coordinate" type="double" indexed="true" stored="true" />
	</fields>

I've defined specific fields for the ``type`` of source (1=node, 2=way/area),
the ``name`` of the amenity (name of the pub, hospital, etc), what sort of
``amenity`` it is (cafe, bank, etc), the ``address`` and ``postcode``, the ``phone`` number, the
type of ``cuisine`` as well as the ``location``. For the location I've also
defined two subtypes: ``location_0_coordinate`` and ``location_1_coordinate``.
Solr requires this for multi-dimensional types such as ``solr.LatLonType``.

The types for each field are also defined in ``schema.xml``. ``text`` is for a
string that is analysed and tokenized (broken down in smaller parts) so that
you can search in only parts of the text, ``string`` is for a string of
characters that is not analysed or tokenized, and ``location`` contains our
latitude/longitude point. Each of those types are associated with a specific
Java class with an entry such as::

	<fieldType name="location" class="solr.LatLonType" subFieldSuffix="_coordinate"/>

The ``subFieldSuffix`` in this line configures the suffix in the fields
``location_0_coordinate`` and ``location_1_coordinate``.

Now we have changed the default schema, we can start Solr with: ``cd example &&
java -jar start.jar``.

On the PHP side, we need to do a bit of set-up as well. I've been using the
`Solr PECL`_ extension to talk to Solr which can be installed with ``pecl
install solr``.  Please do not forget to add the extension to php.ini with
``extension=solr.so``. Check whether it is installed correctly with: ``php --ri
solr``.

.. _`Solr PECL`: http://pecl.php.net/package/solr



**Importing**

For our import, we need to change the script we used previously quite
substantially.  Instead of just using London, I've also downloaded the whole of
the UK from Geofabrik_ and extracted all amenities from the dataset with
Osmosis_::

	wget http://download.geofabrik.de/osm/europe/great_britain.osm.bz2

	./osmosis-0.39/bin/osmosis -v 5 \
	--read-xml file=great_britain.osm.bz2 \
	--tf reject-relations \
	--tf accept-nodes amenity=* \
	--tf reject-ways \
	outPipe.0=POI \
	\
	--read-xml file=great_britain.osm.bz2 \
	--tf reject-relations \
	--tf accept-ways amenity=* \
	--used-node outPipe.0=area \
	\
	--merge inPipe.0=POI inPipe.1=area \
	--write-xml file=great_britain_amenity.osm

This is going to take quite some time, and will result in a 500MB XML file.

.. _Geofabrik: http://geofabrik.de
.. _Osmosis: http://wiki.openstreetmap.org/wiki/Osmosis

The main logic of the importing script remains the same, but instead of adding
each item to a database, we now build a Solr import document and add it to the
search index.  I've also split the address and postcode into two separate
fields, and added the ``amenity`` field to store all the different amenities.
For now, I'm filtering out post boxes, parking areas and grave yards. The
script, ``parsepoi-solr.php`` is available here__.

__ /files/parsepoi-solr.php.txt

After downloading, and renaming the downloaded file to ``parsepoi-solr.php`` we
can run the script with::

	php -dmemory_limit=1G parsepoi-solr.php great_britain_amenity.osm

When done, this should have imported more than 140.000 items into Solr. You can
verify this, by going to http://localhost:8983/solr/select/?q=*:*&rows=0

**Querying**

As you can see, you can query Solr through its HTTP interface quite easily. In
fact, all queries to Solr are done over HTTP. The Solr extension however
abstracts this away from you for at least the import. I couldn't find any
functionality in the extension to do spatial queries, so we'll do it manually.

If we look at the URL above, we can divide it into different parts:

 - ``http://localhost:8983/solr/``: The base URL for this Solr instance.
 - ``select/?``: The action for running search queries.
 - ``q=*:*``: ``q`` is the search query parameter, and its value, ``*:*``
   means all fields (the first ``*``) and all possible values (the second
   ``*``). In this case that means return everything.
 - ``rows=0``: Do not return any rows, so that we just get a count.

We get back the result as an XML file. If we want JSON instead, we can simply
append ``&wt=json``. In case you want a CSV file, append ``&wt=csv``.

In a first example, all we want to do is to return all cafes within 100 meter.
We construct the search query as follows:

 - ``http://localhost:8983/solr/select/?``: The base URL with search action.
 - ``q=amenity:cafe``: We search in the field ``amenity`` and look for the value ``cafe``.
 - ``fq={!geofilt}``: We set a query filter to ``geofilt_``. This filter
   restricts the result set according to the location (``pt``) and the maximum
   distance from this location (``d``) in km.
 - ``sfield=location``: The field to use for location information.
 - ``pt=51.5375,-0.1934``: The point that we center our search around.
 - ``d=0.1``: The maximum distance in kilometers.
 - ``wt=csv``: The format to return, in our case, CSV.

.. _`As query filter`: http://wiki.apache.org/solr/CommonQueryParameters#fq
.. _geofilt: http://wiki.apache.org/solr/SpatialSearch#geofilt_-_The_distance_filter

Together this makes the full GET request:
http://localhost:8983/solr/select/?q=amenity:cafe&fq={!geofilt}&sfield=location&pt=51.5375,-0.1934&d=0.1&wt=csv

And the result is::

	id,phone,cuisine,location,address,location_1_coordinate,name,amenity,type,location_0_coordinate,postcode
	w62088838,,,51.53750834,-0.19329616,75 Kilburn High Road,-0.19329616,Costa,cafe,2,51.53750834,
	w78337118,,,51.53828298,-0.19410346,101 Kilburn High Road,-0.19410346,Caffè Nero,cafe,2,51.53828298,NW6 6JE
	w105467205,,,51.537555925,-0.192445525,274 Belsize Road,-0.192445525,Belsize Cafe,cafe,2,51.537555925,NW6 4BT
	w105467209,+44 207 3724002,,51.537624071429,-0.19228221428571,270 Belsize Road,-0.19228221428571,Lord Jim,cafe,2,51.537624071429,NW6 4BT
	w107710475,+44 20 76245736,,51.53695276,-0.19263662,2 Kilburn Bridge,-0.19263662,Famished Cafe,cafe,2,51.53695276,NW6 6HT
	w107710482,,,51.53717868,-0.19288912,Kilburn Bridge,-0.19288912,Mike's,cafe,2,51.53717868,NW6 6HT
	w107710490,+44 20 76246942,,51.53732946,-0.1930577,12 Kilburn Bridge,-0.1930577,La Dolce Vita,cafe,2,51.53732946,NW6 6HT

Which fields are returned can be configured. In this case, we are only
interested in the location and name of each amenity, so we restrict the number
of returned fields with: ``fl=id,location,name``. The result now becomes::

	id,location,name
	w62088838,51.53750834,-0.19329616,Costa
	w78337118,51.53828298,-0.19410346,Caffè Nero
	w105467205,51.537555925,-0.192445525,Belsize Cafe
	w105467209,51.537624071429,-0.19228221428571,Lord Jim
	w107710475,51.53695276,-0.19263662,Famished Cafe
	w107710482,51.53717868,-0.19288912,Mike's
	w107710490,51.53732946,-0.1930577,La Dolce Vita

Sadly, the results do not come back ordered by distance from our starting
point. In order to do that, we need to add one more query parameter:
``sort=geodist() asc``.  The full query is now:
http://localhost:8983/solr/select/?q=amenity:cafe&fq={!geofilt}&sfield=location&pt=51.5375,-0.1934&d=0.10&wt=csv&fl=id,location,name&sort=geodist()+asc

And the result::

	id,location,name
	w62088838,51.53750834,-0.19329616,Costa
	w107710490,51.53732946,-0.1930577,La Dolce Vita
	w107710482,51.53717868,-0.19288912,Mike's
	w105467205,51.537555925,-0.192445525,Belsize Cafe
	w105467209,51.537624071429,-0.19228221428571,Lord Jim
	w107710475,51.53695276,-0.19263662,Famished Cafe
	w78337118,51.53828298,-0.19410346,Caffè Nero

It is however not possible to return the distance from our starting point
directly with Solr. With the exception being that if you use only the
``geodist()`` function as query argument. In this case, make sure to include
the special field ``score`` in the ``fl=`` argument as well. An URL showing this is:
http://localhost:8983/solr/select/?q={!func}geodist()&fq={!geofilt}&sfield=location&pt=51.5375,-0.1934&d=0.10&wt=csv&fl=id,location,name,score&sort=geodist()+asc
Of course, in this case you can not filter for specific amenities.

**Conclusion**

In currently released versions of Solr (3.1 at the time of writing), it is not
possible to return the distance to the starting point for the search as part
of the result set. The Solr developers have already implemented_ the
capabilities to add the results of function queries to result documents for
version 4.0. You can verify this by downloading a `nightly build`_ and
running the query:
http://localhost:8983/solr/select/?q=amenity:cafe&fq={!geofilt}&sfield=location&pt=51.5375,-0.1934&d=0.10&fl=id,location,name,score,geodist()&sort=geodist()+asc
In this query I've added ``geodist()`` to the ``fl=`` parameter. The result now
includes an extra field called ``geodist()``::

	<?xml version="1.0" encoding="UTF-8"?>
	<response>
	  <!-- missing header -->
	  <result name="response" numFound="7" start="0" maxScore="4.352648">
		<doc>
		  <str name="id">w62088838</str>
		  <str name="name">Costa</str>
		  <str name="location">51.53750834,-0.19329616</str>
		  <float name="score">4.352648</float>
		  <double name="geodist()">0.0072415726934058865</double>
		</doc>
		<doc>
		  <str name="id">w107710490</str>
		  <str name="name">La Dolce Vita</str>
		  <str name="location">51.53732946,-0.1930577</str>
		  <float name="score">4.352648</float>
		  <double name="geodist()">0.030333097323281075</double>
		</doc>

I could not manage to alias it to a different field, or get it to work with the
CSV format (``wt=csv``).

If the return format is ``xml`` (the default) or ``json``, then the result
includes, besides the number of found items, the total search time for this
query.  Look for this information in the ``QTime`` field. In all of the
examples here, the ``QTime`` has been less than 5, meaning 5 milliseconds. Solr
is extremely fast, even with huge amounts of data. I will try to import the
amenities of the whole "planet" at some point, and report back with some
benchmarking information.

In the next installment of this series on storing geospatial data, I will be
looking at using MongoDB_ as data store for geographical information.

.. _`nightly build`: https://builds.apache.org//job/Solr-trunk/lastSuccessfulBuild/artifact/artifacts/
.. _implemented: https://issues.apache.org/jira/browse/SOLR-1298
.. _MongoDB: http://www.mongodb.org
