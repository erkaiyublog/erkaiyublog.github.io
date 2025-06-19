---
title: "SyzGen++: Dependency Inference for Augmenting Kernel Driver Fuzzing"
layout: post
---

Weiteng Chen, Yu Hao, Zheng Zhang, Xiaochen Zou, Dhilung Kirat, Shachee Mishra

* Read: 18 Jun 2025
* Published: 19 May 2024

2024 IEEE Symposium on Security and Privacy (SP)

[https://ieeexplore.ieee.org/document/10646807](https://ieeexplore.ieee.org/document/10646807)

---

# Q&A ([link](https://cseweb.ucsd.edu/~wgg/CSE210/howtoread.html))

**What are the motivations for this work?** 
* See Introduction.
* Despite the substantial attack surface introduced by drivers, the majority of them lack readily available specifications. In fact, Syzkaller contains only a limited number of driver specifications: 60 for Linux, 0 for Darwin, and 5 each for FreeBSD and Android (excluding non-Android specific drivers).
* Existing methods to automatically generate specifications mostly depend on the source code. Some drivers don't have source code available, and some may only periodically release source code snapshots with significant delays.

**What is the proposed solution?**
* See Introduction, Section 4.
* Implemented SyzGen++ atop Angr and SyzGan with 12,587 lines of Python code for interface recovery and dependency inference, and ~1K lines of Go code into Syzkaller for fuzzing and test case generation. 
* The key insight of SyzGen++ is based on the following observations:
    1. In user space, an explicit dependency involves a **integer** (or handler) that corresponds to a complex object maintained by the kernel, e.g., a file descriptor corresponding to a struct file in kernel space;
    2. In kernel space, an explicit dependency requires a **producer** that creates an object and performs an insertion into some global data container (e.g., array or linked list) and a **consumer** that retrieves the corresponding internal object by performing a lookup into the same global data container based on the user input.
* By recognizing the **insertion and lookup operations** that operate on the same global data container, SyzGen++ effectively recovers explicit dependencies 
* SyzGen++ performs **symbolic execution** (with optimizations) on each syscall interface separately to record symbolic access paths and identifies any pairs of insertion and lookup operations against the same data container.

**What is the work's evaluation of the proposed solution?**
* See Introduction.
* SyzGen++ found 245 more dependencies for drivers with no traces.
* Compared SyzGen++’s performance to that of DIFUZE, KSG, and SyzDescribe in terms of code coverage, achieving an average improvement of 71%, 67%, and 39%, respectively. 

**What is your analysis of the identified problem, idea and evaluation?**
* Sounds similar to HFL.

**What are the contributions?**
* See Introduction.
* Proposed a novel approach for the identification of explicit dependencies. Specifically, defined the two fundamental building blocks of insertion and lookup operations and their pairing.
* Implemented SyzGen++, a prototype that supports **both macOS and Linux**.

**What are future directions for this research?**

NONE

**What questions are you left with?**

NONE

**What is your take-away message from this paper?**

NONE