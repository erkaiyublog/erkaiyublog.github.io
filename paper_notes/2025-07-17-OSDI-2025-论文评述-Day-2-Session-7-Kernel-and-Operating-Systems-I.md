---
title: "OSDI 2025 论文评述 Day 2 Session 7: Kernel and Operating Systems I"
layout: post
---

IPADS-SYS

* Read: 17 Jul 2025
* Published: 10 Jul 2025

[https://zhuanlan.zhihu.com/p/1925925001460160336](https://zhuanlan.zhihu.com/p/1925925001460160336)

---

## Extending Applications Safely and Efficiently
[https://www.usenix.org/conference/osdi25/presentation/zheng-yusheng](https://www.usenix.org/conference/osdi25/presentation/zheng-yusheng)
基于eBPF的插件开发优化，将插件转二进制码并且由应用开发者和插件管理者定义权限。

## Tintin: A Unified Hardware Performance Profiling Infrastructure to Uncover and Manage Uncertainty
[https://www.usenix.org/conference/osdi25/presentation/li](https://www.usenix.org/conference/osdi25/presentation/li)
现代处理器配备了Hardware Performance Profiling（HPC），用以测量cache-miss、分支误预测等事件，受硬件限制，该测量方案存在误差。本文通过monitor, scheduler, manager三个模块对profiling进行优化。

## Building Bridges: Safe Interactions with Foreign Languages through Omniglot
