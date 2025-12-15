---
title: "Nice to Meet You: Synthesizing Practical MLIR Abstract Transformers"
layout: post
---

Xuanyu Peng, Dominic Kennedy, Yuyou Fan, Ben Greenman, John Regehr, Loris D'Antoni

* Read: 10 Dec 2025
* Published: 06 Dec 2025

POPL 2026

[https://www.arxiv.org/abs/2512.06442](https://www.arxiv.org/abs/2512.06442)

---

# Q&A ([link](https://cseweb.ucsd.edu/~wgg/CSE210/howtoread.html))

**What are the motivations for this work?** 
* See Abstract, Introduction.
* When writing compiler optimizations, performing static analysis on the results is necessary. Each static analysis is based on a collection of **abstract transformers** that provide abstract semantics for the concrete instructions that make up a program.
* The current static analysis coverage for LLVM backend is limited. 
    1. Abstract transformers are usually started in a imprecise form and get optimized later. This process is very expensive.
    2. Across 17 different LLVM backends, only four have any abstract transformers at all for LLVM instructions representing target-specific (e.g. x86-64, RISC-V, AArch64) intrinsics. In this case, developers who substitute intrinsics for portable code while chasing performance can see degraded dataflow-driven optimizations in nearby code because the portable code could be analyzed but the intrinsics cannot.
* Operations that lack abstract transformers must be analyzed conservatively: they return *top* (concept from abstract representation, of course), the unknown value. 
* The work synthesizes abstract transformer for formally specified (using an MLIR dialect) contrect instruction semantics, it addresses the following research questions in the context of finite, Galois-connection-based abstract domains:
    * Is it practical to automatically generate abstract transformer for multiple domains that are used in real-world compilers using existing formally specified concrete instruction semantics?
    * How precise can it be while remaining sound?
    * Can the synthesis procedure navigate the search space without meaningful help from users? 

**What is the proposed solution?**
* See Introduction.
* Treat synthesis as an optimization problem. Use stochastic search techniques inspired by Stoke, where candidate transformers evolve through a sequence of random modifications guided by the cost function induced by the objective.


**What is the work's evaluation of the proposed solution?**
* See Introduction, Section 2.
* Validated their prototype by synthesizing transformers for three non-relational, compiler-friendly abstract domains (KnownBits, signed and unsigned ConstantRange) for 39 instructions that are present both in LLVM and in MLIR’s Arith dialect. 
* Evaluated by synthesizing abstract transformers for the abstract domains and operations used in the LLVM IR. The results show that NiceToMeetYou complements the precision of LLVM transformers (when measured on 8-bit and 64-bit integers) of 7/47 transformers in the KnownBits domain and 19/47 in ConstantRange. With the addition of a handwritten reduced-product operator for combining synthesized transformers across different abstract domains, the synthesized transformers exceed LLVM’s precision on 22/47 operators.

**What is your analysis of the identified problem, idea and evaluation?**

**What are the contributions?**
* See Introduction.
* Propose a framework for synthesizing abstract transformers that leverages existing formal semantics for instructions, and is not limited to specific abstract domains and does not require program templates
* Design an algorithm that incrementally synthesizes the meet of multiple abstract transformers, which enables an MCMC-based search procedure that can discover individual smaller transformers that can be added to the meet.
* Implement the algorithm in NiceToMeetYou, a tool that effectively balances MCMC-based exploration with SMT-based verification. Apply NiceToMeetYou on the bread-and-butter abstract domains from LLVM, namely KnownBits and ConstantRange
* Conduct an evaluation showing how NiceToMeetYou can synthesize abstract transformers for real LLVM operators.

**What are future directions for this research?**
* See Section 6, 9.
* According to the authors, their furture work is gonna be related to automatic production of additional elements of a compiler bug report.
* In my point of view, the algorithm of C-Reduction is clearly not optimal in terms of efficiency. It is basically:
    1. An outer while loop checking if fixpoint is reached
    2. An inner for loop iterating through all the pre-defined transformations. 
* I believe that the algorithm can be better designed to eliminate certain transformations during the process of search, which should improve the performance.

**What questions are you left with?**

NONE

**What is your take-away message from this paper?**

NONE