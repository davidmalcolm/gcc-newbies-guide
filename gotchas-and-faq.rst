"Gotchas" and FAQs
------------------

Why do the source files have a .c extension?
********************************************

As of version 4.8, GCC is (mostly) implemented in C++, but we haven't
renamed the source files.  Hence you will see source files with a ``.c``
extension throughout the source tree.  These are generally handled by
the build system as C++, rather than C.


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

how to search for them? find . -name "*.h" | xargs grep cp_token


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
