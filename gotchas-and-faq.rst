"Gotchas" and FAQs
------------------

What language is GCC implemented in?
************************************

The short answer: C++98, with some conservative use of C++11.

Longer answer:

GCC is an old project, originally implemented in C (in the late 1980s).

In GCC 4.8 (released in 2013) we migrated our implementation language to
C++.  Although we implement the most recent updates to the C++ language,
for our own implementation we restrict ourselves to a fairly conservative
subset of C++.  This is because GCC is so often used for bootstrapping new
OS and CPU configurations, so we want to make it easy to build GCC itself
using older compilers that might themselves be buggy.

Addititionally, we have some tools which scan the source tree during the
build process, so we must use the subset of C++ that those tools can
handle, in particular, ``gengtype`` (see
:ref:`GCC's garbage collector <ggc>`) - or extend those tools.

We stuck to C++98 from 2013 through 2021, and as of GCC 11 in 2021 started
allowing some C++11 features (such as ``auto`` and range-based ``for`` loops).

In our C++98 era we did allow some support of C++11 features via macros
in ``include/ansidecl.h``, which expand as follows where the underlying
compiler supports it:

============= =============
Macro         Expansion
============= =============
``OVERRIDE``  ``override``
``FINAL``     ``final``
``CONSTEXPR`` ``constexpr``
============= =============

and expanded to nothing for C++98 compilers, so you'll see these in the
source tree.  Doing so allows for catching various problems earlier, and
for better documenting the intent of the code.  Now that C++11 is
required, it makes sense to simply use the lower-case spelling of these.

The official notes on how we use C++ as an implementation language are
at https://gcc.gnu.org/codingconventions.html#Cxx_Conventions


I tried grepping for `struct foo` but can't find it!
****************************************************

It's often difficult to grep for the declaration of, say, ``struct foo``
in the GCC sources due to the presence of "GTY" markers.  For example,
in the C++ frontend, token are handled using ``struct cp_token``, which
is defined as:

.. code-block:: c++

  /* A C++ token.  */

  struct GTY (()) cp_token {
     /* ...fields go here... */
  };

These "GTY" markers are annotations to the types, and are used by GCC's
garbage-collector.  They're stripped away by the preprocessor when building
GCC itself, but get used by a tool during the build called ``gengtype``.

TODO: how to search for them?


How do I debug GCC?
*******************

The ``gcc`` binary is actually a relatively small "driver" program, which
parses some command-line options, and then invokes one or more other
programs to do the real work.  See the notes in :ref:`debugging` for
an explanation of how to debug.


What does "PR" mean, e.g. "PR c/71610"?
***************************************

We use an instance of Bugzilla as our bug tracker.  We refer to bugs
as "Problem Reports", or "PR" for short.  For example, the bug with
ID 71610 affects the "c" component, and might be referred to as
"PR c/71610" in discussions on our mailing lists, and in ChangeLog
entries.

To see the bug report in Bugzilla, go to
https://gcc.gnu.org/PRnnnnn

For example, for PR c/71610, see
https://gcc.gnu.org/PR71610

For more information on the bug tracker and filing bugs, see
https://gcc.gnu.org/bugs/

I want to try my hands on a bug, but cannot assign it to myself, why? 
*********************************************************************

The best way to get familiar with GCC is to find yourself an easy bug
on our Bugzilla you believe you can fix. It will get you more familiar
with the code, debugging GCC, running the testsuite and the whole "patching"
process - while already contributing to GCC. Really all pros no cons ;-).

So here you are, you have found your bug either by requesting one on the
mail list gcc@gcc.gnu.org, on the IRC channel irc://irc.oftc.net/#gcc, or
by skimming through the `Bugzilla`_. Thus you want to assign it to yourself.
However, the *Assignee* field is uneditable, **even though you are connected**.

The reason is simple, using any 'lambda' account created on our Bugzilla 
only provides a limited access. To get access to the whole lot,
you will need to log in using a sourceware.org / gcc.gnu.org email
address, that you can get by following the steps described here
`<https://sourceware.org/cgi-bin/pdw/ps_form.cgi>`_.

.. list-table::
  :header-rows: 1

  * - With 'lambda' account
    - With 'gcc.gnu.org' account
  * - .. image:: images/grayed-out-assignee.bugzilla.png
    - .. image:: images/sourceware-login-assignee.bugzilla.png

Is "ggc" a typo?
****************

"ggc" (as opposed to "gcc") refers to :ref:`GCC's garbage collector <ggc>`.


Compiler basics
***************

If you're interested in GCC's internals, it's probably worth at least
skimming an introductory course on compilers in general.  Some helpful
terms follow:

"build" vs "host" vs "target"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For a compiler, we can talk about:

* the "build" system: the system the compiler is built on

* the "host" system: the system the compiler runs on

* the "target" system: the system that the compiler is generating code
  for

By way of example, many years ago in a previous job, I wrote videogames,
using an older version of gcc.  If I'm remembering things correctly,
this gcc had been built on a Linux box by another company, ran on my
Windows 95 desktop machine, and generated code for a particular games
console that was popular at the time.  In this example:

* the "build" system was whatever Linux system the toolchain provider
  used when building the gcc that got shipped with the console
  development kit

* the "host" system was my desktop PC, a 32-bit i386 system running
  Windows 95

* the "target" system was a games console with a MIPS R3000 CPU (a kind
  of RISC chip), with no real operating system to speak of

Often all three will be the same: when I'm developing GCC I typically
build and run gcc on my x86_64 Fedora box, and it builds binaries for
the same.  We speak of "cross compilation" when the host and target are
different systems.

.. _Bugzilla: https://gcc.gnu.org/bugzilla