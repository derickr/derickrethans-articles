Unicode Collation Sorting
=========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2025-03-25 14:50 Europe/London
   :Tags: blog, php
   :Short: intl-sort

I recently added support to this website to receive Favourites and Replies
through the ActivityPub posts that I create from each article.

Replies show up as comments after moderation, and favourites show up as Likes
under each of the articles.

When fetching all the likes, my internal API returns the following structure —
just like how `PDOStatement::fetchAll()
<https://www.php.net/manual/en/pdostatement.fetchall.php>`_ returns them::

    array(7) {
      [0]=>
      array(8) {
        ["name"]=>
        string(14) "Marcus Bointon"
        ["linked_account"]=>
        string(33) "https://phpc.social/users/Synchro"
      }
      [1]=>
      array(8) {
        ["name"]=>
        string(14) "Derick Rethans"
        ["linked_account"]=>
        string(33) "https://phpc.social/users/derickr"
      }
      [2]=>
      array(8) {
        ["name"]=>
        string(26) "🇵🇸 Álvaro González"
        ["linked_account"]=>
        string(37) "https://mastodon.social/users/kAlvaro"
      }
    }

I wanted to sort the *likes* by name, which turned out harder to be than I had
hoped. Mostly because people do odd things with their names sometimes, such as
not using an upper-case letter, or by prefixing their names with emojis as you
can see in the example above.

If each entry in my array was only a string with a name, and no emoji is
prefixed, you can use `natcasesort()
<https://www.php.net/manual/en/function.natcasesort.php>`_, but I have both
complex objects and an emoji. This means PHP's built-in functions don't cut
it.

However, PHP also has the **Intl** extension, which has a `Collator
<https://www.php.net/manual/en/class.collator.php>`_ class with a `sort
<https://www.php.net/manual/en/collator.sort.php>`_ method.

To create such a collator in code, use::

    $collator = new Collator('root');

The `Root Collation
<http://www.unicode.org/reports/tr35/tr35-collation.html#Root_Collation>`_ is
ICU's `Default Collation
<https://www.unicode.org/reports/tr10/#Default_Unicode_Collation_Element_Table>`_,
but as the ICU documentation says:

    Not all languages have sorting sequences that correspond with the root
    collation order because no single sort order can simultaneously encompass
    the specifics of all the languages. In particular, languages that share a
    script may sort the same letters differently.

This is good enough for my use case there, where I only want to sort names in
a reasonable fashion.

One thing that it **does** do is to sort lower-case letters before upper-case
letters, but this behaviour can be changed by setting `CASE_FIRST
<https://www.php.net/manual/en/class.collator.php#collator.constants.case-first>`_
attribute on the collator to `UPPER_FIRST
<https://www.php.net/manual/en/class.collator.php#collator.constants.upper-first>`_::

    $collator->setAttribute(Collator::CASE_FIRST, Collator::UPPER_FIRST);

Letters in non-Latin script will sort after the Latin letters.

However, the ``Collator->sort()`` method can only handle arrays of strings,
and not complex objects.

To get around this limitation we instead use PHP's built-in `usort()
<https://www.php.net/usort>`_ function with our own callback. In our callback
we can use the `Collator::getSortKey()
<https://www.php.net/collator-getsortkey>`_ method on each sort element's
``name`` array key to create the key that ``Collator::sort()`` would have used
internally. 

For good measure, we also use `trim <https://www.php.net/trim>`_ to remove
preceding and trailing whitespace. The code to sort the elements now looks
like::

    $col = new Collator('root');
    $col->setAttribute(Collator::CASE_FIRST, Collator::UPPER_FIRST);

    usort(
        $allLikes,
        function($a, $b) use ($col) {
            $aName = trim($a['name']);
            $bName = trim($b['name']);
            $aKey = $col->getSortKey($aName);
            $bKey = $col->getSortKey($bName);

            return $aKey <=> $bKey;
        }
    );

Now the only problem is that emojis sort before actual letters, so we need to
scrub them. Each letter defined in the Unicode standard has properties
associated with it. One of those properties is the `General Category
<https://www.unicode.org/reports/tr42/#d1e3191>`_. If you examine the `chart
<https://www.unicode.org/Public/UCD/latest/ucd/UnicodeData.txt>`_ you will
fined that for ``0061;LATIN SMALL LETTER A`` the GC is ``Ll`` (Letter,
Lower-case). The flag emoji from my example is composed of two characters:
``1F1F5;REGIONAL INDICATOR SYMBOL LETTER P`` and ``1F1F8;REGIONAL INDICATOR
SYMBOL LETTER S``. Both of these have as category ``So`` (Symbol, Other).

The PCRE library that PHP uses for regular expressions is aware of these
categories, and you can select for for them with the ``\p{…}`` specifier. The
`documentation <https://www.php.net/manual/en/regexp.reference.unicode.php>`_
has a page on it. 

In our case we want to remove all Symbols (with category ``S``). Changing our
``trim`` lines to strip this character class accomplishes this.

When we now combine all of this, we get::

    <?php
    $allLikes = [
        [ 'name' => 'Marcus Bointon', 'linked_account' => 'https://phpc.social/users/Synchro' ],
        [ 'name' => 'Derick Rethans', 'linked_account' => 'https://phpc.social/users/derickr' ],
        [ 'name' => '🇵🇸 Álvaro González', 'linked_account' => 'https://mastodon.social/users/kAlvaro' ],
    ];

    $col = new Collator('root');
    $col->setAttribute(Collator::CASE_FIRST, Collator::UPPER_FIRST);

    usort(
        $allLikes,
        function($a, $b) use ($col) {
            $aName = trim(preg_replace('/\p{S}+/u', '', $a['name']));
            $bName = trim(preg_replace('/\p{S}+/u', '', $b['name']));
            $aKey = $col->getSortKey($aName);
            $bKey = $col->getSortKey($bName);

            return $aKey <=> $bKey;
        }
    );

    var_dump($allLikes);
    ?>

When we now run the example with the ``$allLikes`` array from previously, our
result is::

    array(3) {
      [0] =>
      array(2) {
        'name' =>
        string(26) "🇵🇸 Álvaro González"
        'linked_account' =>
        string(37) "https://mastodon.social/users/kAlvaro"
      }
      [1] =>
      array(2) {
        'name' =>
        string(14) "Derick Rethans"
        'linked_account' =>
        string(33) "https://phpc.social/users/derickr"
      }
      [2] =>
      array(2) {
        'name' =>
        string(14) "Marcus Bointon"
        'linked_account' =>
        string(33) "https://phpc.social/users/Synchro"
      }
    }

And from that, I can then generate the HTML code to render who liked my
article, which you can hopefully see below.
