---
published: true
title: Executable - Some Basic Concepts
tags: OS software-analysis
---
---
_Ever since I started learning operating system, I've been struggling with some basic concepts involving executable files (e.g. what is the difference between static and dynamic libraries? how does the memory layout look like for an executable? how does an executable fit in different operating systems?). Having some vague answers to these questions in mind might be enough for me to write decent codes. However, since I've recently been working on a project related to software analysis, I decided to do some search to draw a clearer picture of these topics and write this blog to record what I've learned._

---

questions:
	1. What is a static library, what is a dynamic library?
	2. What is an ELF file?
	3. What is the typical memory layout of an executable, who is deciding that?
	4. What is system V?
	5. What is the process of loading an executable? 
	6. What are some common tools to analyze a given executable file? 

references:
	1. https://www.geeksforgeeks.org/static-vs-dynamic-libraries/
	2. https://unix.stackexchange.com/questions/120015/how-to-find-out-the-dynamic-libraries-executables-loads-when-run
	3. https://stackoverflow.com/questions/1124571/get-list-of-static-libraries-used-in-an-executable
	4. https://amir.rachum.com/shared-libraries/

Instead of writing a well-structured article to cover everything, I will write this blog in the form of **Q&A**. Some of the questions have been there in my mind for a long time, some popped up when I was doing some search.

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

The first thing we did was to use ***#include <stdio.h>*** to include the ***stdio*** header, so that we could make use of ***printf*** function later. What happens during compilation is that the compiler will find the functions from C standard library that are included in ***stdio.h***, either by 

1. copying the code of ***stdio*** library and replace ***#include <stdio.h>*** with the code; 

*or* 2. leave some marks in the executable and expect the library to be found somewhere in the system. 

The two different approaches result in the difference between static and dynamic libraries. **Static library** (also known as *archive*) is a collection of precompiled *object files* that are bundled together into a single file, it contains the compiled code and symbols of various functions. As you may recall, one way of generating *object files* is to use gcc with command like ***gcc -c sample.c***. Later when compiler needs to generate executable, it will combine multiple object files into one single executable, such process of combining object files to obtain an executable is called *static linking*. **Dynamic library** (also known as *shared library*) is a collection of compiled functions, instead of being statically linked to produce executable, dynamic libraries are not added to the executable during compilation. During execution of the program (**runtime**), operating system itself will load the dynamic libraries to memory for the executable to use, such process is called *dynamic linking*.

## 1.2 What are some typical static and dynamic library extensions?
Static libraries: *.a* on Linux, *.lib* on Windows.
Dynamic libraries: *.so* on Unix-like systems, *.dll* on Windows.

## 1.3 How to tell if a program uses static or dynamic library?
To see if an executable contains **dynamic library**, we can use ***ldd*** command on Linux system, an example of that is shown below. 

```
$ ldd /bin/ls
    linux-vdso.so.1 =>  (0x00007fff87ffe000)
    libselinux.so.1 => /lib/x86_64-linux-gnu/libselinux.so.1 (0x00007ff0510c1000)
    librt.so.1 => /lib/x86_64-linux-gnu/librt.so.1 (0x00007ff050eb9000)
    libacl.so.1 => /lib/x86_64-linux-gnu/libacl.so.1 (0x00007ff050cb0000)
    libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007ff0508f0000)
    libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007ff0506ec000)
    /lib64/ld-linux-x86-64.so.2 (0x00007ff0512f7000)
    libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007ff0504ce000)
    libattr.so.1 => /lib/x86_64-linux-gnu/libattr.so.1 (0x00007ff0502c9000)
```

The hexadecimal numbers in the result above are the memory addresses where the shared libraries are loaded into when process is running. For Mac OS users, ***otool -L*** may work as an alternative of ***ldd***. For Windows users, I have no idea ;)

To see if an executable contains **static library** is less straightforward, since ***ldd*** can give us the dynamic libraries used, we can use ***nm*** command to display the symbol table of the executable and the dynamic libraries it uses. Then eliminate the matchings, so statically linked functions are the ones left.

## 1.4 Is there a difference between the position where library codes will be loaded in memory between static and dynamic libraries?
Yes.

Static libraries are loaded during compilation, so they have a fixed address relative to main code of the executable. Dynamic libraries are loaded during run time, it's possible that dynamic libraries are loaded to different memory addresses in different execution even though the executable is the same. 

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
0000000000201010 B __bss_start
0000000000201010 b completed.7698
                 w __cxa_finalize@@GLIBC_2.2.5
0000000000201000 D __data_start
0000000000201000 W data_start
0000000000000640 t deregister_tm_clones
00000000000006d0 t __do_global_dtors_aux
0000000000200db0 t __do_global_dtors_aux_fini_array_entry
0000000000201008 D __dso_handle
0000000000200db8 d _DYNAMIC
0000000000201010 D _edata
0000000000201018 B _end
0000000000000824 T _fini
0000000000000710 t frame_dummy
0000000000200da8 t __frame_dummy_init_array_entry
00000000000009ec r __FRAME_END__
0000000000200fa8 d _GLOBAL_OFFSET_TABLE_
                 w __gmon_start__
0000000000000880 r __GNU_EH_FRAME_HDR
00000000000005a8 T _init
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

We can see that ***hello*** contains a symbol called ***terriblesqr***, which corresponds to the static library function.

When running ***ldd*** command on ***hello***, we can see the following results,

```
$ ldd hello 
linux-vdso.so.1 (0x00007ffe5cfca000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007ff907111000)
/lib64/ld-linux-x86-64.so.2 (0x00007ff907704000)
```



 
