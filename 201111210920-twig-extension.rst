Twig extension
==============

.. articleMetaData::
   :Where: London, UK
   :Date: 2011-11-21 09:20 Europe/London
   :Tags: blog, php, extensions
   :Short: twig-ext

A while ago, Fabien_ asked me to have a look at porting one of `Twig's`_
slowest methods, ``TwigTemplate::getAttribute()``, into a PHP extension. It is
a complex method that does a lot of different checks and look-ups. Fabien's
benchmarks showed that this method was responsible for quite a large amount of
time. On top of that, it didn't seem that it could be optimised any further as
PHP code itself. Seeing that it was very likely that this method would be a lot
faster when written in C, SensioLabs_ decided to sponser me for the development
of a port of this method into a PHP extension.

Over the next few months I've worked on re-implementing just this method as a
C-function in twig-ext_: ``twig_template_get_attributes()``. From initial
benchmarks it looks like this function as implemented in the extension
gives about a 15% speed increase while rendering templates. Twig's
compiled templates directly contain a call to this function (as opposed to
marshal it through the original ``TwigTemplate::getAttribute()`` method) for
additional performance. This probably means that you'll have to remove your
compiled templates cache while upgrading.

The extension at http://github.com/derickr/twig-ext has now been merged,
via http://github.com/derickr/Twig, into http://github.com/fabpot/Twig.
The plan is that this new improvement is going to be part of Twig 1.4.

If you are also interested in having some of your PHP code ported into a
PHP extension in C, please feel free to contact_ me.

.. _Fabien: http://github.com/fabpot
.. _`Twig's`: http://twig.sensiolabs.org/
.. _SensioLabs: http://sensiolabs.com/
.. _twig-ext: https://github.com/derickr/twig-ext
.. _contact: https://derickrethans.nl/who.html
