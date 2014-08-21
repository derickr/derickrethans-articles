On Backwards Compatibility and not Fucking Over Users
=====================================================

.. articleMetaData::
   :Where: London, UK
   :Date: 2014-08-21 16:31 Europe/London
   :Tags: blog, php
   :Short: onbc

This is a repost of an email_ I sent to PHP internals as a reply to:

	And since you're targetting[sic] the next major release, BC isn't an issue.

This sort of blanket statements that "Backwards Compatibility is not an
issue" with a new major version is extremely unwarranted. *Extreme care*
should be taken when deciding to break Backwards Compatibility. It
should not be "oh we have a major new version so we can break all the
things"™.

There are two main types of breaking Backwards Compatibility:

1. The obvious case where running things trough ``php -l`` instantly 
   tells you your code no longer works. Bugs like the two default cases,
   fall in this category. I have no problem with this, as it's very easy
   to spot the difference (In the case of allowing multiple "default"
   cases, it's a fricking bug fix too).

2. Subtle changes in how PHP behaves. There is large amount of those
   things currently under discussion. There is the nearly undetectable
   change of the "Uniform Variable Syntax", that I already `wrote
   about`_, the current discussion on `"Binary String Comparison"`_,
   and now changing the `behaviour`_ on ``<<`` and ``>>`` in a subtle
   way. These changes are *not okay*, because they are nearly
   impossible to test for. 
   
   Changes that are so difficult to detect, mean that our users need to
   re-audit and check their whole code base. It makes people not want to
   upgrade to a newer version as there would be more overhead than
   gains. Heck, even changing the ``$`` in front of variables to ``£``
   is a better change, as it's *immediately* apparent that stuff
   changed. And you can't get away with "But Symfony and ZendFramework
   don't use this" either, as there is so much code out there 
 
As I said, the first type isn't much of a problem, as it's easy to find
what causes such Backwards Compatibility break, but the second type is
what causes our users an enormous amount of frustration. Which then
results in a lot slower adoption rate—if they bother upgrading at all.
Computer Science "purity" reasons to "make things better" have little to
no meaning for PHP, as it's clearly not designed in the first place.

Can I please urge people to not take Backwards Compatibility issues so
lightly. Please think really careful when you suggest to break Backwards
Compatibility, it should only be considered if there is a real and
important reason to do so. Changing binary comparison is not one of
those, changing behaviour for everybody regarding ``<<`` and ``>>`` is
not one of those, and subtle changes to what syntax means is certainly
not one of them.

**Don't be Evil**

.. _ email: http://news.php.net/php.internals/76752
.. _`wrote about`: /uniform-variable-syntax.html
.. _`"Binary String Comparison`: https://wiki.php.net/rfc/binary_string_comparison
.. _behaviour: https://wiki.php.net/rfc/integer_semantics
