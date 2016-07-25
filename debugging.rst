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

  # Invoke "cc1" (and "as", etc) under gdb:
  $ gcc hello.c -wrapper gdb,--args

The ``gcc`` driver will then invoke ``cc1`` under ``gdb``, and you can
set breakpoints, and step through the code.

.. note::

   If you ever need to debug the driver itself, you can simply run it under
   gdb in the normal way:

   .. code-block:: console

     # Invoke the "gcc" driver under gdb:
     $ gdb --args gcc hello.c

  I find myself doing this much less frequently than the
  ``-wrapper gdb,--args`` invocation for debugging ``cc1`` though.

You can invoke other debugging programs this way, for example, valgrind:

.. code-block:: console

  # Invoke "cc1" (and "as", etc) under valgrind:
  $ gcc hello.c -wrapper valgrind

.. note::

  For good results under valgrind, it's best to configure your build
  of gcc with :option:`--enable-valgrind-annotations`, which automatically
  suppresses various known false positives.


TODO:

  * howto: stepping through the compiler, stepping through a pass

  * talk about dumpfiles also

  * similar for valgrind
