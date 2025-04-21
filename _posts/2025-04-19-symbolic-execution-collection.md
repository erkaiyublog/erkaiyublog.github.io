---
published: true
title: Symbolic Execution Paper Collections
tags: paper-reading
---

This blog presents a growing collection of papers and tools related to the topic of symbolic execution. I (hopefully ;p) plan to update it regularly, since I want it to serve as a handy index whenever I revisit this topic. 

For each paper or tool listed here, I’ll include a link to a separate page with my notes from reading or studying it. The notes are definitely not *complete*, but I do aim for them to be *sound* (hopefully!).

I collected the related papers by refering to the following surveys/summaries:
1. *A Survey of Symbolic Execution Techniques*, Baldoni et al. [[link](https://arxiv.org/pdf/1610.00502)]
2. *Symbolic execution for software testing: Three decades later*, Cadar et al. [[link](https://people.eecs.berkeley.edu/~ksen/papers/cacm13.pdf)]

# High-Level View

After skimming the article *A Survey of Symbolic Execution Techniques*, I summarized the topics related to symbolic execution as the diagram below.

![Topics Related to Symbolic Execution](../images/posts/symbolic_execution_index/symbolic_exec.drawio.png)

I picked some of the symbolic execution tools mentioned in the diagram above, and wrote notes about them:

* IN PROGRESS

# Advanced Directions
*A Survey of Symbolic Execution Techniques* was published in 2018. In the survey, some further directions for symbolic execution were listed. Below is a summarized version.

1. **Separation Logic**: SL extends Hoare logic to facilitate reasoning about programs that maniputlate pointer data structures, and allows expressing compplex invariants of heap configurations in a succinct manner.
2. **Invariants**: Leveraging invariants of loops can be beneficial to symbolic executors, there was no relavent work discoverd by the authors of the survey.
3. **Function Summaries**: Calysto static checker is an example of using the call graph to construct a symbolic representation of effects on each function.
4. **Program Analysis and Optimization**: The developments in programming languages realm could be beneficial to symbolic execution. Some examples are loop coalescing, loop unfolding, program synthesis.
5. **Symbolic Computation**: Only Z3 and SMT-RAT can reason about non-linear real and integer arithmetic. The developments in symbolic computation can be used to help SMT solvers. SC^2 is an interesting project aiming to bridge the gap between symbolic computation and satisfiability checking.