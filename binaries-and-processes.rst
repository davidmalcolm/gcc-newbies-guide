Binaries and processes
----------------------

Given this trivial C program as :file:`hello.c`::

  #include <stdio.h>
  int main ()
  {
    printf ("hello world\n");
    return 0;
  }

what happens when you compile it with GCC?

If we run this command line:

.. code-block:: sh

  # compile hello.c to an executable named "hello"
  $ gcc hello.c -o hello

we end up with an executable:

.. code-block:: sh

  $ ls
  hello  hello.c

which we can run:

.. code-block:: sh

  $ ./hello
  hello world

but a lot has happened "under the hood".

The :program:`gcc` binary is actually just a relatively small "driver"
program.  It looks at what inputs you have provided, and what output you
have asked for, and tries to figure out what to do.

In this case we have a C source file as input (:file:`hello.c`), and
an executable as output.

The :program:`gcc` driver "knows" that it can compile :file:`.c` files
to assembler by invoking GCC's C compiler, which is a binary called
:file:`cc1`.  This will take :file:`hello.c` as input, and generate
an assembler file with a :file:`.s` suffix.  The driver will then
invoke the assembler on that :file:`.s` file, generating an object file.
This is a binary file format, containing machine code and metadata,
typically in a file format called ELF.  However, this :file:`.o` can't
be run directly; it needs to go through the linker, which will generate
the final executable that can be run (also an ELF file).

In diagram form::

           cc1          as          linker
  hello.c -----> tmp.s ----> tmp.o --------> "hello" executable

We can investigate by adding the :option:`-v` option, which makes :program:`gcc`
print copious amounts of information to stderr.

Here's the output when I run it (your output may look a little
different):

.. code-block::

  $ gcc hello.c -o hello -v
  Using built-in specs.
  COLLECT_GCC=gcc
  COLLECT_LTO_WRAPPER=/usr/libexec/gcc/x86_64-redhat-linux/10/lto-wrapper
  OFFLOAD_TARGET_NAMES=nvptx-none
  OFFLOAD_TARGET_DEFAULT=1
  Target: x86_64-redhat-linux
  Configured with: ../configure --enable-bootstrap --enable-languages=c,c++,fortran,objc,obj-c++,ada,go,d,lto --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-shared --enable-threads=posix --enable-checking=release --enable-multilib --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-gcc-major-version-only --with-linker-hash-style=gnu --enable-plugin --enable-initfini-array --with-isl --enable-offload-targets=nvptx-none --without-cuda-driver --enable-gnu-indirect-function --enable-cet --with-tune=generic --with-arch_32=i686 --build=x86_64-redhat-linux
  Thread model: posix
  Supported LTO compression algorithms: zlib zstd
  gcc version 10.3.1 20210422 (Red Hat 10.3.1-1) (GCC)
  COLLECT_GCC_OPTIONS='-o' 'hello' '-v' '-mtune=generic' '-march=x86-64'
   /usr/libexec/gcc/x86_64-redhat-linux/10/cc1 -quiet -v hello.c -quiet -dumpbase hello.c -mtune=generic -march=x86-64 -auxbase hello -version -o /tmp/cckqYOSJ.s
  GNU C17 (GCC) version 10.3.1 20210422 (Red Hat 10.3.1-1) (x86_64-redhat-linux)
  	compiled by GNU C version 10.3.1 20210422 (Red Hat 10.3.1-1), GMP version 6.2.0, MPFR version 4.1.0-p9, MPC version 1.1.0, isl version isl-0.16.1-GMP
  
  GGC heuristics: --param ggc-min-expand=100 --param ggc-min-heapsize=131072
  ignoring nonexistent directory "/usr/lib/gcc/x86_64-redhat-linux/10/include-fixed"
  ignoring nonexistent directory "/usr/lib/gcc/x86_64-redhat-linux/10/../../../../x86_64-redhat-linux/include"
  #include "..." search starts here:
  #include <...> search starts here:
   /usr/lib/gcc/x86_64-redhat-linux/10/include
   /usr/local/include
   /usr/include
  End of search list.
  GNU C17 (GCC) version 10.3.1 20210422 (Red Hat 10.3.1-1) (x86_64-redhat-linux)
  	compiled by GNU C version 10.3.1 20210422 (Red Hat 10.3.1-1), GMP version 6.2.0, MPFR version 4.1.0-p9, MPC version 1.1.0, isl version isl-0.16.1-GMP
  
  GGC heuristics: --param ggc-min-expand=100 --param ggc-min-heapsize=131072
  Compiler executable checksum: b30993865ac347030daf9d56a2db69cd
  COLLECT_GCC_OPTIONS='-o' 'hello' '-v' '-mtune=generic' '-march=x86-64'
   as -v --64 -o /tmp/ccXNu6jN.o /tmp/cckqYOSJ.s
  GNU assembler version 2.35 (x86_64-redhat-linux) using BFD version version 2.35-18.fc33
  COMPILER_PATH=/usr/libexec/gcc/x86_64-redhat-linux/10/:/usr/libexec/gcc/x86_64-redhat-linux/10/:/usr/libexec/gcc/x86_64-redhat-linux/:/usr/lib/gcc/x86_64-redhat-linux/10/:/usr/lib/gcc/x86_64-redhat-linux/
  LIBRARY_PATH=/usr/lib/gcc/x86_64-redhat-linux/10/:/usr/lib/gcc/x86_64-redhat-linux/10/../../../../lib64/:/lib/../lib64/:/usr/lib/../lib64/:/usr/lib/gcc/x86_64-redhat-linux/10/../../../:/lib/:/usr/lib/
  COLLECT_GCC_OPTIONS='-o' 'hello' '-v' '-mtune=generic' '-march=x86-64'
   /usr/libexec/gcc/x86_64-redhat-linux/10/collect2 -plugin /usr/libexec/gcc/x86_64-redhat-linux/10/liblto_plugin.so -plugin-opt=/usr/libexec/gcc/x86_64-redhat-linux/10/lto-wrapper -plugin-opt=-fresolution=/tmp/cchv3sYK.res -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_s -plugin-opt=-pass-through=-lc -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_s --build-id --no-add-needed --eh-frame-hdr --hash-style=gnu -m elf_x86_64 -dynamic-linker /lib64/ld-linux-x86-64.so.2 -o hello /usr/lib/gcc/x86_64-redhat-linux/10/../../../../lib64/crt1.o /usr/lib/gcc/x86_64-redhat-linux/10/../../../../lib64/crti.o /usr/lib/gcc/x86_64-redhat-linux/10/crtbegin.o -L/usr/lib/gcc/x86_64-redhat-linux/10 -L/usr/lib/gcc/x86_64-redhat-linux/10/../../../../lib64 -L/lib/../lib64 -L/usr/lib/../lib64 -L/usr/lib/gcc/x86_64-redhat-linux/10/../../.. /tmp/ccXNu6jN.o -lgcc --push-state --as-needed -lgcc_s --pop-state -lc -lgcc --push-state --as-needed -lgcc_s --pop-state /usr/lib/gcc/x86_64-redhat-linux/10/crtend.o /usr/lib/gcc/x86_64-redhat-linux/10/../../../../lib64/crtn.o
  COLLECT_GCC_OPTIONS='-o' 'hello' '-v' '-mtune=generic' '-march=x86-64'

