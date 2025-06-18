---
title: "KSG: Augmenting Kernel Fuzzing with System Call Specification Generation"
layout: post
---

Hao Sun, Yuheng Shen, Jianzhong Liu, Yiru Xu, Yu Jiang

* Read: 18 Jun 2025
* Published: 11 July 2022

2022 USENIX Annual Technical Conference. July 11–13, 2022 • Carlsbad, CA, USA

[https://www.usenix.org/conference/atc22/presentation/sun](https://www.usenix.org/conference/atc22/presentation/sun)

---

# Q&A ([link](https://cseweb.ucsd.edu/~wgg/CSE210/howtoread.html))

**What are the motivations for this work?** 
* See Introduction.
* Kernel fuzzers like Syzkaller depend on system call specifications to generate test cases. Writing such specifi- cations requires an immense amount of domain knowledge while being extremely laborious. Meanwhile, automated generation of the specification is still an open problem due to the complexity of the kernel, including entry function extraction and input type identification.

**What is the proposed solution?**
* See Introduction, Section 4.
* Proposed KSG to generate system call specifications for kernel fuzzers automatically. 
    1. First, utilizes **probe-based tracing** to extract entry functions accurately. 
    2. Then, it uses path-sensitive analysis to collect **precise input types** and **range constraints** in each execution path of entry functions. 
    3. Based on the information in step 1 and 2, KSG generates specifications in the domain language Syzlang, which is used by most kernel fuzzers.
* An overview of KSG.
![overview](/images/posts/ksg/overview.png) 

**What is the work's evaluation of the proposed solution?**
* See Introduction.
* Evaluated KSG on several versions of the Linux kernel. It automatically generated **2433** unique specifications.
* Based on the coverage data reported by Syzbot’s dashboard, Syzkaller covers **an average of 38% of Linux kernel code** with the current Syzlang specifications for a prolonged time of fuzzing.
* Each experiment was repeated 3 times and executed over a period of 72 hours
* KSG generates 8 specialized calls per minute, with a total of 2433 specialized calls generated in 5 hours.
* With 1460 new specialized calls, Syzkaller+ and Moonshine+ achieved 22% and 23% coverage improvement, respectively.

**What is your analysis of the identified problem, idea and evaluation?**
> Entries can be registered dynamically in many scenarios, for instance, during kernel initialization and module loading.
> Consequently, it is challenging to extract entries using current static analysis methods. (*From Introduction Section*)
* For the challenge above, is it possible to brute force all the potential registrations before booting (or even during booting)?

**What are the contributions?**
* See Introduction.
* KSG, Kernel Specification Generator that automatically generates system call specifications for kernel fuzzers.
* The evaluation result shows that KSG generated 2433 specifications in total, which can improve the coverage of Syzkaller and Moonshine by 22% and 23% respectively, and assisted fuzzers to find 26 new bugs.

**What are future directions for this research?**
* See Section 7.
* Assembly code in kernel is not well handled by CSA, thus leading to the range constraints and type information being missed during the analysis procedure.

**What questions are you left with?**

NONE

**What is your take-away message from this paper?**

NONE