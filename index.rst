.. GCC Contributors Guide documentation master file, created by
   sphinx-quickstart on Thu Jul 21 17:39:25 2016.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

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
