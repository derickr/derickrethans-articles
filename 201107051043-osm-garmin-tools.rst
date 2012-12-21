OpenStreetMap Quality Assurance with a Garmin GPS
=================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2011-07-05 10:43 Europe/London
   :Tags: blog, openstreetmap
   :Short: osm-qa


I've recently bought myself a new `Garmin eTrex Vista HCx`_ GPS to replace my
older Garmin eTrex Legend unit. Because I don't want to shell out money for
outdated maps, I have spend some time building my own map images for the
Garmin with mkgmap_ and OpenStreetMap_ data. I will write more on that
later.

.. _`Garmin eTrex Vista HCx`: http://wiki.openstreetmap.org/wiki/Garmin/eTrex_Vista_HCx
.. _mkgmap: http://wiki.openstreetmap.org/wiki/Mkgmap
.. _OpenStreetMap: http://wiki.openstreetmap.org/

.. image:: /images/content/garmin-osm.jpg
   :align: left

Because most of the data in OpenStreetMap has been surveyed by volunteers,
it is always possible that things are missing or done incorrectly. There are
quite a few different tools on-line for quality assurance of the
OpenStreetMap data, but I will be looking at two of them only.

**OSM Analysis**

First of all there is `ITO World`_'s `OSM Analysis`_. This tool compares the
OpenStreetMap data in the UK to the Ordnance Survey's Locator_ data. The
latter is supposed to be the authoritative source of road names in the UK.
For each of the 417 areas in the UK, the tool produces two lists; for
example the one for Brent is here:
http://www.itoworld.com/product/data/osm_analysis/area?name=Brent.

.. _`ITO World`: http://www.itoworld.com/
.. _`OSM Analysis`: http://www.itoworld.com/static/osm_analysis.html
.. _Locator: http://www.ordnancesurvey.co.uk/oswebsite/products/os-locator/

The first column lists all the corrections that Ordnance Survey ought to
make in an updated version because one of OpenStreetMap's surveyors found it
incorrect. The second column lists all the streets that are missing from
OpenStreetMap.

Because it would be really useful to have this sort of data available on a
hand-held GPS to be able to notice any of the highlighted missing roads on
the go, I set out to convert this data to a map file suitable for the Garmin
devices. I've published the scripts on github__ so that you can try it for
yourself, but I've also put a ready made ``osmanal.img`` file on-line for
download at http://derick.dev.openstreetmap.org/. In the near future I will
use cron to run this conversion about once a week automatically.

__ https://github.com/derickr/osm-tools/tree/master/osm-analyis-to-garmin

**Musical Chairs**

Another tool that compares OpenStreetMap data with Ordnance Survey's Locator
data is `OS Locator Musical Chairs`_. Documentation on this project is
available on the OpenStreetMap wiki__.

.. _`OS Locator Musical Chairs`: http://ris.dev.openstreetmap.org/oslmusicalchairs/map
__ http://wiki.openstreetmap.org/wiki/OS_Locator_Musical_Chairs/FAQ.

The author of the Musical Chairs tool has provided a `data dump`_ highlighting all the disagreements it
finds (including missing roads) while comparing OpenStreetMap data against
Ordnance Survey's Locator data.  I've written a tool that converts this data
to another Garmin map image files with the scripts that I have published at
github__. With those scripts you can make your own image files, but like
with the OSM Analysis I have made a ready-made file ``muschair.img``
available at http://derick.dev.openstreetmap.org/.

.. _`data dump`: http://wiki.openstreetmap.org/wiki/OS_Locator_Musical_Chairs/FAQ#Can_I_have_a_dump_of_the_data.3F
__ https://github.com/derickr/osm-tools/tree/master/musical-chairs-to-garmin

**Using the map image files**

Before you can use the two Garmin map image files, you either need to
convert it to a stand-alone ``gmapsupp.img`` file by running::

    java -jar /backup/osm/try4/bin/mkgmap-r1946/mkgmap.jar --gmapsupp osmanal.img

or::

    java -jar /backup/osm/try4/bin/mkgmap-r1946/mkgmap.jar --gmapsupp muschairs.img

These commands produce a ``gmapsupp.img`` file that you can upload to your
Garmin device. If you do this, you will **not** have any road data.

You can also merge this file with all your other maps to keep the road data,
by running something like::

    java -Xmx2048M -jar /backup/osm/try4/bin/mkgmap-r1946/mkgmap.jar \
        --gmapsupp /tmp/music.img /tmp/osmanal.img \
        UK/gmapsupp.img \
        contours/gmapsupp.img

Which merges the Musical Chairs image file (``music.img``), the OSM Analysis
image file (``osmanal.img``), the UK maps (``UK/gmapsupp.img``) as well as
my UK contours line file (``contours/gmapsupp.img``). This again produces a
``gmapsupp.img`` file that you can upload__ to your Garmin GPS.

__ http://wiki.openstreetmap.org/wiki/OSM_Map_On_Garmin#Installing_the_map_onto_your_GPS

For further ``mkgmap`` usage details I would like you to point to it's page
on the OpenStreetMap wiki: http://wiki.openstreetmap.org/wiki/Mkgmap or more
generally to http://wiki.openstreetmap.org/wiki/OSM_Map_On_Garmin. I am also
intending to publish my UK and contour map creation scripts after I've
streamlined this process.

And now the maps have been on my GPS for a few days, I've identified and
fixed_ a "disagreement_" from Musical Chairs and a few others_ from the ITO
World OSM analysis tool.

.. _fixed: http://www.openstreetmap.org/browse/changeset/8628284
.. _disagreement: http://ris.dev.openstreetmap.org/oslmusicalchairs/map?osl_id=890373
.. _others: http://www.openstreetmap.org/browse/changeset/8580300
