Memory Malfeasance
==================

.. articleMetaData::
   :Where: London, UK
   :Date: 2022-03-29 14:30 Europe/London
   :Tags: blog, php
   :Short: memerr

A while ago I started getting weird crashes on my desktop machine —
Gargleblaster. Once in a while, PHP, or Node, would crash. And browser tabs
kept turning blank, with Firefox crashing altogether once in a while too.

At first I thought there was some memory corruption in a system library, but
neither valgrind or GDB would show any issues — if the problem could be
reproduced at all. It was also very random, but the problem went away for a
short while after a reboot.

I suspected the worst: Broken memory.

In the past I had used tools like ``memtest86``, and ``memtest86+`` — both
available as packages on my Debian system. There are some `complications
<https://askubuntu.com/questions/917961/can-i-boot-memtest86-if-im-using-uefi>`_
with both of these on newer UEFI BIOS systems. This meant that when I tried
them, the system would not even boot. A new version of ``memtest86+`` was
supposed to fix this, but that did not work either for me.

I decided to live with it for a while, but after another total loss of tabs
(oh dear!), I `stumbled upon
<https://unix.stackexchange.com/questions/439674/is-there-a-free-libre-open-source-alternative-to-memtest86-that-works-with-ue/439769#439769>`_
a different tool: `PCMemTest <https://github.com/martinwhitaker/pcmemtest>`_.
This **did** boot, but their documentation page says `"The UHCI USB controller
is not yet supported"
<https://github.com/martinwhitaker/pcmemtest#known-limitations-and-bugs>`_,
which is needed for USB keyboards.

I was happily surprised that Debian's APT repository also included a package
for this memory testing tool. After I installed it, I rebooted my machine to
see what it would say. The result:

.. image:: /images/content/pcmemtest.jpg
   :title: PCMemTest showing broken memory

PCMemTest allows you to create a configuration line for the Grub configuration
which the Linux kernel uses while booting up to exclude certainly parts of
physical memory from being used. However, without the USB keyboard working, I
could not not navigate to that feature.

Then I `read
<https://askubuntu.com/questions/908925/how-do-i-tell-ubuntu-not-to-use-certain-memory-addresses>`_
that the kernel itself also has a memory test tool built in: the `memtest
<https://www.kernel.org/doc/html/latest/admin-guide/kernel-parameters.html>`_
kernel parameter.

To include the memory test when the system boots, update the
``GRUB_CMDLINE_LINUX_DEFAULT`` line in ``/etc/default/grub`` to::

	GRUB_CMDLINE_LINUX_DEFAULT="quiet pcie_aspm=off memtest=4"

And then run ``update-grub``.

Now when the system starts, the kernel will run a memory test and
automatically exclude any memory that it finds not working.

On my system this looks in the ``dmesg`` output like::

	[    0.000000] early_memtest: # of tests: 4
	[    0.000000]   0x0000000000100000 - 0x0000000001000000 pattern aaaaaaaaaaaaaaaa
	[    0.000000]   0x0000000001020000 - 0x0000000004000000 pattern aaaaaaaaaaaaaaaa
	[    0.000000]   0x000000000401e000 - 0x0000000009df0000 pattern aaaaaaaaaaaaaaaa
	…
	[    0.000000]   0x0000000100000000 - 0x0000000180000000 pattern 5555555555555555
	[    0.000000] ------------[ cut here ]------------
	[    0.000000] Bad RAM detected. Use memtest86+ to perform a thorough test
				   and the memmap= parameter to reserve the bad areas.
	…
	[    0.000000]   5555555555555555 bad mem addr 0x000000016dbc8450 - 0x000000016dbc8458 reserved
	[    0.000000]   0x000000016dbc8458 - 0x0000000180000000 pattern 5555555555555555
	[    0.000000]   0x0000000180410000 - 0x0000000727200000 pattern 5555555555555555
	[    0.000000]   0x000000072980d000 - 0x000000107f300000 pattern 5555555555555555
	[    0.000000]   0x0000000000100000 - 0x0000000001000000 pattern ffffffffffffffff
	…
	[    0.000000]   0x000000072980d000 - 0x000000107f300000 pattern 0000000000000000

The line ``bad mem addr 0x000000016dbc8450 - 0x000000016dbc8458 reserved`` is
saying that the kernel excluded that section of memory because it found it to
be broken.

Since I booted my system 16 days ago, I have no longer seen any unexplained
crashes. Yay!

At some point I will need to replace this memory, if I find out which of the
four memory modules it is. That is a job for some other time.
