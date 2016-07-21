.. GCC Contributors Guide documentation master file, created by
   sphinx-quickstart on Thu Jul 21 17:39:25 2016.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

GCC for New Contributors
========================

This is an unofficial guide to GCC's internals, aimed at new developers,
and at plugin authors.

"Gotchas"
---------

It's worth knowing about the following:

* As of FIXME version GCC is (mostly) implemented in C++, but we haven't
  renamed the source files.  Hence you will see source files with a ``.c``
  extension throughout the source tree.  These are generally handled by
  the build system as C++, rather than C.

* It's often difficult to grep for the declaration of, say, ``struct foo``
  in the GCC sources due to the presence of "GTY" markers.  These are
  annotations to the types, and are used by GCC's garbage-collector.
  They're stripped away by the preprocessor when building GCC itself, but
  get used by a tool during the build called ``gengtype``.

  TODO: how to search for them?

* The ``gcc`` binary is actually a relatively small "driver" program, which
  parses some command-line options, and then invokes one or more other
  programs to do the real work.  If compiling a C source file, this will
  be ``cc1`` (the C compiler), then ``as`` (the assembler), then the linker.

  Given that, how do we debug the C compiler?   The easier way is:
  TODO:
    * ``-wrapper gdb,--args``

  TODO:
    howto: stepping through the compiler, stepping through a pass

TODO
----

  * license for this document

  * Building GCC

    * have a hacking copy, a pair of verification copies, and an svn checkout

    * (I use git for everything other than the final commit)

 * Option macros

  * "^L"

  * Memory management and lifetimes

    * obstacks

    * new/delete and new[]/delete[]

    * malloc/free

    * ggc

  * tree, gimple, RTL

  * How to run just one testcase

  * Running under the debugger:

    * ``-wrapper gdb,--args``

    * gdbhooks

  * global state

The bug tracker


Contents:

.. toctree::
   :maxdepth: 1

   how-to-improve-the-location-of-a-diagnostic.rst

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

