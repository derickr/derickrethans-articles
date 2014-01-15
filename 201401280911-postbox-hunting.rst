Hunting for Postboxes (part 1)
==============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2014-01-28 09:11 Europe/London
   :Tags: blog, openstreetmap, maps, php, mongodb
   :Short: postbox1

.. image:: /images/content/postbox-pebble.png
   :align: left

The new year brings new hobby projects! In this year, I am going to try to
photograph as many UK postboxes as I can. For this I am going to build some
apps and overview maps, but I will write more about those later. A teaser for
a future post is here to the left ;-).

The first thing that I want my application(s) to do is to give a description
of where a postbox is located and this post is the tale of how I went about
just doing that.

As my data source, I am using an OpenStreetMap_ extract for the `London
region`_. I import that through my "Import OSM data into MongoDB" script that
I wrote about in an `earlier article`_. The script is available on GitHub as
part of the 3angle repository. The latest version is at
https://raw.github.com/derickr/3angle/master/import-data.php and it also
requires https://raw.github.com/derickr/3angle/master/classes.php for some
GeoJSON helper classes and
https://raw.github.com/derickr/3angle/master/config.php where you can set the
database name and collection name (in my case, ``demo`` and ``poiConcat``).

My goal is to get from a longitude/latitude pair to the closest postbox's
reference number (``EC2 201``), a description like (``On Paul Street, on the
corner with Epworth Street``), and a direction and distance (``East, 86m``).

The first thing I do is to look-up the closest postbox. I can very easily do
that with the following aggrgation query in MongoDB::

	<?php
	$m = new MongoClient( 'mongodb://localhost' );
	$d = $m->selectDb( 'demo' );
	$c = $d->selectCollection( 'poiConcat' );

	$center = new GeoJSONPoint( (float) $_GET['lon'], (float) $_GET['lat'] );

	$res = $c->aggregate( array(
		'$geoNear' => array(
			'near' => $center->getGeoJson(),
			'distanceField' => 'distance',
			'distanceMultiplier' => 1,
			'maxDistance' => 5000,
			'spherical' => true,
			'query' => array( TAGS => 'amenity=post_box' ),
			'limit' => 1,
		)
	) );

I am using the aggregation framework instead of a normal query because the
aggregation framework also can return the distance to the found object. The
above returns ``$res``, and our first result is located in
``$res['result'][0]``, which looks like::

	array(
		'_id' => 'n905143645',
		'l' => array(
			'type' => 'Point',
			'coordinates' => array( -0.1960, 51.5376 ),
		),
		'm' => array(
			'v' => 5,
			'cs' => 14576087,
			'uid' => 37137,
			'ts' => 1357659884,
		),
		'ts' => array(
			0 => 'amenity=post_box',
			1 => 'operator=Royal Mail',
			2 => 'ref=NW6 14',
		),
		'ty' => 1,
		'distance' => 162.15002059040299,
	),

So postbox ``NW6 14`` is ``162.15`` meters away from ``-0.1979, 51.5385`` at
``-0.1960, 51.5376``, as you can see in this image:

.. image:: /images/content/postbox1.png

To find a description of where the postbox is, we first find the closest
road::

	$r = $res['result'][0];
	$query = [
		LOC => [ 
			'$near' => $r['l'] 
		],
		TAGS => new	MongoRegex( 
			'/^highway=(trunk|pedestrian|service|primary|secondary|tertiary|residential|unclassified)/' 
		)
	];
	$road = $c->findOne( $query );

``highway`` is an OpenStreetMap_ tag that is also used for footpaths, service
roads and alleys. These generally don't have names, so with the regular
expression we restrict the query to only return "normal" roads. After
executing this query, ``$road`` now contains::

	array (
		'_id' => 'w4442243',
		'ty' => 2,
		'l' => array (
			'type' => 'LineString',
			'coordinates' => array (
				array (
					-0.2046823,
					51.5346008,
				),
				…
				array (
					-0.1940129,
					51.5384693,
				),
			),
		),
		'ts' => array (
			'hgv=destination',
			'highway=secondary',
			'lit=yes',
			'name=Brondesbury Road',
			'note=Signed as maxweight 7.5T for goods vehicles except for access, so have tagged as hgv=destination',
			'ref=B451',
			'sidewalk=both',
			'source:ref=OS OpenData StreetView',
		),
		'm' => array (
			'v' => 15,
			'cs' => 18802367,
			'uid' => 37137,
			'ts' => 1384017096,
		),
	)

