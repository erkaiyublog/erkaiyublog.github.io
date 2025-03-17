---
published: false
title: Source Code Study - QEMU cpu_exec.c
tags: qemu SCS
---

Lately, I've been trying to record assembly execution trace of QEMU emulations. To grasp a better understanding of this process, I decided to take a look at the source code of QEMU, ```accel/tcg/cpu_exec.c``` in particular. As I'm reading the source code, I write this blog as a record of what I've lreant from it. 

# Table of Contents
* TOC
{:toc}

# Background Knowlege 
WIP: TCG introduction

The QEMU version I studied is the [source code for stable release 9.2.2](https://github.com/qemu/qemu/tree/ea35a5082a5fe81ce8fd184b0e163cd7b08b7ff7). The file I'm focusing on is at [accel/tcg/cpu_exec.c](https://github.com/qemu/qemu/blob/ea35a5082a5fe81ce8fd184b0e163cd7b08b7ff7/accel/tcg/cpu-exec.c).  

# Functions by Order
Here's an overview of functions 


