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
### Step 0. BIOS
1. A special hardware circuit raises the logical value of RESET pin of the CPU to **start booting**.
2. After RESET is asserted, some registers of the processor (including cs and eip) are set to fixed values, and the code found at physical address 0xfffffff0 is executed.
	* This address is mapped by the hardware to ROM.
	* The set of programs stored in ROM is traditionally called the Basic Input/Output System (**BIOS**) in the 80×86 architecture, because it includes several interrupt-driven low-level procedures used by all operating systems in the booting phase to handle the hardware devices that make up the computer.
	* The BIOS uses **Real Mode addresses** because they are the only ones available when the computer is turned on. A real mode address is composed of a **seg** and an **off**, the corresponding physical address is given by **seg * 16+off**. So no GDT, LDT, page tables are involved when using real mode addresses.
3. Linux retrieves the **kernel image** from disk or from some other external device under the assistance of BIOS. The BIOS bootstrap procedure essentially performs the following four operations:
	1. Power-On Self-Test (POST): Executes a series of tests on the computer hardware to establish which devices are present and whether they are working properly.
    	* During this phase, several messages, such as the BIOS version banner, are displayed.
        * Recent 80×86, AMD64, and Itanium computers make use of the Advanced Configuration and Power Interface (ACPI) standard. The bootstrap code in an ACPI-compliant BIOS builds several tables that describe the hardware devices present in the system. These tables have a vendor-independent format and can be read by the operating system kernel to learn how to handle the devices.
	2. **Initializes the hardware devices**. This phase is crucial in modern **PCI-based** architectures, because it guarantees that all hardware devices operate without conflicts on the IRQ lines and I/O ports. At the end of this phase, a table of installed PCI devices is displayed.
    3. **Searches** for an operating system **to boot!!** Depending on the BIOS setting, the procedure may try access (in a predefined, customizable order) the first sector (boot sector) of every floppy disk, hard disk, and CD-ROM in the system.
    4. As soon as a valid device is found, it **copies the contents of its first sector into RAM**, starting from physical address 0x00007c00, and then jumps into that address and executes the code just loaded.

### Step 1. Boot Loader
The **boot loader** is the program invoked by the BIOS to load the image of an operating system kernel into RAM.

Here’s an example of how boot loaders work in IBM’s PC architecture:
1. Recall that in the BIOS section above (I’m not saying the BIOS is ended now), the instructions stored in some device’s first sector is detected and copied to the RAM, followed by execution of it. These instructions in the first sector contains a **boot loader** (or part of a boot loader).
2. The things that happen next depends on the hardware containing the kernel image:
	* To boot from a **floppy disk**: The instructions in the first sector will copy **all the remaining sectors** containing the kernel image into RAM.
    * To boot from a **hard disk**: Linux uses **two-stage** boot loader to boot its kernel form disk. A well-known boot loader on 80×86 systems is named LInux LOader (**LILO**), another well-known one is GRand Unified Bootloader (**GRUB**).
    * To boot from a hard disk (cont’d): LILO may be installed either on the MBR or in the boot sector of every disk partition. In both cases, the final result is the same: when the loader is executed **at boot time, the user may choose which operating system to load**.
    * To boot from a hard disk (cont’d): LILO boot loader is in fact too large for a **single sector**, so it’s broken into two parts (two-stage), the MBR or the partition boot sector includes a small boot loader, which is loaded into RAM starting from address 0x0007c00 by the BIOS. This small boot loader moves itself to 0x00096a00, sets up the Real Mode stack (ranging from 0x00098000 to 0x000969ff), **loads the second part of the LILO boot loader** into RAM starting from 0x00096c00, and jumps into it.
    * To boot from a hard disk (cont’d): This latter part of boot loader reads a map of bootable operating systems from disk and offers the user a prompt so she can **choose one of them**. After the user picked the operating system to boot, the boot loader may either copy the boot sector of the corresponding partition into RAM and execute it or directly copy the kernel image into RAM.