Ignite London: Crowd Sourcing a Map of the World
================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2012-05-15 09:47 Europe/London
   :Tags: blog, openstreetmap, conference
   :Short: ignite-osm

Almost  two weeks ago, I gave a talk at `Ignite London`_ about OpenStreetMap_,
titled "Crowd Sourcing a Map of the World". Ignite's presentation style is
20 slides which automatically advance every 15 seconds. Having never done this_
before I actually wrote the whole talk out. The presentation that I gave
slightly diverges from this but I thought it'd still be good to reproduce here.
I did add some links to more information, and if you want to see the recording,
you can find it at the end of this post.

.. _`Ignite London`: http://ignitelondon.net
.. _OpenStreetMap: http://openstreetmap.org
.. _this: /talks.html

.. image:: /images/content/osm-01-project-openstreetmap.png

1. This talk is about a project, started here in the UK with as its major goal
to create a free map of the whole planet. From roads and motorways to
country-side footpaths, restaurants and of course pubs. This talk is about
OpenStreetMap, the free map of the world.

.. image:: /images/content/osm-02-ordnance-survey.png

2. There are of course already plenty of mapping solutions available. Maybe
one of the best maps can be acquired through Ordnance Survey. They can be
regarded as the national authority on this subject. It's however expensive to
get access to their maps, especially the very detailed maps from OS MasterMap.
Additionally, it's only for the UK.

.. image:: /images/content/osm-03-google-maps.png

3. Besides the commercial solutions, you might wonder why we simply can't do
with GoogleMaps? It's mostly freely available for use and also provides you
with satellite imagery and StreetView. They even allow you in some areas to
update the map through Google MapMaker.

.. image:: /images/content/osm-04-the-data.png

4. But one thing Google doesn't give you access to, is the data behind the
map. All you will ever see, is the rendered map tiles and perhaps some APIs
to lookup locations and points of interest. Even for data that you have added
yourself through MapMaker.

.. image:: /images/content/osm-05-free-and-open-data.png

5. Both aspects; the cost of commercial maps, as well as the access to
the data that is behind the map tiles is something that the
OpenStreetMap project addresses. But which steps have to be taken to obtain
this enormous amount of geographical data?

.. image:: /images/content/osm-06-survey.png

6. We start by getting our wellies and trusty GPS out. Maybe even some pen and
paper. We find a location that looks rather empty on the map and travel to that
area to see what's on the ground. This is step one: data gathering in the
field.

.. image:: /images/content/osm-07-urban.png

7. In urban areas such as London the roads have often already been mapped
and a GPS is not accurate enough to be useful. Then we just use pen and
paper to record points of interest, such as shops, landmarks, restaurants
and postboxes, my personal favourite.

.. image:: /images/content/osm-08-countryside.png

8. In the country side, donated aerial imagery makes it possible for us to
easily trace tracks and footpaths. However, you can't be sure whether the
imagery is up-to-date, and you can't always see where fences, streams and
local wild life create barriers.

.. image:: /images/content/osm-09-mapping-party.png

9. In both situations, surveys are best done in groups: at mapping parties. It
helps spread the workload and a larger area can be surveyed in one go. As an
additional benefit, it allows us to go the pub and discuss our mapping
adventures!

.. image:: /images/content/osm-10-recording.png

10. Doing a survey is important. We take photographs, video and notes with pen
and paper of everything that seems to be of interest. This leaves a record
that everything we map is actually existing and we can prove that nothing
has been copied from other copyrighted maps.

.. image:: /images/content/osm-11-database.png

11. After collecting the data, we enter it into the database. This includes
basic information such as street names, but we also record whether a café has
wheelchair access, or whether a pub has wifi. Updates to the map show up on
the site close to real time.

.. image:: /images/content/osm-12-tagging.png

