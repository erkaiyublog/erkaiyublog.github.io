---
title: "Ninja: towards transparent tracing and debugging on ARM"
layout: post
---

Zhenyu Ning, Fengwei Zhang

* Read: 01 Jul 2025
* Published: 16 Aug 2017

SEC'17: Proceedings of the 26th USENIX Conference on Security Symposium Pages 33 - 49

[https://dl.acm.org/doi/10.5555/3241189.3241193](https://dl.acm.org/doi/10.5555/3241189.3241193)

---

# Q&A ([link](https://cseweb.ucsd.edu/~wgg/CSE210/howtoread.html))

**What are the motivations for this work?** 
* See Introduction.
* Most of the existing mobile malware analysis systems are based on emulation or virtualization technology, a series of anti-emulation and anti-virtualization techniques have been developed to challenge them.
* Although bare-metal based approaches eliminate the detection of the emulator or hypervisor, the artifacts introduced by the analysis tool itself are still detectable by malware.
* Transparency problem still challenges the state-of-the-art malware analysis systems.

**What is the proposed solution?**
* See Introduction.
* Consider the analysis system as an Environment + an Analyzer:
    * The Environment can be operating system, emulator, hypervisor, or sandbox.
    * The Analyzer can be instruction analyzer, API tracer, or application debugger.
* Three requirements:
    * The Environment must be isolated.
    * The Environment exists on an off-the-shelf bare-metal platform without modifying the software or hardware.
    * The Analyzer should not leave any detectable footprints to the outside of the Environment.
* Use **TrustZone technology, Performance Monitoring Unit (PMU), and Embedded Trace Macrocell (ETM)**.

**What is the work's evaluation of the proposed solution?**
* See Introduction.
* Evaluated the performance of the trace subsystem with several popular benchmarks, and the result shows that the overheads of the instruction trace and system call trace are less than 1% and the Android API trace introduces **4 to 154 times slowdown**.

**What is your analysis of the identified problem, idea and evaluation?**

NONE

**What are the contributions?**
* See Introduction.
* Ninja, implement a prototype of NINJA that embodies a trace subsystem with different tracing granularities and a debug subsystem with a GDB-like debugging protocol on ARM Juno development board.

**What are future directions for this research?**

NONE

**What questions are you left with?**

NONE

**What is your take-away message from this paper?**

NONE