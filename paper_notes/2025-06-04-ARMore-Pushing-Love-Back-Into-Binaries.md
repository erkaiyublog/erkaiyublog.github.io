---
title: "ARMore: Pushing Love Back Into Binaries"
layout: post
---

Luca Di Bartolomeo, Hossein Moghaddas, Mathias Payer

* Read: 07 Jun 2025
* Published: 09 Aug 2023

SEC '23: Proceedings of the 32nd USENIX Conference on Security Symposium Article No.: 353, Pages 6311 - 632

[https://dl.acm.org/doi/10.5555/3620237.3620590](https://dl.acm.org/doi/10.5555/3620237.3620590)

---

See also: [A survey on binary rewriting](/paper_notes/2025-06-05-From-hack-to-elaborate-technique-A-survey-on-binary-rewriting).

# Q&A ([link](https://cseweb.ucsd.edu/~wgg/CSE210/howtoread.html))

**What are the motivations for this work?** 

* See Abstract, Section 2.
* Most research on static rewriting has so far focused on the x86 architecture.
* One of the main challenges for static instrumentation is resolving pointer targets. Due to aliasing, the target for references is unknown during translation. Without runtime information, static rewriters need to rely on complex static analysis, which is inherently imprecise and often resorts to heuristics. Consequently, static rewriting cannot support packed binaries or self-modifying code.

**What is the proposed solution?**

* See Introduction, Section 3.
* A challenge for rewriting ARM binaries: ARM binaries rely on compressed jump tables, where for each possible entry in the jump table, only the offset of each case is stored (usually, only 1 or 2 bytes), making jump tables particularly hard to statically distinguish from random data. A static binary rewriter must detect and adjust those offsets to accommodate the instrumentation that might change the distance from the base case.
* ARMore leverages symbolization to reconstruct control flow after the insertion of instrumentation.
* Employs only **sound techniques** (layout replication and rebound table) rather than using heuristics for pointer recovery.
* **Layout Replication**: Enforces the original virtual address space layout of the binary, so data pointers do not need to be rewritten. Only code pointers need to be corrected to point to the new section that contains instrumentation. See the graph below, note how ```.text``` got copied as both ```.rebound_table``` and ```.old_text```.

![layoutrep](/images/posts/armore/layoutrep.png)

* **Rebound Table**: Static rewriting approaches cannot universally reliably resolve indirect control-flow transfers. Instead of overwriting the code with trap instructions, ARMore leverages direct branches whose destination in the instrumented code section is already known at link-time. Consequently, an undetected pointer construction **always ends up in the rebound table** from where the control flow is redirected to the corresponding target instruction. See the diagram below, the rebound table is nothing but a copy of the original ```.text``` section, which acts as a jump table to the instrumented instructions. ARMore’s rebound table translation comes at the cost of only one additional branch, instead of an expensive lookup through a dynamic structure (some other rewriters use hash table).

![rebound](/images/posts/armore/rebound.png)

* For PC-relative pointer constructions, ARMore forces dynamically computed pointers to target the replicated layout by symbolizing the initial adrp/adr—the prologue of a pointer construction—to point it to the same address on the replicated layout. The remaining instructions that fix the last 12 bits of a pointer remain untouched.

* While many rewriters use call emulation (substitute every ```bl``` and ```blr``` with a ```adr+mov+b``` sequence that puts the original return address in the link register and then uses a direct branch to a function instead of a call).

* Rebound table is on XOM (executable-only pages), so reading data from it triggers a segmentation fault. ARMore catches the segfault through a signal handler. The handler returns the correct value by reading it from a copy of the original text section (```.old_text```).

* Heuristics as an opportunity (Section 3.4). The main idea is that potentially the instrumented code can detect and redirect jumps correctly without involving the rebound table, meanwhile all undetect cases cause is but some overhead.

![appc](/images/posts/armore/appc.png)

**What is the work's evaluation of the proposed solution?**

See Introduction, Section 5.
* In total, rewrote 486MB of code adding up to over 121 million instructions.
* Rewrote binaries from 239 Debian packages (including C++ and Go binaries), and ran the relevant testsuite of each package, passed 97.5% tests.
* Rewrote SQLite, coreutils, passed all tests.
* Used SPEC CPU2017 to measure overhead. The average overhead introduced by the rebound table is 0.99%. The average overhead introduced by call emulation (calculated only on C++ binaries) is, on average, 10.74%.
* Performed **case studies** for address sanitization, coverage information, fuzzing real world closed-source software, and binary patching.
    * ARMore fuzzes code about 3 times faster than AFL- QEMU, on average. Compared to AFL++ (compiler-based instrumentation), ARMore is around 25% slower.
    * Evaluated the Nvidia CUDA toolkit for Linux, a set of utilities to compile, debug and inspect CUDA applications.

**What is your analysis of the identified problem, idea and evaluation?**

* The evaluation is super detailed and strong. The idea of rebound table is straightforward.

**What are the contributions?**

* See Introduction, Section 4.
* A new and heuristic-free approach of instrumenting aarch64 binaries containing arbitrary pointer arithmetic or data interleaved with code.
* A new mechanism, the rebound table,to recover from statically-unresolvable indrect control-flow transitions.
* Safe-fallback mechanisms that enable sound optimization passes.
* ARMore, a fully-precise efficient aarch64 static binary rewriter that scales to large COTS binaries with support for arbitrarily large instrumentation.
* ARMore forks RetroWrite and adds about 3,000 lines of Python code to implement the rebound table and layout replication techniques, along with support for C++ binaries, Go binaries, stripped binaries, data inside text, non-PIE binaries, and aarch64-specific pointer analyses.

**What are future directions for this research?**

NONE

**What questions are you left with?**

NONE

**What is your take-away message from this paper?**

* See Section 2.
* Three popular techniques for static rewriting are briefly introduced in Section 2: Trampoline, Direct, Lifting.
* ARM pointer construction involving ```ldr, adrp``` are introduced in Section 2. It's difficult to recover these pointer addresses after compiler optimization. Mainstream ARM64 static rewriters exclusively rely on data flow to recover pointers, which is inherently imprecise and cannot cover all edge cases of real-world binaries.
* A rough estimate of the frequency of **data inside text** is p/q, where p is the ratio of invalid instructions (disassembled 44 million instructions, 0.0011% of which were invalid.), and q is the chance of random data being a valid instruction (calculate the chance that random data represents a valid aarch64 instruction by disassembling all 2^32 possible 4-bytes values, and the result is around 33.80%). The lower-bound estimate of data inside text sections is thus **0.0032%**.