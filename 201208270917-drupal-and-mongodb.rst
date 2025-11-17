MongoDB and Drupal (@ DrupalCon)
================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2012-08-27 09:17 Europe/London
   :Tags: php, mongodb
   :Short: drupalmongo

Last week I spent in Munich, Germany attending and speaking at DrupalCon_.
This was my first DrupalCon, and I have to say I was amazed with both the 
size (1800+ people) as well as the great community they have built.
My main reason for going was to see how Drupal_ and MongoDB_ work together, and
what we can do to make their integration better.

From my side, MongoDB related events were me presenting at the `Munich MongoDB
Usergroup`_ on Indexing_ on Tuesday evening, a birds of a feather session on
Wednesday, where I showed how Drupal 7 can be used with MongoDB at the moment
and my presentation_ "Introduction to MongoDB" where I outlined what makes
MongoDB such a good fit for Drupal. The presentation on the conference were
standing room only.

I have taken quite a few things back from this conference. First of all, it
seems clear that there is a lot of interest in the Drupal community to have
MongoDB working as a first-class citizen, especially for higher traffic sites.
Right now, there is a module_ that "just" integrates into only a few places in
Drupal. We also support the EntityFieldQuery_ module.

But, as most of Drupal 7 is using hard coded SQL queries, it is hard
(though possible) to run everything with MongoDB instead of MySQL. Due to the
inherent design issues with Drupal 7 it is not easy to use and this then
results that sites such as the WhiteHouse's petition website_ moving back
from MongoDB to MySQL. In their own words:

	*"The current release depends on MongoDB. When we first created the
	application, we wanted to make sure we had a highly scalable application
	and database to meet our anticipated performance needs under high loads. We
	have been running MongoDB in production for over a year, but we have
	decided that the performance benefits it provides are outweighed by the
	complexity of trying to extend Drupal features backed by MongoDB."*

Of course, we_ would like MongoDB to be used with Drupal and we are putting
effort into making this a lot better for Drupal 8. Besides more interaction
with the Drupal community such as sponsoring and attending DrupalCon, we also
financially support Károly Négyesi (or chx_ as most people know him) to make
sure that Drupal 8 will work a **lot** better with MongoDB. Nevertheless, there
is still a lot of work to be done there.

.. _DrupalCon: http://munich2012.drupal.org/
.. _Drupal: http://drupal.org
.. _MongoDB: http://mongodb.org
.. _`Munich MongoDB Usergroup`: http://www.meetup.com/Muenchen-MongoDB-User-Group/
.. _Indexing: http://derickrethans.nl/talks/mongo-index-munich-mug
.. _presentation:  http://derickrethans.nl/talks/mongo-drupalcon12
.. _module: http://drupal.org/project/mongodb
.. _EntityFieldQuery: http://drupal.org/project/efq_views
.. _we: http://10gen.com
.. _website: https://github.com/WhiteHouse/petition/blob/7.x-1.x/readme.md
.. _chx: http://drupal.org/user/9446
