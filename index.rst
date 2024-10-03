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

GCC for New Contributors
========================

This is an unofficial guide to GCC's internals, aimed at new developers,
and at plugin authors.

Source: https://github.com/davidmalcolm/gcc-newbies-guide

I'm a relative newcomer to GCC, so I thought it was worth documenting
some of the hurdles I ran into when I started working on GCC, to try
to make it easier for others to start hacking on GCC.  Hence this guide.

Other sources of information:

- The official "Contributing to GCC" guide on the GCC website:
  https://gcc.gnu.org/contribute.html

- The official GCC "internals" guide is:
  https://gcc.gnu.org/onlinedocs/gccint/

- Another excellent resource is the material available at
  the website of the GCC Resource Center at IIT Bombay:
  https://www.cse.iitb.ac.in/grc/

- There is also much information on the GCC wiki:
  https://gcc.gnu.org/wiki/HomePage
  though sadly much of it is out-of-date -
  GCC is (at the time of writing) a 30-year-old project.

Contents:

.. toctree::
   :maxdepth: 1

   diving-into-gcc-internals.rst
   gotchas-and-faq.rst
   getting-started.rst
   readying-a-patch.rst
   debugging.rst
   working-with-the-testsuite.rst
   how-to-improve-the-location-of-a-diagnostic.rst
   memory-management.rst
   todo.rst
