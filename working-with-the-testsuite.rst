.. Copyright (C) 2017-2023 Free Software Foundation, Inc.
   Originally contributed by David Malcolm <dmalcolm@redhat.com>

   This is free software: you can redistribute it and/or modify it
   under the terms of the GNU General Public License as published by
   the Free Software Foundation, either version 3 of the License, or
   (at your option) any later version.

   This program is distributed in the hope that it will be useful, but
   WITHOUT ANY WARRANTY; without even the implied warranty of
   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
   General Public License for more details.

   You should have received a copy of the GNU General Public License
   along with this program.  If not, see
   <http://www.gnu.org/licenses/>.

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


What tests did I just run?
**************************

By default DejaGnu merely emits the names of unexpected results: typically
FAILs results for test failures (and XPASS for unexpected successes), along
with a summary at the end.

If you want to see all of the results, look for the ``.sum`` file (e.g.
in ``testsuite/gcc/gcc.sum``.

For example, whilst working on a bug fix, I invoked the tests via::

  make check-gcc RUNTESTFLAGS="-v -v --target_board=unix\{-m32,-m64\} dg.exp=*sarif*.c"

and got a ``testsuite/gcc/gcc.sum`` containing::

  Test run by david on Fri Mar 24 10:10:20 2023
  Native configuration is x86_64-pc-linux-gnu

                 === gcc tests ===

  Schedule of variations:
      unix/-m32
      unix/-m64

  Running target unix/-m32
  Running /home/david/coding/gcc-sarif-support/src/gcc/testsuite/gcc.dg/dg.exp ...
  FAIL: gcc.dg/diagnostic-input-charset-sarif-1.c (test for excess errors)
  PASS: c-c++-common/diagnostic-format-sarif-file-1.c  -Wc++-compat  (test for excess errors)
  PASS: c-c++-common/diagnostic-format-sarif-file-1.c  -Wc++-compat   scan-sarif-file "version": "2.1.0"
  [...snip...]
  PASS: c-c++-common/diagnostic-format-sarif-file-FIXME.c  -Wc++-compat   scan-sarif-file "text": "invalid UTF-8 character <80>"
  PASS: c-c++-common/diagnostic-format-sarif-file-FIXME.c  -Wc++-compat   scan-sarif-file "text": "invalid UTF-8 character <99>"
  PASS: c-c++-common/diagnostic-format-sarif-file-Winvalid-utf8.c  -Wc++-compat  (test for excess errors)
  XFAIL: c-c++-common/diagnostic-format-sarif-file-bad-encoding.c  -Wc++-compat  (test for excess errors)
  XFAIL: c-c++-common/diagnostic-format-sarif-file-malformed-utf8.c  -Wc++-compat  (test for excess errors)

                === gcc Summary for unix/-m32 ===

  # of expected passes          71
  # of unexpected failures      1
  # of expected failures        4
  Running target unix/-m64
  Running /home/david/coding/gcc-sarif-support/src/gcc/testsuite/gcc.dg/dg.exp ...
  FAIL: gcc.dg/diagnostic-input-charset-sarif-1.c (test for excess errors)
  PASS: c-c++-common/diagnostic-format-sarif-file-1.c  -Wc++-compat  (test for excess errors)
  PASS: c-c++-common/diagnostic-format-sarif-file-1.c  -Wc++-compat   scan-sarif-file "version": "2.1.0"
  [...snip...]
  PASS: c-c++-common/diagnostic-format-sarif-file-FIXME.c  -Wc++-compat   scan-sarif-file "text": "invalid UTF-8 character <99>"
  PASS: c-c++-common/diagnostic-format-sarif-file-Winvalid-utf8.c  -Wc++-compat  (test for excess errors)
  XFAIL: c-c++-common/diagnostic-format-sarif-file-bad-encoding.c  -Wc++-compat  (test for excess errors)
  XFAIL: c-c++-common/diagnostic-format-sarif-file-malformed-utf8.c  -Wc++-compat  (test for excess errors)

                  === gcc Summary for unix/-m64 ===

  # of expected passes          71
  # of unexpected failures      1
  # of expected failures        4

                  === gcc Summary ===

  # of expected passes          142
  # of unexpected failures      2
  # of expected failures        8
  /home/david/coding/gcc-sarif-support/build/gcc/xgcc  version 13.0.1 20230321 (experimental) (GCC)

I never get the wildcard invocation correct on the first time, so this is
useful for double-checking that the tests I hoped to run are actually
running and passing.

If you want even more detail, see the ``.log`` file (such as
``testsuite/gcc/gcc.log``).  You can use this to find the command lines
used to invoke gcc, which may be helpful when debugging why a test case
is failing.


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
