Humans in the LLM Loop
======================

.. articleMetaData::
   :Where: London, UK
   :Date: 2026-06-09 19:00 Europe/London
   :Tags: blog, php, xdebug, ai
   :Short: xdebug-llm-reports

In the last few weeks, I have been working through some bug reports for
Xdebug, that resulted in the `Xdebug 3.5.3 release
<https://xdebug.org/announcements/2026-06-08>`_. 

These bug reports did not come solely from humans, but rather from a mix of
humans using LLM assistant tools, focussing on security related problems, from
two different sources and methodologies. 

Although all of these issues where indeed bugs that I now have fixed, I don't
think any of them can be classified even as having a ``low`` security impact.

But there was a whole host of other issues with these reports. The reports
themselves can be unnaturally verbose — and also fairly alarmist using terms
like "victim" and "attacker". The tests that were present in the reports were
often minimal, and sometimes incomplete, and so were some of their suggested
fixes.

The humans forwarding the reports took care not to flood the issue tracker
with reports out of the blue, and reached out to me first. They've also been
helpful discussing the reported issues.

The first four cases were reported by `Ilia Alshanetsky <https://ilia.ws/>`_,
a long time, and recently returned, contributor to PHP.

The first report, `#2421 <https://bugs.xdebug.org/2421>`_, deals with sending
wrong option characters with commands through the DBGp protocol, that an IDE
and Xdebug use to communicate.

Xdebug would allocate an array for 27 of these, representing the 26 lower case
letters of the Latin alphabet, and the ``-`` character. What Xdebug did not do
is to make sure the option letters were indeed in the range ``[a-z-]``, and
would happily accept ``-@`` or ``-\x00``. This makes it possible to overwrite
locations in memory.

The suggested patch was fine, but the test that went with it was very hard to
read. It didn't use already exiting framework for testing the step debugger
either — I had to `add my own
<https://github.com/xdebug/xdebug/pull/1080/changes/3be755477abfcb3dcbcd9e8b08faed9f0a58d203>`_.

I also believe that this issue could as easily have been found by a
fuzzer, which I `added now as well <https://github.com/xdebug/xdebug/pull/1080/changes/686eff35190230133969c59fa6dd0386a39ee1de>`_.
The fuzzer found the same problem in about five seconds, and luckily, nothing
else either.

The second issue, `#2422 <https://bugs.xdebug.org/2422>`_ complained that 
there was no limit in the debugger's code that reads commands from the
network.

The patch was mostly fine, but the test was wholly cumbersome again — it didn't use
the already existing testing framework either. It also picked a funnily large (arbitrary)
limit of 64MB for DBGp commands, where 64KB would easily suffice — in most
cases, 256 bytes would have been fine.

The third issue, `#2423 <https://bugs.xdebug.org/2423>`_, argues that Xdebug
shouldn't follow symlinks when creating profiling or trace files.

The patch was OK and trivial, but the test was again very hard to read
making it hard to figure out what it as trying to do. It also did not make use
of some existing helpers to skip tests. It came up with::

	--SKIPIF--
	<?php
	if (PHP_OS_FAMILY === 'Windows') die('skip: Linux-only symlink semantics');
	?>

Instead of what is used everywhere else::

	--SKIPIF--
	<?php
	require __DIR__ . '/../utils.inc';
	check_reqs('!win');
	?>

The fourth and last issue through Ilia, `#2424
<https://bugs.xdebug.org/2424>`_, deals with Xdebug's Control Socket
functionality, where its parser would not handle empty or large command
packets correctly. The LLM proposed patch fixed the symptoms, but not the
actual cause of this issue.

The second set of reports were shared with me in a private gist by Volker, as
part of the PHP Foundation's `Ecosystem Security Team
<https://thephp.foundation/blog/2026/05/18/announcing-ecosystem-security-team/>`_
effort.

