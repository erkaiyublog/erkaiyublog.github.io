---
title: "Xen and the Art of Virtualization"
layout: post
---

Paul Barham, Boris Dragovic, Keir Fraser, Steven Hand, Tim Harris, Alex Ho, Rolf Neugebauer, Ian Pratt, Andrew Warfield

* Read: 11 Jul 2025
* Published: 19 Oct 2003

SOSP '03: Proceedings of the nineteenth ACM symposium on Operating systems principles Pages 164 - 177

[https://dl.acm.org/doi/10.1145/945445.945462](https://dl.acm.org/doi/10.1145/945445.945462)

---

# Q&A ([link](https://cseweb.ucsd.edu/~wgg/CSE210/howtoread.html))

**What are the motivations for this work?** 
* See Abstract, Introduction.
* Currently available virtualization systems face limitations due to overhead and security concerns.
* Multiplexing in between processes can be applied in between operating systems. Although running an OS is more heavyweight than running a process, both in terms of initialization and in terms of resource consumption, it's worth paying this price for virtualized operating systems.  

**What is the proposed solution?**
* See Section 2, Section 3.
* A set of design principles:
    * Support for unmodified application binaries is essential, or users will not transition to Xen. Hence we must virtualize all architectural features required by existing standard ABIs.
    * Supporting full multi-application operating systems is important, as this allows complex server configurations to be virtualized within a single guest OS instance.
    * Paravirtualization is necessary to obtain high performance and strong resource isolation on uncooperative machine architectures such as x86.
    * Even on cooperative machine architectures, completely hiding the effects of resource virtualization from guest OSes **risks both correctness and performance** (an example is the TCP timeout and RTT estimation by the guest OS needs real time to be more precise).
* **Paravirtualization** done for x86 architecture:
![table1](/images/posts/xen/table1.png)
* The overall system structure of Xen:
![arch](/images/posts/xen/arch.png)
* The hypervisor itself provides **only basic control operations**. 
* *Domain0* runs control software that manages the entire server (e.g. can create and destroy domains, set network filters and routing rules, monitor per-domain network activity at packet and flow granularity, and create and delete virtual network interfaces and virtual block device). 
* For interactions between domain and Xen:
```
        --- synchronous hypercall ---->
Domain                                   Xen       
        <----- asynchronous event -----
```
* Data transfer is realized using I/O **ring buffer** (per-guest).

![ioring](/images/posts/xen/ioring.png)

* More detailed subsystem virtualization is introduced in Section 3.3.

**What is the work's evaluation of the proposed solution?**
* See Section 4.
* A thorough performance evaluation of Xen:
    * Benchmark Xen against a number of alternative virtualization techniques.
    * Compare the total system throughput executing multiple applications concurrently on a single native operating system against running each application in its own virtual machine.  
    * Evaluate the performance isolation Xen provides between guest OSes, and assess the total overhead of running large numbers of operating systems on the same hardware. 
* Used XenoLinux port for measurements. Single CPU in all tests (neither Xen nor any of the VMware products currently support multiprocessor guest OSes).
* Results for relative performance (multiple test suites used):

![dia1](/images/posts/xen/dia1.png)

* Detailed experiments on subsystem overhead are done in Section 4.2.
    * It's worth noting that the authors pointed out "the dangers of taking microbenchmarks too seriously" in Section 4.2 as the results may be effected by cache performance.
* For performance isolation, ran 4 domains configured with equal resource allocations, with two domains running normal workloads, two others each running a pair of extremely antisocial processes. Turned out that the performance of normal workload domains were only marginally affected by the behaviour of the two domains running disruptive processes. VMware Workstation achieves similar levels of isolation, but at reduced levels of absolute performance.

**What is your analysis of the identified problem, idea and evaluation?**

NONE

**What are the contributions?**
* See Introduction.
* Xen hypervisor which partitions the resources of a computer between domains running guest operating systems.

**What are future directions for this research?**

* See Section 6.1.

**What questions are you left with?**

NONE

**What is your take-away message from this paper?**

NONE