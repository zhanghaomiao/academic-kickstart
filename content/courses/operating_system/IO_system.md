---
title: I/O system
toc: true
type: docs
date: "2019-05-05T00:00:00+01:00"
draft: false

menu:
  operating_system:
   parent: Concepts
   weight: 12
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 11
---

## Overview

- The two main jobs of a computer I/O and computation
- I/O devices: tape, HD, mouse, joystick, screen
- I/O subsytems: the methods to control all I/O devices
- Two conflicting trends
  - Standardization of HW/SW interfaces
  - Board variety of I/O devices
- Device drivers: a uniform device-access interface to the I/O subsystem(Simliar to system calls between apps and OS)
- I/O Hardware
  - Port: A connection point between I/O diveces and the host(E.g. USB ports)
  - Bus: A set of wires and a well-defined protocal that specifies message sent over the wires (E.g. PCI bus)
  - Controller: A collecton of electronics that can operate a port, a bus, or a device (A controller could have its own processor, memory, etc(E.g. SCSI controller))

<img src="https://i.loli.net/2019/03/22/5c94719d776dd.png" width="400px"/>

## Basic I/O method (Port-mapped I/O)

- Each I/O port (device) is identified by a unique port address
- Each I/O port consists of four registers (1~4 Bytes)
  - Data-in regsiter: read by the host to get input
  - Data-out register: written by the host to send output
  - status register: read by the host to check I/O status
  - Control register: written by the host to control the device
- Program interact with an I/O port through special I/O instructions (differnet from memory access)

## I/O methods Categorization

- Depends on how to address a deive
  - port-mapped I/O: use different address space from memory, access by special I/O instruction(e.g. IN, OUT)
  - Memory-mapped I/O: Reserve specific memory space for device, Access by standard data-transfer instruction (e.g. MOV) (more efficient for large memory I/O, vulnerable to accidental modification)
- Depends on how to interact with a deivce:
  - Poll(busy-waiting): processor periodically check status register of a device
  - Interrupt: device notify processor of its completion
- Depending on who to control the transer:
  - Programmed I/O: transfer controlled by CPU
  - Direct memory access(DMA) I/O: controlled by DMA controller(A special purpose controller, design for large data transfer)
  <img src="https://i.loli.net/2019/03/22/5c94740d3444a.png" width="400px"/>

## Kernel I/O Subsystem

- I/O scheduling -- improve system performance by ordering the jobs in I/O queue (e.g. disk I/O order scheduling)
- Buffering -- store data in memory while transferring between I/O deivces
  - Speed mismatch between devices
  - Devices with different data-transfer sizes
  - Support copy semantics
- Caching -- fast momory that holds copies of data (Key to performance)
- Spooling -- holds output for a device(e.g. printing, cannot accept interleaved files)
- Error handlig -- when I/O error happens(e.g. SCSI device returns error information)
- I/O protection(privileged instructions)

### Blocking and Nonblocking I/O

- Blocking -- process suspended until I/O completed
  - Easy to use and understand
  - Insufficient for some needs
  - Use for synchronous communication & I/O
- Nonblocking
  - Implemented via multi-threading
  - Returns quickly with count of bytes read or wriiten
  - Use for asynchronous communication & I/O

### Transforming I/O requests to Hardware operations

<img src="https://i.loli.net/2019/03/22/5c947676a3809.png" width="400px"/>

### Performance and Improving

- I/O is a major factor in system performance
  - It placs heavy demancs on the CPU to execute device driver code
  - The resulting context switches stress the CPU and its hareware caches
  - I/O loads dwon the memory bus during data copy bewteen controllers and physcial memory
  - Interrupt handling is a relatively expensive task
  - Busy-waiting could be more efficient than interrupt-driven if I/O time is small
- Improving Performance
  - Reduce number of context switches
  - Reduce data copying
  - Reduce interrupts by using large transers, smart controllers, polling
  - Use DMA
  - Balance CPU, memory, bus and I/O performance for highest throughput

## Application I/O interface

- Device drivers: a uniform device-access interface to the I/O subsystem; hide the differences among device controllers from the I/O sub-system of OS
