Selecting Time Zones
====================

.. articleMetaData::
   :Where: London, UK
   :Date: 2025-09-24 18:35 Europe/London
   :Tags: blog, php
   :Short: select-tz

On the `phpc.chat <https://phpc.chat/>`_ Discord, a user of a website was
complaining that a site required them to select their home time zone from the
500 item long list of `IANA Time Zone Identifiers
<https://www.iana.org/time-zones>`_. You might have encountered these in the
form of ``Europe/London``, ``America/Los_Angeles``, or ``Asia/Gaza``.

But the fact is, these identifiers are not intended to be shown to users
directly. They indicate the largest populated city in the are that the time
zone covers, when the time zone identifier was established. It is therefore
not even the 'capital' city, and henceforth we have ``Asia/Shanghai``, and not
``Asia/Beijing``.

This is a useful feature, because sometimes it is necessary to split a zone in
two, and then there is no need to argue what the name of each of the two split
zones should be.

Although these identifiers are **stable**, they shouldn't be used in
front-ends to let users of websites choose what their zone is. And certainly
not in a single big list.

Instead, the Time Zone Database carries extra `information for zones
<https://github.com/eggert/tz/blob/main/zone.tab#L399>`_ to indicate to which
country/countries they belong, their centroid coordinates, and often a
comment. These comments are user-presentable and used to distinguish between
the various options in a specific country.

PHP exposes this functionality too.

For example, you can list all the identifiers belonging to a country with the
following code::

	<?php
	$tzids = DateTimeZone::listIdentifiers(DateTimeZone::PER_COUNTRY, 'PS');
	var_dump($tzids);
	?>

Which outputs::

	array(2) {
	  [0] => string(9) "Asia/Gaza"
	  [1] => string(11) "Asia/Hebron"
	}

So now we know that there are two time zones associated with Palestine.

Where there is more than one time zone per country, the TZDB also includes a
comment with each time zone. You can query these with PHP as well::

	<?php
	$tz = new DateTimeZone('Asia/Hebron');
	echo $tz->getLocation()['comments'], "\n";
	?>

Which outputs::

	West Bank

Each ``getLocation()``'s return value also includes the ``country_code``
element::

	<?php
	$tz = new DateTimeZone('Europe/Kyiv');
	echo $tz->getLocation()['country_code'], "\n";
	?>

Which outputs: ``UA``.

These features together make it possible to create a user-friendly time zone
selector. You can either first ask for the country, and then present all
time zones for this country, like I have shown above. For this you would use
``DateTimeZone::listIdentifiers(DateTimeZone::PER_COUNTRY, $country_code)``
and ``DateTimeZone->getLocation()['comments']``, or you can build up an array
of all zones organised by country code (and cache this information)::

	<?php
	$tzs = [];

	foreach (DateTimeZone::listIdentifiers() as $tzid) {
		$tz = new DateTimeZone($tzid);
		$location = $tz->getLocation();

		$tzs[$location['country_code']][$tzid] =
			$location['comments'] ?: 'Whole Region';
	}
	
	ksort($tzs);
	var_export($tzs);
	?>

This outputs a very big list, an extract looks like::

	…
	'GB' =>
	array (
	  'Europe/London' => 'Whole Region',
	),
	…
	'PS' =>
	array (
	  'Asia/Gaza' => 'Gaza Strip',
	  'Asia/Hebron' => 'West Bank',
	),
	…
	
Of course, you **also** shouldn't show country codes to users, and translate
``Whole Region`` into your user's preferred language.

The Unicode Consortium's `Unicode Common Data Repository
<https://cldr.unicode.org/>`_ project can help you with expressing the
country code as a name. I have used `icanboogie/cldr
<https://packagist.org/packages/icanboogie/cldr>`_ for this before.

With::

	composer require icanboogie/common:^6.0-dev icanboogie/accessor:^6.0-dev icanboogie/cldr:^6.0-dev``

The full example would look like::

	<?php
	require 'vendor/autoload.php';

	use ICanBoogie\CLDR\Repository;
	use ICanBoogie\CLDR\Cache\CacheCollection;
	use ICanBoogie\CLDR\Cache\RuntimeCache;
	use ICanBoogie\CLDR\Provider\CachedProvider;
	use ICanBoogie\CLDR\Provider\WebProvider;

	$locale = 'en';

	$provider = new CachedProvider(
		new WebProvider,
		new CacheCollection([
			new RunTimeCache,
		])
	);
	$repository = new Repository($provider);

	$tzs = [];

	foreach (DateTimeZone::listIdentifiers() as $tzid) {
		$tz = new DateTimeZone($tzid);
		$location = $tz->getLocation();

		try {
			$territory = $repository->territory_for($location['country_code']);
			$countryName = $territory->name_as($locale);
		} catch (Exception $e) {
			$countryName = 'Unknown';
		}

		$tzs[$countryName][$tzid] =
			$location['comments'] ?: 'Whole Region';
	}

	ksort($tzs);
	var_export($tzs);
	?>

With as excerpted output (using the ``en`` locale)::

	…
	'Palestinian Territories' => array (
	   'Asia/Gaza' => 'Gaza Strip',
	   'Asia/Hebron' => 'West Bank',
	 ),
	…
	'United Kingdom' => array (
	  'Europe/London' => 'Whole Region',
	),
	…

Or with the ``cy`` (Welsh) locale::

	…
	'Tiriogaethau Palesteinaidd' =>
	array (
	  'Asia/Gaza' => 'Gaza Strip',
	  'Asia/Hebron' => 'West Bank',
	),
	…
	'Y Deyrnas Unedig' =>
	array (
	  'Europe/London' => 'Whole Region',
	),
	…

Now you just need to find out how to translate ``Whole Region`` into each
language. Pob Lwc!
