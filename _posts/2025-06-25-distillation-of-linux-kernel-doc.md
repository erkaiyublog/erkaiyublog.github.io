---
published: true
title: My Distillation of The Linux Kernel Documentation
tags: OS
---

In this blog, I'm trying to summarize some of the topics in Linux kernel development that I learned while reading its official documentation. 

I’m currently working with the v6.11 version of the Linux kernel. You can find the main documentation page [here](https://www.kernel.org/doc/html/v6.11/).

***Table of Contents***
* TOC
{:toc}

# [Static Keys](https://www.kernel.org/doc/html/v6.11/staging/static-keys.html#static-keys)
Static keys allows the inclusion of **seldom used features** in performance-sensitive fast-path kernel code, via a GCC feature and a **code patching technique**.

To understand static keys, we need to first introduce the concept of "jump label", which is enabled by ```asm goto``` in gcc (v4.5). Using the ‘asm goto’, we can create branches that are either taken or not taken by default, **without the need to check memory**. Then, at run-time, we can patch the branch site to change the branch direction.

For example, if we have a simple branch:

```c
if (static_branch_unlikely(&key))
        printk("I am the true branch\n");
```

By default, the if statemnet should be false, and we don't want to evaluate this condition each time this statement is reached. The solution is to simply encode ```nop```s into this part of code without any condition check, which means by default nothing happens with these two lines of code. During runtime, if we actually want to execute the branch, we can enable it by **replacing the ```nop```s with a ```jmp```** to the ```printk``` statement compiled at somewhere nearby.

Static keys is just an API that allows us to use such feature to implement "jump label patching" mechanism conveniently. An example is:

```c
DEFINE_STATIC_KEY_FALSE(key);

...

if (static_branch_unlikely(&key))
        do unlikely code
else
        do likely code

...
static_branch_enable(&key);         // Switching the jump label
...
static_branch_disable(&key);        // Switching the jump label
...
```

Note that the if condition is no longer checked during runtime, but controlled manually by enabling/disabling the static branch.

Note that switching branches results in some locks being taken, particularly the CPU hotplug lock (in order to avoid races against CPUs being brought in the kernel while the kernel is getting patched).

Since statics key is based on jump labels, one can disable static keys entirely by setting ```CONFIG_JUMP_LABEL=n``` when configuring kernel compilation.

# [Livepatch](https://www.kernel.org/doc/html/v6.11/livepatch/livepatch.html)
Livepatching allows users to redirect function calls so that patches can be applied to critical functions without rebooting the system.

There are multiple mechanisms in the Linux kernel that are directly related to **redirection of code execution**; namely: **kernel probes**, **function tracing**, and **livepatching**. All three approaches need to **modify the existing code at runtime**. Therefore they need to be aware of each other and not step over each other’s toes. See [this section](https://www.kernel.org/doc/html/v6.11/livepatch/livepatch.html#id4) for more details.

Livepatch has a consistency model (such consistency model tend to be super complicated, think about the case when an updated function involves changes of locking, and needs one or more other functions to be updated at the same time) which is a hybrid of kGraft and kpatch: it uses kGraft’s per-task consistency and syscall barrier switching combined with kpatch’s stack trace switching. There are also a number of fallback options which make it quite flexible. 

A diagram demonstrating a naive understanding of livepatch:

![livepatch](/images/posts/distillation_linux/livepatching.png)

Limitations of livepatch:
* Only functions that can be traced could be patched.
    * **Livepatch is based on the dynamic ftrace**. In particular, functions implementing ftrace or the livepatch ftrace handler could not be patched. Otherwise, the code would end up in an infinite loop :)
* Livepatch works reliably only when the dynamic ftrace is located at the very beginning of the function.
* Kretprobes using the ftrace framework conflict with the patched functions.

# [CoreSight - ARM Hardware Trace](https://www.kernel.org/doc/html/v6.11/trace/coresight/index.html)
Coresight is an umbrella of technologies allowing for the debugging of ARM based SoC. It includes solutions for JTAG and HW assisted tracing.