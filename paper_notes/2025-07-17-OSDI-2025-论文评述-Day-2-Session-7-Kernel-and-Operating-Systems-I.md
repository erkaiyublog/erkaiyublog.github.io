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
**会议的两篇best paper之一。**

该论文旨在解决在与外部非安全语言交互时，如何系统性地维护主机语言（Rust）的Soundness这一核心问题。

## KRR: Efficient and Scalable Kernel Record Replay
现有的RR技术开销较高，KRR只记录内核。由于只需要重放内核，KRR只需要确保所有内核逻辑串行执行，用户态应用仍然可以在多核上并行执行。

## Deterministic Client: Enforcing Determinism on Untrusted Machine Code
Determinism在智能合约，区块链和可复制状态机等场景下非常重要。主旨是同一程序在给定相同输入的情况下，无论何时、何地或在哪个节点执行，都必须产生完全一致的输出与状态变化。现有工作基于WASM和JIT，有性能开销且TCB较大。本论文推出DeCl (deterministic client)，不依赖中间语言，直接在原生机器码上强制determinism。利用了SFI技术。
