No to a Uniform Variable Syntax 
===============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2014-07-16 16:48 Europe/London
   :Tags: blog, php
   :Short: nobc

As you might have heard, PHP developers voted on an RFC called `"Uniform
Variable Syntax"`_. This RFC "proposes the introduction of an internally
consistent and complete variable syntax". In general, this RFC argues for
making PHP's parser more complete for all sorts of variable dereferences.
For example::

    $foo()['bar']()
    [$obj1, $obj2][0]->prop
    $foo->bar()()
    $foo::$bar::$baz

Thirty people voted for, and one against: **Me**.

Does that mean that I am against a unified variable syntax? No, I am not.
I am actually quite a fan of having a consistent language, but we need to be
careful when this hits existing users.

The already accepted RFC also has some negative aspects, in the form of
backwards compatibility (BC) breaks. For example (quoted from the RFC)::

    // syntax               // old meaning            // new meaning
    $$foo['bar']['baz']     ${$foo['bar']['baz']}     ($$foo)['bar']['baz']
    $foo->$bar['baz']       $foo->{$bar['baz']}       ($foo->$bar)['baz']
    $foo->$bar['baz']()     $foo->{$bar['baz']}()     ($foo->$bar)['baz']()
    Foo::$bar['baz']()      Foo::{$bar['baz']}()      (Foo::$bar)['baz']()

This basically says that the RFC author **knows** there are BC breaks, but
choses to ignore how this might annoy users.

Unlike keyword additions, or
functions and/or settings being removed, this change in **semantics** is
probably one of the worst BC breaks you can imagine. You can't really write a
scanner for it, as the code could already have been converted. A *tiny*
change like this however, can create very hard to debug issues within
existing code. And this is exactly why people whine that PHP breaks BC and
does not care about its users. In many cases, breaking BC happens by accident,
and I'm no stranger to breaking BC due to some oversight. Accidents like this_
are certainly annoying, but slightly unavoidable as we do not have test cases
for *everything*.

However, when you know for certain that you are going to break BC, there is no
excuse. With such a marginal new "feature" as is outlined in this RFC, 
antagonising our users is not a good thing.

.. _this: https://bugs.php.net/bug.php?id=66985
.. _`Uniform Variable Syntax`: https://wiki.php.net/rfc/uniform_variable_syntax
