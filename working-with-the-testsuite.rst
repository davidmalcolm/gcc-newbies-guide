Working with the testsuite
==========================

Running just one test (or just a few)
*************************************

1) Find the pertinent Tcl script that runs the test: a .exp script in
the same directory, or one of the ancestors directories.  The significant
part is the filename.

For example, for test case
``gcc/testsuite/gcc.dg/pragma-diag-1.c``, it's
``gcc/testsuite/gcc.dg/dg.exp``, where the filename is ``dg.exp``.

2) Figure out the appropriate "make" target, normally based on the
source language for the test.  For the above example, it's ``check-gcc``.

3) Run make in your ``BUILDDIR/gcc``, passing in a suitable value for
``RUNTESTFLAGS`` based on the filename found in step 1 above.
For this case, giving it a couple of ``-v`` flags for verbosity (so that
we can see the command-line of the compiler invocation) it would be::

  $ make -jN && make check-gcc RUNTESTFLAGS="-v -v dg.exp=pragma-diag-1.c"

for some N; I like the ``make && make check-FOO`` construction to ensure
that the compiler is rebuilt before running the tests.

You can also use wildcards e.g.::

  make -j64 && make check-gcc RUNTESTFLAGS="-v -v dg.exp=pragma-diag-*.c"

(and can use ``-jN`` for some N on the ``make check-FOO`` invocation if
there are a lot of tests; I tend not to use it for a small number of tests,
to avoid interleaving of output in the logs).

Testcases that apply to both C and C++ should live in ``c-c++-common``.
You can run them by making both the ``check-gcc`` and ``check-g++``
Makefile targets.  Adapting the idea from above, here's an invocation I
used whilst working on a patch::

  make -k -j64 \
    && make check-gcc check-g++ \
         RUNTESTFLAGS="-v -v dg.exp=Wformat-pr88257.c"

Here's a yet more complicated example, which runs various tests from both
``dg.exp`` and ``tree-ssa.exp`` (with multiple wildcards), and runs them
for both 32-bit and 64-bit ABIs::

  make check-gcc \
    RUNTESTFLAGS="--target_board=unix\{-m32,-m64\} dg.exp='pr10474.c pr15698*.c' tree-ssa.exp=20030530-2.c"


"test for excess errors"
************************

If a DejaGnu test is failing with "test for excess errors"
you can go into gcc/testsuite/lib/prune.exp and uncomment this line
within ``proc prune_gcc_output``:

.. code-block:: tcl

    #send_user "After:$text\n"

to:

.. code-block:: tcl

    send_user "After:$text\n"

This will print any messages that weren't pruned by ``dg-`` directives.