12. Every map object has tags associated with it. Tags tell whether a line is
a road, or perhaps a fence. All the tags are free form so you can generally
add as much information about an object as you want. Sometimes however, this
gets slightly out of hand and people tag pandas in trees and eyes on postboxes.

.. image:: /images/content/osm-13-display.png

13. Once the data has been added to the map, we can make use of it. One
of the primary uses is obviously showing the data as map tiles. But
with all the extra data, we can generate maps that show all the information
you're interested in-and nothing more.

.. image:: /images/content/osm-14-visualisation.png

14. Clockwise, we have four different visualisations of the map data: we have a
cycling-specific_ style, a style that shows transport_ routes, a rendering with
`MapQuest's`_ style sheets and even a `water colours`_ inspired style.

.. _cycling-specific: http://www.openstreetmap.org/?lat=51.51647&lon=-0.16968&zoom=17&layers=C
.. _transport: http://www.openstreetmap.org/?lat=51.51647&lon=-0.16968&zoom=17&layers=T
.. _`MapQuest's`: http://www.openstreetmap.org/?lat=51.51647&lon=-0.16968&zoom=17&layers=Q
.. _`water colours`: http://maps.stamen.com/watercolor/#12/51.5168/-0.1515

.. image:: /images/content/osm-15-mapquest.png

15. Having mentioned MapQuest; they were one of the first companies to make
use of OpenStreetMap data. They provide, free of charge, map tiles with
their own rendering style as well as an instance of Nominatim_, OpenStreetMap's
geolocation sister project.

.. _Nominatim: http://wiki.openstreetmap.org/wiki/Nominatim

.. image:: /images/content/osm-16-switching.png

16. Lots of companies have already switched to OpenStreetMap.  The property
search site Nestoria_ recently switched from using GoogleMaps to OpenStreetMap.
Partly because of their costs, but also partly because "The maps are equal or
better". geocaching.com, TfL's countdown website and Apple also use
OpenStreetMap maps and data.

.. _nestoria: http://blog.nestoria.co.uk/why-and-how-weve-switched-away-from-google-ma
.. _countdown: http://countdown.tfl.gov.uk/#|searchTerm=brick%20lane|stopCode=54814
.. _apple: http://idealab.talkingpointsmemo.com/2012/05/how-openstreetmap-got-apple-to-give-it-due-credit.php

.. image:: /images/content/osm-17-creative-commons.png

17. Although OpenStreetMap provides a free and editable map of the world,
there are certain requirements for using the data as well. The most important
one is that you always need to attribute the OpenStreetMap project.

.. image:: /images/content/osm-18-switch2osm.png

18. In order to help people start using OpenStreetMap for their mapping needs,
the Switch2OSM site has been launched. This website provides background
information, case studies and technical information on how to use OpenStreetMap
data.

.. image:: /images/content/osm-19-we-need-you.png

19. Right now, OpenStreetMap has very good data coverage in the country, but
we are not nearly finished. A lot of work still has to be done, and we rely on
you to improve the data too, even if you add just a postbox.

.. image:: /images/content/osm-20-openstreetmap.png

20. In the last 5 minutes we have looked at what OpenStreetMap is, how the data
is gathered and how the data is added to the map. Further more, we had a look
at different use cases of the data. OpenStreetMap in the UK: Footpaths and pubs
a speciality!

And then I planned showing the `"Year of Edits"`_ video, but that sadly didn't
work out. I'm including it for good measure here though. (If you want it in HD,
follow the link_).

.. vimeo::
   :ID: 34404102
   :Width: 599
   :Height: 337
   :Title: OpenStreetMap: A Year of Edits (2011)

The video of the talk itself is at http://vimeo.com/41626116 and is embedded
here:

.. vimeo::
   :ID: 41626116
   :Width: 599
   :Height: 337
   :Title: Crowdsourcing a Map of the World

.. _`Year of Edits`: http://vimeo.com/derickr/osm-2011
.. _link: http://vimeo.com/derickr/osm-2011
