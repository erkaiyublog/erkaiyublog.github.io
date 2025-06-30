---
published: true
title: "Paper Review - Leveraging Binary Coverage for Effective Generation Guidance in Kernel Fuzzing"
tags: paper-reading firmware fuzzing arm 
---

This is a paper review of the work *Leveraging Binary Coverage for Effective Generation Guidance in Kernel Fuzzing*. 

Some basic info of [this paper](https://dl.acm.org/doi/10.1145/3658644.3690232):

```
Authors: Jianzhong Liu, Yuheng Shen, Yiru Xu, Yu Jiang

Published: 09 Dec 2024

CCS '24: Proceedings of the 2024 on ACM SIGSAC Conference on Computer and Communications Security Pages 3763 - 3777

```

***Table of Contents***
* TOC
{:toc}

Following [*How to Read an Engineering Research Paper*](https://cseweb.ucsd.edu/~wgg/CSE210/howtoread.html), I tried to summarize this paper by answering the questions below.

**What are the motivations for this work?** 

* See Abstract, Introduction, Section 3.
* ***Code coverage is not sufficient for operating system kernel fuzzers***, since kernels contain many untracked but interesting features, such as comparison operands, kernel state identifiers, flags, and executable code, within its data segments, that reflects different execution patterns.
* Current kernel fuzzing technologies can only allow fuzzers to explore a less-than-desired amount of the kernel's code and logic, typically ***reaching their limits within several days*** into a fuzzing campaign.
* AFL++ has included context (where the state of program was when the code was executed) into coverage metrics, but it is still ***restricted to covering the code section*** alone.
* As the figure shown below, a Linux v6.6.8 compiled to the amd64 target has 43% executable code and 57% data, so adding data to coverage feedback potentially allows 2.3x coverage than pure code-based approaches.
![kernel image binary](/images/posts/kbincov/binary.png)
* As the figure shown below, a significant amount of source code for ```ioctl``` syscall from Linux kernel v6.9 is compiled to data section, and it determines whether a certain handler in code section can be executed.
![code](/images/posts/kbincov/ioctl.png)


**What is the proposed solution?**

* See Introduction, Section 4.
* Abstract kernel behavior as its temporal and spatial memory access pattern during execution. Use this pattern as code coverage metrics.
* Optimizations needed for efficient representation of memory access pattern.
    * Address approximation for data embedded in instructions (faster, less accurate addr).
    * Use static analysis during instrumentation, ***tracing elision***, to remove tracing points that contribute redundant coverage info.
    * Multi-level cache-friendly coverage storage.
* Overview of the feedback system.
![overview](/images/posts/kbincov/overview.png)

**What is the work's evaluation of the proposed solution?**

* See Abstract, Section 6.
* Compared against state-of-the-art kernel fuzzers on recent versions of the Linux kernels. 
* Code and binary coverage increases of 7%, 7%, 9% and 87%, 34%, 61%, compared to Syzkaller, StateFuzz, and IJON.
* 1.74x overhead, while StateFuzz is 2.5x, IJON is 2.2x.
* In Section 6, the authors presents four eval criteria:
    * Overall effectiveness in coverage improvements.
    * Overall efficiency and accuracy.
    * Effectiveness of individual and component-wise design choices.
    * Real-world bug detection capabilities.
* Branch coverage with KVM gets to an average of 124141 branches on the three versions of Linux kernel.
![branchcov](/images/posts/kbincov/branchcov.png)
* Binary coverage statistics, with units in KiB.
![bincov](/images/posts/kbincov/bincov.png)
* Combining binary coverage and branch coverage.
![branchbincov](/images/posts/kbincov/branchbincov.png)
* The coverage experiments are conducted for 72 hours. All bugs listed were found in this experiment. Each experiment was repeated 5 times to reduce statistical errors. 

**What is your analysis of the identified problem, idea and evaluation?**

They didn't open source their code.

**What are the contributions?**
* See Abstract.
* A new execution feedback and novelty detection mechanism, Kernel Binary Coverage Feedback.
* A prototype tool KBinCov integrated into kernel fuzzer syzkaller. 
* Higher code and binary coverage, lower overhead.
* Found 21 previously unknown bugs using KBinCov with Syzkaller.

**What are future directions for this research?**

NONE

**What questions are you left with?**

NONE

**What is your take-away message from this paper?**

* Including memory pattern in kernel code coverage metrics can be benefitial as it includes more info. 
* The memory overhead introduced by memory pattern is affordable, while the run-time overhead might be higher than state-of-the-art implementations. In the experiments, it takes KBinCov longer execution time to really achieve a better result.
* In Section 6.2, the authors explained that QEMU-KVM has significantly higher coverage statistics than QEMU-TCG, due to the fact that ***QEMU enabled a different set of devices under the two modes, thus affecting the number of modules the kernel loads during initialization***, and in turn resulting in a difference in the figures when saturated.