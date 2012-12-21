Embrace Inheritance
===================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20081018 1613 CEST
   :Tags: blog, php, work

In the last week I've been working on writing a tutorial for `our`_ MvcTools component. For this
tutorial I've also written a very simple `"HelloMvc" application`_ , not only to have something to write the tutorial
about, but also to test how the component would do in a more real life
situation, outside of the unit testing framework.

I realized during the development of this sample application, is that
while writing software, you can never really implement things so that it
does exactly what everybody wants and expects. During the development of
our `MvcTools`_ component we're run into a few of those occasions. One of those is the
naming conventions our default controller uses for action's methods. In
our `first implementation`_ we would always require the method naming of
"doActionName()".

Some of our users might want to do something else, but would now have to
rewrite the whole class. That is of course unnecessary, as Object
Oriented programming excels at reuse of code. However, in order for
inheritance to work, all the functionality need to be separated as much
as possible into their own methods.

In the re-factored `version`_ we now have split out the method name algorithm to its own method,
createActionMethodName(). This allows for easier customization of code
by inheriting from this class and just overriding this method. It also
makes testing easier, as now it is possible to test the name generating
algorithm. To illustrate this, we actually found an issue while writing
the tests for this new method.


.. _`our`: http://ezcomponents.org
.. _`"HelloMvc" application`: http://svn.ezcomponents.org/viewvc.cgi/docs/examples/applications/HelloMvc/
.. _`MvcTools`: http://ezcomponents.org/docs/tutorials/MvcTools
.. _`first implementation`: http://svn.ezcomponents.org/viewvc.cgi/trunk/MvcTools/src/interfaces/controller.php?revision=8970&view=markup
.. _`version`: http://svn.ezcomponents.org/viewvc.cgi/trunk/MvcTools/src/interfaces/controller.php?revision=9132&view=markup