The first one was a duplicate of `bug #2421 <https://bugs.xdebug.org/2421>`_.
The test focussed on the Control Socket functionality instead of the Step
Debugger, but the underlying issue and fix were the same.

The second issue I added as `bug #2433
<https://bugs.xdebug.org/2433>`_. When you enable
``xdebug.collect_assignments`` with tracing, Xdebug needs to re-create the
variable name from several opcodes in order to show this name in a readable way.

But the issue is not a real time problem, insofar this can only happen if you
run PHP code on the command line through the ``-r`` option, **and**
``xdebug.start_with_request`` is ``yes``. For some reason, when PHP runs code
through ``-r``, the CLI binary does not generate ``EXT_STMT`` opcodes (Xdebug
uses these for breaking during step-debugging), which
would otherwise prevent the out-of-bound memory read from happening.

The LLM tool also hadn't realised that the third argument to the function
responsible for reassembling the variable name was always ``NULL``, and hence
superfluous.

I addressed these both through the same commit, and added a test, which would
not exhibit the problem in most situations either. It is still good to have
the expected outcome documented.

Another report resulted in two issues in Xdebug's tracker. `#2427
<https://bugs.xdebug.org>`_ addresses an incorrect memory read if the
``xdebug.file_link_format`` setting ends with a lone ``%``, and `#2429
<https://bugs.xdebug.org>`_, a similar report, but then for
``xdebug.trace_output_name``.

Although the report mentioned three locations, the accompanying test only
covered one of three situations where this was a problem: for trace file
names, but not formatting link files through ``xdebug.file_link_format``, nor
profiling files. 

The patch it suggested was also wrong, as it would remove
the trailing ``%`` instead of keeping it. One of three locations where the
trailing ``%`` was not handled, was internal only, and hence couldn't be
triggered by making configuration errors.

The test that came with this report did not help me trying to show the
problem. It relied on AddressSanitizer to show any problems, but I could
not get that to happen. All the tests through this tool also provided tests
that tested that the **bug** was present, and not what the correct result
ought to be.

Luckily, using the Xdebug test suite with the ``valgrind`` tool showed
the problem.

A further report, `#2430 <https://bugs.xdebug.org/2430>`_ showed a problem if either an IDE through the step debugging
protocol, or a developer directly, would request the contents of a "variable"
named ``:::``. The step debugger uses ``::`` to indicate "all the static
variables for this class", and following that up with a ``:`` isn't valid.

The fix was good, but I couldn't directly use the test case, as it tested for
the *broken* behaviour. The test was fairly trivial to write as the
reproduce case in the reported test case was correct.

And the last issue from the second list, `#2431 <https://bugs.xdebug.org>`_,
again reinvented its own way for doing DBGp tests, and also tested
that the behaviour was *wrong*, instead of a test to show that it now works.

Even with the code fixed, the new correct test would also surface another issue, as
it would have resulted in Xdebug to open a directory as it was a file, and then
fail.

**Conclusion**: Although the LLM tools did find bugs, they were not
particularly groundbreaking. Some of the bugs would also have been found by
fuzzing, and used a lot less resources in that process.

Most of the crashes and potential security issues would only be a problem if
an attacker didn't already have access to the machine that the code runs on
itself, or have an IDE talking to Xdebug already. 

If you have access to the machine, you can do worse without these bugs
present. If you have client access to Xdebug through DBGp, you would have all
the functionality that PHP provides, including reading all files on the file
system and running code.

The generated test cases were generally hard to read, or incomplete. The patches
that the models came up with were not always comprehensive, or correct. I also
spent too much time getting AddressSanitizer to do anything, unsuccessfully.

I think I would have been as quick writing these patches and actual test cases
myself, when provided with the issues' causes and the reasoning that was
provided.

I don't think I'll be spending time trying to get these tools to work myself,
but in the right hands with people that know what they're doing, they can find
issues that needs to be addressed. But the value comes from the humans
interpreting their results.