That's a lot of text.  Let's break it down a bit to see what's going on.

Immediately below the command-line I typed, :program:`gcc` has emitted
some version information about itself, and how it has been configured:

.. code-block::

  $ gcc hello.c -o hello -v
  Using built-in specs.
  COLLECT_GCC=gcc
  COLLECT_LTO_WRAPPER=/usr/libexec/gcc/x86_64-redhat-linux/10/lto-wrapper
  OFFLOAD_TARGET_NAMES=nvptx-none
  OFFLOAD_TARGET_DEFAULT=1
  Target: x86_64-redhat-linux
  Configured with: ../configure --enable-bootstrap --enable-languages=c,c++,fortran,objc,obj-c++,ada,go,d,lto --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-shared --enable-threads=posix --enable-checking=release --enable-multilib --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-gcc-major-version-only --with-linker-hash-style=gnu --enable-plugin --enable-initfini-array --with-isl --enable-offload-targets=nvptx-none --without-cuda-driver --enable-gnu-indirect-function --enable-cet --with-tune=generic --with-arch_32=i686 --build=x86_64-redhat-linux
  Thread model: posix
  Supported LTO compression algorithms: zlib zstd
  gcc version 10.3.1 20210422 (Red Hat 10.3.1-1) (GCC)
  COLLECT_GCC_OPTIONS='-o' 'hello' '-v' '-mtune=generic' '-march=x86-64'

Next is the :program:`cc1` invocation:

.. code-block:: sh

   /usr/libexec/gcc/x86_64-redhat-linux/10/cc1 -quiet -v hello.c -quiet -dumpbase hello.c -mtune=generic -march=x86-64 -auxbase hello -version -o /tmp/cckqYOSJ.s

That command-line is very long, so let's reformat it to make it easier
to see what's going on:

.. code-block:: sh

   /usr/libexec/gcc/x86_64-redhat-linux/10/cc1 \
     -quiet \
     -v \
     hello.c \
     -quiet \
     -dumpbase hello.c \
     -mtune=generic \
     -march=x86-64 \
     -auxbase hello \
     -version \
     -o /tmp/cckqYOSJ.s

