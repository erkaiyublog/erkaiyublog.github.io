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

**What is the proposed solution?**
* See Abstract.
* Characterize the silent bugs from the testing process and identify 3D information:
    1. test program
    2. optimizations
    3. test execution
* Systematically conduct causal analysis to identify bug-causal features from each of the 3D for more accurate bug de-duplication.
* Rank the test failures that are more likely to be caused by different silent bugs higher by measuring the distance among test failures based on the 3D bug-causal features.
* The two bug-triggering test programs shown below are corresponding to the same bug in GCC-4.4.0, and they both produce inconsistent outputs under the optimization levels ```-O0``` and ```-O1```.
![example](/images/posts/compiler3d/example.png)

**What is the work's evaluation of the proposed solution?**

**What is your analysis of the identified problem, idea and evaluation?**


**What are the contributions?**

**What are future directions for this research?**


**What questions are you left with?**

NONE

**What is your take-away message from this paper?**

NONE