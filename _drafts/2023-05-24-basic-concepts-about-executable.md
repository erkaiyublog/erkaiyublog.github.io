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
	1. https://wiki.osdev.org/Object_Files

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

The first thing we did was to use ***#include <stdio.h>*** to include the ***stdio*** library, so that we could make use of ***printf*** function later. What happens during compilation is that the compiler will do the job of including ***stdio*** library, either by 

1. copying the code of ***stdio*** library and replace ***#include <stdio.h>*** with the code; 

*or* 2. leave some marks in the executable and expect the library to be found somewhere in the system. 

The two different approaches result in the difference between static and dynamic libraries. **Static Library** 

## 1.2 How to tell if a program uses static or dynamic library?

