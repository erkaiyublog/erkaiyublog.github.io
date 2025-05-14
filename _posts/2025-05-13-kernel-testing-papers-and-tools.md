---
published: true
title: Kernel Testing Papers and Tools Collection
tags: paper-reading
---

This blog presents a growing collection of papers and tools related to the topic of kernel testing. I (hopefully ;p) plan to update it regularly, since I want it to serve as a handy index whenever I revisit this topic. 

For each paper or tool listed here, I’ll include a link to a separate page with my notes from reading or studying it. The notes are definitely not *complete*, but I do aim for them to be *sound* (hopefully!).

I collected the related papers by refering to the following surveys/summaries:
1. *A Survey of Fuzzing Open-Source Operating Systems*, Kun et al. [[link](https://arxiv.org/abs/2502.13163)]
2. *Demystifying OS Kernel Fuzzing with a Novel Taxonomy*, Jiacheng et al. [[link](https://arxiv.org/abs/2501.16165)]


# List of Online Documents
1. [Kernel Testing Guide by kernel.org](https://docs.kernel.org/dev-tools/testing-overview.html)

# List of Tools
1. [syzkaller: an unsupervised coverage-guided kernel fuzzer by Google.](https://github.com/google/syzkaller)
2. [kAFL: a fast guided fuzzer for the x86 VM.](https://intellabs.github.io/kAFL/)
3. [KUnit: a Kernel-native unit testing framework introduced in Linux 5.5.](https://kunit.dev/)
4. [kselftest: a set of “self tests” under the tools/testing/selftests/ directory.](https://docs.kernel.org/dev-tools/kselftest.html)
5. [LTP: Linux Test Project.](https://linux-test-project.readthedocs.io/en/latest/)
6. [sparse: a semantic checker for C programs.](https://docs.kernel.org/dev-tools/sparse.html)
7. [smatch: static checker for the kernel.](https://smatch.sourceforge.net/)
1. [An automated, continuous kernel testing pipeline by Codethink.](https://www.codethink.co.uk/articles/2021/automated-linux-kernel-testing/)

# List of Paper Notes
## Surveys
1. [Symbolic execution for software testing: Three decades later](/paper_notes/2025-04-20-Symbolic-execution-for-software-testing-Three-decades-later)(2013)
1. [A Survey of Symbolic Execution Techniques](/paper_notes/2025-04-20-A-Survey-of-Symbolic-Execution-Techniques)(2018)

## Classical Ones
1. [DART: directed automated random testing](/paper_notes/2025-04-22-DART-Directed-Automated-Random-Testing)(2005)
1. [CUTE: A Concolic Unit Testing Engine for C](/paper_notes/2025-04-22-CUTE-A-Concolic-Unit-Testing-Engine-for-C)(2005)
3. [EXE: Automatically Generating Inputs of Death](/paper_notes/2025-04-21-EXE-Automatically-Generating-Inputs-of-Death)(2006)
4. [KLEE: Unassisted and Automatic Generation of High-Coverage Tests for Complex Systems Programs](/paper_notes/2025-04-22-KLEE-unassisted-and-automatic-generation-of-high-coverage-tests-for-complex-systems-programs)(2008)

## Modern Ones
1. [QSYM: a practical concolic execution engine tailored for hybrid fuzzing](/paper_notes/2025-04-23-QSYM-A-practical-concolic-execution-engine-tailored-for-hybrid-fuzzing)(2018)
1. [Symbolic execution with SymCC: Don’t interpret, compile!](/paper_notes/2025-04-23-Symbolic-execution-with-SYMCC-dont-interpret-compile)(2020)
1. [Coyote C++: An Industrial-Strength Fully Automated Unit Testing Tool](/paper_notes/2025-04-22-Coyote-C-An-Industrial-Strength-Fully-Automated-Unit-Testing-Tool)(2023)

