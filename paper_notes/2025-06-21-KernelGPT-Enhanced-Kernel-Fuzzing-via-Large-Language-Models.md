---
title: "KernelGPT: Enhanced Kernel Fuzzing via Large Language Models"
layout: post
---

Chenyuan Yang, Zijie Zhao, Lingming Zhang

* Read: 21 Jun 2025
* Published: 30 Mar 2025

ASPLOS '25: Proceedings of the 30th ACM International Conference on Architectural Support for Programming Languages and Operating Systems, Volume 2 Pages 560 - 573


[https://doi.org/10.1145/3676641.3716022](https://doi.org/10.1145/3676641.3716022)

---

# Q&A ([link](https://cseweb.ucsd.edu/~wgg/CSE210/howtoread.html))

**What are the motivations for this work?** 
* See Introduction.
* Key insight is that LLMs have seen massive kernel code, documentation, and use cases during pre-training, and thus can automatically distill the necessary information for making valid syscalls.

**What is the proposed solution?**
* See Introduction.
* KernelGPT leverages an iterative approach to automatically infer the specifications, and further debug and repair them based on the validation feedback.

**What is the work's evaluation of the proposed solution?**
* See Section 5.

**What is your analysis of the identified problem, idea and evaluation?**

NONE

**What are the contributions?**
* See Introduction.
* Proposed the first automated approach to leveraging the potential of LLMs for kernel fuzzing.
* Implemented KernelGPT to infer syscall specifications with a novel iterative strategy.
* Detected 24 previously unknown bugs.

**What are future directions for this research?**

NONE

**What questions are you left with?**

NONE

**What is your take-away message from this paper?**

NONE