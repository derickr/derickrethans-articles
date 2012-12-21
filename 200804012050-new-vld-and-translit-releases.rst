New VLD and translit releases
=============================

.. articleMetaData::
   :Where: Skien, Norway
   :Date: 20080401 2050 CEST
   :Tags: blog, cms, php

They're two little extension for very distinctive purposes. `VLD`_ is a tool for hard core PHP hackers that want
to figure out what is going on in PHP's engine. This would be an
extremely useful tool for students that want to work on the `Google Summer of Code project`_ to
implement and finish "Ilia's" `Optimizer`_ .

The `translit`_ extension focuses on
transliterating scripts into different representations. It contains many
filters for different tasks. For example the
"normalize_numbers" filter can convert
"１２３４５６７８９０" into "1234567890"
with the following script:

::

	<?php
	$input = "１２３４５６７８９０";
	var_dump(transliterate(
	    $input, 
	    array('normalize_numbers'), 
	    'utf-8', 'ascii'));
	?>

But many other filters exist, such as converting Chinese text
(大平矿难死者增至66人) to pinyin
(dapingkuangnansǐzhezengzhi66ren) or stripping out accents (á -> a
)and converting ligatures (© -> (c), æ -> ae) etc. The translit
extension is also used on this website to create "nice" urls
with the following code:

::

	$blurp['url'] = transliterate($blurp['title'],
	    array(
	        'cyrillic_transliterate', 'lowercase_latin',
	        'normalize_ligature', 'diacritical_remove',
	        'normalize_punctuation', 'remove_punctuation',
	        'spaces_to_underscore', 'compact_underscores'),
	    'utf8', 'us-ascii');

Both extensions are available through PECL and installable like:

::

	pecl install vld
	pecl install translit


.. _`VLD`: /vld.php
.. _`Google Summer of Code project`: /gsoc_2008_optimizer.php
.. _`Optimizer`: http://wiki.php.net/gsoc/2008#make_ilia_s_optimizer_production_ready
.. _`translit`: /translit.php

