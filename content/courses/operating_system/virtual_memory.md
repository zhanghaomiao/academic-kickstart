---
title: Virtual Memory
# linktitle: Tips 1-2
toc: true
type: docs
date: "2019-05-05T00:00:00+01:00"
draft: false

menu:
  operating_system:
   parent: Concepts
   weight: 4
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 3
---


## Background

- We don't want to run a program that is entirely in memory
  1. Many code for handling unusual errors or conditions
  2. Certain program routines or features are rarely used
  3. The same library code used by many programs
  4. Arrays, lists and tables allocated but not used
- Virtual memory — separation of user logical memory from physical memory
  - To run a extremely **large process**, logical address space can be much larger than the physical address space
  - To increase **CPU/resources utilization** (higher degree of multiprogramming degree)
  - To **simplify programming** tasks (Free programmer from memory limitation)
  - To run programs **faster** (less I/O would be needed to load or swap)
- Virtual memory can be implemented by
  - Demand paging
  - Demand segmentation: more complicated due to variable size
<img src="https://i.loli.net/2019/03/17/5c8de51380ac7.png" width="400px"/>

## Demand Paging

- A program rather than the whole process is brought into memory only when it is needed
  - Less I/O needed $\rightarrow$ Faster response
  - Less memory needed $\rightarrow$ More users
- Page is needed when there is a reference to the page
  - Invalid reference $\rightarrow$ abort
  - Not-in-memory $\rightarrow$ bring to memory via paging
- Pure demand paging
  - Start a process with no page
  - Never bring a page into memory until it is required
- A swapper (midterm scheduler) manipulates the entire process, whereas a pager is concerned with individual page of a process
- Hardware support
  - Page table : a valid-invalid bit (1 $\rightarrow$ page in memory, $0 \rightarrow$ page no in the memory)
  - Secondary memory (swap space, backing store), usually, a high-speed disk (swap device) is use
<img src="https://i.loli.net/2019/04/01/5ca1897edd6c1.png" width="400px"/>

### Page Fault

First reference to a page will trap to OS (page fault trap)

1. OS looks at the internal table (PCB) to decide
    - Invalid reference $\rightarrow$ abort
    - Just not in memory $\rightarrow$ continue
2. Get an empty frame
3. Swap the page from disk (swap) space into the frame
4. Reset page table, invalid-valid bit = 1
5. Restart instruction
  <img src="https://i.loli.net/2019/03/17/5c8dea0ea45c7.png" width="400px"/>

### Page replacement

If there is no free frame when a page fault occurs

- swap a frame to backing store
- swap a page from backing store into the frame
- different page **replacement algorithms** pick different frames for replacement

### Demand Paging Performance

- Effective Access Time (EAT): $(1-p) \times ma + p \times PFT$
  - P: page fault rate, ma: memory, access time, PFT: page fault time
- Example ma=200ns, PFT=8ms
  - EAT = 200ns + 7999800 ns $\times$ p
- Access time is proportional to the page fault rate
  - If one access out of 1000 causes a page fault, then EAT = 8.2 ms $\rightarrow$ slowdown by a factor of 40
- Programs tend to have locality of reference
- Locality means program often accesses memory addresses that are close together
  - A single page fault can bring in 4KB memory content
  - Greatly reduce the occurrence of page fault
- Major components of page fault time (about 8ms)
  1. serve the page-fault interrupt
  2. **read in the page from disk** (most expensive)
  3. Restart the process

### Process Creation and Virtual memory

- Demand paging: only bring in the page containing the first instruction
- Copy-on-Write: the parent and the child process share the same frames initially, and frame-copy when a page is written
- Memory-Mapped File: map a file into the virtual address space to bypass file system calls (e.g. read(), write())

### Copy-on-Write

- if either process modifies a frame, only then a frame is copied
- COW allows efficient process creation
- Free frames are allocated from a pool of zeroed-out frames, the content of a frame is erased to 0
- Figure: a child process is forked
  <img src="https://i.loli.net/2019/03/17/5c8defdbc3767.png" width="400px"/>
- After parent modifies page C
  <img src="https://i.loli.net/2019/03/17/5c8df0553a9f2.png" width="400px"/>

### Memory-Mapped Files

- Approach:
  - MMF allows file I/O to be treated as routine memory access by mapping a disk block to memory frame
  - A file  is initially read using demand paging. Subsequent read/writes to/from the file are treated as ordinary memory accesses
- Benefit:
  - Faster file access by using memory access rather than `read()` and `write()` system calls
  - Allows several process to map the SAME file allowing the pages in memory to be SHARED
- Concerns:
  - Security(access control), data lost, more programming efforts
  <img src="https://i.loli.net/2019/03/17/5c8df3f01bddd.png" width="400px"/>

## Page replacement

- when a page fault occurs with on free frame
  1. swap out a process,  freeing all its frames
  2. page replacement, find one not currently used add free it.
     Use **dirty bit** to reduce overhead of page transfers -- only modified pages are written to disk
- Solve two major problems for demand paging
  - Frame-allocation algorithm, determine how many frames to be allocated to a process
  - Page-replacement algorithm, select which frame to be replaced

