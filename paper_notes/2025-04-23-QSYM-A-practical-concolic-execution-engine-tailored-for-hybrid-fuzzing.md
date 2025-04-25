---
title: "QSYM: A practical concolic execution engine tailored for hybrid fuzzing"
layout: post
---

Insu Yun, Sangho Lee, Meng Xu, Yeongjin Jang, Taesoo Kim

* Read: 23 Apr 2025
* Published: 15 Aug 2018

SEC'18: Proceedings of the 27th USENIX Conference on Security Symposium Pages 745 - 761

https://dl.acm.org/doi/10.5555/3277203.3277260

---

Distinguished Paper Award Winner, USENIX Security 18.

The presentation for QSYM in USENIX Security 18: [link](https://www.youtube.com/watch?v=cXr1ZXp40jA) [slides](https://www.usenix.org/sites/default/files/conference/protected-files/usesec18_slides_yun.pdf).

# Q&A ([link](https://cseweb.ucsd.edu/~wgg/CSE210/howtoread.html))

**What are the motivations for this work?** 

* See Abstract, Introduction, Section 2.
* Want to use hybrid fuzzing to address the limitations of fuzzing and concolic execution.
* Hybrid fuzzers suffer from scaling to find real bugs in non-trivial, real-world applications. The ***performance bottlenecks of hybrid fuzzers' concolic executors*** are the main limiting factor that deters their adoption beyond the synthetic benchmarks.
* Symbolic emulation is ***too slow in formulating path constraints***. Current concolic executors mainly employ coarse-grained, basic-block-level taint tracking and symbolic emulation, which incur non-negligible overheads to the concolic execution.
* Ineffective snapshot.
* Slow and inflexible sound analysis.


**What is the proposed solution?**

* See Introduction, Section 2, Section 3, Section 4. 
* Hybrid fuzzer QSYM.
* Take a look at section 2 for motivations and proposed solutions. This is one of the most well-organized paper I have ever read!
* In Section 4, the authors mentioned that they implemented the concolic executor from scratch, and it took 16K lines of code in total.
* QSYM relies on Intel Pin for DBT, and its core components are implemented as Pin plugins written in C++: 12K LoC for the concolic execution core, 1.9K LoC for expression generation, and 1.5K LoC for handling system calls. QSYM also exposes Python APIs (0.5K LoC) such that users can easily extend the concolic executor; the hybrid fuzzer is built as a showcase using these APIs.
* A figure from Section 3 featuring an overview of QSYM's architecture.

![QSYM architechture](/images/posts/qsym/qsym.png)

* Some optimizations include:
    * Instruction-level symbolic execution.
    * Solving only relevant constraints.
    * Preferring re-execution to snapshoting.
    * Concrete external environment.
    * Optimistic solving.
    * Basic block pruning.

**What is the work's evaluation of the proposed solution?**

* See Section 5.
* Experiments ran on Ubuntu 14.04 LTS equipped with Intel Xeon E7-4820 (eight 2.0GHz cores) and 256GB RAM.
* Experiments performed on ***real-world software***, including the applications tested by OSS-Fuzz. OSS-Fuzz generated 10 trillion test inputs a day for a few months to fuzz the applications, but QSYM ran them for 3 hours using a single workstation and ***found 13 previously unknown bugs***.
* When testing real-world applications, compared with the state-of-the-art hybrid fuzzer, Driller. Driller generated less than 10 test cases on average for 30 minutes of running, whereas QSYM generated hundreds.
* Studied the code coverage result of QSYM on real-world applications. On libpng, with a 6-hour run, acheived 26.8% of code coverage provided 141 png images as initial seeds. 
* Used DARPA CGC dataset to compare runtime performance between QSYM and Driller (mainly due to constraint solving efficiency).

**What is your analysis of the identified problem, idea and evaluation?**

The paper is one of the most well-organized academic paper I have ever read. The idea of generating the constraints directly from source code rather than on top of LLVM IR is very innovative, the idea of generating constraints with LLVM IR dates back to [KLEE](/paper_notes/2025-04-22-KLEE-unassisted-and-automatic-generation-of-high-coverage-tests-for-complex-systems-programs), 2008. One of the limitation of this work might be the unhandled floating point constraints. Besides, it might be interesting to make use of MLIR to generate constraints, which may strike a balance in between efficiency of generating constraints for different programming languages and the runtime performance. 

**What are the contributions?**
* See Introduction, Section 3.
* Hybrid fuzzer QSYM.

**What are future directions for this research?**

NONE

**What questions are you left with?**

NONE

**What is your take-away message from this paper?**

NONE
