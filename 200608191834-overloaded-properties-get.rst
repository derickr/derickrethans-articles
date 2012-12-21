Overloaded properties (__get)
=============================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20060819 1834 CEST
   :Tags: php, work

While checking whether the `eZ components`_ would run with the latest PHP 5.2 release candidate we
noticed that there are some things that are not backwards compatible
with PHP 5.1.

The first issue is an extra notice in some cases. In our ( `ezcMailTools`_ )
class we implement a method that allows you to "reply" to a
parsed e-mail message. In this method we have the folowing code:

::

	static function replyToMail( ezcMail $mail )
	{
	// ...
	    foreach ( $mail->to as $address )
	    {
	// ...
	    }
	// ...
	}

As you can see we loop over one of the seemingly public variables of the
$mail class. However, the `ezcMail`_ class does not have this as a public member variable, but instead uses `overload`_ as you can see in this code snippet:

::

	public function __get( $name )
	{
	  switch ( $name )
	  {
	    case 'to':
	      return $this->properties[$name];
	  }
	}

This all works 'fine' with PHP 5.1, however with PHP 5.2 the following
notice was generated for this code:

Notice: Indirect modification of overloaded property ezcMail::$to has no
effect in ../Mail/src/tools.php on line 364

The reason for this is that __get() only returns variables in read mode,
while foreach() wants a variable in read/write mode as it tries to
modify the internal array pointer. As it can't do this PHP 5.2 will now
throw a warning on this.

There is a workaround however in the form of casting it to an array.
After our `changes`_ the code now looks like:

::

	public function __get( $name )
	{
	  switch ( $name )
	  {
	    case 'to':
	      return (array) $this->properties[$name];
	  }
	}

The second issue is related to this. In this case we did not see an
extra notice but instead a fatal error:

Fatal error: Cannot assign by reference to overloaded object in
.../trunk/Url/src/url.php on line 242

The code around this line is:

::

	if ( array_key_exists( $index, $this->path ) )
	{
	    $this->path[$name] =& $this->path[$index];
	}

In this case $this->path is also an overloaded property returning an
array. Because __get() returns a read-only variable assigning a
reference can not work as that requires a variable to be in read/write
mode. Most likely the code didn't work properly in PHP 5.1 either but
luckily this is still in an unreleased component. However, I am not
aware of a work-around here.


.. _`eZ components`: http://components.ez.no
.. _`ezcMailTools`: http://ez.no/doc/components/view/latest/(file)/Mail/ezcMailTools.html
.. _`ezcMail`: http://ez.no/doc/components/view/latest/(file)/Mail/ezcMail.html
.. _`overload`: http://no.php.net/manual/en/language.oop5.overloading.php
.. _`changes`: http://lists.ez.no/pipermail/svn-components/2006-August/002539.html

