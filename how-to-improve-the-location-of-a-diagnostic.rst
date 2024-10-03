.. Copyright (C) 2016-2023 Free Software Foundation, Inc.
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

How to improve the location of a gcc diagnostic
===============================================

A user filed a bug about a bad location in a warning.  It was marked as
an "easyhack" in bugzilla, and I had a go at fixing it.

I thought it may be useful for new GCC developers if I document what I
did to fix it.

FWIW, the bug was PR c/71610
i.e. https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71610)
("Improve location for "warning: ISO C restricts enumerator values to
range of 'int' [-Wpedantic]"?").


Step 1: create a minimal reproducer for the diagnostic
------------------------------------------------------

This was easy for this bug: I copied it from the bug report.  Try to
avoid ``#include`` if possible; start with the simplest possible reproducer
(to minimize effort in the debugger), and work from there.

If you're working on a bug in our bugzilla it's good to click on the
"take" button next to "Assignee" to mark yourself as assignee, and to
set the status to ASSIGNED, so that we don't duplicate effort.


Step 2: create a file under the relevant subdirectory of gcc/testsuite
----------------------------------------------------------------------

This will be under one of ``gcc.dg``, ``g++.dg`` or ``c-c++-common``.
If possible, create it under ``c-c++-common`` so that we can test both the
C and C++ frontends.

I created a file called ``pr71610.c`` under ``c-c++-common``.  (If this was
for a specific warning, the name of the warning may have been better,
but my case was part of ``-Wpedantic``, which is too broad for a testcase
title).


Step 3: run the reproducer under DejaGnu
----------------------------------------

With a freshly built gcc, I ran the following from the "gcc" build
directory:

.. code-block:: console

  make -jN -k && make check-gcc RUNTESTFLAGS="-v -v dg.exp=pr71610*"

changing the "N" in ``-jN`` to reflect the number of cores on my box.

This ensures that the compiler is rebuilt (useful when making changes),
and then runs the C compiler testsuite, filtering to just the new tests
that I'd added.

Given that the test case is for ``c-c++-common``, it's also worth running
``make check-g++`` to run the C++ compiler on the same tests.

I added DejaGnu directives to the test file to ensure that the test was
run with appropriate command-line flags to trigger the behavior in
question.  In this case, -Wpedantic is needed, and we want to ensure
that the correct source ranges are printed so ``-fdiagnostics-show-caret``
is also needed (otherwise the testsuite defaults to
``-fno-diagnostics-show-caret``).

Hence I added:

.. code-block:: c

  /* { dg-options "-pedantic -fdiagnostics-show-caret" } */

to the test case.

Given that I wanted to test ranges, I renamed some of the identifiers
in the test case to avoid using 1-character names.  That way it's easy
to verify that all of the identifier is properly underlined (since
finish != the start for a multi-character identifier).

At this point you should have a reproducer that runs within our DejaGnu
-based test suite, and demonstrates the problem.

Adding ``-v -v`` to ``RUNTESTFLAGS`` gives us the command-line invocations
of the test cases.  Copy and paste this and verify that the reproducer can
be run directly from the command-line.  This brings us to...


Step 4: run the reproducer under gdb
------------------------------------

Having created a reproducer and identified a command-line invocation
for running it under the testsuite, add the following to the end of the
command-line:

.. code-block:: console

  -wrapper gdb,--args

This will make the gcc driver run "cc1" under gdb.

A good breakpoint location is diagnostic_show_locus.  This is run
whenever a diagnostic is actually emitted (as opposed to, say, warning
and warning_at, which are called many times as the compiler runs, with
most of the calls doing nothing):

.. code-block:: console

  (gdb) break diagnostic_show_locus
  (gdb) run

Hopefully the debugger now hits the breakpoint.  At this point, go up
the stack until you reach the frontend call that emits the diagnostic
(in my case, this was a call to pedwarn).  It's usually helpful to now
put a breakpoint on the actual emission point (to save a step when
re-running).


Step 5: figure out where the location is coming from, and improve it
--------------------------------------------------------------------

In the debugger you can investigate which location is being used for
the diagnostic, and use a better one.

Many older calls to the diagnostic subsystem implicitly use the
``input_location`` global, which is typically the location (and range) of
the token currently being parsed.  (We're aiming to eventually eliminate
implicit uses of ``input_location``).

In my case, the code was using:

.. code-block:: c++

  value_loc = c_parser_peek_token (parser)->location;

which gets the location (and range) of the first token within an
expression.

Usually it's better to display the range of an entire expression rather
than just a token.

Some expression tree nodes have locations, others do not.  I hope to
fix this for gcc 7.  In the meantime, the C frontend has a
``struct c_expr`` which captures the range of an expression during
parsing (even for those that don't have location information), and the
C++ frontend has a ``class cp_expr`` which works similarly, capturing the
location and range (again, only during parsing).  Or you can use:

.. code-block:: c++

  EXPR_LOC_OR_LOC (expr, fallback_location)

If you want to emit multiple locations from a test, or emit fix-it
hints, try using the rich_location class (see libcpp/include/line
map.h).


Step 6: add more test cases
---------------------------

Assuming you've made a change to update the location that's used for a
diagnostic, think about what could break, and try to extend the test
case beyond the minimal reproducer to cover other cases.  It's good to
think from two perspectives: developer ("make it work") and QA ("try to
break it").

If you're using ``-fdiagnostics-show-caret`` to capture the range
information, you can use dg-{begin|end}-multiline-output to express
testing for this output.  For example:

.. code-block:: c

  /* { dg-options "-fdiagnostics-show-caret" } */
  void test (void)
  {
    enum { c1 = "not int", /* { dg-error "14: enumerator value for .c1. is not an integer constant" } */
    /* { dg-begin-multiline-output "" }
     enum { c1 = "not int",
                 ^~~~~~~~~
       { dg-end-multiline-output "" } */

        c_end };
  }

It's preferable to consolidate multiple related tests into one source
file, where practical, to minimize the per-source-file overhead when
running the test suite.


Step 7: bootstrap & regression test
-----------------------------------

Assuming you have a patch that works, follow the usual GCC testing
steps: https://gcc.gnu.org/contribute.html to ensure it doesn't break anything.

FWIW I have a script for building both unpatched ("control") and
patched ("experiment") versions of gcc, bootstrapping and running the
regression-test suite, and comparing the results: https://github.com/davidmalcolm/gcc-build

Write a ChangeLog.  I do this whilst it's bootstrapping, using a script
here: https://github.com/davidmalcolm/gcc-refactoring-scripts/blob/master/generate-changelog.py
which makes it easier to generate a ChangeLog (by parsing the output of "git show").


Step 8: post to gcc-patches
---------------------------

FWIW, my patch for the issue described above is here:
  https://gcc.gnu.org/ml/gcc-patches/2016-06/msg01658.html


Step 9: respond to patch review
-------------------------------


Step 10: push to git
--------------------

Once the patch is approved, push it to master.

