Patching Alpine
===============

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20070430 1637 CEST
   :Tags: php, work

Being a fervent `pine`_ user
since the early nineties I was always annoyed that Debian didn't have it
because of licensing issues (this time I agreed with the Debian folks
though). Pine has different ways of sorting indexes of mailboxes,
however, their "treaded sort" is done in a very annoying way
as it sorts threads in such a way that they appear in the order of their *first* message. This means that old threads with a new message do
not end up at the end of the index.

In addition to that it tries to be smart and "thread" even
mail messages depending on the subject. This is something that does not
work very well on CVS commit lists, as there are often commits to the
same file while they are totally not related. This in combination with
the "sort mailbox by first mail in thread first" problem is
quite annoying, and it makes me miss new messages.

Luckily there was a site " `Patches for Pine`_ which provided a patch " `Fancy Thread Interface`_ " that addresses this issue.

Pine has now been `discontinued`_ in favor of `alpine`_ -
now also `available`_ in
Debian as it is under the Apache 2 license. Unfortunately the old patch
for Pine did not apply anymore because of rigorous refactoring of the
code base. As I can not really do without nice threaded sorting I spend
the weekend porting the old patch to alpine. So far, it seems to run all
stable here and I am providing the patch `here`_ .

I will spend some more time on cleaning up the patch (white-space
related) and then submit it to the developers and hope they want to
include it in upcoming versions of alpine.


.. _`pine`: http://en.wikipedia.org/wiki/Pine_(e-mail_client)
.. _`Patches for Pine`: http://staff.washington.edu/chappa/pine/
.. _`Fancy Thread Interface`: http://staff.washington.edu/chappa/pine/info/fancy.html
.. _`discontinued`: http://mailman1.u.washington.edu/pipermail/pine-info/2006-November/055017.html
.. _`alpine`: http://www.washington.edu/alpine/
.. _`available`: http://packages.qa.debian.org/a/alpine.html
.. _`here`: http://files.derickrethans.nl/patches/alpine-fancy-thread-interface-2007-04-03.diff.txt

