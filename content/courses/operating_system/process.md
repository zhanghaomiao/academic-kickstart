---
title: Process
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


## Concept

- Program: passive entity: binary stored in disk
- Process: active entity: a program in execution in memory
- A process includes:
  - Code segment
  - Data section --global variables
  - Stack -- temporary local variables and functions
  - Heap -- dynamic allocated variables or classes
  - Current activity (program counter, register contents) 用来管理 process
  - A set of associated resources (e.g. open file handlers)
- Process in memory
    <img src="https://i.loli.net/2019/03/15/5c8b3959e8cd0.png" width="40%"/>
- Threads
  - lightweight process
  - All threads belonging to the same processes share **code section, data section and OS resources (e.g. open files and signals)**
  - Each thread has it own **thread ID, program counter, register set and stack**
- Process States
  - New: process is being created
  - Ready: process is in the memory waiting to be assigned to a processor, 放到 waiting **queue** 中
  - Running: instructions are being executed by CPU
  - Waiting: the process is waiting for events to occur (e.g. IO)
  - Terminated: the process has finished execution
    <img src="https://i.loli.net/2019/03/15/5c8b3b57c8609.png" width="40%"/>

## Process switch

- Process control Block
  <img src="https://i.loli.net/2019/03/15/5c8b3e1bb0ea9.png" width="20%"/>
- context switch
  Context-switch time is purely overhead
  <img src="https://i.loli.net/2019/03/15/5c8b3f9767fd1.png" width="50%"/>
  <div class="note warning"><p>Switch time depends on memory speed, number of registers, existence of special instructions (a single instruction to save/load all registers), hardware support (multiple sets of registers) </p></div>

## Process Scheduling

- Concept
  - Multiprogramming: CPU runs process at all times to maximize CPU utilization
  - Time sharing: switch CPU frequently such that users can interact with each program while it is running
  - Processes will have to wait until the CPU is free and  can be re-scheduled
- Scheduling Queues
  - Job queue (New State) -- set of all processes in the system
  - ready queue (Ready state) -- set of all processes residing in main memory, ready and waiting to execute
  - device queue (Wait state) -- set of processes waiting for an I/O device
- Diagram
  <img src="https://i.loli.net/2019/03/15/5c8b452e66392.png" width="40%"/>
- Schedulers
  - Short- term scheduler (CPU scheduler) -- selects which process should be executed and allocated CPU (Ready state $\longrightarrow$ Run state) 很频繁
  - Long-term scheduler (job scheduler) -- selects which processes should be loaded into memory and brought into ready queue (New state $\longrightarrow$ Ready state)
  - Middle-term scheduler -- selects which processes should be swapped in/out memory (Ready state $\longrightarrow$ Wait state) 与 virtual memory 相结合
  <img src="https://i.loli.net/2019/03/15/5c8b46dfcfc3b.png" width="50%"/>
- Long-Term Scheduler (现在memory 很大， 现在变为 middle-term scheduler)
  - control degree of multiprogramming (Degree 很少时，cpu 会 idle， Degree 很多时， 会竞争CPU 资源)
  - Execute less frequently (e.g. invoked only when a process leaves the system or once several minutes)
  - Select a **good mix of CPU-bound & I/O- bound** processes to increase system overall performance
- Short-Term Scheduler
  - Execute quite frequently (e.g. once per 100ms)
  - Must be efficient (averaging wait time)
     <img src="https://i.loli.net/2019/03/15/5c8b49ed102a6.png" width="50%"/>
- Medium-Term Scheduler
  - Swap out: removing processes from memory to reduce the degree of multiprogramming
  - Swap in: reintroducing swap-out processes into memory
  - Purpose: improve process mix, free up memory
  - Most modern OS doesn't have medium-term scheduler because having sufficient physical memory or using virtual memory

## Operations on Processes

### Tree of process

Each process is identified by a **unique** processor identifier (**pid**)
<div class="note info"><p> `ps-ael` will list complete info of all active processes  in unix </p></div>

### Process Creation

- Resource sharing
  - Parent and child processes share **all** resources
  - Child process shares **subset** of parent's resources
  - parent and child share **no** resources
- Two possibilities of execution
  - Parent and children **execute concurrently**
  - parent **waits until children terminate**
- Two possibilities of address space
  - Child duplicate of parent (sharing variables)
  - Child has a program loaded into it (message passing)

### UNIX/LINUX Process Creation

- `fork` system call
  - Create a new child process
  - The new process duplicates the address space of its parent
  - Child & parent execute **concurrently** after fork
  - Child: return value of fork is 0
  - Parent: return value of fork is PID of the child process
- `execlp` system call
  **Load a new binary file** into memory, **destroying the old code** (memory content reset)
- `wait` system call
  The parent waits for one of its child processes to complete
- Memory space of `fork()`,  A 调用 `fork()` 时
  - old implementation: A's child is an extra copy of parent
  - current implementation: use copy-on-write technique to store differences in A's child address space
  <img src="https://i.loli.net/2019/03/15/5c8b4f7e2ea39.png" width="40%"/>

