---
published: false
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

The paper *FrankenTrace: Low-Cost, Cycle-Level, Widely Applicable Program Execution Tracing for ARM Cortex-M SoC* introduces a method for collecting traces without relying on ETM.


# References
1. FrankenTrace: Low-Cost, Cycle-Level, Widely Applicable Program Execution Tracing for ARM Cortex-M SoC [https://doi.org/10.1145/3576914.3587521](https://doi.org/10.1145/3576914.3587521) 
