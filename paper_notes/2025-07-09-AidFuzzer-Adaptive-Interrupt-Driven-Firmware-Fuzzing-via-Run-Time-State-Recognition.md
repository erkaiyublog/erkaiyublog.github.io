---
title: "AidFuzzer: Adaptive Interrupt-Driven Firmware Fuzzing via Run-Time State Recognition"
layout: post
---

Jianqiang Wang, Qinying Wang, Tobias Scharnowski, Li Shi, Simon Woerner, Thorsten Holz

* Read: 09 Jul 2025
* Published: May 2025

34th USENIX Security Symposium (USENIX Security)

[https://www.usenix.org/conference/usenixsecurity25/presentation/wang-jianqiang](https://www.usenix.org/conference/usenixsecurity25/presentation/wang-jianqiang)

---

# Q&A ([link](https://cseweb.ucsd.edu/~wgg/CSE210/howtoread.html))

**What are the motivations for this work?** 
* See Abstract, Introduction.
* A proper mechanism for triggering and handling interrupts is a crucial yet under-researched aspect of firmware fuzzing.
* Tools like P2IM and μEmu attempt to model peripheral behavior by extracting information from the MCU documentation or using symbolic execution. Unfortunately, these techniques are unstable and imprecise.
* Need to answer three questions when dealing with the interrupt-triggering problem:
    * When should the interrupts be triggered?
    * How often should the interrupts be triggered? 
    * Which interrupts should be triggered?

**What is the proposed solution?**
* See Introduction, Section 3, Section 4.
* AidFuzzer, an Adaptive Interrupt-Driven Fuzzing framework that provides a proper interrupt triggering mechanism for firmware fuzzing.
* Key insight to solve the interrupt triggering problem: **the run-time state transition cycle** of a running firmware and the relations between the interrupt triggering and run-time state.
* To solve the interrupt triggering problem, need to identify 
    * the IRQ status (ready or unready)
    * IRQ types (effective or ineffective)
    * the firmware run-time state (waiting or not waiting)
* Two challenges from real-world firmware:
    * run-time data dependency: solved monitoring and intercepting the changes of the interrupt vector table base, vector tbale entries, and the function pointers used in the ISR during the whole fuzzing campaign.
    * state recognition: observed that most firmware share a common run-time transition cycle, concluded that the firmware enters a **waiting state** if one of the certain conditions is satisified (this assumption is aligned with 83% of the tested 110 firmware samples).


**What is the work's evaluation of the proposed solution?**
* See Section 6.
* Higher coverage.

**What is your analysis of the identified problem, idea and evaluation?**

NONE

**What are the contributions?**
* See Introduction.
* AidFuzzer, an Adaptive Interrupt-Driven Fuzzing framework that provides a proper interrupt triggering mechanism for firmware fuzzing.

**What are future directions for this research?**

NONE

**What questions are you left with?**

NONE

**What is your take-away message from this paper?**

NONE