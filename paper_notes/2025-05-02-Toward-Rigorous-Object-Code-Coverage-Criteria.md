---
title: "Toward Rigorous Object-Code Coverage Criteria"
layout: post
---

Taejoon Byun, Vaibhav Sharma, Sanjai Rayadurgam, Stephen McCamant, Mats P. E. Heimdahl

* Read: 02 May 2025
* Published: 23 Oct 2017

2017 IEEE 28th International Symposium on Software Reliability Engineering (ISSRE)

[https://doi.org/10.1109/ISSRE.2017.33](https://doi.org/10.1109/ISSRE.2017.33)

---

See also: [Discovering instructions for robust binary-level coverage criteria](/paper_notes/2025-05-02-Discovering-instructions-for-robust-binary-level-coverage-criteria)

# Q&A ([link](https://cseweb.ucsd.edu/~wgg/CSE210/howtoread.html))

**What are the motivations for this work?** 

* See Abstract.
* To address the need for a robust object coverage criterion, this paper proposes a rigorous definition of OBC such that it captures well the semantics of source code branches for a given instruction set architecture.

**What is the proposed solution?**

* See Abstract.
* A rigorous definition of OBC (called Flag-Use Object Branch Coverage) to capture the semantics of source code branches for a given instruction set architecture.
* Define ***Flag-Use Instruction*** and ***Flag-Use Object Branch Coverage***.
    * Flag-Use Instruction: Any instruction that ***reads*** the value of one or more flag registers is called a Flag-Use Instruction.
    * Flag-Use Object Branch Coverage: A test- suite is said to achieve Flag-Use Object Branch Coverage (Flag-Use OBC) if for each Flag-Use Instruction in the object- code, if each distinct behavior of the instruction that is conditional on the flag values read, is exercised by some test- case in the test-suite.

**What is the work's evaluation of the proposed solution?**

See Section 4.

**What is your analysis of the identified problem, idea and evaluation?**

NONE

**What are the contributions?**

* See Abstract, Introduction.
* A rigorous definition of OBC (called Flag-Use Object Branch Coverage) to capture the semantics of source code branches for a given instruction set architecture.

**What are future directions for this research?**

* See Introduction.
* Flag-Use OBC is sensitive to the structure of the object-code—albeit far less so than OBC—and the fault- finding ability of test suites satisfying Flag-Use OBC (as well as MC/DC for that matter) is not as strong as one would like.


**What questions are you left with?**

NONE

**What is your take-away message from this paper?**

NONE