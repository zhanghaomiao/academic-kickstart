---
title: OS structures
# linktitle: Tips 1-2
toc: true
type: docs
date: "2019-05-05T00:00:00+01:00"
draft: false

menu:
  operating_system:
   parent: Concepts
   weight: 2
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 1
---


## OS services

- OS services: Communication,  Error detection,  User Interface,  Resource allocation,  Program Execution
- User Interface
  - CLI (Command Line Interface)
    Shell: Command-line interpreter (CSHELL, BASH)
  - GUI (Graphic User Interface)
    Icons,  Mouse
- Communication Models
  - Message passing
    例如， process A 与 process B 交换信息： 把 processA 的 信息 copy 到 os 的 memory,  再从 os 把 process A 的信息 copy 到 process B。(Protection 机制, 需要使用 system call)， 缺点是较慢
  - Shared memory
    有一块公用的 memory, 两个 process 可以直接使用，需要先通过 system call 分配好。 (Multi-thread programming)， 缺点是有可能有 dead-lock problem

## System Calls & API

Request OS services:  Process control, File management, Device management, Information maintenance, Communication

- System calls (简单， bug-free)
  - **OS interface** to running program
  - An explicit request to the kernel made via a **software interrupt**
  - Available as **assembly-language** instructions
- API: Application Program Interface (方便使用者, simplicity, portability, efficiency)
  - Users mostly program against API instead of system call
  - Commonly implemented by language libraries, e.g. C library
  - An api call could involve zero or multiple system call
    - `malloc()` and `free()` use system call `brk()`
    - `abs()` don't need to involve system call
- 不会直接产生 Interrupt, 由 system call 产生 interrupt
- Three most common APIs:
  - Win32API for windows
  - POSIX API for POSIX-based systems (Portable Operating System Interface for Unix)  所有的 API 都一样， 但是实作(Library)不一样
  - Java API for the java virtual machine
- System calls: passing parameters
  - Pass parameters in registers
  - Store the parameters in a table in memory, and the table address is passed as a parameter in a register
  - Push the parameters onto the stack by the program, and pop off the stack by operating system

## System Structure

User goals and system goals is different
User goals: easy to use and learn, as well as reliable, safe and fast
System goals: easy to *design*, *implement* and *maintain* as well as reliable, error-free, and efficient

- Simple OS Architecture
  - unsafe, difficult to enhance
  <img style="margin: auto;" src=https://i.loli.net/2019/03/15/5c8b08039972e.png width="40%" />
- Layered OS Architecture
  - lower levels independent of upper levels
  - Easy to debugging and maintenance
  - less efficient, difficult to define layers
  <img src="https://i.loli.net/2019/03/15/5c8b0e784d3f5.png" width="40%" />
- Microkernel OS (有新的系统朝着这个方向做）
  - Moves as much from the kernel info "user" space
  - Communications is provided by message passing, 避免 synchronization
  - Easier for extending and porting
  <img src="https://i.loli.net/2019/03/15/5c8b0a3e2836d.png" width="40%"/>
- Modular OS Architecture
  - Modern OS implement kernel modules
  - Object-oriented Approach
  - Each core component is separate
  - Each to talk each other over known interfaces
  - Each is loadable as needed within the kernel
    <img src="https://i.loli.net/2019/03/15/5c8b0b7fbd37c.png" width="40%"/>
- Virtual Machine
  Hard: Critical Instruction，指的是一些指令在 User space 和 Kernel space 执行的结果不一样， 因此需要知道一些指令是否是 critical instruction.
  <img src="https://i.loli.net/2019/03/15/5c8b1c7b544c4.png" width="40%"/>
  - 虚拟化指的是如何加 virtual-machine implementation layer。
  - 工作流程: 虚拟机的Kernel 在运行的时候是假设是在 kernel space, 因此才可以执行privileged instruction. 但从架构来说，虚拟机的 kernel 应该是存在于 user space 中。 Privileged instruction 在 虚拟机的 Kernel space 执行时，系统会产生 一个 signal (exception)， interrupt 会回到底层的OS， 因此系统会得知虚拟机想要执行privileged instruction， 底层的OS 会重复执行这个命令。
  - 缺点:  虚拟机执行效率比较低。
  - 现在很多 CPU 都支持虚拟机， 除了 User mode, kernel mode, 还有 VM mode。
  - Usage: Protection, compatibility problems (系统兼容)， research and development， honeypot (A virtual honeypot is software that emulates a vulnerable system or network to attract intruders and study their behavior), cloud computing (不需要直接用虚拟机提供， container)
  - Full Virtualization (VMware, 需要有 hardware support, 执行效率才会快)
    - Run in user mode as an application on top of OS
    - Virtual machine believe they are running on bare hardware but in fact are running inside a user-level application
  - Para-virtualization: Xen (存在一个 global zone)
    - Presents guest with system similar but not identical to the guest's preferred systems (Guest must be modified)
    - Hardware rather than OS and its devices are virtualized
  - Java Virtual Machine
    - Code translation, compile 执行完成后, 会有自己的 binary code. 再进行一次translation, 在host os执行。
    - Just-In-Time(JIT) compliers, 会记录执行的命令，然后 reuse.