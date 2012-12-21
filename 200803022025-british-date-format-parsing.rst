British date format parsing
===========================

.. articleMetaData::
   :Where: London, UK
   :Date: 20080302 2025 GMT
   :Tags: blog, conference, php, travel, work, datetime

During the `PHP London`_ conference (slides are `here`_ as usual), a
number of British people were "complaining" that PHP's `strtotime()`_ and `date_create()`_ /new DateTime()
functions do not understand the British date format
"dd/mm/yyyy". Historically, the "English" text to
date parsers in PHP only accept the illogical American format of
"mm/dd/yyyy". Having seen this problem before, I recently
added a new function/method to PHP's Date/Time support to address
exactly this issue. From PHP 5.3 the new date_create_from_format()
function and the DateTime::createFromFormat() factory method are
available. As first argument they accept the expected format, and as
second argument the string to parse. To parse a British date string, you
would for example use:

::

	<?php
	$dt = date_create_from_format( 'd/m/Y', "02/03/2008" );
	echo $dt->format( 'd/m/Y' ), "\n";
	?>

In case the passed string does not match the format, the function will
return false. With the also new function date_get_last_errors() method
you then can request more information about which part of the string
could not be passed. The following example illustrates that:

::

	<?php
	$dt = date_create_from_format( 'Y-m-d', "02/03/2008" );
	if ( !$dt )
	{
	    $errors = date_get_last_errors();
	    var_dump( $errors['errors']);
	}
	?>

The $errors contains a list of errors, indexed by the position in the
string where the error was found. The above script would produce:

::

	array(3) {
	  [2]=>
	  string(22) "Unexpected data found."
	  [5]=>
	  string(22) "Unexpected data found."
	  [8]=>
	  string(13) "Trailing data"
	}

The format specifiers are mostly similar to the ones used for the `date()`_ function. Documentation on the
new functions should make it into the documentation soon.


.. _`PHP London`: http://phpconference.co.uk
.. _`here`: /talks.php
.. _`strtotime()`: http://php.net/strtotime
.. _`date_create()`: http://php.net/date_create
.. _`date()`: http://php.net/date

