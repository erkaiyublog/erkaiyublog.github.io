---
title: "Container Breakouts: Escape Techniques in Cloud Environments"
layout: post
---

Yosef Yaakov, Bar Ben-Michael

* Read: 10 Jul 2025
* Published: 18 Jul 2024

[https://unit42.paloaltonetworks.com/container-escape-techniques/](https://unit42.paloaltonetworks.com/container-escape-techniques/)

---

People use containers for efficient resource utilization because they allow the use of multiple systems on a single server. Containers achieve this by creating an **isolated process tree, network stack, file system and various other user-space components** using the **namespace mechanism** provided by the operating system.

The **container runtime** is responsible for initiating a process and adjusting its attributes to limit and isolate not only the process itself but also all its child processes.

The attributes subject to modification by the container runtime to perform process isolation include the following:
* Credentials
* **Capabilities**: By strategically removing unnecessary and high-privilege capabilities from the processes involved in container creation, the container engine can execute containers securely, even with root privileges. This security mechanism is made possible through the inheritable capabilities mechanism of Linux. 
* Linux Security Modules (LSM)
* Secure computing (seccomp)
* **Namespaces**: In process management, if capabilities define *what* a process can do, then namespaces define *where* these actions can be performed. Essentially, namespaces provide a layer of abstraction that enables a process and its children to operate as if they possess their own exclusive instance within a global resource.
* Control groups (cgroups).

5 examples of container escape were provided by the article. Some notes taken while reading: 
![exmpl1](/images/posts/container_breakouts/expl1.jpeg)
![exmpl2](/images/posts/container_breakouts/expl2.jpeg)