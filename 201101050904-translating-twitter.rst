Translating Twitter
===================

.. articleMetaData::
   :Where: London, UK
   :Date: 2011-01-05 09:04 Europe/London
   :Tags: php, opensource

*Note: Google deprecated_ the API used in this article. See the `new article`_
that uses `Bing Translate`_ instead.*

*Note: I've made an update_ on January 6th to this article after a comment
by a reader.*

.. _deprecated: http://googlecode.blogspot.com/2011/05/spring-cleaning-for-some-of-our-apis.html
.. _`new article`: http://drck.me/bing-translate-8nq
.. _`Bing Translate`: http://www.microsofttranslator.com/dev/

As the author of Xdebug_ I am interested in finding out what people think of
it, and whether they have problems or compliments. I've set-up a twitter_ account
for Xdebug, `@xdebug`_, and my twitter client Haunt_ also shows me all tweets
with the `search term xdebug`_.

However, sometimes I get tweets in a language I can't read; for example
Brazilian Portuguese:

	Debugando aplicações PHP com Xdebug e Eclipse PDT: http://bit.ly/ffJC4G

	- junichi_y

or Japanese:

	@pomu0325 ありがとうございます！このXdebugの書き方と各場所を調べてたんですよ！こんなふうに書くんですね。

	- Ken

Once in a while, I would send these tweets through `Google's language
tools`_  but then my friend Elizabeth_ tweeted:

	Hey Lazyweb, is there a twitter client that lets me filter tweets by
	language?

	- Elizabeth Naramore

Instead of a manual copy and paste in into the language tools, I thought
it'd be nice to embed it directly into the client when it is requested.

Sadly, tweets don't have a language associated with them, so the first step
is to actually find out which language a tweet is in. Google provides a web
service called "`Language Detect`_". To use this service, you only have to
query a specific URL containing the text you want to guess the language off.
and parse the returned JSON_ structure. The Google website has an example__
which basically boils down to requesting the following URL:
`https://ajax.googleapis.com/ajax/services/language/detect?v=1.0&q=Hola,+mi+amigo`__

It returns the following JSON struct::

	{
		"responseData":
		{
			"language":"es",
			"isReliable":false,
			"confidence":0.08829542
		},
		"responseDetails": null,
		"responseStatus": 200
	}

If the ``responseStatus`` is 200, then it worked. ``responseData->language``
contains the found language, and
``responseData->isReliable``/``responseData/confidence`` describe how sure
Google is that the language found is actually correct. The larger the text,
the easier it is to find out of course. In this case, although the
confidence is low, the language is guessed correctly: ``es``, for Spanish.

Now we have the language, we can use another web service from Google to
translate the text from the guessed language to our target language which in
my case is English. This Translate_ service wants the text and a language
pair for translations. Google suggests_ you add a ``key``, and an
``userip``, but this is not strictly necessary. The language pair has the
format ``source-language-code|destination-language-code``; which is in our
case ``es|en``. The service is again very simple to use as you can see in
this example__. It boils again down to requesting an URL, such as:
`https://ajax.googleapis.com/ajax/services/language/translate?v=1.0&q=Hola, mi amigo!&langpair=es|en`__

It returns the following JSON struct::

	{
		"responseData": 
		{
			"translatedText":"Hello, my friend!"
		},
		"responseDetails": null,
		"responseStatus": 200
	}

If the ``responseStatus`` is 200, then it worked and
``responseData->translatedText`` contains the translated text.

A screenshot shows the translation feature of Haunt_ in
action:

.. image:: /images/content/haunt.gif

You can find the implementation here_. Haunt also has a `project page`_.

Update
------

A reader of this article, Jan, pointed out that you don't actually have to
do the translation in two steps. If you simply leave out the language before
the | when you pass it to ``language/translate``, then the web service will
automatically guess the original language. This URL demonstrates this:
`https://ajax.googleapis.com/ajax/services/language/translate?v=1.0&q=Hola, mi amigo!&langpair=|no`__

This translates the input text "Hola, mi amigo!" to Norwegian (language code
``no``). The returned JSON struct has an extra element in this case too::

	{
		"responseData": 
		{
			"translatedText":"Hei, min venn!",
			"detectedSourceLanguage":"es"
		},
		"responseDetails": null,
		"responseStatus": 200
	}

The ``responseData->detectedSourceLanguage`` element shows which language
Google thought the original text was in (``es`` in our case). It does not
however state its confidence level. I've also `updated Haunt`_.

.. _Xdebug: http://xdebug.org
.. _twitter: http://twitter.com
.. _`@xdebug`: http://twitter.com/xdebug
.. _Haunt: http://derickrethans.nl/projects.html#haunt
.. _`search term xdebug`: http://twitter.com/#!/search/xdebug
.. _`Google's language tools`: http://www.google.com/language_tools?hl=en
.. _Elizabeth: http://www.naramore.net/blog
.. _`Language Detect`: http://code.google.com/apis/language/translate/v1/using_rest_langdetect.html
.. _JSON: http://json.org/
__ http://code.google.com/apis/language/translate/v1/using_rest_langdetect.html#json_snippets_php
__ https://ajax.googleapis.com/ajax/services/language/detect?v=1.0&q=Hola,+mi+amigo
__ http://code.google.com/apis/language/translate/v1/using_rest_translate.html#json_snippets_php
.. _Translate: http://code.google.com/apis/language/translate/v1/using_rest_translate.html
.. _suggests: http://code.google.com/apis/language/translate/v1/using_rest_translate.html#prereqs
.. _here: http://svn.xdebug.org/cgi-bin/viewvc.cgi/twitter/trunk/twitter.php?annotate=34&root=openmoko#l209
.. _`project page`: http://derickrethans.nl/projects.html#haunt
.. _`updated Haunt`: http://svn.xdebug.org/cgi-bin/viewvc.cgi?view=revision&root=openmoko&revision=35 
__ https://ajax.googleapis.com/ajax/services/language/translate?v=1.0&q=Hola, mi amigo!&langpair=es|en
__ https://ajax.googleapis.com/ajax/services/language/translate?v=1.0&q=Hola, mi amigo!&langpair=|no
