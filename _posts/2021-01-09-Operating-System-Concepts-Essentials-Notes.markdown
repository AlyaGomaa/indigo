---
title: "Operating System Concepts Essentials Notes"
layout: post
date: 2021-01-09 10:00
image: /assets/images/markdown.jpg
headerImage: false
tag:
- Notes
category: blog
author: coreflood
description: ~.~
---

Here's a small summary i made of the [Operating System Concepts Essentials](http://dusithost.dusit.ac.th/~juthawut_cha/download/Operating_System_Concepts_Essentials_2nd_Edition.pdf) book

I hope it helps

---

(CPU), the memory, and the input/output (I/O) devices are **basic computing resources**

maximize resource utilization— to assure that all available resources are used efficiently and that no individual user takes more than his fair share

Each **device controller** is in charge of a specific type of device (for example, disk drives, audio devices, or video displays)

Any OS consists of : Kernel + system programs[daemons]

how do **software** and **hardware** make events/interrupts/traps/break points?
**In software** -> sys call
**In hardware** -> interrupt signal to the CPU using the bus

**When an interrupt occurs**, each interrupt has a specific number in a table of interrupt type, function pointers and device number. the CPU checks that table before executing the function associated with the interrupt

**RAM** commonly is implemented in a semiconductor technology called dynamic random-access memory **(DRAM)**

electrically erasable programmable read-only memory,  **EEPROM** , it can be changed. in smartphones it stores their factory-installed Programs

Because **ROM cannot be changed**, only static programs, such as the bootstrap program are stored there.

**Load** -> loads from RAM to registers
**Store** -> from regs to ram

The CPU immediately executes from RAM
storage order according to speed and cost:

img

Volatile storage loses its content when the power is removed  like RAM

from ssd downwars -> non volatile 

Typically, operating systems have a device driver for each device controller. [one for the GRAPHICS CARD , KEYBOARD ETC..] 

disk i/o uses direct memory access (DMA) , which means that the device controller transfers an entire block of data directly to or from its own buffer storage to memory, with no intervention by the CPU. 

there are always special purpose processors next to the single main  processor (general purpose processor) for eg : the keyboeard's microprocessor that translate each char to its ascii

multiprocessor systems (also known as parallel systems or multicore systems) 

**graceful degradation**: fault tolerant which means that the pc has to distribute tasks among the rest of the cores if one of them failed

The multiple-processor systems in use today are of two types: 
- **asymmetric multiprocessing **: the boss processor gives commands  and tasks to  other processors , can be implemented using hardware or software 

- **symmetric multiprocessing (SMP)** : means each processor performs a different task  

**UMA uniform memory access** : the situation in which access to any RAM from any CPU takes the same amount of time. 
 
**NUMA**: some parts of memory may take longer to access than other parts 
 
multiple cores are faster and less power consuming than multiple CPU chips because on-chip communication is faster than between-chip communication. 
 
Each core has its own regs and (not all designs )( own cache memory) 
 
**Multicore CPUs** appear to the operating system as processors. 
 
clustering : nafs 7war el distributed systems en kaza computer aw server mtwsl bb3dhom bLAN 
 wel app el wahed byt2sm eno yrun 3lehom kolhom 
**Clustering**: means one app is devided into tasks that run on different LAN connected servers 

**asymmetric clusters:** one server is in stand-by mode and monitors the rest to replace them in case something happens/
**symmetric vlusters:** they're all monitoring each other 

clusters give very high computational power 
 
**Parallelization : ** divides a program into separate components that run in parallel on individual cores in a computer, it can be implemented using hardware or software.
 
**storage-area networks (SANs)** allow many systems to attach to a pool of storage. provides access to consolidated, block-level data storage.

**In time-sharing {multiprogramming} systems: ** the CPU executes multiple jobs by switching among them 

**software-generated interrupt** is caused either by an error (division by zero or invalid memory access) or by a syscall .
 
user mode (**mode bit = 1**) 
kernel mode( **mode bit =0**) 
that tells the os whether i'm in the userdefined code or the os code 
 
CPUs that support **virtualization** has a user mode, a kernel mode and another mode kman that has its own bit set whenever there's a VM running because it needs more privileges than user mode and less than the kernel mode.

**Intel 64 family** of CPUs supports** four privilege levels (rings)** and supports virtualization but does not have a separate mode for virtualization. 
 
MS-DOS was written for the **Intel 8088 architecture** which has no mode bit. A user program can wipe out the operating system by writing over it with data. 
 
**if a program gets stuck in an infinite loop** A timer can be set to interrupt the computer after a specified period. control transfers automatically to the operating system after that time 
 
consider a process to be a job or a time-shared program 

When a process terminates, the operating system will reclaim any reusable resources. 
**Multithreaded programs** have one eip afor each thread 

----

**Firmware == bios == bootsrap program:** resides in ROM : first prog to execute when a computer boots. initializes the hardware ( regs and memory ) loads the os and the kernel into memory and waits for an event 

**BIOS (basic i/o system)** is the software that resides in rom 
**firmware** used to be called **bios** , now it's called uefi or efi.  
network cards , ssds , video cards all have firmware 

changes made to the bios are stored in the **cmos chip **

uefi has a feature called **secure boot** that restricts any unsigned drivers from booting and protects the computer from rootkits 

**cache coherency :**  means updating every cache as long as the value i want to change in still in use

---

we need VRAM bc main memory is too small and volatile 
 
**tertiary-storage ** any removable media like usbs disks CDs theyre usually slower
 
when the cpu needs data it first checks the cache , if it's unable to find it there it stores it there.
**chache memory** has an instruction cache that hhols the next instruction. without it the cpu would wait several cycles to fetch the instruction from RAM
 
registers are controlled by the compiler  
cache by the hardware 
ram, ssd, disks by OS 

**singly linked list: ** each item points to its successor 
**doubly linked list:** either to its predecessor or to its successor 

**bitmap :** binay string where every bit represents a  state 

every **disk drive** is divided into several thousand individual units, called** disk blocks **
 
when **emulating a different CPU** evey instruction has its own function that gets executed on the actual CPU thats why its slower
 
**Interpretation is a form of emulation**, emulating not another CPU but a theoretical virtual machine on which that language could run natively (python and java) 
 
**iOS** is open-source kernel,  closed-source components 
**Darwin,** the core kernel component of Mac OS X,is based on BSD 
 
**BSD** is a 'unix-like' complete OS, with its own kernel and its own userland (**no linux kernel nor GNU**). GNU/Linux and *BSD family (FreeBSD, OpenBSD and NetBSD) are 'unix-like' OS, they behave like Unix. 

 
the gnu project owner  never released an open source kernel, linus trovalds is the one who published the kernel  
 
a **LiveCD**--an OS thats boots and run from a CD-ROM without being installed on a system’s boot disk 

**kernel mode:** 
- Certain instructions could be executed 
- hardware devices could only be accessed  

**run-time support system: **(a set of functions built into libraries included with a compiler) 

A running program needs to be able to halt its execution either normally (**end()**) or abnormally (**abort()**). 

---


