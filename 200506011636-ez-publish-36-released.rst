eZ publish 3.6 released
=======================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20050601 1636 CEST
   :Tags: cms, php, work

We just released `eZ publish 3.6`_ .
In this release we have a couple of cool new features:

`Static Caching of content`_ : allows you to create static HTML code whenever you
publish a new document. This is then served directly through Apache (by
means of mod_rewrite) and PHP is bypassed. This gives a huge performance
increase.

`Support for transactions in databases`_ : Does not improve performance, but
makes sure your database never ends up in a broken state.

Real preview of content: So that you can see how a page is going to look
like before it's actually published.

`Improved template syntax`_ : A new more inituitive syntax has been added. Now
you can use if(), for() and while() just like you would in PHP code. Of
course, support for the old syntax has been kept.

`Improved support for internal links`_ : Now you can simply use
eznode://galleries/misc_flowers in links. These links are automatically
updated when you move this node to a new location.


.. _`eZ publish 3.6`: http://ez.no/community/news/ez_publish_3_6
.. _`Static Caching of content`: http://ez.no/ez36_staticcache
.. _`Support for transactions in databases`: http://ez.no/ez_publish/download/changelogs/ez_publish_3_6/new_features/database_transactions
.. _`Improved template syntax`: http://ez.no/community/developer/specs/improved_template_syntax
.. _`Improved support for internal links`: http://ez.no/ez_publish/download/changelogs/ez_publish_3_6/new_features/changes_in_xml_tags

