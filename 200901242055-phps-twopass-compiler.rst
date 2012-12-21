PHP's two-pass compiler
=======================

.. articleMetaData::
   :Where: Istanbul, Turkey
   :Date: 20090124 2055 EET
   :Tags: cms, php, travel, work

On my way to Istanbul I was looking at Xdebug bug `#422`_ . For some
reason Xdebug was crashing while doing code-coverage analysis, in the
part that analyses which code was dead (ie. opcodes that could never be
reached). The crash occurred with a JMPZ (jump-if-zero) instruction,
that suddenly saw a jump-to position of 572222864. That position
resembles more a jump-address.

Xdebug uses the same branch analysis implementation as `VLD`_ so I used the latter
tool to find out why it would crash. Unfortunately, it was working just
all nice and fine with VLD. After digging around some more, I saw from
the back trace that the crash in `Xdebug`_ only occurred when a user-defined
error-handler was called while parsing a file. The latter gave me the
insight of looking at which phase the compiler was in. I remembered that
PHP has a two phase compiler. The first pass is quick and dirty, and
only records the opcode line *number* to jump to. Xdebug however
was expecting an *memory address* as jump target. Because a memory
address is a much larger number than an opcode number—the latter
usually not being much higher than a thousand—Xdebug was setting the
"visited" flag in a part of memory that wasn't allocated. And
writing to unallocated memory makes a process die with a segmentation
fault.

The compiler in PHP is two-pass. During the first pass, it will find out
to which opcode it needs to jump in the jump instructions. However, the
PHP engine (and Xdebug) expects a memory address to jump to while *executing* your script. In the second pass, the compiler will then
go over the generated opcodes and calculate the memory address to jump
to from the jumps to opcode numbers. It will also do a few other things,
such as collapsing sequential EXT_STMT opcodes, calling Zend extension's
functions to finalize the opcode arrays—Xdebug uses this for caching
whether an opcode array has been scanned already—and re-allocating the
opcode array itself to save space.

Now, the thing is, that usually VLD and Xdebug kick in *after* the
whole opcode array has been created, which includes running the second
pass of the compiler. However, Xdebug also tries to analyze opcode
arrays while executing them. In the case of a user defined error
handler, that happens before the second pass has been run. Preventing
the crash was therefore as easy as making sure that the compiler's
second pass had been run while scanning the opcode arrays for executable
code.


.. _`#422`: http://bugs.xdebug.org/view.php?id=422
.. _`VLD`: http://derickrethans.nl/vld.php
.. _`Xdebug`: http://xdebug.org

