Spatial Indexes: Fetching Data/SQLite
=====================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2011-03-31 09:17 Europe/London
   :Tags: blog, php, openstreetmap
   :Short: spat-osm-sqlite

In a previous article_ I introduced 'The Flat Earth Model' and the 'The
spherical Earth model'. In this article we're going to have a look at fetching
a data set and importing them into a SQLite database to query from PHP. What
better data set is there to import than all of the UK's pubs?

.. _article: https://drck.me/spat-dist-8kf

**Getting the Data**

In order to get a suitable data set we are going to use the data from the
OpenStreetMap_ project. This project is mainly concerned with making an open
map of the entire globe, but it also contains a vast database of
points-of-interest (POI). OpenStreetMap contributes seems to be a big fan of
pubs, and hence they are mapped really well. POIs can either be stored as a
node (a single point with a geographical location) such as `The Long Acre`_ or 
as a closed way (an ordered collection of nodes where the first and last node
are the same), such as `Brondes Age`_.

.. _OpenStreetMap: http://openstreetmap.org
.. _`The Long Acre`: http://www.openstreetmap.org/browse/node/603112458
.. _`Brondes Age`: http://www.openstreetmap.org/browse/way/72773168

I will demonstrate two methods to fetch the pubs in an XML format containing
nodes and ways.  The first method is with XAPI_. XAPI is an interface to the
OpenStreetMap database to allow users to query and filter items. In order to
fetch data through it, you specify a bounding box and a predicate. A bounding
box specifies the most Eastern, Southern, Western and Northern coordinates.
The UK has roughly as bounding box *9.05°W, 48.77°N, 2.19°E, 58.88°N*, or
short: *-9.05, 48.77, 2.19, 58.88*. Fetching the data can simply be done by
querying the server with wget::

  wget -O pubs.xml 'http://xapi.openstreetmap.org/api/0.6/*[amenity=pub][bbox=-9.05,48.77,2.19,58.88]'

This is going to take a long time; and will most likely just not work or
time-out. The current XAPI server, written in an obscure programming language
called MUMPS_, is extremely unreliable and is really slow. A new version of
XAPI build in Java is on the way, but right now it's limited to 10 square
degrees.

.. _XAPI: http://wiki.openstreetmap.org/wiki/Xapi
.. _MUMPS: http://en.wikipedia.org/wiki/MUMPS

Luckily, there is an alternative in the form of parsing (an extract of) the
planet file. The planet file is an enormous database dump of OpenStreetMap's
data. The people at Geofabrik_ have extracts for specific parts of the world
at http://download.geofabrik.de/osm/. From them I downloaded the
``europe/great_britain.osm.pbf`` file (274 MB) for my example. With a tool
called Osmosis_ we can then filter out all pubs into a similar formatted XML
file. I am only giving the command to do our task at hand, but a detailed
usage guide for Osmosis is available_ too. The command runs in about four
minutes and looks like::

  ./osmosis-0.38/bin/osmosis -v 5 \
  \
  --read-pbf file=great_britain.osm.pbf \
  --tf reject-relations \
  --tf accept-nodes amenity=pub,bar \
  --tf reject-ways \
  outPipe.0=POI \
  \
  --read-pbf file=great_britain.osm.pbf \
  --tf reject-relations \
  --tf accept-ways amenity=pub,bar \
  --used-node \
  outPipe.0=area \
  \
  --merge inPipe.0=POI inPipe.1=area \
  --write-xml file=great_britain_pubs.osm

.. _Geofabrik: http://geofabrik.de
.. _Osmosis: http://wiki.openstreetmap.org/wiki/Osmosis
.. _available: http://wiki.openstreetmap.org/wiki/Osmosis/Detailed_Usage

**Importing the Data**

The resulting XML file has two important elements: nodes and ways. The XML for
The Long Acre looks like::

  <node id="603112458"
    version="1" timestamp="2010-01-02T16:12:57Z"
    uid="1185" user="dankarran" changeset="3519803" 
    lat="51.511831" lon="-0.126789">

    <tag k="amenity" v="pub"/>
    <tag k="name" v="The Long Acre"/>
  </node>

Important here are the latitude (51.51°N) and longitude (0.12°W) and of
course, the name of the pub in the *tag* element.

And the XML for Brondes Age (a way) is a bit more complex, and looks like::

  <way id="72773168"
    version="2" timestamp="2011-01-31T10:07:45Z"
    uid="346" user="Tom Chance" changeset="7143502">

    <nd ref="863936779"/>
    <nd ref="863936774"/>
    <nd ref="863936777"/>
    <nd ref="863936796"/>
    <nd ref="863936791"/>
    <nd ref="863936768"/>
    <nd ref="863936779"/>
    <tag k="addr:housenumber" v="328"/>
    <tag k="addr:postcode" v="NW6 2QN"/>
    <tag k="addr:street" v="Kilburn High Road"/>
    <tag k="amenity" v="pub"/>
    <tag k="building" v="yes"/>
    <tag k="contact:email" v="brondesage@aol.com"/>
    <tag k="contact:phone" v="+44 20 76249010"/>
    <tag k="contact:website" v="http://www.brondesage.com/"/>
    <tag k="name" v="Brondes Age"/>
    <tag k="toilets" v="yes"/>
    <tag k="toilets:access" v="customers"/>
  </way>

