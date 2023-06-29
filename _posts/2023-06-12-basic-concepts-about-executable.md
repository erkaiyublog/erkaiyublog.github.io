---
published: true
title: Executable - Some Basic Concepts
tags: OS software-analysis
---
---
_Ever since I started learning operating system, I've been struggling with some basic concepts involving executable files (e.g. what is the difference between static and dynamic libraries? how does the memory layout look like for an executable? how does an executable fit in different operating systems?). Having some vague answers to these questions in mind might be enough for me to write decent codes. However, since I've recently been working on a project related to software analysis, I decided to do some search to draw a clearer picture of these topics and write this blog to record what I've learned._

---

Instead of writing a well-structured article to cover everything, I will write this blog in the form of **Q&A**. Some of the questions have been there in my mind for a long time, some popped up when I was doing some search.

***Table of Contents***
* TOC
{:toc}

# 1. Static Library & Dynamic Library
## 1.1 What are they?
As we all know, ***library*** refers to precompiled code modules or functions that can be used to perform specific tasks or provide certain functionalities. For example, suppose we write a very basic "Hello World" program in C as shown below,

```
#include <stdio.h>
int main() {
	printf("Hello World\n");
	return 0;
}
``` 

The first thing we did was to include the ***stdio*** header, so that we could make use of ***printf*** function later. What happens during compilation is that the compiler will find the functions from C standard library that are included in ***stdio.h***, either by 

1. copying the code of ***stdio*** library and replace ***#include*** macro with the code; 

*or* 2. leave some marks in the executable and expect the library to be found somewhere in the system. 

The two different approaches result in the difference between static and dynamic libraries. **Static library** (also known as *archive*) is a collection of precompiled *object files* that are bundled together into a single file, it contains the compiled code and symbols of various functions. As you may recall, one way of generating *object files* is to use gcc with command like ***gcc -c sample.c***. Later when compiler needs to generate executable, it will combine multiple object files into one single executable, such process of combining object files to obtain an executable is called *static linking*. **Dynamic library** (also known as *shared library*) is a collection of compiled functions, instead of being statically linked to produce executable, dynamic libraries are not added to the executable during compilation. During execution of the program (**runtime**), operating system itself will load the dynamic libraries to memory for the executable to use, such process is called *dynamic linking*.

## 1.2 What are some typical static and dynamic library extensions?
Static libraries: *.a* on Linux, *.lib* on Windows.

Dynamic libraries: *.so* on Unix-like systems, *.dll* on Windows.

## 1.3 How to tell if a program uses static or dynamic library?
To see if an executable contains **dynamic library**, we can use ***ldd*** command on Linux system, an example of that is shown below. 

```
$ ldd /bin/ls
	linux-vdso.so.1 (0x00007ffccf3a8000)
	libselinux.so.1 => /lib/x86_64-linux-gnu/libselinux.so.1 (0x00007f178fdf0000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f178f9ff000)
	libpcre.so.3 => /lib/x86_64-linux-gnu/libpcre.so.3 (0x00007f178f78e000)
	libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f178f58a000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f179023a000)
	libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f178f36b000)
```

The hexadecimal numbers in the result above are the memory addresses where the shared libraries are loaded into when process is running. For Mac OS users, ***otool -L*** may work as an alternative of ***ldd***. For Windows users, I have no idea ;)

To see if an executable contains **static library** is less straightforward, since ***ldd*** can give us the dynamic libraries used, we can use ***nm*** command to display the symbol table of the executable and the dynamic libraries it uses. Then eliminate the matchings, so statically linked functions are the ones left.

## 1.4 Is there a difference between the position where library codes will be loaded in memory between static and dynamic libraries?
Yes.

Static libraries are loaded during compilation, so they have a fixed address relative to main code of the executable. Dynamic libraries are loaded during runtime, it's possible that dynamic libraries are loaded to different memory addresses in different execution even though the executable is the same. 

## 1.5 Can I have some hands on experience of static and dynamic libraries?
We can start with writing a C program in a file called ***hello.c***, 

```
// This section of code is in file hello.c

#include <stdio.h>
#include "sqr.h"

int main() {
	int a;
	printf("Input an integer: ");
	scanf("%d", &a);
	printf("Hello, here's the square of your integer: %d\n", terriblesqr(a));
	return 0;
}
```

Then we implement the ***terriblesqr*** function in a file called ***sqr.c***,

```
// This section of code is in file sqr.c

#include "sqr.h"
int terriblesqr(int a) {
	return a*a;
}
```

Lastly, write a header file for ***sqr.c***, call it ***sqr.h***,

```
// This section of code is in file sqr.h

int terriblesqr(int a);
```

So far, we got three files, ***hello.c, sqr.c, sqr.h***. 

We first illustrate static library compilation. Compile ***sqr.c*** to generate an object file with the following command,

```
gcc -c sqr.c -o sqr.o  
```

