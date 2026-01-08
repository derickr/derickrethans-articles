E_RECOVERABLE_ERROR
===================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20050826 1608 CEST
   :Tags: cms, php, work

Following my `posting`_ and the
following discussion about "Type Hints throwing a
fatal-non-catchable error when a wrong object is passed", I wrote
up a patch that adds a new error level to PHP (6): E_RECOVERABLE_ERROR.
This error level can be used for cases where the engine decides that
something is wrong, but when it doesn't end up in an unstable state.
This means that we have yet another level, and they should be used IMO
for the following cases:

E_NOTICE: Just for little notices to inform the user that something
might have gone wrong.

E_WARNING: Something went wrong, probably resulting in unwanted behaviour
- should be fixed by the script's writer.

E_RECOVERABLE_ERROR: An error situation occurred, which is probably
dangerous for a script to continue, but does not leave the Engine itself
in an unstable state. If this one is not caught in a user defined error
handler, the application aborts.

E_ERROR: The engine is in an unstable state, we have to abort. Not
catchable by a user defined error handler.

For now the new error level E_RECOVERABLE_ERROR is only implemented for
type-hints, but lots of extensions are still using E_ERROR, when there
is no need for it. They need updating to make those errors
E_RECOVERABLE_ERROR, or preferably E_WARNING (or even
E_NOTICE).

Update: We renamed E_RECOVERABLE to E_RECOVERABLE_ERROR.

The patch is online for ``6.0`` (already committed) and ``5.1`` (not committed).


.. _`posting`: http://news.php.net/php.internals/17581
