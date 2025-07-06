---
published: true
title: Collecting Execution Trace for ARM Chips
tags: arm firmware
---

In this blog, I’ll walk through the basics of execution tracing for ARM chips. I will mainly focus on methods based on ETM - a powerful hardware feature found in many ARM processors that enables **non-intrusive**, **cycle-accurate** tracing of instruction execution. Unlike breakpoints or printf-style debugging, ETM allows developers to observe exactly what code was executed without altering the timing or behavior of the program. 

This blog only focuses on methods **based on hardware** features.

***Table of Contents***
* TOC
{:toc}

# Tracing Without ETM
Before introducing execution tracing based on ETM, I'd like to first introduce some alternatives.

## FrankenTrace
The paper [*FrankenTrace: Low-Cost, Cycle-Level, Widely Applicable Program Execution Tracing for ARM Cortex-M SoC*](https://doi.org/10.1145/3576914.3587521) introduces a method for collecting traces without relying on ETM.

FrankenTrace collects full cycle-level traces by **repeatedly executing** the target software with varied tracing configurations and then merging the results. This approach overcomes the bandwidth limitations of low-speed debug interfaces (like SWO) without needing expensive ETM-based hardware.

The difficulty in trace collection without using ETM can be illustrated by the following example from the paper: "As a point of reference, let us consider the Texas Instruments CC2650 SoC, deployed as an experimental node in the 1KT testbed. This SoC is clocked at **48 MHz** and features two single-cycle memories. A tracing packet with a value of PC, as sampled by DWT, has a size of 5 B. A timestamp packet of ITM, in turn, has a size of 1–5 B. With the 6.4 Mbps of SWO throughput, tracing is inherently **limited to about 100K samples per second**. In effect, PC can be sampled approximately only once every 512 CPU cycles."

Hardware features required: 
* DWT (Debug Watchpoint and Trace): Configured to sample PC or monitor memory accesses.
* ITM (Instrumentation Trace Macrocell): Receives trace packets and timestamps them.
* SWO (Single Wire Output): Used to output trace data over UART (low bandwidth).

The trace can be collected with cheap logical analyzers or UART-USB chips.

FrankenTrace supports generating two types of traces: a noninvasive cycle-level PC trace and a cycle-level LSU trace of varying invasiveness.

Below is a screeshot from the paper, illustrating **half** of the workflow (I included only the workfow involving cycle-level PC trace).
![frankentrace overview](/images/posts/trace_arm/frankentrace.png)
In the paper, the authors used DWT to sample PC at exactly N-cycle intervals, where the value of N is configurable. Since PC is sampled upon a transition of a configured tap bit of the cycle counter, to control the sampling offset for different *i*, the authors adjusted the point at which the sample is taken by "carefully manipulating the DWT configuration and by modifying the initial value of the counter".

Despite the fact that FrankenTrace can collect execution traces at much lower cost and without requiring ETM, I personally find this method not entirely reliable. The instruction trace is reconstructed based on N **independent** executions (via ```system reset```), and the nondeterminism in these executions is simply non-negligible. The authors also point out that, due to nondeterminism, it often takes **more than N executions** to obtain consistent and useful traces. To determine consistency, the authors insert a special piece of code with a known trace before the analyzed code and later use this to check for validity.

If the goal of collecting an execution trace is to obtain a precise reflection of the entire execution process, then trace collection based on re-execution seems somewhat unreliable to me.

That said, I can certainly imagine scenarios where FrankenTrace would be useful:
1. When one wants to collect execution traces for an ARM chip **without ETM** support (this is somewhat common among Cortex-M3-based SoCs, a related table can be found in the paper).
2. When one wants to reverse engineer some closed-source hardware by efficiently collecting its execution trace, without requiring strong consistency.

For the actual experiments, the authors employed a **USB logic analyzer** based on a popular Cypress CY7C68013A chip, which allows for reliable sampling at up to **24 MHz**. For robust symbol recovery they sampled 3 times per symbol, achieving a maximum effective UART symbol rate of **8 Mbaud**. The captured data was then processed in software by two decoders from the open-source sigrok project: first by a UART decoder and then by an ARM ITM decoder. They modified the latter to save the decoded tracing packets to a CSV that can be subsequently processed to generate the final trace.

> Mbaud stands for Mega Baud, which means 1 million baud. It is a unit of symbol rate. ***Baud rate*** refers to the **number of symbols** transmitted per second. A symbol is a "unit" of communication that the receiver can distinguish from others. In binary communication (like UART): 1 symbol = 1 bit

# Tracing with ETM
## What is ETM?
ETM stands for Embedded Trace Macrocell. It is a real-time trace module providing **instruction and data tracing** of a processor. 

People like ETM since it provides **non-intrusive, cycle-accurate, real-time tracing** 🍺.  

There is more than one webpage introducing ETM on the Arm website, the high-level one is called [Embedded Trace Macrocell Architecture Specification](https://developer.arm.com/documentation/ihi0014/q?lang=en) (I will refer to it as ***ETM Arch Spec*** from here on). 

## Where Do I Find the Specs?
One thing to keep in mind is that online documents by Arm are usually quite large. A good way to start is by reading the *Preface* section and search for subtitles like *Intended Audience* and *How to Use This Specification*.

Noticeably, in the [*Additional Reading*](https://developer.arm.com/documentation/ihi0014/q/preface/Additional-reading/The-ETM-documentation-suite?lang=en) section of ***ETM Arch Spec***, there is a guide for what **specific documentation** to look for within the so-called *ETM documentation suite*. ***ETM Arch Spec*** is part of the *ETM documentation suite* and contains information that is relevant to all implementations of the ETM. The other manuals in the ETM documentation suite are **implementation specific**. Two points from this section that attract my attention are:
1. There is an ETM Technical Reference Manual (TRM) describing the implementation defined behavior of the ETM.
2. For the Cortex-M3 processor, the CoreSight ETM-M3 is described in the Cortex-M3 Technical Reference Manual (ARM DDI 0337).

## What is CoreSight?
**CoreSight** is another concept that comes closely with **ETM**. At a high level, ETM is a feature that captures trace data from a specific CPU core, and CoreSight is an infrastructure that collects trace data from sources like ETM, routes it through components (e.g., funnels, replicators), and outputs it to a trace sink (e.g., TPIU or memory). 

The CoreSight architecture provides **a set of** standard **interfaces** and programmer model **views**. ETM is one of them.

![coresight overview](/images/posts/trace_arm/coresight_overview.png)


## Ninja
[Ninja](https://dl.acm.org/doi/10.5555/3241189.3241193) leverages a hardware-assisted isolated execution environment Trust-Zone to transparently trace and debug a target application with the help of Performance Monitor Unit (PMU) and Embedded Trace Macrocell (ETM). 

An overview of the Ninja architecture:
![arch](/images/posts/trace_arm/ninja_arch.png)

The main idea of this design is that the "normal OS" (Linux or Android) on the left is being executed in **normal domain**, and the Ninja system on the right is being ran in **secure domain**. Such isolation makes use of the hardware-based **TrustZone** feature provided by ARM.

The experiments were conducted on a **64-bit ARMv8 Juno r1 board**, which has two **ARM Cortex-A57** cores and four ARM Cortex-A53 cores.

As mentioned in Section 5.3.1, there are **funnels** controlled by a group of CoreSight Trace Funnel (CSTF) registers, the ETM trace is filtered by these funnels. For Ninja's experiment, the authors only collected traces for the core 0 in the Cortex-A57 cluster. The filtered result is output to **Embedded Trace FIFO (ETF)** which is controlled by Trace Memory Controller (TMC) registers. 

The ETM is controlled by a group of trace registers. The following setting is applied to the Ninja system:
1. Set all ```EXLEVEL_S``` bits and clear all ```EXLEVEL_NS``` bits of the ```TRCVICTLR``` register to collect trace for non-secure EL0 and non-secure EL1.
2. Set the ```EN``` bit of the ```TRCPRGCTLR``` register to start the instruction trace.
3. Clear the ```EN``` bit of the ```TRCPRGCTLR``` register to disable ETM.
4. Set the ```StopOnFl``` bit and the ```FlushMan``` bits of ```FFCR``` register in the ```TMC``` registers to stop the ETF. 
5. To **read the trace**, keep reading from the ```RRD``` (RAM Read Data) register until ```0xFFFFFFFF``` is fetched.

The authors also mentioned in Section 5.3.2 how to trace system calls with ETM and PMU.

Experiments related to the tracing subsystem is introduced in Section 7.1. The authors wrote a simple Android application that uses Java Native Interface to read the ```/proc/self/status``` file line by line. The file contains 38 lines in total, it takes about 0.22 ms to finish executing. After the execution, the ETF contains **9.92 KB** encoded trace data, and the datarate is approximately **44.03 MB/s**. After decoding with ptm2human, the decoded data contains **1341 signpost instructions**.

> **Signpost instructions** are used to help tools or developers identify and interpret events in the trace or log data. Examples of what they mark: Beginning and end of critical sections; Context switches; Specific events like acquiring a lock or sending a message. 

## HATBED
HATBED from [HATBED: A Distributed Hardware Assisted Testbed for Non-invasive Profiling of IoT Devices](https://dl.acm.org/doi/10.1145/3312480.3313172) is a testbed designed for IoT devices equipped with ARM Cortex-M3/M4 processors, utilizing standardized built-in debugging units and general hardware-assisted tracing technologies.

They used a CY7C68013A based logic analyzer (controlled by sigrok) to collect the ETM trace.

Unfortunately, I failed to understand the technical details provided by the authors in the paper, and there was no clear description of their workflow. 

I found this [repository](https://github.com/PetteriAimonen/STM32_Trace_Example) which presumably inspired the authors of HATBED.

# References
1. FrankenTrace: Low-Cost, Cycle-Level, Widely Applicable Program Execution Tracing for ARM Cortex-M SoC [https://doi.org/10.1145/3576914.3587521](https://doi.org/10.1145/3576914.3587521) 

2. HATBED: A Distributed Hardware Assisted Testbed for Non-invasive Profiling of IoT Devices [https://dl.acm.org/doi/10.1145/3312480.3313172](https://dl.acm.org/doi/10.1145/3312480.3313172)

3. Tracing on STM32 discovery [https://essentialscrap.com/tips/arm_trace/theory.html](https://essentialscrap.com/tips/arm_trace/theory.html)

4. Ninja: Towards Transparent Tracing and Debugging on ARM [https://dl.acm.org/doi/10.5555/3241189.3241193](https://dl.acm.org/doi/10.5555/3241189.3241193)

5. Embedded Trace Macrocell Architecture Specification ETMv1.0 to ETMv3.5 [https://developer.arm.com/documentation/ihi0014/q](https://developer.arm.com/documentation/ihi0014/q) (***ETM Arch Spec***)