Aliases and Scripts
===================

If you are going to be doing a lot of development on GCC, it can become a huge
hassle to type, and remember, the same complicated commands over and over again.
These aliases and scripts are designed to save you a lot of time, and headaches.

Aliases
-------

Aliases are like custom commands that act as "shortcuts". These "shortcuts" are
used to represent a command, or set of commands. If you use Ubuntu, chances are
you are already using some aliases, as Ubuntu defaultly defines a few. If you
would like to see a list of aliases you might have, type ``alias`` into your 
terminal.

Aliases live inside the bashrc (typically at ``~/.bashrc``), so will have to 
edit the file to gain our nice custom shortcuts! 
I will be going over each alias and describing what it does.

.. note::

   I will assume that your source and build
   directory paths are as follows: ``~/src/gcc/``, and ``~/bld/trunk/``


.. code-block:: bash

   alias gt='cd ~/src/gcc/gcc' # source files
   alias gtls='cd ~/src/gcc/libstdc++-v3' # C++ standard
   alias gtt='cd ~/src/gcc/gcc/testsuite/' # testsuite
   alias gtu='cd ~/svn/trunk/; svn up' # svn directory for commiting
   alias gb='cd ~/bld/trunk' # build directory
   alias gbg='cd ~/bld/trunk/gcc' # where the compiler lives

These are all pretty straight forward. They are all shortened "cd" style
commands.

* ``gt`` will bring you to where most of the source files for GCC's 
  front end is.
* ``gtls`` brings you to where the GNU Standard C++ Library V3
  lives.
* ``gtt`` brings you to the testsuite.
* ``gtu`` is a little more interesting. Once you have write access to GCC you 
  will make a directory for the svn repository. That is where you would commit 
  your patches. So what this alias does is, it cds into the directory,
  updates it.
* ``gb`` brings you to the build directory.
* ``gbg`` brings you to where all the object files live. More importantly, it
  brings you to where you where the compiler (``cc1``, and ``cc1plus``) lives.

.. code-block:: bash

   alias mg='make CXXFLAGS=-g3'
   export MAKEFLAGS='-j4'

This is just to make the building process go faster. It saves a lot of time.

.. code-block:: bash

   alias gdba='gdb -q --args'

``gdba`` is my quick way of running the debugger. You will be debugging the
compiler a lot, so, it saves a lot of time to not have to write that 
command every singe time. ``-q`` is so it ruins quietly. GDB would usually
print annoying messages everytime you run it, having ``q`` enabled, turns that
off. ``--args`` is what lets you debug inside of the compiler. With that, you
can now make breakpoints and step through the compiler as you would a normal
program.

.. code-block:: bash

   dgxx ()
   {
       ( cd ~/bld/trunk/gcc;
       GXX_TESTSUITE_STDS=98,11,14,17,2a make check-c++ ${1:+RUNTESTFLAGS="$*"} )
   }
   
   dg ()
   {
        ( cd ~/bld/trunk/gcc;
	make check-c ${1:+RUNTESTFLAGS="$*"} )
   }

These two are for running tests. ``dgxx`` is used for C++ tests and 
``dg`` is used for C tests. To run these you can either run them alone to
run all tests, or you can run it with an extra argument to run one specific
test. For example:

.. code-block:: console

   dgxx dg.exp="name-of-test"

.. code-block:: bash

   gcc_configure ()
   {
     ~/src/gcc/configure
   }

This last one is used to configure gcc inside of a build directory. This can be
heavily customized as everyone will have different configuration options. For
example, mine looks like this:

.. code-block:: bash

   gcc_configure ()
   {
     ~/src/gcc/configure --enable-languages=c,c++ 
     --enable-checking=yes -with-system-zlib --disable-bootstrap 
     --disable-libvtv --disable-libcilkrts --disable-libitm --disable-libgomp 
     --disable-libcc1 --disable-libstdcxx-pch --disable-libssp --disable-isl 
     --disable-libmpx --disable-libsanitizer --disable-libquadmath 
     --disable-libatomic
   }

You can find the configuration options `here`_.

.. _here: https://gcc.gnu.org/install/configure.html

.. code-block:: console

   alias 'cc1'='./cc1 -quiet' # lazy C compile
   alias 'cc1p'='./cc1plus -quiet' # lazy C++ compile

These last two are more for convience than anything. I hated having to type out
the command everytime I compiled something (which was a lot), so I decided to
shorten it.

Scripts
-------

The first thing you are going to want to do is make a bin directory in your
home directory (``~/bin/``). This is so your $PATH can catch it so you wont have
to type out each scripts path everytime you run one. The way all of these work
is by typing the scripts name and a string you want to search.

.. note::

   Once again, I am going to assume your source and build directorys' paths are
   as follows: ``~/src/gcc/``, and ``~/bld/trunk``

Searching scripts
^^^^^^^^^^^^^^^^^

**gcf**

