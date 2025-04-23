---
title: "DART: Directed Automated Random Testing"
layout: post
---

Patrice Godefroid, Nils Klarlund, Koushik Sen

* Read: 22 Apr 2025
* Published: 12 Jun 2005

ACM SIGPLAN Notices, Volume 40, Issue 6, Pages 213 - 223

https://doi.org/10.1145/1064978.1065036

---

# Q&A ([link](https://cseweb.ucsd.edu/~wgg/CSE210/howtoread.html))

**What are the motivations for this work?** 

* See Abstract, Introduction.
* Want to have ***completely automatic testing on any program that compiles*** -- there is no need to write any test driver or harness code.
* Unit testing in practice is very hard and expensive, since in order to be able to execute and test a component in isolation, one needs to write test driver/harness code to simulate the environment of the component.
* DART was the ***first*** tool that combines automatic interface extraction with random testing and dynamic test generation.

**What is the proposed solution?**

* See Abstract, Section 2, Section 3. 
* 3 main techniques are combined: 1. automated extraction of the interface of a program with its external environment using static source-code parsing; 2. automatic generation of a test driver for this interface that performs random testing to simulate the most general environment the program can operate in; 3. dynamic analysis of how the program behaves undr random testing and automatic generation of new test inputs to direct systematically the execution along alternative program paths.
* DART uses ***lp_solve*** constraint solver, which can only solve ***linear constraints***.

**What is the work's evaluation of the proposed solution?**

* See Section 4.
* Experiments performed on a tiny C code example simulating an AC-controller. The experiment showed that DART can effectively find critical inputs that are also impossible to be found in random testing.
* Experiments performed on a C implementation of the Needham-Schroeder public key authentication protocol. The source code is around 400 lines in C. With 328459 iterations taking 18 minutes, an error is detected.
* Experiments performed on a large library called oSIP (about 30000 lines of C code). Detected many vulnerabilities with just a small number of iterations.

**What is your analysis of the identified problem, idea and evaluation?**

The paper was published two decades ago, obviously there're many limitations in today's point of view, including the assumptions on no-side-effect external functions (Section 3.4) and the ability of the constraint solver. Also, the goal of DART is to automate the test generation process rather than guarantee a code coverage.

**What are the contributions?**
* See Introduction, Section 3, Section 4.
* Symbolic execution tool DART.

**What are future directions for this research?**

NONE

**What questions are you left with?**

NONE

**What is your take-away message from this paper?**

NONE

# GPT-generated Keynotes
DART is a white-box testing approach that automatically generates test inputs to explore program execution paths. It combines three main techniques:

1. ***Automated Interface Extraction***: DART begins with a given program, often a C program with a specified API or entry function. It automatically identifies how to invoke the program (i.e., the interface), including parameters and input types.

2. ***Concrete and Symbolic Execution*** The program is run with concrete inputs, like traditional testing. At the same time, DART performs symbolic execution, tracking the path constraints (i.e., conditions that guided the control flow, like ```if (x > 5)```).

3. ***Systematic Exploration via Path Constraints***: After each execution, DART negates one of the path conditions and solves it using a constraint solver. This generates a new input that will steer the next run down a different execution path. This cycle continues, incrementally exploring different paths through the program.

4. ***Handling of External Inputs & Libraries***: Unknown external functions (e.g., library calls) are treated as returning random values. These are constrained and tracked during execution.

Limitations:

1. ***Path Explosion***: As the number of branches grows, the number of possible paths grows exponentially, making exhaustive exploration infeasible for large programs.

2. ***Constraint Solving Bottleneck***: Solving symbolic path constraints (especially for complex conditions or data structures) can be computationally expensive or undecidable in some cases.

3. ***Limited to Deterministic Programs***: DART assumes deterministic behavior. Programs with non-determinism (e.g., concurrency, network behavior) are harder to test effectively.

4. ***External Functions and Environments***: When calling external or OS-level functions, DART cannot symbolically reason about their internal logic. It must approximate or mock their behavior.

5. ***Input Space Coverage***: While DART is better than pure random testing, it may still miss paths due to limitations in constraint solving or infeasible paths being selected.

6. ***No Support for Loops with Large Iteration Counts***: For programs with loops that behave differently across many iterations, DART may fail to explore deep loop behaviors.

7. ***Focus on C-like Programs***: Original implementation targets C; adapting DART-style tools to other languages with different memory models or features (e.g., Java, Python) may not be straightforward.