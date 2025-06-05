---
title: "You Can’t Judge a Binary by Its Header: Data-Code Separation for Non-Standard ARM Binaries using Pseudo Labels"
layout: post
---

Hadjer Benkraouda, Nirav Diwan, Gang Wang

* Read: 04 Jun 2025
* Published: 12 May 2025

2025 IEEE Symposium on Security and Privacy (SP)

https://www.computer.org/csdl/proceedings-article/sp/2025/223600a036/21B7QpveAHC

---

# Q&A ([link](https://cseweb.ucsd.edu/~wgg/CSE210/howtoread.html))

**What are the motivations for this work?** 

* See Abstract, Introduction, Section 2.
* Customized and non-standard binary formats cannot be readily processed by existing tools.
* Due to the lack of standardization and the need to keep their software proprietary, IoT vendors often create customized file formats without public documentation.
* As mentioned in section 2, **inline data** is also a target for this work.

**What is the proposed solution?**

* See Introduction, Section 3.
* Start with data-code separation in binaries.
* Trained a ML classifier based on standard binaries, and then adapted the classifier to the target non-standard binary formats.
* Perform a linear sweep over the binaries and classify each byte as code or data (using the fact that ARM binaries have fixed-length instructions).


**What is the work's evaluation of the proposed solution?**

See Introduction, Section 5.
* Use XDA as the baseline. XDA doesn't perform code and data separation therefore adapt their instruction boundary detection task for this purpose.

**What is your analysis of the identified problem, idea and evaluation?**

Choosing ARM is indeed a good idea.

**What are the contributions?**

* See Introduction.
* A system called Loadstar, it is a pseudo-label-based learning model that performs data-code separation for non-standard binaries.

**What are future directions for this research?**

NONE

**What questions are you left with?**

NONE

**What is your take-away message from this paper?**

NONE