.. code-block:: bash

   #!/bin/bash
   cd ~/src/gcc/gcc
   grep -nE --color=tty "$@" c/*.{c,h} c-family/*.{c,cc,h,def,opt}

This one searches inside of the C/C++ front end files.

**gcx**

.. code-block:: bash

   #!/bin/bash
   cd ~/src/gcc/gcc
   grep -nE --color=tty "$@" cp/*.{c,cc,h,def} c-family/*.{c,cc,h,def,opt}

This one searches inside the gcc/cp/ and gcc/c-family/ directories.

**gf**

.. code-block:: bash

   #!/bin/bash
   cd ~/src/gcc/gcc
   grep -nE --color=tty "^""$@" *.{c,h,def,opt,pd} doc/*.texi* c/*.{c,h} c-family/*.{c,h,def,opt} cp/*.{c,h,def} ginclude/*.h config/i386/*.{c,h,def,opt}

This one searches for function declarations.

**gg**

.. code-block:: bash

   #!/bin/bash
   cd ~/src/gcc/gcc
   grep -nE --color=tty "$@" *.{c,cc,h,def,opt,pd} doc/*.texi* c/*.{c,h} c-family/*.{c,cc,h,def,opt} cp/*.{c,h,def,cc} ginclude/*.h config/i386/*.{c,h,def,opt}
   #grep -nE --color=tty $@ *.{c,h,def,opt} doc/*.texi* c/*.{c,h} {c-family,java,fortran}/*.{c,h,def,opt} {cp,objc,objc}/*.{c,h,def} lto/*.{c,h,opt} go/*.{c,cc,h} ada/*.{adb,ads} ada/gcc-interface/*.{c,h,def,opt} config/*.{c,h,def,opt} config/*/*.{c,h,md,def,opt} common/config/*/*.c
   #grep -nEr --exclude-dir=".svn" --color=tty $@ ../libgcc/*
   #grep -nEr --exclude-dir=".svn" --color=tty $@ ../libcpp/*

This one searches the entire compiler.

**ggh**

.. code-block:: bash

   #!/bin/bash
   cd ~/src/gcc/gcc
   grep -nE --color=tty "$@" *.{h,def,opt} doc/*.texi* c/*.h c-family/*.{h,def,opt} cp/*.{h,def} ginclude/*.h

Compiling scripts
^^^^^^^^^^^^^^^^^

These are really only useful if you have set up older versions of GCC on your
system. The paths I use within the scripts assume you have set up the older
versions in the way of step 4 in "How to Build GCC from Source".

**xg++-7**

.. code-block:: bash

   #!/bin/sh
   ~/bld/gcc7/gcc/xg++ -B ~/bld/gcc7/gcc/ -B ~/bld/gcc7/x86_64-pc-linux-gnu/libstdc++-v3/bld/.libs/ `~/bld/gcc7/x86_64-pc-linux-gnu/libstdc++-v3/scripts/testsuite_flags --build-includes` -B ~/bld/gcc7/x86_64-pc-linux-gnu/libsanitizer/ubsan/.libs/ -Wl,-rpath=~/bld/gcc7/x86_64-pc-linux-gnu/libsanitizer/ubsan/.libs/ "$@"

**xg++-8**

.. code-block:: bash

   #!/bin/sh
   ~/bld/gcc8/gcc/xg++ -B ~/bld/gcc8/gcc/ -B ~/bld/gcc8/x86_64-pc-linux-gnu/libstdc++-v3/bld/.libs/ `~/bld/gcc8/x86_64-pc-linux-gnu/libstdc++-v3/scripts/testsuite_flags --build-includes` -B ~/bld/gcc8/x86_64-pc-linux-gnu/libsanitizer/ubsan/.libs/ -Wl,-rpath=/home/mbelivea/bld/gcc8/x86_64-pc-linux-gnu/libsanitizer/ubsan/.libs/ "$@"

**xg++-9**

.. code-block:: bash

   #!/bin/sh
   ~/bld/gcc9/gcc/xg++ -B ~/bld/gcc9/gcc/ -B ~/bld/gcc9/x86_64-pc-linux-gnu/libstdc++-v3/bld/.libs/ `~/bld/gcc9/x86_64-pc-linux-gnu/libstdc++-v3/scripts/testsuite_flags --build-includes` -B ~/bld/gcc9/x86_64-pc-linux-gnu/libsanitizer/ubsan/.libs/ -Wl,-rpath=/home/mbelivea/bld/gcc9/x86_64-pc-linux-gnu/libsanitizer/ubsan/.libs/ "$@"

Other Scripts
^^^^^^^^^^^^^

This last script is one of the most used scripts I have used while developing
for GCC. This patch generates a script by taking the git diff, and checking its
contents with a style guide that is built into ``~/src/gcc/contrib``.

**do_patch**

.. code-block:: bash

   #!/bin/sh
   P=$1.patch
   if test -e $P; then
   echo "Patch already exists, bailing out."
   exit 1
   fi
   git diff  > $P
   ~/src/gcc/contrib/mklog /tmp/$P | cat - /tmp/$P > $P
   pr=`echo $P | sed 's/pr\([0-9]\+\).*/\1/'`
   if test -n "$pr"; then
     sed -i "3i\\\tPR c++/$pr" $P
   fi
   sed -i "s/\t\*\ gcc\//\t\*\ /" $P
   sed -i '1i Bootstrapped/regtested on x86_64-linux, ok for trunk?\n' $P
   echo -e "Wrote $P\nDon't forget to remove c/, cp/, c-family/, testsuite/ prefixes."
   ~/src/gcc/contrib/check_GNU_style.sh $P
   $EDITOR $P

The way to run this script is as follows: ``do_patch pr55555555`` then it will
generate a patch called ``pr55555555.patch``.
