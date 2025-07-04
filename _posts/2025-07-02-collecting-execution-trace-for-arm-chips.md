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

![frankentrace overview](/images/posts/trace_arm/frankentrace.png)


# Tracing with ETM
## What is ETM?

## HATBED

# References
1. FrankenTrace: Low-Cost, Cycle-Level, Widely Applicable Program Execution Tracing for ARM Cortex-M SoC [https://doi.org/10.1145/3576914.3587521](https://doi.org/10.1145/3576914.3587521) 

2. HATBED: A Distributed Hardware Assisted Testbed for Non-invasive Profiling of IoT Devices [https://dl.acm.org/doi/10.1145/3312480.3313172](https://dl.acm.org/doi/10.1145/3312480.3313172)
