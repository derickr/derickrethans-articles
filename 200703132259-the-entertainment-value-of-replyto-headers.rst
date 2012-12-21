The entertainment value of Reply-To headers
===========================================

.. articleMetaData::
   :Where: Above the Atlantic
   :Date: 20070313 2259 CET
   :Tags: blog, php, work

In the past two weeks I have been discussing the use of Reply-To headers
on mailing lists on several occasions. Some people are afraid that
people are not be able to use their MUA (mail client) in a proper way
and insist that mailing list software administrators configure that
lists in such a way that the software adds a Reply-To header so that
replies automatically end up on the list. They argue that this is
important in order to keep discussions on the mailing list. Those same
people also often argue that it is a good thing to do because it
prevents duplicates being received when somebody replies to a message
that was sent to the list. There are a few major flaws in this
argumentation which I would like to illustrate.

Mail clients have often two ways of replying to e-mail messages:
reply-to-sender and reply-all. In default cases (case 1), if a person
sends a mail to a mailing list all participants see a mail with a From
header from the user, and as To header the mailing list. In case
Reply-To headers are added (case 2) the participants see a From header
with the sender's email address, a Reply-To header containing the
mailing list's address and a To header containing the mailing list's
address.

If in case 1 a message is to-be-replied to the original sender, a user
only has to hit "reply" . In almost every MUA
"reply" defaults to "reply-to-sender". To reply to
the whole mailing list the user has to use "reply-all". In
that case the original sender *might* receive a duplicate mail
message (once as normal recipient through the To header, and once
through the mailing list). Luckily all modern mailing list managers
(like mailman) allow you to configure a feature that excepts users from
receiving a mail through the mailing list if there address is found in a
To or Cc header. This "filter-out-duplicates" feature
conveniently breaks down the argument of the duplicate received messages
that Reply-To-header-adding proponents bring up.

In case 2 the user just have to use "reply" to send his reply
to the whole mailing list, "reply-all" will do the same. This
clearly shows that using a reply-to header makes things easier for
users. There will also be no duplicate as the original sender's address
won't appear in the headers of the message that is replied. However,
some mail clients (like `pine`_ and mutt) do only have
"reply" (which defaults to "reply-to-all"). For
those clients there is now suddenly one extra question whether the reply
should use the address in the Reply-To header, or the address from the
original From header. This is somewhat of a nuisance that can be fixed
by a simple procmail filter in combination with the formail utility that
removes the Reply-To header:

::

	:0 f:formail4.dummy
	* ^List-Id: <phpdbabstraction
	| formail -I "Reply-To"
	:0:
	* ^List-Id: <phpdbabstraction
	mail/dbabstraction

There is a technical argument against munging headers, which is written
down in some RFC (which I can't find right now unfortunately). This RFC
states that headers should *only* be added to mail messages in case
it is of vital importance to the delivery and routing of the mail
message.

Adding a Reply-To header will overwrite an already existing Reply-To
header, meaning that that the sender might not actually the reply on the
address he/she wanted a reply to be send back to. Some people are
unfortunately misguided and don't care so much about this technical
concept.

However there is a sociological problem with Reply-To header munging as
well, which I would like to illustrate with some examples. Just like
other somewhat larger companies do `we`_ have an internal mailing list where we as employees can discuss issues
of various matter (both "useful" and "fun" things).
Subjects that can be found on this list are for example subjects that
concern everybody in the company, such as new marketing material, but
also invitations to film showings, "interesting" news items,
etcetera.

In some cases not everybody is fully agreeing with some of the ideas or
questions that are shared on this list, and this can result in somewhat
heated discussions­. During heated discussions people pay by nature
less attention on what they are sending with their mail client and
forget to reply to the original sender *only* (by editing the
headers in the reply message) and instead reply to the list with words
that are not thought out well enough to be shared with everybody on the
mailing list. A worse example that I encountered myself is a misdirected
mail calling some of the people on the list (a business partner in this
case) ass holes.

There is no doubt that this can result in quite a bit of damage for the
person that was supposed to send the reply only to the original sender.
For others this might be a form of entertainment because besides the
"personal attack" replies there are also examples of more
humorous nature where a reply message ends up on the whole mailing list.
For example a reply with a question if the intended person of the reply
will go out on a date, or a reply that almost suggest there is some
lesbian relation going on.

To prevent the kind of madness that gets people into problems, the
principle of least damage should be applied. There can never be a
problem if a mail is reply by accident only to a single person and not
the whole mailing list, because it is trivial to resend the message to
the whole mailing list. (It is still an art to do that properly though
as people forget about "threading" - more about that in a
later post). However, it is *always* a problem if a personal reply
ends up on the whole mailing list with text that should not have been
disclosed to all mailing list recipients. Therefore I would *urge* people to *not* to use Reply-To munging on their mailing
lists--even if this decreases the entertainment value :).

There is also some more information `here`_ . (This
resource contains more information about older mail clients and Usenet
usage, the arguments being made still apply for normal mailing lists as
well).


.. _`pine`: http://www.washington.edu/pine/
.. _`we`: http://ez.no
.. _`here`: http://www.unicom.com/pw/reply-to-harmful.html

