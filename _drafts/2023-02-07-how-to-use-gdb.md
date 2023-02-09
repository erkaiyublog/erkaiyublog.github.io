---
published: false
---
Here's a brief walkthrough of [GDB](https://www.sourceware.org/gdb/) (GNU Debugger). I'm assuming that you already know what GDB is, and you agree that GDB is a powerful tool for debugging.

I will use C programs to give example in this article.
## Where Can GDB be Used?
Clearly GDB is not a tool for every programming language. It's usually used for debugging **C, C++, Assembly and Go** programs.

An important thing to notice is that *you can't just use GDB to debug arbitrary programs* even if they're written in appropriate languages. For example, if you use [GCC](https://gcc.gnu.org/) to compile a C program, like:

	gcc example.c
    
You'll probably get an object file called "a.out". However, when you try to debug this file with 
	
    gdb a.out
    
The feedback will tell you that GDB can't obtain sufficient information of the program. You need to add a **-g** flag in GCC to compile the C program with debugging information, like:

	gcc -g example.c
    
With that, you'll be able to run GDB on the new a.out program. It's also a common practice to pass **-O** flag together with **-g** to GCC,

	gcc -Og example.c
    
With **-O** optimization of compiler will be disabled, for a better debugging experience. 
## Basic GDB Commands
In this section, some basic GDB commands will be introduced. For a detailed list of GDB commands, you may search online for GDB cheatsheets.
### Start GDB
Suppose "a.out" is the program that we want to debug, we can start GDB with the following command:
	
    gdb a.out
    
If there're arguments for "a.out", we can specify them like this:

	gdb --args a.out arg1 arg2 ...
    

## References
https://gcc.gnu.org/onlinedocs/gcc/Debugging-Options.html
https://developer.apple.com/library/archive/documentation/DeveloperTools/gdb/gdb/gdb_18.html
