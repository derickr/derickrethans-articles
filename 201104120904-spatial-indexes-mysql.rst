Spatial Indexes: MySQL
======================

.. articleMetaData::
   :Where: London, UK
   :Date: 2011-04-12 09:04 Europe/London
   :Tags: blog, php, openstreetmap
   :Short: spat-mysql

In two previous articles I introduced `The spherical Earth model`_ and 
`importing data`_ into SQLite for querying geographical data. 
In this article we're going to have a look at importing the data into
MySQL_ and finding out how to best store and query spatial data in 
the databases.

.. _`The spherical Earth model`: http://drck.me/spat-dist-8kf
.. _`importing data`: http://drck.me/spat-osm-sqlite-8la
.. _MySQL: http://dev.mysql.com/
.. _PostgreSQL: http://www.postgresql.org/

**MySQL**

MySQL has some support for `Spatial Extensions`_, but it's not particularly
useful. For example, there is no way to query anything within the radius
around a specific point. Their community pages list a method_ of implementing
it, but it only calculates for a flat Earth model.

.. _`Spatial Extensions`: http://dev.mysql.com/doc/refman/5.1/en/spatial-extensions.html
.. _method: http://forge.mysql.com/tools/tool.php?id=41

Instead, we'll have to implement our own algorithms again. But first of all,
let us import the data into MySQL. First we create the MySQL database::

    derick@whisky:~$ mysqladmin -u root -p create poi
    mysql> CREATE TABLE poi(id int, type int, lat float, lon float, name char(255), address char(255), cuisine char(64), phone char(18));

And then we take the script_ from the previous article on
`importing data`_, and change the third line from::

    $d = ezcDbFactory::create( 'sqlite://' . dirname( __FILE__ ) . '/pois.sqlite' );

to::

    $d = ezcDbFactory::create( 'mysql://root:root@localhost/poi' );

Of course, substitute the username and password (``root``, ``root``) and the
database name (``poi``) to one that suits yourself. We then run again::

    php parseoi.php.txt great_britain_pubs.osm

And check that our import is complete::

    mysql> SELECT count(*) from poi\G
    *************************** 1. row ***************************
    count(*): 28147

Now that we have all the POIs imported, we can query the database. Because MySQL
doesn't have working spatial extensions, nor the register-function
capabilities from SQLite such as we saw in the previous article, we have to
come up with a new solution. The most obvious one is writing a stored
procedure for the task, another (non-explored option) would be to write a
`user defined function`_. Writing the stored procedure is simple enough; we
just have to convert the ``distance()`` function to SQL. On the MySQL command
line you can enter::

    delimiter //

    CREATE FUNCTION distance (latA double, lonA double, latB double, LonB double)
        RETURNS double DETERMINISTIC
    BEGIN
        SET @RlatA = radians(latA);
        SET @RlonA = radians(lonA); 
        SET @RlatB = radians(latB); 
        SET @RlonB = radians(LonB); 
        SET @deltaLat = @RlatA - @RlatB; 
        SET @deltaLon = @RlonA - @RlonB; 
        SET @d = SIN(@deltaLat/2) * SIN(@deltaLat/2) + 
            COS(@RlatA) * COS(@RlatB) * SIN(@deltaLon/2)*SIN(@deltaLon/2);
        RETURN 2 * ASIN(SQRT(@d)) * 6371.01;
    END//

Fetching all the pubs within a 250 meter radius around 51.5375°N, 0.1933°W is
than as easy as running::

    mysql> SELECT name, address, phone 
           FROM poi
           WHERE DISTANCE(lat, lon, 51.5375, -0.1933) < 0.25;

With the result being::

    | name            | address                        | phone           |
    +-----------------+--------------------------------+-----------------+
    | Mrs Betsy Smith | Kilburn High Road 77 , NW6 6HY | +44 20 76245793 |
    | The Cock Tavern | Kilburn High Road 125          | NULL            |
    | The Old Bell    | NULL                           | NULL            |
    | The Westbury    | Kilburn High Road 34 , NW6 5UA | +44 20 76257500 |
    +-----------------+--------------------------------+-----------------+
    4 rows in set (0.72 sec)

If we want to also include the distance from the centre point in the result,
we need to modify the query to::

    mysql> SELECT name, DISTANCE(lat, lon, 51.5375, -0.1933) AS dist
           FROM poi
           HAVING dist < 0.25
           ORDER BY dist;

with as result::

    | name            | dist                |
    +-----------------+---------------------+
    | Mrs Betsy Smith | 0.00825473345748987 |
    | The Cock Tavern |     0.2420193460511 |
    | The Old Bell    |   0.103123484090313 |
    | The Westbury    |   0.150294300836645 |
    +-----------------+---------------------+
    4 rows in set (0.72 sec)

In both cases, the query takes 0.72 seconds. This is not overly fast, and the
main reason for this is that the ``distance()`` function has to be called for
every row in the table. An index can not be created on this either. However,
what we can do is to filter out the results roughly first, by calculating a
bounding box of latitude/longitude pairs around the centre point. Calculating
the latitude boundaries can be done by::

    d = (distInKm / 6371.01 * 2π) * 360
    lat1 = centreLat - d
    lat2 = centreLat + d

