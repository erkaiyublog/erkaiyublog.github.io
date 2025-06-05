---
title: "From hack to elaborate technique - A survey on binary rewriting"
layout: post
---

Matthias Wenzl, Georg Merzdovnik, Johanna Ullrich, Edgar Weippl

* Read: 05 Jun 2025
* Published: 18 Jun 2019


ACM Computing Surveys (CSUR), Volume 52, Issue 3 Article No.: 49, Pages 1 - 37

https://doi.org/10.1145/3316415

---

# Q&A ([link](https://cseweb.ucsd.edu/~wgg/CSE210/howtoread.html))

**What are the motivations for this work?** 

* See Abstract, Section 2.
* Most research on static rewriting has so far focused on the x86 architecture.
* One of the main challenges for static instrumentation is resolving pointer targets. Due to aliasing, the target for references is unknown during translation. Without runtime information, static rewriters need to rely on complex static analysis, which is inherently imprecise and often resorts to heuristics. Consequently, static rewriting cannot support packed binaries or self-modifying code.

**What is the proposed solution?**

* See Introduction.
* A challenge for rewriting ARM binaries: ARM binaries rely on compressed jump tables, where for each possible entry in the jump table, only the offset of each case is stored (usually, only 1 or 2 bytes), making jump tables particularly hard to statically distinguish from random data. A static binary rewriter must detect and adjust those offsets to accommodate the instrumentation that might change the distance from the base case.
* ARMore leverages symbolization to reconstruct control flow after the insertion of instrumentation.
* Employs only **sound techniques** (layout replication and rebound table) rather than using heuristics for pointer recovery.

**What is the work's evaluation of the proposed solution?**

See Introduction.
* Rewrote binaries from 239 Debian packages (including C++ and Go binaries), and ran the relevant testsuite of each package, passed 97.5% tests.

**What is your analysis of the identified problem, idea and evaluation?**

NONE

**What are the contributions?**

* See Introduction.
* A new and heuristic-free approach of instrumenting aarch64 binaries containing arbitrary pointer arithmetic or data interleaved with code.
* A new mechanism, the rebound table,to recover from statically-unresolvable indrect control-flow transitions.
* Safe-fallback mechanisms that enable sound optimization passes.
* ARMore, a fully-precise efficient aarch64 static binary rewriter that scales to large COTS binaries with support for arbitrarily large instrumentation.

**What are future directions for this research?**

NONE


**What questions are you left with?**

NONE

**What is your take-away message from this paper?**

* See Section 2.
* Three popular techniques for static rewriting are briefly introduced in Section 2: Trampoline, Direct, Lifting.