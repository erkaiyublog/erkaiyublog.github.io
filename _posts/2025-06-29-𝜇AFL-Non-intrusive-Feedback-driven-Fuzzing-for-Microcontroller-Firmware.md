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
A presentation given by one of the authors can be found at [this link](https://www.youtube.com/watch?v=hGIrsQqHXDg). 

***Table of Contents***
* TOC
{:toc}

Following [*How to Read an Engineering Research Paper*](https://cseweb.ucsd.edu/~wgg/CSE210/howtoread.html), I tried to summarize this paper by answering the questions below.

## What are the motivations for this work?

See Introduction, Section 2, Section 3.
### Limitation of Existing Works
There are limitations in the existing firmware security testing approaches when applied to embedded firmware testing:
![table1](/images/posts/microafl/table1.png)
Specifically, in the Introduction section, the authors categorized the existing firmware security testing approaches as:
* **Emulation-based rehosting technique** (extensively studied recently): the main challenge is to accurately model the behavior of diverse peripherals. Some attempted to learn an approximate peripheral model using symbolic execution, access-pattern matching, and machine learning techniques, but they are inaccurate in general.
* **Porting based on HAL**: leverage the HAL available in MCU firmware to avoid modeling peripherals. Limited to peripheral drivers that are available in HAL.
* **Hardware in the loop**: QEMU + peripheral I/O fowarding. Require frequent and expensive switching (and syncing) between QEMU and hardware.

### Difficulty with MCU
In Section 2, the authors introduced the difficulty in dynamic analysis of MCU firmware. Unlike traditional software which assumes an OS layer that provides an abstract view of hardware, MCU firmware **runs on bare metal** or only includes an OS library (e.g. RTOS) for simple multi-task management. Therefore, it compiles the driver code of peripherals and the application code **together to form a single-address-space program**. The peripheral I/O operation is performed by accessing the memory-mapped registers.

## What is the proposed solution?
See Introduction, Section 2.
### Hardware
Use ARM ETM (Embedded Trace Macrocell) for non-intrusive feedback collection.

ETM is a hardware instruction trace feature for ARM. The design of ETM is: A dedicated hardware component emits a stream of control flow packets, a decoder reconstructs a unique execution path by matching the control flow data to the disassembled machine code.
### Trace Collection
For trace collection, in the documentation of ETM there were two different ways. However, the authors found that ETB (Embedded Trace Buffer) is **rarely supported on real chips**. They instead use a physical parallel port called [Cortex Debug+ETM connector](https://documentation-service.arm.com/static/5fce6c49e167456a35b36af1) to stream the trace data to an external debugger.
### Control Flow Packets
ETM emits packets that contain information for: conditional branches, indrect branches, asynchronous events, and optionally direct branches. Exception entries and returns can be properly paired using the packets.
### Trace Filtering
Trace generation can be controlled in three ways:
1. Event-based.
2. A code region can be included or excluded from tracing. (By setting a pair of address comparators)
3. By the trace start/stop block.
The authors found that **only the last method** is supported by ARM MCUs.

### Design Overview
An online trace collector + an offline trace analyzer, interact with AFL.
![overview](/images/posts/microafl/overview.png)
### Lovel-level Device Control and Fuzzing Scheduling
Use the JTAG or SWD interface for low-level control of the target device, which enables the dbug dongle to directly access the processor registers and the device memory.

Use the SEGGER RTT protocol for high-speed transmission of the testcases. Transmit the subsequent testcase in the background while the target is running against the current testcase.
### Trace Collection and Filtering
Address-based filter and event-based filter can be implemented in the online trace collector using DWT, but with limitations.

"Instruction trigger" is a type of event-based filter that treats the execution of an instruction in a particular memory range as an event. Using this trigger, the authors managed to set the trace filter to filter out the trace for device booting process.
![instruction trigger](/images/posts/microafl/instructiontrigger.png)

"Data trigger" is a type of event-based filter that treats R/W a particular value from/to a particular address as an event. Using this trigger and the fact that in the RTOS environment each task has its task control block (TCB) in a fixed location, the authors managed to filter out the execution trace of uninterested tasks and the OS kernel, such as interrupt handlers and scheduling.
### Calculate Branch Information without Decoding ETM
Attempted to use simply branch target addresses to reconstruct branching info, but by default ETM doesn't emit addresses for direct branches. The authors found in the ETM manual that this behavior can be overriddent by setting the eighth bit of the ETMCR register, this enabled them to differentiate tricky paths.

### LCSAJ
Introduced LCSAJ_BB, an instruction sequence starting from the last taken branch target and ending with the following branch instruction which is actually taken. In the example shown below, the path A -> C has A as a LCSAJ_BB, while the path A -> B -> C has A concatenated with B as a LCSAJ_BB.

![bb](/images/posts/microafl/bb.png)

Used an algorithm to hash LCSAJ_BB transmission info into BB_ID, which captures the same path sensitivity as AFL's bitmap, without too many collisions.

### Offline Filtering
Since exception is nondeterministic, it cannot be filtered by the online trace collector. The authors leverage the exception information embedded in the ETM branch packets to figure out the exception entry points and exit points.

### Crash/Hang Detection
Use the vector catching feature to mark the exceptions in concerns. With vector catching, when such an exception happens, instead of trapping to the corresponding handlers, the chip enters debug state which can be automatically captured by the debug dongle.

## What is the work's evaluation of the proposed solution?

See Section 4.

### Implementation
Implemented a prototype of 𝜇AFL on top of AFL2.56 by adding ∼2,000 lines of C code on the PC side. Used the SEGGER J-Trace Pro debug dongle to control the communication between the host PC and the target ARM Cortex-M evaluation boards.

Experiments were done to NXP TWR-K64F120M and STM32H7B3I-EVAL. Used the sample code provided in NXP SDK and STM32 SDK.

Following is background info about the two boards generated by GPT
![nxp](/images/posts/microafl/nxp.png)
![stm](/images/posts/microafl/stm.png)

### Firmware Samples Used in Evaluation
Used sample code provided in the NXP SDK, including Fibonacci, I2C, UART, USB, SD Card, Enet, MMCAU. Most of the samples are used for testing communication protocols.

A noticible issue with the samples is that the application-level logic of all the samples is very simple (this is pointed out by the authors themselves).

### Overhead of Fully Recovering Instruciton Flow
Compared with Capstone to demonstrate how using LCSAJ_BB-based approach can reduce performance overhead on PC.

### Overhead Breakdown
A figure presents the overhead breakdown.
![overhead](/images/posts/microafl/overhead_break.png)

## What is your analysis of the identified problem, idea and evaluation?

The fact that the ETM tracing module doesn't instrument the code sounds pretty nice.

## What are the contributions?

* See Introduction.
* 𝜇AFL, the first fuzzing tool that is applicable to the driver code of MCU firmware.
* Proposed using ARM ETM for non-intrusive feedback collection. Used Linear Code Sequence And Jump (LCSAJ) analysis to directly process raw ETM data without expensive disassembling.
* Some bugs detected.

## What are future directions for this research?

To be added

## What questions are you left with?

For multi-tasked MCU firmware (briefly mentioned in Section 3.3), will the execution trace be correctly annotated? How is this achieved?

## What is your take-away message from this paper?

To be added