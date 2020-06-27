PHP 8: A Quick Look at JIT
==========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-06-27 11:30 Europe/London
   :Tags: blog, php, phpinternalsnews
   :Short: jit1

Following on from a `PHP 8/JIT benchmark on twitter
<https://twitter.com/IvanChepurnyi/status/1276576154254786561>`_, I decided
to have a look myself.

I've picked an example that I know speeds up really well when reimplementing
it in C. I wrote about this `RDP algorithm </advent12.html>`_ some time ago.

What it does is to take a line of geospatial points (lon/lat coordinates), and
simplifies it. It's my go-to example to show raw algorithmic performance,
which is probably the *best* place to use a JIT for non-trivial code. I
actually use this in production.

With PHP 7.4::

	$ pe 7.4dev; time php -n \
		-dzend_extension=opcache -dopcache.enable=1 -dopcache.enable_cli=1 \
		-dopcache.jit=1235 -dopcache.jit_buffer_size=64M \
		bench-rdp.php 1000
	Using array (
	  0 => 'RDP',
	  1 => 'simplify',
	)

	real	0m8.778s
	user	0m8.630s
	sys	0m0.117s

(I realise that the opcache arguments do nothing on the command line here).
This runs ``RDP::simplify`` (my PHP implementation) 1000 times in about 8
seconds.

With PHP 8.0 and JIT::

	$ pe trunk; time php -n \
		-dzend_extension=opcache -dopcache.enable=1 -dopcache.enable_cli=1 \
		-dopcache.jit=1235 -dopcache.jit_buffer_size=64M \
		bench-rdp.php 1000
	Using array (
	  0 => 'RDP',
	  1 => 'simplify',
	)

	real	0m4.640s
	user	0m4.627s
	sys	0m0.008s

It jumps from *~8.8s* to *~4.6s*, a reduction in time of *~4.2s* (or *48%*),
which is pretty good.

Now if I run the same with the `geospatial extension
<https://github.com/php-geospatial/geospatial>`_ which has a C implementation.

With PHP 7.4 and the extension::

	$ pe 7.4dev; time php -n -dextension=geospatial \
		-dzend_extension=opcache -dopcache.enable=1 -dopcache.enable_cli=1 \
		-dopcache.jit=1235 -dopcache.jit_buffer_size=64M bench-rdp.php 1000
	Using 'rdp_simplify'

	real	0m0.695s
	user	0m0.675s
	sys	0m0.021s

Which gives a reduction in speed compared to PHP 7.4 of *~8.1s* (or *92%*).

So it looks like the JIT does do some good work for something that's highly
optimisable, but still nowhere near what an implementation in C could do.

The code that I used is in this
`Gist <https://gist.github.com/derickr/74889c388a1667961cafec1f4a27fdfe>`_.

This ran on a 4th gen ThinkPad X1 Carbon, making sure my CPU was pinned at its
maximum speed of 3.3Ghz. Although I've pasted only one result for each, I did
run them several times with very close outcomes.