### Process Termination

- Terminate when the last statement is executed or `exit()` is called
- Parent may terminate execution of children processes by specifying its PID (abort)
- Cascading termination
  killing (exiting) parent $\longrightarrow​$ killing all its children

<div class="note info" <p> Control-C , OS在启动时，会产生 console process, 在执行程序时，是 console 再去产生其他程序，按 Control-C 时是用 console 来终止程序</p></div>
<div class = "note info" <p> kill, 通过 OS 来终止程序, 需要权限</p></div>

## Interprocess Communication (IPC)

- IPC: a set of methods for the exchange of data among multiple threads in one or more processes
- Independent process: cannot affect or be affected by other process
- Cooperating process:  otherwise
- Purposes: information sharing, computation speedup, convenience, modularity

### Communication Methods

- shared memory:
  - Require more careful user synchronization
  - implemented by memory access: faster speed
  - use memory address to access data
- Message passing:
  - No conflict : more efficient
  - Use send/recv  message
  - Implemented by system call : slower speed
- Sockets:
  - A network connection identified by IP & port (port number is process)
  - Exchange unstructured stream of bytes
- Remote Procedure Calls:
  - Cause a procedure to execute in another space
  - Parameters and return values are passed by message
     <img src="https://i.loli.net/2019/03/15/5c8b6d362e44b.png" width="40%"/>

### Shared Memory

- Establishing a region of shared memory
  - Typically, a shared-memory region resides in the address space of the process creating the shared-memory segment
  - Participating processes must agree to **remove memory access constraint** from OS
- Determining the form of the data and the location
- Ensuring data are not written simultaneously by processes
- Consumer and Producer problem (系统里面都有很多这样的问题, Compile(Producer), Link(Consumer))
  - **Producer** process produces information that is consumed by a **Consumer** process
  - Buffer as a circular array with size B
    - next free: in
    - first available : out
    - empty: in = out
    - full (in + 1) % B = out
  - The solution allows at most (B-1) item in the buffer, otherwise, cannot tell the buffer is fall or empty
<img src="https://i.loli.net/2019/03/15/5c8b71433d5d2.png" width="30%"/>

```C
while(1){
    while (((in+1) % BUFFER_SIZE) == out); // wait if buffer is full
    buffer[in] = nextProduced;
    in = (in + 1) % BUFFER_SIZE;
}

while(1){
        while(in == out); // wait if buffre is empty
    nextConsumed = buffer[out];
    out = (out+1) % BUFFER_SIZE
}
```

### Message-Passing System

- Mechanism for processes to communicate and synchronize their actions
- IPC facility provides two operations:
  - Send (Message) -- message size fixed or variable
  - Receive(Message)
- Message system -- process communicate without resorting to shared variables
- To communicate, processes need to
  - Establish a communication link
  - Physical (HW bus, network)
    - Logical (logical properties)  1. Direct or indirect communication 2. Blocking and non-blocking
  - Exchange a message via send/receive
- Direct Communication
  - Process must name each other explicitly
  - Links are **established automatically**
  - **One-to-One** relationship between links and processes
  - The link may be unidirectional, but is usually bi-directional
  <div class="note warning"><p> Limited modularity, if the name of a process is changed, all old names should be found</p></div>
- Indirect communication (E-mail)
  - Messages are directed and received from mailboxes
  - Each mailbox has a unique ID
  - Processes can communicate if the share a mailbox
  - Send(A, message) , Receive(A, message), communicate through mailbox A
  - **Many-to-Many** relationship between links and processes
  - Link established only if processes share a common mailbox
  - Mailbox can be owned either by OS or processes
  - MailBox 怎么解决一对一的问题？
    <img src="https://i.loli.net/2019/03/15/5c8b88e397ddd.png" width="50%"/>
    Solutions:
    - Allow a link to be associated with at most two processes
    - Allow only one process at a time to execute a receive operation
    - Allow the system to select arbitrarily a singe receiver. Sender is notified who the receiver was
- Synchronization
  - Message passing may be either blocking (synchronous) or non-blocking (asynchronous)
  - **Blocking send:** send is blocked until the message is received by receiver or by the mailbox
  - **Nonblocking send:** sender sends the message and resumes operation
  - **Blocking receive:** receiver is blocked until the message is available
  - **Nonblocking receive:** receiver receives a valid message or a null (存在一个 token, 来判断是否收到信息)
- Buffer implementation (中间存在一个 buffer 来进行消息的储存)
  - Zero capacity: blocking send/receive
  - Bounded capacity: if full, sender will be blocked
  - Unbounded capacity: sender never blocks
- Sockets
  <img src="https://i.loli.net/2019/03/15/5c8b906940d23.png" width="45%"/>
- Remote Procedure Calls: RPC
  Stubs, client-side proxy for the actual procedure on the server
  <img src="https://i.loli.net/2019/03/15/5c8b90e017de2.png" width="40%"/>