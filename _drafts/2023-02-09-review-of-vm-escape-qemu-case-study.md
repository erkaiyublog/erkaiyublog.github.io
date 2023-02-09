---
published: false
---
---
*This is a review of the article [VM escape - QEMU Case Study](http://www.phrack.org/issues/70/5.html#article) by Mehdi Talbi and Paul Fariello published on [Phrack](http://www.phrack.org/).*

*This is my first time reading & understanding (a decent portion of) an article related with computer security, so I'd like to write this review to briefly record what I learned from this article.*

---
## Introduction
At HITB 2016, Xu Liu and Shengping Wang from Qihoo 360 have showcased a successful exploit on KVM/QEMU, by exploiting **two vulnerabilities** (CVE-2015-5165 and CVE-2015-7504) present in two different network card device emulator models. 

However, the presentation they gave didn't include the technical details of reproducing the attack, so the Prhack article was written to provide a in-depth analysis of CVE-2015-5165 and CVE-2015-7504, and discuss the technical details to exploit the vulnerabilities on QEMU's network card device emulation, and provide generic techniques that could be re-used to exploit future bugs in QEMU.

My review will 