***-o*** command specifies the name of output file, ***-c*** flag tells GCC to compile without linking. If you're wondering why ***sqr.h*** is not mentioned in the command, it's because ***sqr.c*** does include it at the beginning, so GCC will automatically look for ***sqr.h***, no need us to tell it. 

Then we compile the executable with the following command,

```
gcc hello.c sqr.o -o hello
```
Now we have two more files, ***sqr.o, hello***. Here ***hello*** is the executable, ***sqr.o*** is the static library. Use ***nm*** command to list their symbol table. 

```
$ nm sqr.o
0000000000000000 T terriblesqr

$ nm hello
0000000000201010 b __bss_start
0000000000201010 b completed.7698
                 w __cxa_finalize@@glibc_2.2.5
0000000000201000 d __data_start
0000000000201000 w data_start
0000000000000640 t deregister_tm_clones
00000000000006d0 t __do_global_dtors_aux
0000000000200db0 t __do_global_dtors_aux_fini_array_entry
0000000000201008 d __dso_handle
0000000000200db8 d _dynamic
0000000000201010 d _edata
0000000000201018 b _end
0000000000000824 t _fini
0000000000000710 t frame_dummy
0000000000200da8 t __frame_dummy_init_array_entry
00000000000009ec r __frame_end__
0000000000200fa8 d _global_offset_table_
                 w __gmon_start__
0000000000000880 r __gnu_eh_frame_hdr
00000000000005a8 t _init
0000000000200db0 t __init_array_end
0000000000200da8 t __init_array_start
0000000000000830 R _IO_stdin_used
                 U __isoc99_scanf@@GLIBC_2.7
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
0000000000000820 T __libc_csu_fini
00000000000007b0 T __libc_csu_init
                 U __libc_start_main@@GLIBC_2.2.5
000000000000071a T main
                 U printf@@GLIBC_2.2.5
0000000000000680 t register_tm_clones
                 U __stack_chk_fail@@GLIBC_2.4
0000000000000610 T _start
0000000000000792 T terriblesqr
0000000000201010 D __TMC_END__
```

We can see that ***hello*** contains a symbol called ***terriblesqr***, which corresponds to the static library function. Here "T" means the symbol is a text, "D" means the symbol is a data, "U" means the symbol is undefined.

When running ***ldd*** command on ***hello***, we can see the following results,

```
$ ldd hello 
linux-vdso.so.1 (0x00007ffe5cfca000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007ff907111000)
/lib64/ld-linux-x86-64.so.2 (0x00007ff907704000)
```

***linux-vdso.so.1*** is a virtual dynamic shared object provided by the Linux kernel. It allows certain system calls to be executed within the context of the calling process without switching to kernel mode. It is not a physical library but rather a mechanism for optimized system call handling. ***libc.so.6*** is the so called "libc" on Linux. It contains the basic functions and definitions required by most programs, including standard input/output operations, string manipulation, memory allocation, and more. ***/lib64/ld-linux-x86-64.so.2*** is the dynamic linker/loader, responsible for dynamically linking the executable with the required libraries at runtime. It is necessary for the proper execution of dynamically linked executables.

It might be a little bit of confusing to see these three dependencies in the result of ***ldd***, in fact, we can use the following command as an alternative which gives a more neat version of dynamic libraries used by an executable,

```
$ readelf -d hello | grep NEEDED
 0x0000000000000001 (NEEDED)             Shared library: [libc.so.6]
```

Now, back to the three files ***hello.c, sqr.c, sqr.h***, we try to generate an executable ***hello*** with ***sqr*** library dynamically linked. Start with generating *.o* file for ***hello.c*** using following command,

```
gcc -o hello.o -c hello.c
```

Then generate the dynamic library of ***sqr*** with the following command, note that naming of ***libsqr.so*** is forced,

```
gcc -shared -o libsqr.so sqr.o
```

The ***-shared*** flag indicates shared library, ***libsqr.so*** follows the naming convention of ***lib[name].so***. 

Next, we compile the **new** executable ***hello*** (replace the old one) with dynamic linkage,

```
gcc -o hello hello.o -lsqr -L .
```

Here ***-lsqr*** tells compiler to look for ***sqr*** dynamic library. ***-L*** tells the compiler where to find dynamic libraries, in our case, ***-L*** is followed by a dot to indicate the dynamic library (***libsqr.so***) should be found right here. Note that these flags are only used for compilation, not runtime.

Now we have our executable with dynamic linkage, use ***readelf*** command on it, we can see an extra line indicating ***libsqr.so*** is needed by ***hello***,

```
$ readelf -d hello | grep NEEDED
 0x0000000000000001 (NEEDED)             Shared library: [libsqr.so]
 0x0000000000000001 (NEEDED)             Shared library: [libc.so.6]
```

However, running it directly will get an error message like this:

```
./hello: error while loading shared libraries: libsqr.so: cannot open shared object file: No such file or directory
```

That is because the OS doesn't know where to find ***libsqr.so*** when executing ***hello***. Also, if we run ***ldd*** command, we can see the dynamic library is missing,

