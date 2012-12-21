4th Major eZ Components release
===============================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20070702 1424 CEST
   :Tags: blog, cms, php, work

.. image:: /images/content/ezc-2007.1.png
   :align: center

Earlier today we, the `eZ Components`_ development team, put out our fourth major release.
This new release comes with two new components ( `Authentication`_ and `Workflow`_ ) as
well as more than `200 other updates`_ .

The Authentication component allows you to authenticate against many
different backends, such as database, LDAP, Open ID and TypeKey. The
core of the new Workflow engine is a virtual machine that executes
workflows represented through object graphs. These object graphs can be
created programmatically through the software component's Workflow
Definition API. Alternatively, a workflow definition can be loaded from
an XML file. Object graph and XML file are two different representations
of a workflow definition that uses the so-called backend language of the
workflow engine's core.

In the past two years that we've been developing the eZ Components we've
pioneered with a new development method - Test Driven Development - and
implemented more than 25 components that can be used as generic building
blocks in PHP applications. By providing extensive `documentation`_ and a
cross-component consistent API the eZ Components can serve as a solid
foundation for any PHP application out there, and will also form the
base of eZ Publish 4.0.

One of the most interesting components are the `Mail`_ component that parses
and constructs e-mail messages and supports `many different RFCs`_ . Describing all the different components that we've
developed goes to far now, but have a look at the `overview`_ on our web
site. Feel free to drop by in the `eZ Components channel`_ on IRC,
or ask questions and/or provide feedback through our `mailing lists`_ .


.. _`eZ Components`: http://ez.no/ezcomponents
.. _`Authentication`: http://components.ez.no/doc/Authentication
.. _`Workflow`: http://components.ez.no/doc/Workflow
.. _`200 other updates`: http://ez.no/ezcomponents/download/ez_components_2007_1_stable/2007_1/ez_components_2007_1/changelog
.. _`documentation`: http://components.ez.no/doc
.. _`Mail`: http://components.ez.no/doc/Mail
.. _`many different RFCs`: http://ez.no/doc/components/view/latest/(file)/Mail_rfcs.html
.. _`overview`: http://ez.no/ezcomponents/overview
.. _`eZ Components channel`: http://ez.no/ezcomponents/irc
.. _`mailing lists`: http://ez.no/ezcomponents/mailinglists

