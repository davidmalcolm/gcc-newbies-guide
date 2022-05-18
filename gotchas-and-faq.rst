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


Is "ggc" a typo?
****************

"ggc" (as opposed to "gcc") refers to :ref:`GCC's garbage collector <ggc>`.
