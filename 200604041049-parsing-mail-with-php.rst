Parsing Mail with PHP
=====================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20060404 1049 CEST
   :Tags: cms, php, work

Many PHP applications require to parse e-mail messages. For example bug
systems and ticket systems that want to allow input by e-mail. For
sending e-mail there are already decent implementations, `ones`_ that even allow sending multi-part and mixed text/html messages with
attachments and so on.

Parsing e-mail is a whole different story, and definitely not an easy
task. As `we`_ see this task as something
important we decided to add e-mail parsing functionality to the Mail `component`_ . We
just released an alpha release of this component as `PEAR package`_ which you should be able
to install with (not sure if you have to add the components channel
first, for information on how to do that see the "PEAR
Installer" section in `this`_ article.

::

	pear install components.ez.no/mail-beta

The Mail parsing part of the component can currently use a POP3 server
or a file to parse e-mail messages from. A small example to parse
e-mail from a POP 3 server looks like (if you use the PEAR installer to
install the component):

::

	<?php
	require_once "ezc/Base/base.php";
	function __autoload( $className )
	{
	    ezcBase::autoload( $className );
	}
	$pop3 = new ezcMailPop3Transport( "pop3.example.com" );
	$pop3->authenticate( "user", "password" );
	$set = $pop3->fetchAll();
	$parser = new ezcMailParser();
	$mails = $parser->parseMail( $set );
	foreach ( $mails as $mail )
	{
	    echo "From: {$mail->from->email}\n";
	    echo "To: ";
	    foreach ( $mail->to as $to ) {
	        echo "{$to->name} ({$to->email}) ";
	    }
	    echo "\n";
	    echo "Subject: {$mail->subject}\n";
	    switch ( get_class( $mail->body ) )
	    {
	        case 'ezcMailText':
	            echo "Text part, ".
	                "type={$mail->body->subType}\n--\n";
	            echo $mail->body->text;
	            echo "\n--\n";
	            break;
	        case 'ezcMailMultipartMixed':
	            echo "Multipart mail\n";
	            break;
	    }
	    echo "\n";
	}
	?>

The $mail variable now holds an array of `ezcMail`_ objects. For more information on how to access the information in the
mail classes, please refer to the `documentation`_ .
Currently not all the ezcMailPart decendents document the available
properties yet, but this will ofcourse be addressed before the first
beta.

In the near future we want to expand the component with an IMAP
transport, more authentication mechanisms and add methods that allow
you to "reply" to a parsed e-mail message or
"forward" one. Those methods then set the correct headers in
the e-mail object, including the correct handling of
"References" and "In-Reply-To" headers.


.. _`ones`: http://ez.no/doc/components/view/1.0.1/(file)/introduction_Mail.html
.. _`we`: http://ez.no
.. _`component`: http://ez.no/products/ez_components
.. _`PEAR package`: http://pear.php.net
.. _`this`: http://ez.no/community/articles/an_introduction_to_ez_components
.. _`ezcMail`: http://ez.no/doc/components/view/latest/(file)/Mail/ezcMail.html
.. _`documentation`: http://ez.no/doc/components/view/latest/(file)/classtrees_Mail.html

