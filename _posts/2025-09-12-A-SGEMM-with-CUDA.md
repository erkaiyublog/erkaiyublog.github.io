---
published: false
title: An SGEMM with CUDA 
tags: CUDA
---

This blog introduces my implementation of an SGEMM (Single-precision 
GEneral Matrix Multiplication) with CUDA. This is motivated by the Lab 3 exercise of 
*CS 508: Manycore Parallel Algorithms* in UIUC.

# Background
SGEMM stands for single-precision general matrix multiplication, it is a widely used 
technique especially in neural networks.

The naive implementation for an SGEMM would be a serial algorithm on a single-core 
CPU. 

The pseudo code for that would be like 

```
# Calculate C = A x B
# A dimension: N x M 
# B dimension: M x K
for i in (0, N)
    for j in (0, K)
        C[i, j] = 0
        for k in (0, M)
            C[i, j] += A[i, k] * B[k, j]
```

This is apparently too slow for large matrices. Meanwhile, the simple computation 
task together with the massive parallelism makes the SGEMM super suitable for
GPU computation. 

# Optimization with GPU
Firstly, it is important to have an overall picture of the **problem-size** before 
considering potential optimizations. 



















