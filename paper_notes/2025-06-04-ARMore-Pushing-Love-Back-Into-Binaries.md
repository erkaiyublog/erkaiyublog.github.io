---
title: "ARMore: Pushing Love Back Into Binaries"
layout: post
---

Luca Di Bartolomeo, Hossein Moghaddas, Mathias Payer

* Read: 04 Jun 2025
* Published: 09 Aug 2023

SEC '23: Proceedings of the 32nd USENIX Conference on Security Symposium Article No.: 353, Pages 6311 - 632

https://dl.acm.org/doi/10.5555/3620237.3620590

---

# Q&A ([link](https://cseweb.ucsd.edu/~wgg/CSE210/howtoread.html))

**What are the motivations for this work?** 

* See Abstract.
* Most research on static rewriting has so far focused on the x86 architecture.

**What is the proposed solution?**

* See Introduction.
* A challenge for rewriting ARM binaries: ARM binaries rely on compressed jump tables, where for each possible entry in the jump table, only the offset of each case is stored (usually, only 1 or 2 bytes), making jump tables particularly hard to statically distinguish from random data. A static binary rewriter must detect and adjust those offsets to accommodate the instrumentation that might change the distance from the base case.
* ARMore leverages symbolization to reconstruct control flow after the insertion of instrumentation.
* Employs only sound techniques (layout replication and rebound table) rather than using heuristics for pointer recovery.

**What is the work's evaluation of the proposed solution?**

See Section 4.

**What is your analysis of the identified problem, idea and evaluation?**

NONE

**What are the contributions?**

* See Introduction.
* ARMore, a new and heuristic-free approach of instrumenting aarch64 binaries containing arbitrary pointer arithmetic or data interleaved with code.

**What are future directions for this research?**

* See Introduction.
* Flag-Use OBC is sensitive to the structure of the object-code—albeit far less so than OBC—and the fault- finding ability of test suites satisfying Flag-Use OBC (as well as MC/DC for that matter) is not as strong as one would like.


**What questions are you left with?**

NONE

**What is your take-away message from this paper?**

NONE