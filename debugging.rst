.. _debugging:

Debugging GCC
=============

The ``gcc`` binary is actually a relatively small "driver" program, which
parses some command-line options, and then invokes one or more other
programs to do the real work.

Consider compiling a simple hello world C program:

.. literalinclude:: hello.c
   :language: c

to generate an ``a.out`` binary:

.. code-block:: console

  $ gcc hello.c
  $ ./a.out
  Hello world

Internally, the driver will invoke ``cc1`` (the C compiler), which
converts the .c code to a .s assembler file.  Assuming this succeeds the
driver will typically then invoke ``as`` (the assembler), then the linker.

Given that, how do we debug the C compiler?   The easier way is to add
``-wrapper gdb,--args`` to the `gcc` command-line:

.. code-block:: console

  $ gcc hello.c -wrapper gdb,--args

TODO:

  * howto: stepping through the compiler, stepping through a pass

  * talk about dumpfiles also

  * similar for valgrind
