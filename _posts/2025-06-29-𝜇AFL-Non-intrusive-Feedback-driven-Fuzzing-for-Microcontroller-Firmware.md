---
published: true
title: "Paper Review - 𝜇AFL: Non-intrusive Feedback-driven Fuzzing for Microcontroller Firmware"
tags: paper-reading firmware fuzzing arm 
---

This is a paper review of the work *𝜇AFL: Non-intrusive Feedback-driven Fuzzing for Microcontroller Firmware*. 

Some basic info of [this paper](https://doi.org/10.1145/3510003.3510208):

```
Authors: Wenqiang Li, Jiameng Shi, Fengjun Li, Jingqiang Lin, Wei Wang, Le Guan

Published: 05 July 2022

ICSE '22: Proceedings of the 44th International Conference on Software Engineering, Pages 1 - 12
```
A presentation given by one of the authors can be found at [this link](https://www.youtube.com/watch?v=hGIrsQqHXDg). Unfortunately, I failed to find the slides in this talk, so I had to include them as screenshots in this blog.


Following [*How to Read an Engineering Research Paper*](https://cseweb.ucsd.edu/~wgg/CSE210/howtoread.html), I tried to summarize this paper by answering the questions below.

## What are the motivations for this work?

* See Introduction.
* There are limitations in the existing firmware security testing approaches when applied to embedded firmware testing:
![table1](/images/posts/microafl/table1.png)

## What is the proposed solution?

* See Introduction, Section 2.
* Use ARM ETM (Embedded Trace Macrocell) for non-intrusive feedback collection.
* ETM is a hardware instruction trace feature for ARM. The design of ETM is: A dedicated hardware component emits a stream of control flow packets, a decoder reconstructs a unique execution path by matching the control flow data to the disassembled machine code.

## What is the work's evaluation of the proposed solution?

See Section 4.
* Implemented a prototype of 𝜇AFL on top of AFL2.56 by adding ∼2,000 lines of C code on the PC side.
* Used the SEGGER J-Trace Pro debug dongle to control the communication between the host PC and the target ARM Cortex-M evaluation boards.
* Experiments were done to NXP TWR-K64F120M and STM32H7B3I-EVAL. Used the sample code provided in NXP SDK and STM32 SDK.

## What is your analysis of the identified problem, idea and evaluation?

To be added

## What are the contributions?

* See Introduction.
* 𝜇AFL, the first fuzzing tool that is applicable to the driver code of MCU firmware.
* Some bugs detected.

## What are future directions for this research?

To be added

## What questions are you left with?

To be added

## What is your take-away message from this paper?

To be added