As an image this looks like:

.. image:: /images/content/postbox2.png

We are interested only in the name (``name=Brondesbury Road``) and the
geometry (``l``). Right now, we can already assemble the description ``NW6
14, on Brondesbury Road``. But we also want to know the closest cross road,
which we can find by finding all roads that intersect with our geometry (in
``l``) by running the following query::

	$q = $c->find( [
		'l' => [
			'$geoIntersects' => [ '$geometry' => $road['l'] ]
		],
		'ts' => new MongoRegex(
			'/^highway=(trunk|pedestrian|service|primary|secondary|tertiary|residential|unclassified)/' 
		),
		'_id' => [ '$ne' => $road['_id'] ],
	] );

This returns nineteen roads. An extract looks like::

	array(19) {
		'w4211713' => array(5) {
			'_id' => string(8) "w4211713"
			'ty' => int(2)
			'l' => array(2) {
				'type' => string(10) "LineString"
				'coordinates' => array(22) { ... }
			}
			'ts' =>
			array(4) {
				[0] => string(19) "highway=residential"
				[1] => string(15) "maxspeed=20 mph"
				[2] => string(23) "name=Brondesbury Villas"
				[3] => string(13) "sidewalk=both"
			}
			…
		}
		'w245650577' =>
		array(5) {
			'_id' => string(10) "w245650577"
			'ty' => int(2)
			'l' => array(2) {
				'type' => string(10) "LineString"
				'coordinates' => array(5) { ... }
			}
			'ts' =>
			array(6) {
				[0] => string(15) "highway=primary"
				[1] => string(7) "lit=yes"
				[2] => string(15) "maxspeed=30 mph"
				[3] => string(22) "name=Kilburn High Road"
				[4] => string(6) "ref=A5"
				[5] => string(13) "sidewalk=both"
			}
			…
		}
	}

As an image this looks like:

.. image:: /images/content/postbox3.png

We are only interested in the roads that have a name and have a **different**
name than the road we have run the intersection query for. In some cases,
OpenStreetMap_ splits up a road in more than one segment carrying the same
name. We discard both those in a loop and are then left with an array of
intersecting road IDs in the ``$intersectingWays`` variable::

	$intersectingWays = array();
	foreach ( $q as $crossRoad )
	{
		$crossTags = Functions::split_tags( $crossRoad[TAGS] );
		if ( !in_array( "name={$roadName}", $crossRoad ) && array_key_exists( 'name', $crossTags ) )
		{
			$intersectingWays[] = $crossRoad['_id'];
		}
	}

With these IDs, we then search for the closest road(s) to the initially found
postbox location::

	$res = $c->aggregate( array(
		'$geoNear' => array(
			'near' => $r['l'],
			'distanceField' => 'distance',
			'distanceMultiplier' => 1,
			'maxDistance' => 5000,
			'spherical' => true,
			'query' => [
				'_id' => [ '$in' => $intersectingWays ], 
				'ts' => [ '$ne' => "name={$roadName}" ] 
			],
			'limit' => 1,
		)
	) );

Again, the result in ``$res`` is in a similar format as before, so I won't
repeat that. We use the aggrgation framework again so that we also get the
distance of this intersecting road to the originally found postbox location.
Depending on the distance to the intersecting road, we either use ``on the
corner of <roadname>`` (less thatn 25m) or ``near <roadname>`` if it's further
away than 25m. For our example postbox, that makes ``NW6 14, on Brondesbury
Road, near Algernon Road`` which is illustrated by this image:

.. image:: /images/content/postbox4.png

The full code for this example can be found at
https://github.com/derickr/3angle/tree/master/maps-postbox and you see it in
action (for London) at:
http://maps.derickrethans.nl/?l=postbox,lat=51.5&lon=-0.128&zoom=17

.. _OpenStreetMap: http://openstreetmap.org
.. _`London region`: http://metro.teczno.com/#london
.. _`earlier article`: /importing-osm-into-mongodb.html