There is no latitude and longitude, but instead there are references to nodes
(the *nd* elements). There is also a large collection of descriptive tags,
such as *addr:housenumber* and *contact:phone*. In order to calculate the
latitude and longitude of Brondes Age we can either take the *correct*
approach by calculating the centroid_ or we can simply take the average
latitude and longitude of all the nodes. To make things simple, I will take
the simple approach.

.. _centroid: http://paulbourke.net/geometry/polyarea/

In order to find the latitude and longitude of all the nodes, we simply scan
through the file again and find all the nodes that correspond to the *ref*
attribute of each *nd* element. We should disregard the last one of each *way*
though, as it is the same as the first one.

**SQLite**

To start, we will use a very simple RDBMS: SQLite_. We will still have to
define a database schema. We can simply do that with::

  derick@whisky:~$ sqlite pois.sqlite
  sqlite> CREATE TABLE poi(id int, type int, lat float, lon float, name char, address char, cuisine char, phone char);

The importing of data then can be done by this "simple" script_ (after
adjusting the path to eZ Components/ `Zeta Components`_) with::

	php parsepoi.php.txt great_britain_pubs.osm

After running the import script, there should be about 28000 POIs in the
database.

.. _script: /files/parsepoi.php.txt
.. _`Zeta Components`: http://incubator.apache.org/zetacomponents/
.. _SQLite: http://www.sqlite.org/

**Querying the Data**

Once we have imported the POIs into our SQLite database, we are ready to query
them. SQLite does not have a very extensive set of functions_ so we can not
do the calculation in the query directly. Just to iterate from the previous_
article in this series, the formula for calculating the distance in a
spherical Earth model is::

  <?php
  function distance($latA, $lonA, $latB, $lonB)
  {
    // convert from degrees to radians
    $latA = deg2rad($latA); $lonA = deg2rad($lonA);
    $latB = deg2rad($latB); $lonB = deg2rad($lonB);

    // calculate absolute difference for latitude and longitude
    $dLat = ($latA - $latB);
    $dLon = ($lonA - $lonB);

    // do trigonometry magic
    $d =
      sin($dLat/2) * sin($dLat/2) + 
      cos($latA) * cos($latB) * sin($dLon/2) *sin($dLon/2); 
    $d = 2 * asin(sqrt($d));
    return $d * 6371;
  }
  ?>

.. _functions: http://www.sqlite.org/lang_corefunc.html
.. _previous: https://drck.me/spat-dist-8kf

One solution would be to query the database, and then use the ``distance()``
function to filter out unwanted elements, like::

  <?php
  include 'distance.php';
  require '/home/derick/dev/zetacomponents/trunk/Base/src/ezc_bootstrap.php';
  $d = ezcDbFactory::create( 'sqlite://' . dirname( __FILE__ ) . '/pois.sqlite' );

  // Centre point
  $lat = 51.5375;
  $lon = -0.1933;

  // Distance (in km)
  $wantedD = 0.25;

  $q = $d->createSelectQuery();
  $q->select( '*' )->from( 'poi' );
  $s = $q->prepare();
  $s->execute();

  foreach ( $s as $res )
  {
    $e = distance( $lat, $lon, $res['lat'], $res['lon'] );
    if ( $e < $wantedD )
    {
      echo sprintf( '%.3f,%.3f %-40s %.2f km away',
        $res['lat'], $res['lon'], $res['name'],
        $e ), "\n";
    }
  }
  ?>

This will show all pubs in a 250 meter radius around 51.53°N, 0.19°W::

  derick@whisky:/home/httpd/html/test/maps$ php fetch-sqlite-simple.php 
  51.538,-0.193 Mrs Betsy Smith                          0.01 km away
  51.539,-0.195 The Cock Tavern                          0.24 km away
  51.537,-0.192 The Old Bell                             0.10 km away
  51.537,-0.192 The Westbury                             0.15 km away

Of course, this is not very efficient as **all** items are selected from the
database, and then filtered out depending on their calculated distance.
Luckily, SQLite supports user defined functions written in PHP. This would
mean that we can increase performance a bit by letting PHP's internals call
the distance function for every row::

  <?php
  include 'distance.php';
  require '/home/derick/dev/zetacomponents/trunk/Base/src/ezc_bootstrap.php';
  $d = ezcDbFactory::create( 'sqlite://' . dirname( __FILE__ ) . '/pois.sqlite' );
  
  // Register SQLite function "dist" to our PHP function "distance".
  $d->sqliteCreateFunction( 'dist', 'distance' );
  
  // Centre point
  $lat = 51.5375;
  $lon = -0.1933;
  
  // Distance (in km)
  $wantedD = 0.25;
  
  $q = $d->createSelectQuery();

  // Use the user defined dist() function as additional column
  $q->select( "*, dist($lat, $lon, lat, lon) as e" )
    ->from( 'poi' )
    ->where( "e < $wantedD" );
  
  $s = $q->prepare();
  $s->execute();
  
  foreach ( $s as $res )
  {
    echo sprintf( '%.3f,%.3f %-40s %.2f km away',
      $res['lat'], $res['lon'], $res['name'],
      $res['e'] ), "\n";
  }
  ?>

The result is naturally the same as before.

**Conclusion**

In this installment we have seen how to retrieve information from
OpenStreetMap_'s database and import them into SQLite for querying. In the
next installment we will have a look at how to import and query with MySQL and
PostgreSQL.
