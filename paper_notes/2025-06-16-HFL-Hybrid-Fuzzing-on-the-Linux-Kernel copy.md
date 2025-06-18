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
    * **Nested structures** in syscall arguments make interface templating difficult.

**What is the proposed solution?**

* See Section 3, Section 4.
* HFL is built based on Syzkaller and S2E.
* An overview of HFL.
![overview](/images/posts/hfl/overview.png)
* For the **indirect control transfer** problem, HFL designed an **offline translator**, operating based on the source code of the kernel, which transforms indirect control-flows to direct control-flows. The translator ensures that an index variable of the function pointer table originates from syscall parameters (so that HFL can control it). The translation is done by an iteration of all instructions in **compile-time**, with the help of inter-procedural dataflow analysis. An example of branch transformation:
![branch](/images/posts/hfl/branch.png) 
* For **syscall sequence inference**, HFL uses the following three steps to determine the order of syscalls:
    1. Static Analysis to Find Candidate Dependency Pairs.
    2. Runtime Validation to Identify True Dependencies.
    3. Symbolic Checking to Detect Parameter Dependency.
* For **nested syscall arguments**, the observation is that the following two pieces of information are the key to re-construct the nested syscall arguments: 1. memory location connecting to nested input structures; 2. the length of memory buffer arguments. To this end, HFL keeps monitoring invocations of the transfer functions during concolic execution. Once invoked, it check if its source buffer is **symbolically tainted**. This allows concolic executor to focus on certain transfer functions that come from the syscall of interest. Workflow:
![nested](/images/posts/hfl/nested.png)

**What is the work's evaluation of the proposed solution?**
* See Section 6.
* A table of syscalls tested.
![syscalltested](/images/posts/hfl/syscalltested.png)
* Experiments conducted over *3 runs of 50 hours each*.
* HFL has the highest coverage (measured by KCOV), 10.5%.
* HFL managed to find bugs faster than competitors.

**What is your analysis of the identified problem, idea and evaluation?**

I'm curious how the authors managed to distill the problems faced by hybrid kernel fuzzing into the three concrete ones listed in the paper. Are these three problems the only ones left to be explored?

**What are the contributions?**
* See Introduction.
* HFL, the first hybrid kernel fuzzer.

**What are future directions for this research?**
* See Section 8.
* Some kernel's non-deterministic program behaviors were obeserved.

**What questions are you left with?**

NONE

**What is your take-away message from this paper?**

NONE