---
title: "Coyote C++: An Industrial-Strength Fully
Automated Unit Testing Tool"
layout: post
---

Sanghoon Rho, Philipp Martens, Seungcheol Shin, Yeoneo Kim, Hoon Heo, SeungHyun Oh

* Read: 22 Apr 2025
* Published: 23 Oct 2023

arXiv:2310.14500 

https://doi.org/10.48550/arXiv.2310.14500


---

# Q&A ([link](https://cseweb.ucsd.edu/~wgg/CSE210/howtoread.html))

**What are the motivations for this work?** 

* See Abstract, Introduction.
* While concolic testing has proven effective for languages such as C and Java, tools have struggled to achieve a practical level of ***automation for C++*** due to its many syntactical intricacies and overall complexity.
* Coyote C++ is the ***first*** automated testing tool to breach the barrier and bring automated unit testing for C++ to a practical level suitable for industrial adoption, consistently reaching around 90% code coverage.

**What is the proposed solution?**

* See Introduction, Section III.
* Coyote C++ can be divided into 2 parts. The first part builds executable test files based on harness generation. The second part handles generating test cases through concolic execution.
* "A fundamental design decision made in Coyote C++ is using
LLVM IR as its symbolic execution target." (However, [KLEE](/paper_notes/2025-04-22-KLEE-unassisted-and-automatic-generation-of-high-coverage-tests-for-complex-systems-programs) made use of LLVM IR about 15 years earlier than this paper).
* Implemented offline testing.
* Path search optimization: Adopted a hybrid approach that combines CCS (Code Coverage Search) and DFS to search for candidate paths for exploration.
* Memory model: Similar to MAYHEM, the approach implemented in Coyote C++ reads values from memory symbolically but writes values to concrete memory addresses. 

![overview](/images/posts/coyote_cpp/coyote.png)

**What is the work's evaluation of the proposed solution?**

* See Section IV.
* Tested Coyote C++ with a set of diverse open-source projects as well as several industrial software projects.
* On an Intel Core i7-13700 system with 64GB of RAM running Ubuntu 20.04, reached an overall testing speed of roughly 17,000 statements per hour.
* Decent coverage results, but the total number of statements are pretty low for the open source projects (less than 10000).

**What is your analysis of the identified problem, idea and evaluation?**

NONE

**What are the contributions?**
* See Introduction, Section V.

**What are future directions for this research?**

* See Section V.

**What questions are you left with?**

NONE

**What is your take-away message from this paper?**

NONE
