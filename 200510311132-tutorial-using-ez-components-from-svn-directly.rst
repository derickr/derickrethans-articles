Tutorial: Using eZ components from SVN directly
===============================================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20051031 1132 CET
   :Tags: php, work

1. Create a directory, and chdir to that directory

2. Checkout the Base package: svn co
http://svn.ez.no/svn/ezcomponents/packages/Base Base

3. Checkout the Mail package: svn co
http://svn.ez.no/svn/ezcomponents/packages/Mail Mail

4. Setup the environment with: Base/scripts/setup-env.sh This will
create the symlinks for autoload

5. Create a directory to put your scripts in. f.e.
"/tmp/test-app".

6. Set your include path to include the full absolute path of the
directory that you created in the first step in php.ini. F.e.:
include_path = '/tmp/test-components:/usr/local/bin/php:.'
Alternatively, you can do this at the top of your script with:

::

	set_include_path('/tmp/test-components:.');

7. In that directory, create a script with the following content, this
sets up the autoload environment that the eZ components require:

::

	set_include_path('/tmp/test-components:.');
	require_once 'Base/trunk/src/base.php';
	function __autoload($className)
	{
	    ezcBase::autoload($className);
	}

8. Start writing code, f.e.

::

	$transport = new ezcMailTransportSmtp( "10.0.2.35" );
	$mail = new ezcMail();
	$mail->from = new ezcMailAddress( 'null@ez.no', 'Test' );
	$mail->addTo( new ezcMailAddress( 'derick@tequila' ) );
	$mail->subject = "[Components test] SMTP test";
	$mail->body = new ezcMailTextPart( "Content" );
	$transport->send( $mail );

Ofcourse you need to change the SMTP server's IP (10.0.2.35) and
definitely the email adresses.

As you can see you don't have to use any require or include statements
for any of the ezc classes that you use, this is because of our
autoload mechanism which can locate the classes for you when you
instantiate or use them otherwise.



