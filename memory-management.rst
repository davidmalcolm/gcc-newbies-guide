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

Memory Management in GCC
========================

We have a few different memory management strategies in GCC:

  * RAII types, which manage their own memory.  An example
    is ``auto_vec``.

  * Heap allocation with manual deallocation, where you have
    to use the correct deallocator matching the allocator.
    (Sadly it's not always clear when looking at a pointer
    which allocator to use; it would be good to use wrapper
    classes to automate this).

    * ``new`` and ``delete``

    * ``new[]`` and ``delete[]``

    * ``malloc`` and ``free``

  .. _ggc:

  * A garbage-collected heap: "ggc" (as opposed to "gcc"),
    with automatic deallocation.

    Data is allocated, and, when memory usage is high, a
    mark-and-sweep collector frees up no-longer used data.

    Various pointers and types in the source tree are tagged
    using the ``GTY`` preprocessor macro.  This code is
    processed at build time using a custom tool, ``gengtype``.
    It locates garbage-collector "roots": global variables
    that can reference GC-allocated data - the roots of the
    mark-and-sweep operation.

    This pointer graph metadata is also used by our implementation
    of pre-compiled headers: a precompiled header is essentially
    just a snapshot of the GC-heap, which can be loaded back
    into memory, and the metadata is used to update all of
    the pointer cross-references within it.

    This means that anything that needs to be preserved in
    pre-compiled headers needs to be GC-allocated.

    Sadly, ``gengtype`` only supports a loosely-defined subset
    of C++ and can be fiddly to work with.
    Documentation can be seen at
    https://gcc.gnu.org/onlinedocs/gccint/Type-Information.html

  * obstacks: obstacks support a form of dynamic allocation
    in which various blocks of memory are allocated, and
    then are all released together.  This is effectively a
    stack and thus fast, and only the "watermark" of the release
    point needs to be tracked.   We use them a lot within
    optimization passes for allocating temporary data, releasing
    it in one go when we're done with a function.

  * :file:`alloc-pool.h` provides an optimized way of allocating
    chunks of known size

Running gcc under valgrind
**************************
See the notes on :ref:`invoking the compiler under valgrind <valgrind>`.
