---
layout: post
title:  "Limited Direct Execution"
date:   2023-07-19 10:40:00 +0530
categories: OSTEP OS
---



>CRUX: How To Efficiently Virtualise the CPU with Control
>OS must virtualise CPU **efficiently** while still **retaining control** over the system, this is done by using both software and hardware.

Limited Direct Execution is a representation of the solution of the Crux -> Direct Execution means low overhead, and limited implies that there is control retained by the OS on what is run, when and for how long.

The "Direct Execution" means what you probably think it means, just directly run the program on the CPU, i.e. create an entry in the process list, load some memory for it, load the program into memory from the disk, locate it's entry point(like the main() method), jump to it and start running the users code.

But this process has some problems:
1. How do we make sure the program we load and run does not do something we do not want and still remain ***efficient***
2. How does the OS stop a process and start running a different process, required to implement **time sharing** to virtualise the CPU

Therefore we have to Limit it.

## Problem #1: Restricted Operations

>THE CRUX: HOW TO PERFORM RESTRICTED OPERATIONS?
>A process must be able to perform I/O and some other restricted operations, but without giving the process complete control over the system. How can the OS and hardware work together to do so?

Letting the process do whatever it wants in terms of I/O and other operations is not an option.
So the other approach is to introduce multiple processor modes:
1. **User Mode**: Code that runs in this mode is restricted, it can't to privileged operations like issue I/O requests, doing so would result in the processor raising and exception; and the OS would kill the process
2. **Kernel Mode**: Mode in which the OS(or the kernel) runs in. In this mode code can do whatever it like.

>Problem: How to execute privileged code from a user process?
>Answer: System Calls: This allows the kernel to expose key pieces of functionality to user programs, like accessing File Systems, creating and destroying processes etc. 
>To execute a system call a program must use a special Trap Instruction.


<br>


**TLDR: Trap Instructions**
1. Simultaneously, jumps into the kernel and raises privilege level to kernel mode
2. Once in the Kernel, system can perform whatever operations are needed(if they are allowed), and do the required work for the calling process.
3. When Done, OS calls a special **return-to-trap** instruction, which, returns into the calling user program while simultaneously reducing the privilege level back to user mode.




## Problem #2: Switching Between Processes
Why do we want to this?
 -> To virtualise the CPU
>Problem: When a process is running on a CPU, by definition the OS is not, so it can't really do anything about it.

Two approaches for this problem
### Co-op Approach: Wait for system calls
Here the OS believes in the goodness of the programmer, and takes advantage of system calls, if there are no system calls for restricted actions. It expects a yield system call.

Why? When a system call is initiated, the process gives up the control of the CPU to the OS, this allows the OS to make scheduling decisions. 
If this does not happen a process could theoretically keep running for an infinite amount of time.

It is easy to see why this approach is a disaster(to put it mildly).

Note: applications also transfer control to the OS when they do something illegal. For Example, diving by zero, or trying to access memory that it shouldn't be accessing, this generates a trap to the OS. The OS then gains control of the CPU again and will mostly likely terminate the offending process.

### Non Co-op Approach: Kernel Takes Control
Usually through a hardware interrupt timer. An interrupt is raised every few milliseconds; when the interrupt is raised, the currently running process is halted, and a pre-configured i**interrupt handler** in the OS runs. At this point OS has control over the CPU and can do with the process whatever it pleases, stop it, start a new one etc.
![THIS IS AN IMAGE](/assets/Pasted image 20230712225139.png)
The following image was taken from OSTEP.
