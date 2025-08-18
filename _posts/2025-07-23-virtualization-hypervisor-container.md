---
published: true 
title: Demystifying Virtualization - Emulators, Hypervisors, and Containers Explained
tags: hypervisor container qemu
---

In the world of software and systems, terms like virtualization, emulation, hypervisor, and container are often used interchangeably — but they refer to very different technologies. Whether you're a developer, a student, or just curious about how modern computing environments are built, understanding these concepts is essential. This post will break down what each term means, how they differ, and when you might use one over the other.

***Table of Contents***
* TOC
{:toc}

# What is a Hypervisor?
A hypervisor is a type of software that enables virtualization, allowing multiple operating systems (OSes) to run simultaneously on a single physical machine by **abstracting and managing hardware resources**.

There are two types of hypervisors: Type 1 (bare-metal) and Type 2 (hosted).

## What is a Type 1 Hypervisor (Bare-metal)?
Type 1 hypervisors run directly on hardware, replacing the traditional OS.

Examples: Xen, VMware ESXi, Microsoft Hyper-V, KVM. 

### More on KVM
KVM is special because it turns **the Linux kernel itself into a hypervisor**. 

KVM is not a software, but a set of kernel modules that turn the Linux kernel into a type 1 hypervisor. This means any Linux distribution with KVM support can act as a full-fledged hypervisor.

KVM provides the kernel-level infrastructure for managing and monitoring resources allocated to the VMs, but it needs a user-space component to handle I/O, devices, and system emulation. QEMU usually acts as such a user-space component. QEMU can emulate hardware devices including CPU, motherboard, etc. When used with KVM, QEMU delegates CPU and memory virtualization to the kernel, which is much faster.

## What is a Type 2 Hypervisor (Hosted)?
Type 2 hypervisors run on top of a regular OS, like an application. The host OS manages actual hardware; the hypervisor just manages virtual machines.

Examples: VirtualBox, VMware Workstation, QEMU (in user mode).

## What is Xen? XenServer?
Xen is a hypervisor. XenServer is the platform made by Citrix, and the source of the fork that gave XCP-ng. XenServer is also known as Citrix Hypervisor.

XCP-ng and XenServer are both downstream projects of the Xen project.

### Why Do We Need Xen?
Xen for ARM can be really useful. In the automotive domain, Xen is particularly well-suited due to its ability to statically partition hardware resources. For example, a single motherboard with multiple CPUs can be allocated across distinct virtual machines, such as one dedicated to infotainment and another to electronic control units (ECUs) or other safety-critical functions. This approach ensures strict isolation: Xen enforces complete separation of memory and resources, thereby mitigating risks of security breaches and data leakage.

AWS initially ran many of its servers on Xen. Over time, they developed their own hardware–software stack called Nitro, which consists of custom DPUs (Data Processing Units) and the Nitro Hypervisor. Unlike general-purpose Xen, the Nitro Hypervisor is lightweight and designed specifically to work with AWS’s custom Nitro hardware.

# What is HVM?
HVM stands for Hardware Virtual Machine, a virtualization mode that uses hardware extensions provided by modern CPUs (like Intel VT-x or AMD-V) to enable full virtualization of guest operating systems.

It allows **a guest OS to run unmodified**, as if it were on real hardware.

# References
1. [A brief synopsis of KVM, HVM, and VFIO](https://forums.unraid.net/topic/37013-a-brief-synopsis-of-kvm-hvm-and-vfio/).
2. [Xen Project Beginners Guide](https://wiki.xenproject.org/wiki/Xen_Project_Beginners_Guide).
3. [Xen vs XenServer vs KVM vs Proxmox](https://forums.lawrencesystems.com/t/xen-vs-xenserver-vs-kvm-vs-proxmox/14256).