In our example that becomes::

    d = (0.250 / 6371.01 * 2π) * 360 = 0.0022483
    lat1 = 51.5375 - 0.0022483 = 51.5352.. -> 51.5351
    lat2 = 51.5375 + 0.0022483 = 51.5397.. -> 51.5398

Or in PHP::

    <?php
    function findLatBoundary($dist, $lat, &$lat1, &$lat2)
    {
        $d = ($dist / 6371.01 * 2 * M_PI) * 360;
        $lat1 = $lat - $d;
        $lat2 = $lat + $d;
    }
    ?>

We round slightly up and down to combat inaccuracies in the calculations—it is
a rough estimate after all.  After adding the index on the ``lat`` column, and
the index on the ``lon`` column, we reissue the query::

    mysql> CREATE index poi_lat ON poi(lat);
    mysql> CREATE index poi_lon ON poi(lon);

    mysql> SELECT name, DISTANCE(lat, lon, 51.5375, -0.1933) AS dist
           FROM poi
           WHERE lat BETWEEN 51.5351 AND 51.5398
           HAVING dist < 0.25;

Which returns the same result as before, but faster::

    | name            | dist                |
    +-----------------+---------------------+
    | The Westbury    |   0.150294300836645 |
    | The Old Bell    |   0.103123484090313 |
    | Mrs Betsy Smith | 0.00825473345748987 |
    | The Cock Tavern |     0.2420193460511 |
    +-----------------+---------------------+
    4 rows in set (0.01 sec)

We can pre-filter the result set even more, by also limiting on the longitude
boundary. This involves a few more calculations than for the latitude
boundaries. The distance in degrees longitude that belongs to a distance in km
depends on the latitude of the location. So first we need to calculate the
latitude boundaries as we have done above, and with that information calculate
the longitude boundaries.

.. image:: /images/content/sphere-distance.png

In the first step (red), we calculate the northern and southern boundaries of
the circle. We then calculate the western and eastern boundaries for the
northern boundary (green), and southern boundary (blue).

In PHP this algorithm becomes::

    <?php
    function findLonBoundary($dist, $lat, $lon, $lat1, $lat2, &$lon1, &$lon2)
    {
        $d = $lat - $lat1;

        $d1 = $d / cos(deg2rad($lat1));
        $d2 = $d / cos(deg2rad($lat2));

        $lon1 = min($lon - $d1, $lon - $d2);
        $lon2 = max($lon + $d1, $lon + $d2);
    }
    ?>

If we use both functions we calculate as boundaries::

    <?php
    $dist = 0.25;
    $lat = 51.5375;
    $lon = -0.1933;

    findLatBoundary($dist, $lat, $lat1, $lat2);
    findLonBoundary($dist, $lat, $lon, $lat1, $lat2, $lon1, $lon2);

    echo "SELECT name, DISTANCE(lat, lon, $lat, $lon) AS dist
          FROM poi
          WHERE lat BETWEEN $lat1 AND $lat2
            AND lon BETWEEN $lon1 AND $lon2
          HAVING dist < $dist
          ORDER BY dist;\n";
    ?>

Which returns the query to execute::

	SELECT name, DISTANCE(lat, lon, 51.5375, -0.1933) AS dist
	FROM poi
	WHERE lat BETWEEN 51.535251699514 AND 51.539748300486
	  AND lon BETWEEN -0.19691479627963 AND -0.18968520372037
	  HAVING dist < 0.25
	  ORDER BY dist;

If we run this query, we get as result::

    | name            | dist                |
    +-----------------+---------------------+
    | Mrs Betsy Smith | 0.00825473345748987 |
    | The Old Bell    |   0.103123484090313 |
    | The Westbury    |   0.150294300836645 |
    | The Cock Tavern |     0.2420193460511 |
    +-----------------+---------------------+
    4 rows in set (0.00 sec)

As you can see, we are getting the result a lot faster now—0.00 sec vs 0.72
sec.  Please do note, that if we would just have used the boundaries without
using the ``HAVING dist < 0.25`` clause, we would have gotten one extra
result that is slightly too far away (0.281 km)::

	| name            | dist                |
	+-----------------+---------------------+
	| Mrs Betsy Smith | 0.00825473345748987 |
	| The Old Bell    |   0.103123484090313 |
	| The Westbury    |   0.150294300836645 |
	| The Cock Tavern |     0.2420193460511 |
	| Bar Hemia       |   0.281138534935208 |
	+-----------------+---------------------+

.. _script: /files/parsepoi.php.txt
.. _`user defined function`: http://dev.mysql.com/doc/refman/5.1/en/adding-udf.html

**Conclusion**

In this article we saw how we can use a MySQL stored procedure to find our pubs
and bars within a certain distance from a central location.  In the next part,
we will be looking on how to solve the same problem with PostgreSQL_.

