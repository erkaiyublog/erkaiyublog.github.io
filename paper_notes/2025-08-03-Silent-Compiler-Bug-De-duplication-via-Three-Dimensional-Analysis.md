---
title: "Silent Compiler Bug De-duplication via Three-Dimensional Analysis"
layout: post
---

Chen Yang, Junjie Chen, Xingyu Fan, Jiajun Jiang, Jun Sun

* Read: 03 Aug 2025
* Published: 13 Jul 2003

ISSTA 2023: Proceedings of the 32nd ACM SIGSOFT International Symposium on Software Testing and Analysis Pages 677 - 689

[https://dl.acm.org/doi/10.1145/3597926.3598087](https://dl.acm.org/doi/10.1145/3597926.3598087)

---

# Q&A ([link](https://cseweb.ucsd.edu/~wgg/CSE210/howtoread.html))

**What are the motivations for this work?** 
* See Abstract, Introduction, Section 2.
* Evaluating the quality of compilers is critical.
* Bug duplication problem: Many test failures are caused by the same compiler bug.
* **Silent compiler bugs** make the bug duplication problem worse due to the lack of crash feedback.
* The two bug-triggering test programs shown below are corresponding to the same bug in GCC-4.4.0, and they both produce inconsistent outputs under the optimization levels ```-O0``` and ```-O1```. The test programs in the example give significantly different compiler coverage, which invalidates the code coverage approach by prior works. 
![example](/images/posts/compiler3d/example.png)

**What is the proposed solution?**
* See Abstract, Section 3.
* Characterize the silent bugs (***b***) from the testing process and identify 3D information:
    1. test program ***p***
    2. optimizations ***o***
    3. test execution ***c***
* Systematically conduct causal analysis to identify bug-causal features from each of the 3D for more accurate bug de-duplication (the authors argue that **only a small portion** of information in ***p***, ***o***, and ***c*** is bug-causal).
* Rank the test failures that are more likely to be caused by different silent bugs higher by measuring the distance among test failures based on the 3D bug-causal features.
* For test program ***p***: Use test program reduction tool (C-Reduce) to reduce each bug-triggering test program. -> Mutate test program until passing, take minor differences between fail/success as bug-causal features.
    * Mutation is done by implementing mutation rules. Four categories of them: identifier-level mutation, operator-level mutation, delimiter-level mutation, statement-level mutation.
    * Difference between fail/success is extracted at the AST level, which can more comprehensively represent bug-causal features.
    * AST operation is sorted by frequency and then extracted as vector. D3 takes the distace between test failures.
* For optimization ***o***: Use delta debugging to identify the minimal bug-triggering optimizations (binary search).
* For test execution ***c***: Collect function coverage achieved by the bug-triggering test program as the original information in this dimension (similar to the existing work [20]). Then, it identifies the bug-causal functions from all the covered functions by estimating their suspicious scores via the idea of spectrum-based bug localization, and finally treats highly suspicious functions as the bug-causal features in this dimension. 
* Finally, the test failure prioritization is done by calculating the distance extracted from the three dimensions, and ranks them by the furthest-point-first (FPF) algorithm.

**What is the work's evaluation of the proposed solution?**
* See Section 4.

**What is your analysis of the identified problem, idea and evaluation?**
* The idea of measuring distance shall not be an unusal approach. The subject (compiler test) seems to be the real innovation.

**What are the contributions?**
* See Introduction.
* D3. A novel technique for addressing  the de-duplication problem on silent compiler bugs.

**What are future directions for this research?**
NONE

**What questions are you left with?**

NONE

**What is your take-away message from this paper?**

NONE