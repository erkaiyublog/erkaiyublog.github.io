---
published: true
title: Introduction to Reverse Engineering Tools with an Angry Example
tags: CTF reverse-engineering  
---
---
_Infact, I spent three days in the spring break to work on a single CTF problem, and ended with failure ..._

_This article gives a brief introduction to tools related with reverse engineering, including Ghidra, angr, and Pin, with a concrete_ [CTF example](https://ctf.sigpwny.com/challenges#Meetings/angry-417)
---
# Motivation
Well, I believe you've at least heard about the term "reverse engineering". Reverse engineering is necessary when we want to investigate the behavior of an executable file without the access of its source code. 

When you're given an executable without source code, you may first try to run it with a debugger, since this would allow you to investigate the memory and registers during the execution, and set break points. However, if the executable was compiled without debugging information (stripped), then debugger can no longer be used to it. At this point, you may want to seek help from powerful **reverse engineering tools**. 

Since the executable would always contain **machine code** for the processor to execute, reverse engineering tools can guess the programming source code of an executable based on the machine codes. With the basic translation done by reverse engineering tools, we can manually refine the guessed source code and eventually have a better understanding of the behavior of the executable. 

# CTF Example Description
I guess it might be helpful to introduce tools with examples. [Here](https://ctf.sigpwny.com/challenges#Meetings/angry-417)'s a good one, a CTF puzzle that worths 500 points on [sigpwny CTF website](https://ctf.sigpwny.com/). 

In fact, I've been playing CTF puzzles on sigpwny website for fun during the spring break, I gathered my solutions in a github [repo](https://github.com/silkrow/CTF_sigpwny). The puzzle I use as an example here cost me more than three days to solve ... 

The puzzle is called *Angry*, which perfectly describes my feeling when trying to solve it. The executable takes a string in form ```sigpwny{..}``` and tells you if it's the right flag or not. 

**Spoil Alert: I will introduce my solution to this CTF puzzle in the following post, if you want to figure it out by yourself, you have to stop reading RN :(**

## References
1. [https://research.kudelskisecurity.com/2016/08/08/angr-management-first-steps-and-limitations/](https://research.kudelskisecurity.com/2016/08/08/angr-management-first-steps-and-limitations/)
2. [https://book.hacktricks.xyz/reversing-and-exploiting/reversing-tools-basic-methods/angr/angr-examples](https://book.hacktricks.xyz/reversing-and-exploiting/reversing-tools-basic-methods/angr/angr-examples)
3. [https://docs.angr.io/](https://docs.angr.io/)



--- 
# Drafts

layout:
	angr? 
	basic template
		official doc examples
	problem description
	how to solve it
		length=5 if replace call with hook 
	more angr techniques (2. reference, 1. for python debugging technique)


## How to put multiple inputs received by scanf (should use registers!)
https://book.hacktricks.xyz/reversing-and-exploiting/reversing-tools-basic-methods/angr/angr-examples#registry-values



