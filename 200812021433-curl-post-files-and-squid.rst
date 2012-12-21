Curl POST files and Squid
=========================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20081202 1433 CET
   :Tags: blog, php, work

For `releases`_ we also package a `PEAR`_ package for each component.
We have a channel server at http://components.ez.no that can be
integrated to download each component separately, but with dependency
checking. As server back-end we use Chiara_PEAR_Server, which allows us
to upload each component's release with a web-form. Now that we're
having more and more available components uploading them one by one is
no fun anymore—even more because the Firefox developers thought to be `smart`_ and force you to use the dialog instead of pasting in the filename.

So I wrote a script not too long ago to upload all the .tgz PEAR
packages through `curl`_ . That was
working great for quite a few releases. Unfortunately, when I rolled our
latest `2008.2beta1`_ release this script refused to work. I investigated a bit and saw that
curl posted only the header of the request, and not the POST body. Being
annoyed by that I tried older curl versions to see if those were
working, but no luck. I even tried `PECL`_ 's `HTTP`_ package only in
order to find out that it actually uses curl in order to do requests.

So because all of that failed, I looked a tad more at the headers that
curl was posting, and found this " `Expect: 100-continue`_ " header, which can be used to test whether a web
server will accept a certain request based on headers. Turns out that `Squid`_ doesn't quite
support that and simply rejects the request. As we use Squid to
accelerate our site, we now have to create an SSH tunnel to the web
server so that we can run curl against localhost with a port forward to
the web server. Not fun, but it works.


.. _`releases`: http://ezcomponents.org
.. _`PEAR`: http://pear.php.net
.. _`smart`: http://codangaems.blogspot.com/2008/06/firefox-3s-file-upload-box.html
.. _`curl`: http://curl.haxx.se
.. _`2008.2beta1`: http://ezcomponents.org/resources/news/news-2008-12-01
.. _`PECL`: http://pecl.php.net
.. _`HTTP`: http://pecl.php.net/package/pecl_http
.. _`Expect: 100-continue`: http://www.apps.ietf.org/rfc/rfc2616.html#sec-8.2.3
.. _`Squid`: http://www.squid-cache.org/

