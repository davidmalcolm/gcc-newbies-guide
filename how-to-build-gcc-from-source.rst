How to Build GCC from Source
============================

Step 1: Install Dependancies
----------------------------

.. code-block:: console

   sudo dnf install gcc-c++ \
	libmpc-devel \
	mpfr-devel \
	gmp-devel \
	bison \
	flex \
	dejagnu \
	zlib-devel \
	glibc-devel.i686

Step 2: Set up Directories
--------------------------

Development for GCC is split into two different directories: A source, and 
a build. The source directory is where all the source files will go. This is
where you will clone the git repo, and where you will be changing files for 
development. The build directory is where you are going to configure and
build gcc. This directory contains all of the object files. After every
edit to the source directory, you will have to recompile the object files.

.. code-block:: console

   mkdir -p ~/bld/trunk && mkdir ~/src/

Step 2a: Clone Git Repo into Source Directory
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Make a build and source directory. Here I will make it in my home directory;
however, you can make the directories where you choose.

.. code-block:: console

   cd ~/src/ && git clone git://gcc.gnu.org/git/gcc.git

Then you will want to checkout branch ``trunk`` or ``master``

.. code-block:: console

   git checkout master

.. note::

   If your git username and email are not the same as the one you use to
   contribute to GCC (i.e. work email), it's helpful to set it to the correct
   information inside the git directory.

   .. code-block:: console

      git config user.name "User Name"
      git config user.email "email@address.com"

   This is useful for when you are generating patches later, because the script
   grabs your git information. This means you will not have to change it every
   time you make a new patch.

Step 2b:  Set up Build Directory Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

GCC has a script called ``configure`` inside the source directory at
``~/gcc/configure``. Calling by itself will enable every option by default.
If you want a different configuration you will have to use option flags to
change the configuration settings. For example, I mostly work on front end C and
C++ stuff, so my configuration would look like this:

.. code-block:: console

    ~/src/gcc/configure --enable-languages=c,c++ \
    --enable-checking=yes -with-system-zlib --disable-bootstrap \
    --disable-libvtv --disable-libcilkrts --disable-libitm --disable-libgomp \
    --disable-libcc1 --disable-libstdcxx-pch --disable-libssp --disable-isl \
    --disable-libmpx --disable-libsanitizer --disable-libquadmath \
    --disable-libatomic

You can find the configuration options `here`_.

.. _here: https://gcc.gnu.org/install/configure.html

Next you will have to make the directory:

.. code-block:: console

   make CXXFLAGS=-g3

The CXXFLAGS option will make the process go much faster.

.. note::

   If you make a change to the source directory, you wont have to build the
   entirety of GCC again! For example, if you make a change to the front end
   (``~/src/gcc/gcc``), all you have to do is run the command above in
   ``~/bld/trunk/gcc``. This will save you a load of time!

Step 3: Compile a test program with the newly built GCC to confirm it works
---------------------------------------------------------------------------

Take this test code ``hello.c``:

.. literalinclude:: hello.c
   :language: c

From inside the build directory ``~/bld/trunk/gcc``, we will run the command:

.. code-block:: console

   ./cc1plus -quiet -Iinclude hello.c

The ``-quiet`` option makes it so the compiler doesn't print out annoying
messages to console when compiling. The ``-Iinclude`` option lets you
compile code that contains ``#include``'s when using
either ``cc1`` or ``cc1plus``.

Step 4 (Optional): Set up Older Versions of GCC
-----------------------------------------------

Many times you will find a bug that worked in one version and doesn't work in
another, sometimes you will accept a bug report that is in GCC 9 or GCC 8. In
these cases, it's handy to already have a working source and build directory for
this version. What I like to do is have working directories for ``trunk``,
``gcc-9``, ``gcc-8``, and ``gcc-7``.

To set these directories up, all you have to do is copy your working ``trunk``
source directory into a seperate one and check out ``gcc-7/8/9-branch``
inside those directories.  Then follow the steps above to configure the build
directory, but changing the path of the script to fit the correct version. For
example: ``~/src/gcc9/configure`` instead of ``~/src/gcc/configure``.
