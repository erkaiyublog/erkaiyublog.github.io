---
published: false
---
---

_What happens after you switch on your computer? What is a bootloader? What is BIOS? ..._

_Don't worry, this post will answer all these annoying questions._

---
### References
Appendix A of [ULK(Understanding Linux Kernel), the 3rd Edition](https://www.amazon.com/Understanding-Linux-Kernel-Third-Daniel/dp/0596005652). 
[UEFI vs BIOS: What's the Difference?](https://www.freecodecamp.org/news/uefi-vs-bios/#:~:text=UEFI%20provides%20faster%20boot%20time,booting%20from%20unauthorized%2Funsigned%20applications.)

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
    * To boot from a hard disk (cont’d): If the Linux kernel image has to be copied and booted, here’s what LILO will do:
    	1. Invokes a BIOS procedure to display a “Loading” message.
        2. Invokes a BIOS procedure to load an **initial portion of the kernel image** from disk: the first 512B of the kernel image are put in RAM at 0x00090000, the **setup()** function is put in RAM starting from 0x00090200.
        3. Invokes a BIOS procedure to load the rest of the kernel image from disk and put it in AM starting from either 0x00010000 (for small kernel images compiled with **make zImage**, such process is so called “loaded low”) or 0x00100000 (for big kernel images compiled with **make bzImage**, such process is so called “loaded high”).
        4. Jumps to **setup()**.

### Step 2. setup()
setup() is assembly code that placed by the **linker** at **offset 0x200 of the kernel image file**. Boot loader will copy it into RAM at 0x00090200.

setup() initializes the hardware devices in the computer and set up the environment for execution of the kernel program.

Although the BIOS already initialized most hardware devices, Linux doesn’t rely on it.

setup() does the following things:
* In ACPI-compliant systems, it invokes a BIOS routine that builds a table in RAM describing the layout of the system’s physical memory.
* Sets the **keyboard repeat delay and rate**.
* Initializes the video adapter card.
* Reinitializes the disk controller and determines the hard disk parameter.
* Checks for an IBM Micro Channel bus.
* Checks for a PS/2 pointing device (bus mouse).
* Checks for Advanced Power Management BIOS support.
* If the BIOS supports the Enhanced Disk Drive Services, it invokes the proper BIOS procedure to build a table in RAM describing the hard disks available in the system.
* If the kernel image was **loaded low** in RAM, the setup() function now moves it to 0x00001000 (not the same address with either “loaded low” or “loaded high”).
* Sets the A20 pin located on the 8042 keyboard controller.
* Sets up a provisional IDT and a provisional GDT.
* Resets the floating-point unit (FPU), if any.
* Reprograms the **Programmable Interrupt Controllers** (PIC) to mask all interrupts, except IQR2 (If you don’t know why, check for cascading interrupt between two PICs).
* Switches **the CPU from Real Mode to Protected Mode** by setting the PE bit in the cr0 status register.
* Jumps to the **startup_32()** function.

### Step 3. startup_32() & Step 4. start_kernel()
There're also tedious setup functions following setup(), but I'm not including them here.

## What about UEFI?
UEFI stands for Unified Extensible Firmware Interface, it’s another way to boot the operating system.

The main difference between UEFI and BIOS is that UEFI stores all data about initialization and startup in an **.efi** file, instead of storing it on the firmware.

This **.efi** file is stored on a special partition called EFI System Partition (ESP) on the hard disk. This ESP partition also contains the bootloader.

UEFI usually boots faster than BIOS, and it also supports larger hard disk partition than BIOS does.

UEFI runs in 32-bit or 64-bit mode. BIOS runs only in 16-bit mode and may utilize only 1 MB of executable memory.