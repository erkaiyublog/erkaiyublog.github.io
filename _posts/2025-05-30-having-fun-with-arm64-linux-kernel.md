---
published: false
title: Having Fun with ARM64 Linux Kernel
tags: OS qemu arm
---

Lately, I’ve been working on a project related to recording Linux kernel execution traces at the object code level. During this process, I had the opportunity to work with the ARM64 architecture on Linux. I decided to write this blog to capture some of the highlights I learned along the way.

***Table of Contents***
* TOC
{:toc}

# Objdump ARM Kernel
Since I'm working with an x86-64 server, performing an object dump on ARM64 ELF files can't be done simply using the ```objdump``` program. Instead, I had to use the ```aarch64-linux-gnu-objdump``` program. This tool can be installed on Ubuntu via the ```binutils-aarch64-linux-gnu``` package.

The command I used to objdump an ARM64 kernel image is:

```bash
aarch64-linux-gnu-objdump -d -z vmlinux
```

Note that the ```-z``` flag in the command above ensures that the full object dump is shown, preventing zeroed lines from being collapsed into ellipses (...).

# Some Booting Process Introduction
The beginning of ARM64 kernel image objdump looks like:

```
/linux/vmlinux:     file format elf64-littleaarch64


Disassembly of section .head.text:

ffff800080000000 <_text>:
ffff800080000000:	fa405a4d 	ccmp	x18, #0x0, #0xd, pl	// pl = nfrst
ffff800080000004:	14713c37 	b	ffff800081c4f0e0 <primary_entry>
ffff800080000008:	00000000 	.word	0x00000000
ffff80008000000c:	00000000 	.word	0x00000000
ffff800080000010:	02c20000 	.word	0x02c20000
ffff800080000014:	00000000 	.word	0x00000000
ffff800080000018:	0000000a 	.word	0x0000000a
ffff80008000001c:	00000000 	.word	0x00000000
ffff800080000020:	00000000 	.word	0x00000000
ffff800080000024:	00000000 	.word	0x00000000
ffff800080000028:	00000000 	.word	0x00000000
ffff80008000002c:	00000000 	.word	0x00000000
ffff800080000030:	00000000 	.word	0x00000000
ffff800080000034:	00000000 	.word	0x00000000
ffff800080000038:	644d5241 	.word	0x644d5241
ffff80008000003c:	00000040 	.word	0x00000040
ffff800080000040:	00004550 	.word	0x00004550
ffff800080000044:	0002aa64 	.word	0x0002aa64
ffff800080000048:	00000000 	.word	0x00000000
```

The first two lines are instructions executed when the kernel is entered from the bootloader. ```primary_entry``` function can be found in ```linux/arch/arm64/kernel/head.S``` source file. Let's take a look at the corresponding snippet:

```
/*
 * Kernel startup entry point.
 * ---------------------------
 *
 * The requirements are:
 *   MMU = off, D-cache = off, I-cache = on or off,
 *   x0 = physical address to the FDT blob.
 *
 * Note that the callee-saved registers are used for storing variables
 * that are useful before the MMU is enabled. The allocations are described
 * in the entry routines.
 */
	__HEAD
	/*
	 * DO NOT MODIFY. Image header expected by Linux boot-loaders.
	 */
	efi_signature_nop			// special NOP to identity as PE/COFF executable
	b	primary_entry			// branch to kernel start, magic
	.quad	0				// Image load offset from start of RAM, little-endian
	le64sym	_kernel_size_le			// Effective size of kernel image, little-endian
	le64sym	_kernel_flags_le		// Informative flags, little-endian
	.quad	0				// reserved
	.quad	0				// reserved
	.quad	0				// reserved
	.ascii	ARM64_IMAGE_MAGIC		// Magic number
	.long	.Lpe_header_offset		// Offset to the PE header.

	__EFI_PE_HEADER

```

To better understand the connection between the ```head.S``` snippet and the objdump results, I made the following annotation:

```
/linux/vmlinux:     file format elf64-littleaarch64


Disassembly of section .head.text:

ffff800080000000 <_text>:
ffff800080000000:	fa405a4d 	ccmp	x18, #0x0, #0xd || efi_signature_nop // special NOP to identity as PE/COFF executable
ffff800080000004:	14713c37 	b	ffff800081c4f0e0    || b primary_entry // branch to kernel start, magic
ffff800080000008:	00000000 	.word	0x00000000      || .quad    0 // Image load offset from start of RAM, little-endian
ffff80008000000c:	00000000 	.word	0x00000000                    
ffff800080000010:	02c20000 	.word	0x02c20000      || le64sym _kernel_size_le // Effective size of kernel image, little-endian
ffff800080000014:	00000000 	.word	0x00000000
ffff800080000018:	0000000a 	.word	0x0000000a      || le64sym _kernel_flags_le // Informative flags, little-endian
ffff80008000001c:	00000000 	.word	0x00000000
ffff800080000020:	00000000 	.word	0x00000000      || .quad    0 // reserved
ffff800080000024:	00000000 	.word	0x00000000
ffff800080000028:	00000000 	.word	0x00000000      || .quad    0 // reserved
ffff80008000002c:	00000000 	.word	0x00000000
ffff800080000030:	00000000 	.word	0x00000000      || .quad    0 // reserved
ffff800080000034:	00000000 	.word	0x00000000
ffff800080000038:	644d5241 	.word	0x644d5241      || .ascii	ARM64_IMAGE_MAGIC // Magic number
ffff80008000003c:	00000040 	.word	0x00000040      || .long	.Lpe_header_offset // Offset to the PE header.
ffff800080000040:	00004550 	.word	0x00004550
ffff800080000044:	0002aa64 	.word	0x0002aa64
ffff800080000048:	00000000 	.word	0x00000000
...                                                     || _EFI_PE_HEADER
```

