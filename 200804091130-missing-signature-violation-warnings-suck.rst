Missing signature violation warnings suck
=========================================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20080409 1130 CEST
   :Tags: blog, cms, php, work

I've been working on some code, while developing on PHP 5.3. The code
resembles the following structure:

::

	<?php
	interface ezcSearchQuery
	{
	    public function limit( $limit, $offset = '' );
	}
	interface ezcSearchFindQuery extends ezcSearchQuery
	{
	}
	class ezcSearchFindQuerySolr implements ezcSearchFindQuery
	{
	    public function limit( $limit = 10, $offset = 0 )
	    {
	        $this->limit = $limit;
	        $this->offset = $offset;
	    }
	}
	?>

No problems at all while development, no warnings, no errors. Now when I
deployed this on a PHP 5.2 machine it bombed out, with the following *correct* message:

Fatal error: Declaration of ezcSearchFindQuerySolr::limit() must be
compatible with that of ezcSearchQuery::limit() in /tmp/index.php on
line 11

And this really sucks. I made a mistake in my code (wrongly implemented
interface) and I get no warning (not even E_STRICT)... and then deploy
it and it bails out on me. We can't have this. We need *warnings* (actually, it should be E_FATAL) for those cases in order to avoid
problems. I don't know how this check got removed, but it should be put
back in ASAP!



