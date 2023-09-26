Xdebug Update: August 2023
==========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2023-09-12 09:27 Europe/London
   :Tags: blog, php, xdebug
   :Short: xdebug-23aug

In this monthly update I explain what happened with Xdebug development
in the past month. These are normally published on the first
Tuesday on or after the 5th of each month.

`Patreon <https://www.patreon.com/derickr>`_ and `GitHub
<https://github.com/sponsors/derickr/>`_ supporters will get it earlier,
around the first of each month.

You can `become a patron <https://www.patreon.com/bePatron?u=7864328>`_
or support me through `GitHub Sponsors
<https://github.com/sponsors/derickr>`_. I am currently 36% towards my $2,500
per month goal, which is set to allow continued maintenance of Xdebug.

If you are leading a team or company, then it is also possible to
support Xdebug through `a subscription <https://xdebug.org/support>`_.

In the last month, I spend around 27 hours on Xdebug, with 32 hours funded.

Towards Xdebug 3.3
------------------

In August I mostly spend my time on improving Xdebug's
`xdebug_get_function_stack()` function and stack traces with regard to
chained exceptions.

A `feature request <https://bugs.xdebug.org/1562>`_ asked whether it would be
possible to add the local variables for each stack frame that is returned with
the `xdebug_get_function_stack()` function. Xdebug can already show local
variables for the top most frame when it shows stack traces (through the
`xdebug.show_local_vars
<http://xdebug.org/docs/all_settings#show_local_vars>`_ setting), but the
function's result don't include them.

When implementing this feature, I noticed that arguments were being returned as
strings, instead of actual values, as part of each stack frame. I created `an
issue <https://bugs.xdebug.org/2194>`_ for that and implemented that as well.

When the original requester tried out the new feature, it turned out that he
wanted to do this in a user-defined exception handler. However, at that stage,
the original stack has been destroyed, and Xdebug no longer could access that
information. 

To work around this, I now cache the stack when an exception gets **thrown**
so that the cached version then can be requested when calling
``xdebug_get_function_stack()`` with the new ``from_exception`` option.

That looks like::

    <?php
    class Handlers
    {
        function __construct(private string $title, private float $PIE) {}

        static function exceptionHandler($exception)
        {
            $s = xdebug_get_function_stack( [ 'from_exception' => $exception ] );
            var_dump($s);
        }
    }

    class Error_Entry
    {
        public function __construct($base, $errno)
        {
            throw new Exception();
        }
    }

    set_exception_handler(['Handlers', 'exceptionHandler']);
    $e = new Error_Entry(1, 2);

    ?>

Xdebug's cache is eight items big, which allows for 8 rethrown/chained
exception stacks to be remembered.

Because of this cache it was now also (finally) possible to resolve `issue
#450 <https://bugs.xdebug.org/450>`_ and `issue #476
<https://bugs.xdebug.org/476>`_. This now means that chained and rethrown
exceptions are now displayed when Xdebug shows a stack trace, whether it is on
the CLI, or in an HTML context.

Over the next few months I will continue to work on the features and issues on
the `3.3 roadmap <https://bugs.xdebug.org/roadmap_page.php?version_id=101>`_,
without any guarantees these tickets will be implemented.

If you have comments, suggestions, or if your company wants to help fund 
features, please `reach out <mailto:derick@xdebug.org>`_, or leave comments on
the document.

Xdebug Videos
-------------

I have published one new videos in the last month:

- `PHP: Debugging FFI and PHP <https://www.youtube.com/watch?v=u420A89tIMY>`_

Let me know what you'd like to see!

You can find all previous videos on my `YouTube channel
<https://www.youtube.com/playlist?list=PLg9Kjjye-m1g_eXpdaifUqLqALLqZqKd4>`_.

Business Supporter Scheme and Funding
-------------------------------------

In the last few months, two new business supporters signed up:

`REWE Digital GmbH <https://rewe-digital.com>`_ through the Supporter Scheme,
and `Clever Age <https://www.clever-age.com/en/>`_ on Patreon.

If you, or your company, would also like to support Xdebug, head over to
the `support <https://xdebug.org/support>`_ page!

Besides business support, I also maintain a `Patreon
<https://www.patreon.com/derickr>`_ page, a profile on `GitHub sponsors
<https://github.com/sponsors/derickr>`_, as well as an `OpenCollective
<https://opencollective.com/xdebug>`_ organisation.

Xdebug Cloud
------------

Xdebug Cloud is the *Proxy As A Service* platform to allow for debugging
in more scenarios, where it is hard, or impossible, to have Xdebug make
a connection to the IDE. It is continuing to operate as Beta release.

Packages start at £49/month, and I have recently introduced a package
for larger companies. This has a larger initial set of tokens, and
discounted extra tokens.

If you want to be kept up to date with Xdebug Cloud, please sign up to
the `mailinglist <https://xdebug.cloud/newsletter>`_, which I will use
to send out an update not more than once a month.
