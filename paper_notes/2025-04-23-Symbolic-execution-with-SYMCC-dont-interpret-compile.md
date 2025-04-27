---
title: "Symbolic execution with SYMCC: don't interpret, compile!"
layout: post
---

Sebastian Poeplau, Aurélien Francillon

* Read: 26 Apr 2025
* Published: 12 Aug 2020

SEC'20: Proceedings of the 29th USENIX Conference on Security Symposium, Article No.: 11, Pages 181 - 198

https://dl.acm.org/doi/10.5555/3489212.3489223

---

# Q&A ([link](https://cseweb.ucsd.edu/~wgg/CSE210/howtoread.html))

**What are the motivations for this work?** 

* See Abstract, Introduction.
* Want faster speed with practical symbolic execution.
* Observation of the fact that symolic execution is implemented as essentially an interpreter. 
* Want to replace innterpretation with compilation to acheive better performance.

**What is the proposed solution?**

* See Introduction, Section 3.
* A compilation-based approach to symbolic execution that ***performs better than state-of-the-art implementations by orders of magnitude***.
* Embed the symbolic processing into the target program.
* At each branch point in the program, the "symbolized" binary will ***generate an input*** that deviates from the current execution path.

**What is the work's evaluation of the proposed solution?**

* See Section 5.

**What is your analysis of the identified problem, idea and evaluation?**
To be added.

**What are the contributions?**
* See Abstract.
* SymCC, an LLVM-based C and C++ compiler that builds ***concolic execuition right into the binary***.

**What are future directions for this research?**

NONE

**What questions are you left with?**

NONE

**What is your take-away message from this paper?**

NONE
