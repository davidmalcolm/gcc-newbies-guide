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

Looking at the generated assembler
==================================

In the previous section, we looked at how the :program:`gcc` driver
program invokes various tools to get its job done.  For me, the most
interesting part of that process is the conversion from C source code to
machine code (expressed in the form of assembler) done by the compiler,
:program:`cc1`.

We can see this part of the process more directly by invoking
:program:`gcc` with the :option:`-S` option (upper-case "S"):

.. code-block:: sh

  $ gcc -S hello.c

telling it to merely create an assembler file (with a lower-case
:file:`.s` extension), without doing the later stages we talked about
above::

           cc1
  hello.c -----> hello.s


.. code-block:: sh

  gcc -S hello.c

and in addition to the :file:`hello` executable we built earlier, we now
have a :file:`hello.s` file:

.. code-block:: sh

  $ ls
  hello  hello.c  hello.s

Looking in the :file:`hello.s` file we see the generated assembler:

.. code-block:: asm

  	.file	"hello.c"
  	.text
  	.section	.rodata
  .LC0:
  	.string	"hello world"
  	.text
  	.globl	main
  	.type	main, @function
  main:
  .LFB0:
  	.cfi_startproc
  	pushq	%rbp
  	.cfi_def_cfa_offset 16
  	.cfi_offset 6, -16
  	movq	%rsp, %rbp
  	.cfi_def_cfa_register 6
  	movl	$.LC0, %edi
  	call	puts
  	movl	$0, %eax
  	popq	%rbp
  	.cfi_def_cfa 7, 8
  	ret
  	.cfi_endproc
  .LFE0:
  	.size	main, .-main
  	.ident	"GCC: (GNU) 10.3.1 20210422 (Red Hat 10.3.1-1)"
  	.section	.note.GNU-stack,"",@progbits

It's not important yet to be able to fully understand the assembler,
but for now just know that we have some metadata at the top, followed
by the user's :c:func:`main` function, and then some trailing
metadata.

You can even see how the assembler relates to the C code by passing
:option:`-fverbose-asm` to :program:`gcc`:

.. code-block:: sh

  $ gcc -S hello.c -fverbose-asm

which leads to this in the output :file:`hello.s` file:

.. code-block:: asm

  	.file	"hello.c"
  # GNU C17 (GCC) version 10.3.1 20210422 (Red Hat 10.3.1-1) (x86_64-redhat-linux)
  #	compiled by GNU C version 10.3.1 20210422 (Red Hat 10.3.1-1), GMP version 6.2.0, MPFR version 4.1.0-p9, MPC version 1.1.0, isl version none
  # GGC heuristics: --param ggc-min-expand=100 --param ggc-min-heapsize=131072
  # options passed:  hello.c -mtune=generic -march=x86-64 -fverbose-asm
  # options enabled:  -faggressive-loop-optimizations -fallocation-dce
  # -fasynchronous-unwind-tables -fauto-inc-dec -fdelete-null-pointer-checks
  # -fdwarf2-cfi-asm -fearly-inlining -feliminate-unused-debug-symbols
  # -feliminate-unused-debug-types -ffp-int-builtin-inexact -ffunction-cse
  # -fgcse-lm -fgnu-unique -fident -finline-atomics -fipa-stack-alignment
  # -fira-hoist-pressure -fira-share-save-slots -fira-share-spill-slots
  # -fivopts -fkeep-static-consts -fleading-underscore -flifetime-dse
  # -fmath-errno -fmerge-debug-strings -fpeephole -fplt
  # -fprefetch-loop-arrays -freg-struct-return
  # -fsched-critical-path-heuristic -fsched-dep-count-heuristic
  # -fsched-group-heuristic -fsched-interblock -fsched-last-insn-heuristic
  # -fsched-rank-heuristic -fsched-spec -fsched-spec-insn-heuristic
  # -fsched-stalled-insns-dep -fschedule-fusion -fsemantic-interposition
  # -fshow-column -fshrink-wrap-separate -fsigned-zeros
  # -fsplit-ivs-in-unroller -fssa-backprop -fstdarg-opt
  # -fstrict-volatile-bitfields -fsync-libcalls -ftrapping-math -ftree-cselim
  # -ftree-forwprop -ftree-loop-if-convert -ftree-loop-im -ftree-loop-ivcanon
  # -ftree-loop-optimize -ftree-parallelize-loops= -ftree-phiprop
  # -ftree-reassoc -ftree-scev-cprop -funit-at-a-time -funwind-tables
  # -fverbose-asm -fzero-initialized-in-bss -m128bit-long-double -m64 -m80387
  # -malign-stringops -mavx256-split-unaligned-load
  # -mavx256-split-unaligned-store -mfancy-math-387 -mfp-ret-in-387 -mfxsr
  # -mglibc -mieee-fp -mlong-double-80 -mmmx -mno-sse4 -mpush-args -mred-zone
  # -msse -msse2 -mstv -mtls-direct-seg-refs -mvzeroupper
  
  	.text
  	.section	.rodata
  .LC0:
  	.string	"hello world"
  	.text
  	.globl	main
  	.type	main, @function
  main:
  .LFB0:
  	.cfi_startproc
  	pushq	%rbp	#
  	.cfi_def_cfa_offset 16
  	.cfi_offset 6, -16
  	movq	%rsp, %rbp	#,
  	.cfi_def_cfa_register 6
  # hello.c:4:   printf ("hello world\n");
  	movl	$.LC0, %edi	#,
  	call	puts	#
  # hello.c:5:   return 0;
  	movl	$0, %eax	#, _3
  # hello.c:6: }
  	popq	%rbp	#
  	.cfi_def_cfa 7, 8
  	ret
  	.cfi_endproc
  .LFE0:
  	.size	main, .-main
  	.ident	"GCC: (GNU) 10.3.1 20210422 (Red Hat 10.3.1-1)"
  	.section	.note.GNU-stack,"",@progbits

In particular, looking at the heart of the :c:func:`main` function we
now have comments showing us how the some of the assembler relates to
specific lines in the C source file:

.. code-block:: asm

  # hello.c:4:   printf ("hello world\n");
  	movl	$.LC0, %edi	#,
  	call	puts	#
  # hello.c:5:   return 0;
  	movl	$0, %eax	#, _3
  # hello.c:6: }
  	popq	%rbp	#
  	.cfi_def_cfa 7, 8
  	ret

We'll look at how :program:`cc1` actually turns the C into assembler in
the next section.
