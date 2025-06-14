---
published: true
title: Discover Concurrency in C Programs
tags: C Tips
---

Sometimes, you get the opportunity to implement a C function that will be called by other programs. In some cases, the program may be multi-threaded, and you might want to verify whether your function is being called simultaneously by multiple threads :)

Below is one way to verify this,

```c
#include <stdatomic.h>
#include <pthread.h>
#include <stdio.h>

static atomic_int in_function = 0;

void your_function() {
    if (atomic_fetch_add_explicit(&in_function, 1, memory_order_relaxed) > 0) {
        fprintf(stderr, "Thread %lu: Reentrant or concurrent call detected!\n", (unsigned long)pthread_self());
        fprintf(stderr, "WARNING: Concurrent access detected!\n");
    }

    // --- Function logic goes here ---

    atomic_fetch_sub_explicit(&in_function, 1, memory_order_relaxed);
}
```

I did this for ```qemu_plugin_register_vcpu_tb_trans_cb``` in QEMU plugin. FYI, up to v10.0, the callback function registered will be called by multi-threads in you run QEMU with ```-smp [n>1]```.