### Page Replacement Algorithms

- Goal: lowest page-fault rate
- Evaluation: running against a string of memory references (reference string) and computing the number of page faults
- Reference String: 1,2,3,4,1,2,5,1,2,3,4,5

### FIFO algorithm

1. The oldest page in a FIFO queue is replaced
2. 3 frames (available memory frames = 3) 9 page faults
  <img src="https://i.loli.net/2019/03/17/5c8df74139338.png" width="400px"/>
3. FIFO illustrating Belady's Anomaly
  More allocated frames doesn't guaranteed less page fault
  <img src="https://i.loli.net/2019/03/17/5c8df806c0447.png" width="400px"/>
4. Figure illustration
   <img src="https://i.loli.net/2019/03/17/5c8df86990ad6.png" width="400px"/>

### Optimal (Belady) Algorithm

1. Replace the page that will not be used for the longest period of time, need future knowledge
2. 4 frames (6 page faults)
3. In practice, we don't have future knowledge, only used for reference and comparison
  <img src="https://i.loli.net/2019/03/17/5c8e05f397a22.png" width="400px"/>
  
### LRU Algorithm (Least Recently Used)

- An approximation of optimal algorithm, looking backward, rather than forward
- It replaces the page that has not been used for the longest period of time
- It is often used, and is considered as quite good
- Counter implementation
  1. page referenced: time stamp is copied into the counter
  2. replacement: remove the one with oldest counter, but linear search is required
- Stack implementation
  1. page referenced: move to top of the double-linked list
  2. replacement: remove the page at the bottom
  <img src="https://i.loli.net/2019/03/17/5c8e078801cb9.png" width="400px"/>

#### Stack Algorithm

1. A property of algorithms
2. Stack algorithm: the set of pagers in memory for n frames is always a subset of the set of pages that would be in memory with n+1 frames
3. Stack algorithms do not suffers from Belady's anomaly
4. Both optimal algorithm and LRU algorithm stack algorithm

### LRU approximation algorithms

Few systems provide sufficient hardware support for the LRU page-replacement

1. additional-reference-bits algorithm
2. second-chance algorithm
3. enhanced second-chance algorithm

### Counting Algorithms

1. LFU algorithms (least frequently used)
    - keep a counter for each page
    - idea: an actively used page should have a large reference count
2. MFU algorithms (most frequently used)
    - idea: the page with the smallest count was probably just brought in and has yet to be used

Both counting algorithms are not common

1. implementation is expensive (overflow)
2. do not approximate OPT algorithm very well

## Allocation of frames

- each process needs minimum number of frames
- Fixed allocation
  - Equal allocation -- 100 frames, 5 processes $\rightarrow$ 20 frames/process
  - Proportional allocation -- Allocate according to the size of the process
- Priority allocation
  Using proportional allocation based on priority, instead of size.
- Local allocation: each process select from its own set of allocated frames
- Global allocation : process selects a replacement frame from the set of all frames
  - One process can take away a frame of another process
  - e.g. Allow a high-priority process to take frames from a low-priority process
  - Good system performance and thus is common used
  - A minimum number of frames must be maintained for each process to prevent **trashing**

### Trashing

- If a process dost not have "enough" frames to support pages in active page $\rightarrow$ very high paging activity
- A process is trashing if it spending more time paging than executing
  <img src="https://i.loli.net/2019/03/17/5c8e0d69140ec.png" width="400px"/>
- Performance problem caused by trashing
  Processes queued for I/O to swap (page fault) $\rightarrow$ low CPU utilization $\rightarrow$ OS increased the degree of multiprogramming $\rightarrow$ new processes take frames from old processes $\rightarrow$ CPU utilization drops even further
- To prevent trashing, must provide enough frames for each process (Working-set model, page-fault frequency)

#### Working-Set Model (实现起来比较繁琐， 比较少用)

- Locality: a set of pages that are actively used together
- Locality model: as a process executes, it moves from locality to locality (program structure, data structure)
- Working-set model
  1. working-set window: a parameter $\Delta$
  2. working set: set of pages in most recent $\Delta$ page reference (an approximation locality)
<img src="https://i.loli.net/2019/03/17/5c8e0fedf1e67.png" width="400px"/>
- Explanation
  - $WSS_i:$ Working-set size  for process i
  - $D = \sum WSS_i:$  Total demand frames
  - If $D > m$ (available frames) $\rightarrow$ trashing
  - The OS monitors the $WSS_i$ of each process and allocates to the process enough frames: if $D << m$ , increase degree of MP, if $D > m$, suspend a process.
- Advantages: prevent thrashing while keeping the degree of multiprogramming as high as possible , optimize CPU utilization

#### Page Fault frequency scheme

- page fault frequency directly measures and controls the page-fault rate to prevent trashing
- Establish upper and lower bounds on the desired page-fault rate of a process
- If page fault rate exceed the upper limit, allocate another frame to the process
- if page fault rate fails below the lower limit, remove a frame from the process
<img src="https://i.loli.net/2019/03/17/5c8e249f97b68.png" width="400px"/>