```
ldd hello
	linux-vdso.so.1 (0x00007ffc1c5e5000)
	libsqr.so => not found
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f04dbdb1000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f04dc3a4000)
```

One way to make things work is to explicitly tell the OS where to look for ***libsqr.so*** when executing ***hello***, 

```
LD_LIBRARY_PATH=. ./hello
```

Adding ***LD_LIBRARY_PATH=.*** tells the OS to look for dynamic libraries within the current directory, so that the ***hello*** executable can be run successfully.

Similar to static library example, we run ***nm*** on the new ***hello*** executable, will get the following result,

```
$ nm hello
0000000000201010 B __bss_start
0000000000201010 b completed.7698
                 w __cxa_finalize@@GLIBC_2.2.5
0000000000201000 D __data_start
0000000000201000 W data_start
0000000000000750 t deregister_tm_clones
00000000000007e0 t __do_global_dtors_aux
0000000000200d98 t __do_global_dtors_aux_fini_array_entry
0000000000201008 D __dso_handle
0000000000200da0 d _DYNAMIC
0000000000201010 D _edata
0000000000201018 B _end
0000000000000924 T _fini
0000000000000820 t frame_dummy
0000000000200d90 t __frame_dummy_init_array_entry
0000000000000ac4 r __FRAME_END__
0000000000200fa0 d _GLOBAL_OFFSET_TABLE_
                 w __gmon_start__
0000000000000980 r __GNU_EH_FRAME_HDR
00000000000006a8 T _init
0000000000200d98 t __init_array_end
0000000000200d90 t __init_array_start
0000000000000930 R _IO_stdin_used
                 U __isoc99_scanf@@GLIBC_2.7
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
0000000000000920 T __libc_csu_fini
00000000000008b0 T __libc_csu_init
                 U __libc_start_main@@GLIBC_2.2.5
000000000000082a T main
                 U printf@@GLIBC_2.2.5
0000000000000790 t register_tm_clones
                 U __stack_chk_fail@@GLIBC_2.4
0000000000000720 T _start
                 U terriblesqr
0000000000201010 D __TMC_END__
```

Compared with the ***hello*** executable compiled with static linkage, the difference is that symbol ***terriblesqr*** here is of type "U" instead of "T", which means the dynamic library symbol is of undefined type.

# 2. ELF
## 2.1 What is ELF?
ELF stands for "Executable and Linkable Format", it is a very versatile file format, designed by Unix System Laboratories while working with Sun Microsystems on SVR4. It is a format specified in System V ABI (What is System V??? I will mention that below).  

In short, ELF is **a format for storing programs or fragments of programs on disk**. It contains:

1. **ELF header**: includes identification fields (e.g. magic number, arch type), it also contains details like the entry point address and program header and section header table offsets.
2. **Program header table**: contains a list of entries describing the various segments of the executable file (if this ELF is an executable). Each entry specifies the type of segment (e.g. code, data, dynamic linking info), its virtual memory address, and its size. Used by the system's runtime linker or loader when executing this file (or used as shared library).
3. **Section header table**: contains entries that describe the sections present in the file, each section is a part of a segment. Section headers contain details about each section (e.g. type, offset, size, memory address). Mostly used during the compilation and linking process. 
4. **Section data**: contains the actual data of each section, includes the machine instructions, data values, symbol tables, relocation information, etc.
5. **Symbol table**: holds information about symbols defined or referenced in the file. 
6. **String table**: holds the string data referenced by various components of the ELF file. 
7. **Relocation table**: contains information necessary for adjusting addresses in the executable when it is loaded into memory.
8. **Dynamic linking table**: provides information for runtime linking and dynamic loading of shared libraries. 

Note that some of the components mentioned above may not neccessarily exist in every ELF file, it all depends on the actual purpose of the ELF. In ELF header, there is a "flag" specifying which components are present in this ELF file.

## 2.2 What are some types of sections?
* ***.text***: where code lives, can be shown by ***objdump -drS***.
* ***.data***: where global tables, variables, ***objdump -s -j .data*** will hexdump it.
* ***.bss***: where the uninitialized arrays and variables live.
* ***.rodata***: where the strings live, ***objdump -s -j .rodata*** will hexdump it.
* ***.comment***: just comments.
* ***.stab***: debugging symbols.

## 2.3 What are some typical extensions for ELF files?
* ***.elf***: executable file
* ***.exe***: executable file
* ***.o***: object file
* ***.obj***: similar to ***.o***, often used in Windows
* ***.so***: shared library
* ***.core***: generated as core dump when program crashes

# 3. System V 
# 4. Process of Loading Executable

# References

1. https://www.geeksforgeeks.org/static-vs-dynamic-libraries/
2. https://unix.stackexchange.com/questions/120015/how-to-find-out-the-dynamic-libraries-executables-loads-when-run
3. https://stackoverflow.com/questions/1124571/get-list-of-static-libraries-used-in-an-executable
4. https://amir.rachum.com/shared-libraries/
5. https://wiki.osdev.org/ELF


