---
title: "𝜇AFL: Non-intrusive Feedback-driven Fuzzing for Microcontroller Firmware"
layout: post
---

Wenqiang Li, Jiameng Shi, Fengjun Li, Jingqiang Lin, Wei Wang, Le Guan

* Read: 13 Jun 2025
* Published: 05 July 2022


ICSE '22: Proceedings of the 44th International Conference on Software Engineering, Pages 1 - 12

https://doi.org/10.1145/3510003.3510208

---

# Q&A ([link](https://cseweb.ucsd.edu/~wgg/CSE210/howtoread.html))

**What are the motivations for this work?** 

* See Introduction.
* There are limitations in the existing firmware security testing approaches when applied to embedded firmware testing:
![table1](/images/posts/microafl/table1.png)

**What is the proposed solution?**

* See Introduction, Section 2.
* Use ARM ETM (Embedded Trace Macrocell) for non-intrusive feedback collection.
* ETM is a hardware instruction trace feature for ARM. The design of ETM is: A dedicated hardware component emits a stream of control flow packets, a decoder reconstructs a unique execution path by matching the control flow data to the disassembled machine code.

**What is the work's evaluation of the proposed solution?**

See Section 4.
* Implemented a prototype of 𝜇AFL on top of AFL2.56 by adding ∼2,000 lines of C code on the PC side.
* Used the SEGGER J-Trace Pro debug dongle to control the communication between the host PC and the target ARM Cortex-M evaluation boards.
* Experiments were done to NXP TWR-K64F120M and STM32H7B3I-EVAL. Used the sample code provided in NXP SDK and STM32 SDK.

**What is your analysis of the identified problem, idea and evaluation?**

NONE

**What are the contributions?**

* See Introduction.
* 𝜇AFL, the first fuzzing tool that is applicable to the driver code of MCU firmware.
* Some bugs detected.

**What are future directions for this research?**

NONE


**What questions are you left with?**

NONE

**What is your take-away message from this paper?**

NONE