Looking at the above options in turn, starting with the :program:`cc1` invocation:

  We can see that the :program:`cc1` binary isn't in the :envvar:`PATH`
  but is hidden away in a separate directory
  (:file:`/usr/libexec/gcc/x86_64-redhat-linux/10/`) that the
  :program:`gcc` driver knows to use.

:option:`-quiet`:

  This option was supplied by the driver twice: without it the compiler
  emits debugging messages to stderr about what it's doing, which can be
  handy when debugging it directly.

:option:`-v`:

  This is passed on from the :program:`gcc` invocation to its invocation
  of :program:`cc1`, so :program:`cc1` will, in turn, emit lots of
  verbose information to stderr about what it is doing.

:file:`hello.c`:

  tells :program:`cc1` which source file to compile.

:option:`-dumpbase hello.c`

  tells :program:`cc1` that when it creates any dump files, it should
  use `hello.c` as the base for their filenames.  It won't create any
  dump files by default, but we'll do that below.

:option:`-mtune=generic` and :option:`-march=x86-64`

   are both architecture-specific options, documented in
   https://gcc.gnu.org/onlinedocs/gcc/x86-Options.html

   They affect the kind of machine code that the compiler will generate.

   :option:`-mtune=generic` for x86 means "try to tune the performance
   of the generated code for a blend of popular x86 processors".

   :option:`-march=x86-64` for x86 means generate code for a generic x86
   CPU with 64-bit extensions

:option:`-auxbase hello`:

   was an undocumented option that later releases of GCC don't use
   anymore.

:option:`-version`:

   makes :program:`cc1` emit version information to stderr

:option:`-o /tmp/cckqYOSJ.s`:

   tells :program:`cc1` to use :file:`/tmp/cckqYOSJ.s` as the output file
   when writing the generated assembler.

   The precise temporary file will change from invocation to invocation,
   and the :program:`gcc` driver will delete its temporary files when its
   done.  If you're exploring how gcc works, or debugging, you can use
   the :option:`-save-temps` option to tell :program:`gcc` to keep these
   intermediate files around.

You might see slightly different options; you can see the full
documentation for GCC options at
https://gcc.gnu.org/onlinedocs/gcc/Invoking-GCC.html (though that can
get overwhelming).

Given that :program:`gcc` passed on the :option:`-v` option to
:program:`cc1`, the next thing on stderr is the verbose output from
:program:`cc1`.  This mainly consists of version and configuration
information:

.. code-block::

  GNU C17 (GCC) version 10.3.1 20210422 (Red Hat 10.3.1-1) (x86_64-redhat-linux)
  	compiled by GNU C version 10.3.1 20210422 (Red Hat 10.3.1-1), GMP version 6.2.0, MPFR version 4.1.0-p9, MPC version 1.1.0, isl version isl-0.16.1-GMP
  
  GGC heuristics: --param ggc-min-expand=100 --param ggc-min-heapsize=131072
  ignoring nonexistent directory "/usr/lib/gcc/x86_64-redhat-linux/10/include-fixed"
  ignoring nonexistent directory "/usr/lib/gcc/x86_64-redhat-linux/10/../../../../x86_64-redhat-linux/include"
  #include "..." search starts here:
  #include <...> search starts here:
   /usr/lib/gcc/x86_64-redhat-linux/10/include
   /usr/local/include
   /usr/include
  End of search list.
  GNU C17 (GCC) version 10.3.1 20210422 (Red Hat 10.3.1-1) (x86_64-redhat-linux)
  	compiled by GNU C version 10.3.1 20210422 (Red Hat 10.3.1-1), GMP version 6.2.0, MPFR version 4.1.0-p9, MPC version 1.1.0, isl version isl-0.16.1-GMP
  
  GGC heuristics: --param ggc-min-expand=100 --param ggc-min-heapsize=131072
  Compiler executable checksum: b30993865ac347030daf9d56a2db69cd

We'll go into more detail about what happens in :program:`gcc` below.

Next comes the invocation of :program:`as`, the assembler:

.. code-block:: sh

  COLLECT_GCC_OPTIONS='-o' 'hello' '-v' '-mtune=generic' '-march=x86-64'
   as -v --64 -o /tmp/ccXNu6jN.o /tmp/cckqYOSJ.s
  GNU assembler version 2.35 (x86_64-redhat-linux) using BFD version version 2.35-18.fc33

This is much simpler, where the :program:`gcc` driver invokes :program:`as` with:

.. code-block:: sh

   as -v --64 -o /tmp/ccXNu6jN.o /tmp/cckqYOSJ.s

essentially merely telling it the input :file:`.s` file, the output
:file:`.o` file, and a couple of options.

Next comes the linker invocation; in this case :program:`gcc` driver invokes :program:`collect2`:

