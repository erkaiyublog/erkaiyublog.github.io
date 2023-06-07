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

## 1.2 How to tell if a program uses static or dynamic library?
## 1.3 What are some typical static and dynamic library extensions?
