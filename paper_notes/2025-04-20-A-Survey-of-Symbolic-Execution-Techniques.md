---
title: "A Survey of Symbolic Execution Techniques"
layout: post
---

Roberto Baldoni, Emilio Coppa, Daniele Cono D’elia, Camil Demetrescu, Irene Finocchi

* Read: 20 Apr 2025
* Published: 23 May 2018

ACM Computing Surveys (CSUR), Volume 51, Issue 3
Article No.: 50, Pages 1 - 39

[https://doi.org/10.1145/3182657](https://doi.org/10.1145/3182657)

---

See also: [KLEE](/paper_notes/2025-04-22-KLEE-unassisted-and-automatic-generation-of-high-coverage-tests-for-complex-systems-programs), [EXE](/paper_notes/2025-04-21-EXE-Automatically-Generating-Inputs-of-Death).

After skimming the survey *A Survey of Symbolic Execution Techniques*, I summarized the topics related to symbolic execution as the diagram below.

![Topics Related to Symbolic Execution](/images/posts/symbolic_execution_index/symbolic_exec.drawio.png)

In the survey, some further directions for symbolic execution were listed. Below is a summarized version.

1. **Separation Logic**: SL extends Hoare logic to facilitate reasoning about programs that maniputlate pointer data structures, and allows expressing compplex invariants of heap configurations in a succinct manner.
2. **Invariants**: Leveraging invariants of loops can be beneficial to symbolic executors, there was no relavent work discoverd by the authors of the survey.
3. **Function Summaries**: Calysto static checker is an example of using the call graph to construct a symbolic representation of effects on each function.
4. **Program Analysis and Optimization**: The developments in programming languages realm could be beneficial to symbolic execution. Some examples are loop coalescing, loop unfolding, program synthesis.
5. **Symbolic Computation**: Only Z3 and SMT-RAT can reason about non-linear real and integer arithmetic. The developments in symbolic computation can be used to help SMT solvers. SC^2 is an interesting project aiming to bridge the gap between symbolic computation and satisfiability checking.

