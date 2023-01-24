---
published: false
---
---

_What happens after you switch on your computer? What is a bootloader? What is BIOS? ..._

_Don't worry, this post will answer all these annoying questions._

---
## Story of BIOS
### Bootstrap 
Bootstrapping is the process to bring at least a portion of the operating system into main memory and have the processor execute it, it also denotes the initialization of kernel data structures, the creation of some user processes, and the transfer of control to one of them.

Bootstrap is highly dependent on the computer architecture.
### Step 0.BIOS
1. A special hardware circuit raises the logical value of RESET pin of the CPU to **start booting**.
2. After RESET is asserted, some registers of the processor (including cs and eip) are set to fixed values, and the code found at physical address 0xfffffff0 is executed.
	* This address is mapped by the hardware to ROM.
	* The set of programs stored in ROM is traditionally called the Basic Input/Output System (**BIOS**) in the 80×86 architecture, because it includes several interrupt-driven low-level procedures used by all operating systems in the booting phase to handle the hardware devices that make up the computer.
	* The BIOS uses **Real Mode addresses** because they are the only ones available when the computer is turned on. A real mode address is composed of a **seg** and an **off**, the corresponding physical address is given by **seg * 16+off**. So no GDT, LDT, page tables are involved when using real mode addresses.