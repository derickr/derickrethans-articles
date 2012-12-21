Translating Twitter, part 2
===========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2011-05-31 09:04 Europe/London
   :Tags: blog, php, opensource, twitter
   :Short: bing-translate

A while ago I wrote in `an article`_ about translating tweets in my client
Haunt_. For the translating itself I was using the Google Translate API,
which has sadly be deprecated_. Evil after all I suppose.

.. _`an article`: /translating-twitter.html
.. _Haunt: /projects.html#haunt
.. _deprecated: http://googlecode.blogspot.com/2011/05/spring-cleaning-for-some-of-our-apis.html

I've now rewritten my translation code to use the `Bing Translation APIs`_
instead. You need to register an API key (see http://www.bing.com/developers/appids.aspx)
to be able to use the APIs. The APIs that I am using are fairly simple though.

.. _`Bing Translation APIs`: http://www.microsoft.com/web/post/using-the-free-bing-translation-apis

For a simple translation, requesting
http://api.microsofttranslator.com/V2/Http.svc/Translate?appId=[yourappid]&to=en&text=[yourtext]
is all you need to do. It will auto-detect the language for you as well.

However, it does not return the detected language, so I had to resort to using
two requests in order to reimplement the same functionality that I had before
with the Google APIs. I also found that it was easier to use the Http and not
the Ajax variant of the API. It requires using SimpleXML to get to the data,
but at least you do not have to fight with the BOM_ (Byte-order mark) and
quoting.

.. _BOM: http://en.wikipedia.org/wiki/Byte-order_mark

The full code looks like::

	<?php
	$apiBase = 'http://api.microsofttranslator.com/V2/Http.svc/';
	$appId = 'yourappid';
	$text = urlencode( 'Een hoge boom vangt veel wind' );

	$language = (string) simplexml_load_string(
		file_get_contents(
			"{$apiBase}/Detect?appId={$appId}&text={$text}"
		)
	);

	$inEnglish = (string) simplexml_load_string(
		file_get_contents(
			"{$apiBase}/Translate?appId={$appId}&text={$text}&to=en"
		)
	);

	var_dump( $language, $inEnglish );
	?>

with as output::

	string(2) "nl"
	string(34) "A high tree catches a lot of wind."
