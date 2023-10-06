.. Copyright (C) 2022-2023 Free Software Foundation, Inc.
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

Getting Started
===============

"Hello world" from the compiler
*******************************

Challenge:

* add a diagnostic that says "hello world" and emits the function name
  for each function that is compiled in the source file.

* run the compiler under the debugger, and put a breakpoint on the
  emission of that diagnostic, and single-step through some of the
  compilation

To do this you'll need to:

* build GCC from source

* add a call to::

    inform (UNKNOWN_LOCATION, "hello world: %qE", cfun->decl);

  somewhere inside the compiler.

.. note::
  ``cfun`` is a global pointer referencing the **c**\urrent **fun**\ction being compiled.


Setting up your build environment
*********************************

GCC is a big project, so make sure you have plenty of disk space.

As of May 2022, on my machine:

* a ``git`` clone of the source tree occupies 2.5GB of disk space.

* a "build" tree occupies 6.8GB

for a total of about 9.5GB.

It typically takes 1.5 to 2 hours on my machine to do a full build
and test run when testing a patch - but clearly that's going to be
impractical when hacking on a new feature, or fixing a bug: you want
to be able to make a change to a source file and then debug that
change as quickly as possible.

By default, a build of GCC involves building the source *three times*:
once using the "bootstrapping" compiler, then rebuilding using the GCC
that was just built, then rebuilding a third time, with the second GCC
(built by the first GCC, built by the bootstrapping compiler).  This is
a great safety check that we have a working compiler when testing a
patch, but obviously is a major pain when working on the patch itself.

Hence one important configuration flag is ``--disable-bootstrap``,
which turns off this three-stage bootstrap in favor of simply building
GCC once.

So the goals of "easy hackability" and "robust testing" are somewhat
in tension.  My own way of dealing with this is to spend a lot of
disk space by having multiple working copies, some for working
on patches, and others for *testing* those patches.  I typically have
three kinds of working copy:

(a) a working copy and build directory pair for debugging/development.
    I use ``--disable-bootstrap`` when configuring the build, so that
    I can quickly rebuild the compiler when making changes.  If I
    change one source file in the ``SOURCE/gcc`` subdirectory and
    invoke ``make -jN`` in the ``BUILDDIR/gcc`` subdirectory it typically
    takes about 20 seconds total to rebuild the ``.o`` file, relink the
    compiler and run the ``-fself-test`` unit tests (the bulk of
    the time is spent in the linker).  Without the shortcuts of using
    ``--disable-bootstrap`` and restricting the rebuild to the
    ``BUILDDIR/gcc`` subdirectory the build would take *much* longer
    (measured in minutes or tens of minutes).

(b) a clean working copy/build directory pair for use as a "control"
    when testing patches.  We have many thousands of tests in our
    test suite, and so, unfortunately, there are usually at least
    several of them failing at any given time, so it's helpful when
    testing a patch to have something to compare the test results
    against.  I configure this build with all languages enabled, and
    *without* ``--disable-bootstrap``, so it does a full bootstrap.

(c) a testing source/build directory pair for testing patches, which
    is identical to the "control" pair, except for the patches being
    tested.

Separating out the "control" and "testing" directories from the
debugging/development directories helps isolate the testing, and
means that I can work on something else in the 2 hours it takes to
properly test a candidate patch - but means I'm using about 30GB of
disk space, rather than 10GB (and I have to refresh my "control"
builds when I do a rebase of my development trees).

Whilst you are getting started, I recommend simply creating (a),
using ``--disable-bootstrap`` when configuring the build, and perhaps
only enabling the languages that you care about (to make rebuilds
faster).  As you progress to preparing real patches for GCC, you'll
want to add (b) and (c).


How I'm doing it on x86_64-pc-linux-gnu
#######################################

Following https://gcc.gnu.org/install/, here is how I'd install
a working *development* version to work on the analyzer.

.. code-block:: bash

  # Install prerequisites - to be done once
  sudo apt install perl gawk binutils gcc-multilib \
  python3 python3-pip gzip make tar zstd autoconf automake \
  gettext gperf dejagnu autogen guile-3.0 expect tcl flex texinfo \
  git diffutils patch git-email

  # clone GCC sources
  mkdir -p gcc-control
  cd gcc-control
  git clone git://gcc.gnu.org/git/gcc.git sources
  # install the prerequisites using GCC contrib/ script
  cd gcc-control
  ./sources/contrib/download_prerequisites
  mkdir build && cd build
  # here you would add --disable-bootstrap for your 'draft' directory
  ../gcc/configure --prefix "/path/to/gcc-control/install"
  # I have 16 cores on my box, so I use -j16 to parallelize as much as possible
  # target 'bootstrap' for a bootstrap build. Otherwise omit it.
  make -j16 bootstrap
