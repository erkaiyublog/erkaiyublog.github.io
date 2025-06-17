---
title: "HFL: Hybrid Fuzzing on the Linux Kernel"
layout: post
---

Kyungtae Kim, Dae R. Jeong, Chung Hwan Kim, Yeongjin Jang, Insik Shin, Byoungyoung Lee

* Read: 16 Jun 2025
* Published: 23 Feb 2020

NDSS 2020

[https://www.ndss-symposium.org/ndss-paper/hfl-hybrid-fuzzing-on-the-linux-kernel/](https://www.ndss-symposium.org/ndss-paper/hfl-hybrid-fuzzing-on-the-linux-kernel/)

---

A handwritten note I made when reading this paper:
![handwritten](/images/posts/hfl/notes.jpeg)

# Q&A ([link](https://cseweb.ucsd.edu/~wgg/CSE210/howtoread.html))

**What are the motivations for this work?** 

* See Introduction, Section 3.
* There are three main challenges in kernel hybrid fuzzing:
    * Linux kernel uses many **indirect control transfers** to support polymorphism, random fuzzing cannot efficiently determine a specific index value fetch the function pointer table if that index comes from an input. Symbolic execution can neither easily handle such a case because indexing a table with symbols results in symbolic dereference and it requires the exploration of the entire value space by the symbols.
    * It is difficult to infer the right **sequence (and dependency) of syscalls** so as to build the system states required for triggering a vulnerability.
    * Nested structures in syscall arguments make interface templating difficult.


**What is the proposed solution?**

* See Section 3, Section 4.


**What is the work's evaluation of the proposed solution?**



**What is your analysis of the identified problem, idea and evaluation?**

NONE

**What are the contributions?**



**What are future directions for this research?**

NONE


**What questions are you left with?**

NONE

**What is your take-away message from this paper?**

NONE