.. code-block:: sh

  COMPILER_PATH=/usr/libexec/gcc/x86_64-redhat-linux/10/:/usr/libexec/gcc/x86_64-redhat-linux/10/:/usr/libexec/gcc/x86_64-redhat-linux/:/usr/lib/gcc/x86_64-redhat-linux/10/:/usr/lib/gcc/x86_64-redhat-linux/
  LIBRARY_PATH=/usr/lib/gcc/x86_64-redhat-linux/10/:/usr/lib/gcc/x86_64-redhat-linux/10/../../../../lib64/:/lib/../lib64/:/usr/lib/../lib64/:/usr/lib/gcc/x86_64-redhat-linux/10/../../../:/lib/:/usr/lib/  
  COLLECT_GCC_OPTIONS='-o' 'hello' '-v' '-mtune=generic' '-march=x86-64'
   /usr/libexec/gcc/x86_64-redhat-linux/10/collect2 -plugin /usr/libexec/gcc/x86_64-redhat-linux/10/liblto_plugin.so -plugin-opt=/usr/libexec/gcc/x86_64-redhat-linux/10/lto-wrapper -plugin-opt=-fresolution=/tmp/cchv3sYK.res -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_s -plugin-opt=-pass-through=-lc -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_s --build-id --no-add-needed --eh-frame-hdr --hash-style=gnu -m elf_x86_64 -dynamic-linker /lib64/ld-linux-x86-64.so.2 -o hello /usr/lib/gcc/x86_64-redhat-linux/10/../../../../lib64/crt1.o /usr/lib/gcc/x86_64-redhat-linux/10/../../../../lib64/crti.o /usr/lib/gcc/x86_64-redhat-linux/10/crtbegin.o -L/usr/lib/gcc/x86_64-redhat-linux/10 -L/usr/lib/gcc/x86_64-redhat-linux/10/../../../../lib64 -L/lib/../lib64 -L/usr/lib/../lib64 -L/usr/lib/gcc/x86_64-redhat-linux/10/../../.. /tmp/ccXNu6jN.o -lgcc --push-state --as-needed -lgcc_s --pop-state -lc -lgcc --push-state --as-needed -lgcc_s --pop-state /usr/lib/gcc/x86_64-redhat-linux/10/crtend.o /usr/lib/gcc/x86_64-redhat-linux/10/../../../../lib64/crtn.o
  COLLECT_GCC_OPTIONS='-o' 'hello' '-v' '-mtune=generic' '-march=x86-64'

The command the driver uses to invoke the linker is over 1000 characters
long.  We can reformat it to make it more approachable, but it's still
rather intimidating:

.. code-block:: sh

  /usr/libexec/gcc/x86_64-redhat-linux/10/collect2 \
     -plugin /usr/libexec/gcc/x86_64-redhat-linux/10/liblto_plugin.so \
       -plugin-opt=/usr/libexec/gcc/x86_64-redhat-linux/10/lto-wrapper \
       -plugin-opt=-fresolution=/tmp/cchv3sYK.res \
       -plugin-opt=-pass-through=-lgcc \
       -plugin-opt=-pass-through=-lgcc_s \
       -plugin-opt=-pass-through=-lc \
       -plugin-opt=-pass-through=-lgcc \
       -plugin-opt=-pass-through=-lgcc_s \
     --build-id \
     --no-add-needed \
     --eh-frame-hdr \
     --hash-style=gnu \
     -m elf_x86_64 \
     -dynamic-linker \
     /lib64/ld-linux-x86-64.so.2 \
     -o hello \
     /usr/lib/gcc/x86_64-redhat-linux/10/../../../../lib64/crt1.o \
     /usr/lib/gcc/x86_64-redhat-linux/10/../../../../lib64/crti.o \
     /usr/lib/gcc/x86_64-redhat-linux/10/crtbegin.o \
     -L/usr/lib/gcc/x86_64-redhat-linux/10 \
     -L/usr/lib/gcc/x86_64-redhat-linux/10/../../../../lib64 \
     -L/lib/../lib64 \
     -L/usr/lib/../lib64 \
     -L/usr/lib/gcc/x86_64-redhat-linux/10/../../.. \
     /tmp/ccXNu6jN.o \
     -lgcc \
     --push-state \
       --as-needed \
       -lgcc_s \
     --pop-state \
     -lc \
     -lgcc \
     --push-state
       --as-needed \
       -lgcc_s \
     --pop-state \
     /usr/lib/gcc/x86_64-redhat-linux/10/crtend.o \
     /usr/lib/gcc/x86_64-redhat-linux/10/../../../../lib64/crtn.o

We'll skip the details for now so that we can focus on :program:`cc1`
in the next section, but perhaps the most important options are
:option:`-o hello` specifying the output file, and
:option:`/tmp/ccXNu6jN.o` specifying the file that the assembler just
emitted, this time as an input file to the linker (containing the user's
code built from :file:`hello.c`).  It's also linking in various support
libraries needed by an executable binary.
