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

## HATBED

# References
1. FrankenTrace: Low-Cost, Cycle-Level, Widely Applicable Program Execution Tracing for ARM Cortex-M SoC [https://doi.org/10.1145/3576914.3587521](https://doi.org/10.1145/3576914.3587521) 

2. HATBED: A Distributed Hardware Assisted Testbed for Non-invasive Profiling of IoT Devices [https://dl.acm.org/doi/10.1145/3312480.3313172](https://dl.acm.org/doi/10.1145/3312480.3313172)
