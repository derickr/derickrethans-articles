Using OpenStreetMap tiles with Flickr
=====================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2011-03-01 09:20 Europe/London
   :Tags: blog, php, photography, openstreetmap
   :Short: osm-flickr

**Update** August 4th, 2011: I've changed the script to use any of the three
OpenStreetMap_ tile servers, instead of hard coding it just to one.

I like taking pictures, and I usually take a GPS so that I can place them on a
map on my Flickr_ page. On my last excursion however, the battery of my GPS
had died, so I did not have location information available to store in my
pictures' EXIF_ headers. Flickr can use the EXIF headers to then show the
images on the map.

Because I did not have the location information to automatically place my
pictures on the map, I wanted to do that by hand. Flickr, being owned by
Yahoo!, uses `Yahoo! Maps`_. Yahoo! Maps however, is not terribly great in the
country side. Actually, it is mostly empty, especially compared_ to
OpenStreetMap's version. This made placing photos onto the map by hand quite
impossible. So I set out to have only OpenStreetMap_ tiles as back ground in
Flickr like they already do for Bejing_ and some other places_.

Inspired by the Upside-Down-Ternet_ I installed Squid_, and set the
``url_rewrite_program`` to a PHP script (with execute bit on)::

	url_rewrite_program /home/derick/bin/redirect.php

Squid will fire up a few of those scripts (depending on the
``url_rewrite_children`` setting) and then supplies them with URLs to rewrite.
Each line of input is an URL to rewrite, and the script should echo a
rewritten URL.

The script itself is rather simple. It just needs to account for two different
formats because the Yahoo!Maps URLs are different from the Flickr ones (as
indicated by the ``r`` argument in the original URL). I'm including the script
here (and as download_)::

    #!/usr/local/php/5.3dev/bin/php
    <?php
    do {
        // Open the log file
        $a = fopen( '/tmp/test.log', 'a' );

        // Read the URL line
        $input = fgets( STDIN );
        if ( $input == '' )
        {
            continue;
        }

        // Split the data so that we can access the host and
        // query string elements
        $parts = explode( ' ', $input );
        $urlParts = parse_url( $parts[0] );
        $queryParts = array();
        if ( isset( $urlParts['query'] ) )
        {
            parse_str( $urlParts['query'], $queryParts );
        }

        // The block to test for Yahoo! Maps:
        if ( preg_match( '@maps\d?.yimg.com@', $urlParts['host'] ) )
        {
            if ( !isset( $queryParts['r'] ) || $queryParts['r'] == 0 )
            {
                // This is the format that Flickr uses

                // Do the math to calculate the OSM tile
                // coordinates from the Yahoo!Maps one
                $z = 18 - $queryParts['z'];
                $x = $queryParts['x'];
                $y = pow(2, $z-1) - $queryParts['y'] - 1;

                // Assemble new URL and write log line
                $newUrl = "http://b.tile.openstreetmap.org/$z/$x/$y.png";
                fwrite( $a, "REDIR: $parts[0] => $newUrl\n" );
            }
            else
            {
                // This is the format that Yahoo!Maps uses

                // Do the math to calculate the OSM tile
                // coordinates from the Yahoo!Maps one
                $z = $queryParts['z'] - 1;
                $x = $queryParts['x'];
                $y = pow( 2, $z - 1 ) - $queryParts['y'] - 1;

                // Assemble new URL and write log line
                $range = range('a', 'c');
                $server = $range[rand(0, sizeof($range)-1)];
                $newUrl = "http://{$server}.tile.openstreetmap.org/$z/$x/$y.png";
                fwrite( $a, "REDIR: $parts[0] => $newUrl\n" );
            }
        } else {
            $newUrl = $parts[0];
            fwrite($a, "NORMAL: $newUrl\n");
        }

        // Output the rewritten (or original) URL
        echo $newUrl, "\n";
    } while ( true );

After I configured my browser to use the Squid proxy running on localhost,
Flickr is now shown with OpenStreetMap_ tiles as background:

.. image:: /images/content/flickrosm.png

And with the OpenStreetMap tiles in the background, I could place my photos on
the correct location on the map_.

.. _Flickr: http://www.flickr.com/photos/derickrethans/map
.. _EXIF: http://en.wikipedia.org/wiki/Exif
.. _`Yahoo! Maps`: http://maps.yahoo.co.uk/
.. _compared: http://sautter.com/map/?zoom=16&lat=53.05598&lon=-1.78709&layers=00000B0TFFFFFTFF
.. _Bejing : http://www.flickr.com/map?&fLat=39.9123&fLon=116.4179&zl=7
.. _places: http://en.wikipedia.org/wiki/OpenStreetMap#Flickr
.. _Upside-Down-Ternet: http://www.ex-parrot.com/pete/upside-down-ternet.html
.. _Squid: http://www.squid-cache.org/
.. _url_rewrite_program: http://www.squid-cache.org/Doc/config/url_rewrite_program/
.. _download: /files/redirectYahooMapsToOsm.php.txt
.. _OpenStreetMap: http://openstreetmap.org
.. _map: http://www.flickr.com/photos/36163802@N00/sets/72157625953902689/map?&fLat=53.0599&fLon=-1.7893&zl=4&order_by=recent