So, the .word directives coming after the first two instructions are representing data or instructions used by booting and kernel initial setup process.

## MMU Enabling
When recording the entire booting process of ARM64 Linux kernel, I noticed that the start of the booting process was **not captured** by my tool when using virtual addresses. This happens because the MMU (Memory Management Unit) is not enabled at the very beginning of kernel initialization.

In ```linux/arch/arm64/kernel/head.S``` I located the following snippet at the end of ```primary_entry``` function:

```
SYM_CODE_START(primary_entry)
...
	/*
	 * The following calls CPU setup code, see arch/arm64/mm/proc.S for
	 * details.
	 * On return, the CPU will be ready for the MMU to be turned on and
	 * the TCR will have been set.
	 */
	bl	__cpu_setup			// initialise processor
	b	__primary_switch
SYM_CODE_END(primary_entry)
```

Following to the comment, I found the ```__cpu_setup``` function in ```linux/arch/arm64/mm/proc.S```, it comes with a clear comment saying:

```
/*
 *	__cpu_setup
 *
 *	Initialise the processor for turning the MMU on.
 *
 * Output:
 *	Return in x0 the value of the SCTLR_EL1 register.
 */
```

With this, I can conclude the following take away:

> 1. When ARM64 Linux kernel boots, the bootloader runs at the beginning of the entire process. Then, the execution flow starts in kernel image from its beginning, where ```primary_entry``` function is the first one to be called. The kernel image reserves a section of .word at the beginning which contains data for booting and kernel initialization (e.g. ARM ELF magic number, kernel image size).
> 2. ```primary_entry``` function can be found in ```linux/arch/arm64/kernel/head.S```, at the end of this function there is a function call to ```__cpu_setup```, where the MMU is set up and you get virtual addresses mapped to physical ones.

# ```.word``` Directives and ```udf``` Instructions 
In ARM64 kernel image objdump, one can observe many instances of ```.word``` directives, examples can be found in the [Some Booting Process Introduction](#some-booting-process-introduction) section above. These ```.word``` entries contain data rather than executable code.

A question comes naturally: *How does the ```objdump``` program tell if a 4 byte binary in an ELF file is an instruction or a ```.word``` data?*

Besides ```.word``` directives, one can also observe ```udf``` instances, for example:

```
ffff800080010d48:       cb2063e0        sub     x0, sp, x0
ffff800080010d4c:       f274cc1f        tst     x0, #0xfffffffffffff000
ffff800080010d50:       54001581        b.ne    ffff800080011000 <__bad_stack>  // b.any
ffff800080010d54:       cb2063ff        sub     sp, sp, x0
ffff800080010d58:       d53bd060        mrs     x0, tpidrro_el0
ffff800080010d5c:       14000262        b       ffff8000800116e4 <el0t_64_fiq>
ffff800080010d60:       00000000        udf     #0
ffff800080010d64:       00000000        udf     #0
ffff800080010d68:       00000000        udf     #0
ffff800080010d6c:       00000000        udf     #0
ffff800080010d70:       00000000        udf     #0
ffff800080010d74:       00000000        udf     #0
ffff800080010d78:       00000000        udf     #0
ffff800080010d7c:       00000000        udf     #0
ffff800080010d80:       14000003        b       ffff800080010d8c <vectors+0x58c>
```

These ```udf``` instructions are used as paddings and never got executed.

So, another natural question is: *How does the ```objdump``` program decides if a ```0x00000000``` is translated to ```.word 00000000``` or ```udf #0```?*
## How Are ```.word``` and ```udf``` Recognized?
I did a tiny experiment with the following example as ```example.s```:

```asm
    .section .text
    .global _start
_start:
    mov x0, #1             // set x0 = 1 (exit code)
    .word 0xd2800020
    udf #0
    .word 0x00000000
    mov x1, #2             // set x1 = 2
    mov x8, #93            // syscall number for exit on ARM64
    svc #0                 // syscall
```

On a x86 machine, I assembled and loaded it for ARM64.

```
aarch64-linux-gnu-as example.s -o example.o
aarch64-linux-gnu-ld example.o -o example
```

Next, I disassembled it with ```aarch64-linux-gnu-objdump```:
![disassembled](/images/posts/have_fun_arm/dis_word1.png)

Notice how the two consective ```0x00000000``` got collapsed into ellipses!

I added the ```-z``` flag to ask for the complete objdump:
![disassembled](/images/posts/have_fun_arm/dis_word2.png)

It turned out that the ```aarch64-linux-gnu-objdump``` program correctly recognizes both ```.word``` and ```udf```!!

Now, take a look at how the actual binary looks like in ```example```:

![offset](/images/posts/have_fun_arm/readelf.png)
![binary](/images/posts/have_fun_arm/xxd.png)

Well, it appears that within the ```.text``` section the raw bytes for ```0xd2800020``` and ```0x00000000``` pairs are indistinguishable!

So, the take aways is 
> In ELF files there are **debug info** (like DWARF) and other **symbol info** (like function boundaries, labels, or assembler-generated metadata) that helps ```objdump``` program to recognize ```.word``` and ```udf``` entries in ```.text``` section, the raw bytes themselves don't really reflect such differences.

