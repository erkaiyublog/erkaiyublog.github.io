---
title: "通用Linux x64内核态shellcode编写技巧"
layout: post
---

奇安信天工实验室

* Read: 28 May 2025
* Published: 27 May 2025

https://mp.weixin.qq.com/s/UbznUs2RakpztFumIlu0AQ

---

在内核态漏洞利用、ko模块动态注入、USMA或者ebpf提权时，我们需要编写一段位置无关代码（position-Indenpendent Code ）的shellcode的代码来提供给内核执行，以达成内核态漏洞利用、动态功能注入及系统行为控制等目的。