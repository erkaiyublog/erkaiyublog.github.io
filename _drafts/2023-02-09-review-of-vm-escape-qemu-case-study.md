---
published: false
---
---
*This is a review of the article [VM escape - QEMU Case Study](http://www.phrack.org/issues/70/5.html#article) by Mehdi Talbi and Paul Fariello published on [Phrack](http://www.phrack.org/).*

*This is my first time reading & understanding (a decent portion of) an article related with computer security, so I'd like to write this review to briefly record what I learned from this article.*

---
## Introduction
At HITB 2016, Xu Liu and Shengping Wang from Qihoo 360 have showcased a successful exploit on KVM/QEMU, by exploiting **two vulnerabilities** (CVE-2015-5165 and CVE-2015-7504) present in two different network card device emulator models. 

However, the presentation they gave didn't include the technical details of reproducing the attack, so the Prhack article was written to provide a in-depth analysis of CVE-2015-5165 and CVE-2015-7504, discuss the technical details to exploit the vulnerabilities on QEMU's network card device emulation, and provide generic techniques that could be re-used to exploit future bugs in QEMU.

My review will generally follow the structure of the paper, but I choose to start with a brief outline of the exploit. 
## Outline of the Exploit
The exploit makes use of two vulnerabilites present in two different network card device emulator models. CVE-2015-5165 is on network card RTL8139, a memory-leak vulnerability. CVE-2015-7504 is on network card PCNET, a heap-based overflow vulnerability. 

The exploit will first inject payload into memory, including a fake structure and some other codes for the shell.

Then, with CVE-2015-5165, the exploit will find out:
1. The base address of the **.text** segment
2. The base address of the physical memory

use the memory leak to find out the address of the payload. 

Eventually, use CVE-2015-7504 to overwrite a structure on the heap, so that later interrupt handlers for the PCNET network card will be decided by the attacker. The overwritten interrupt handler will first use **mprotect()** to enable the execution bit of certain pages where the payload resides, then trigger **system()** to execute codes in the payload, which brings up a shell in the host machine.
## Sources
* http://www.phrack.org/issues/70/5.html#article
* https://www.technovelty.org/linux/plt-and-got-the-key-to-code-sharing-and-dynamic-libraries.html
