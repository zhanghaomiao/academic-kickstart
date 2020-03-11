---
title: Process Scheduling
toc: true
type: docs
date: "2019-05-05T00:00:00+01:00"
draft: false

menu:
  operating_system:
   parent: Concepts
   weight: 6
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 5
---

## Introduction

**CPU-I/O burst cycle**: Process execution consists of a cycle of CPU execution and I/O wait

- Generally, there is a large number of short CPU bursts, and a small number of long CPU bursts
- A I/O bound program would typically has many very short CPU bursts
- A CPU-bound program might have a few long CPU bursts
<img src="https://i.loli.net/2019/03/17/5c8e3c19c049e.png" width="400px"/>
CPU scheduler: Select from ready queue to execute (i.e. allocates a CPU for the selected process)

### Preemptive vs. Non-preemptive

- CPU scheduling decisions may take palce when a process:
  1. Switches from running to waitting state (IO)
  2. Switches from running to ready state (Time-sharing)
  3. Swtiches from waiting to ready state
  4. Terminates
- Non-preemptive scheduling (不会打断)
  - Scheduling under 1 and 4 (no choice in terms of scheduling)
  - The process keeps the CPU until it is terminated or switched to the waitting state
- Preemptive scheduling, Scheduling under all cases 使用率很高
- Preemptive Issues
  - Inconsistent state of shared data, require process synchronization, incurs a cost assocated with access to shared data
  - Affect the design of OS kernel
    Unix solution: waiting either for a system call to complete or for an I/O block to take palce before doing a context switchd (disable interrupt)

### Dispatcher

Dispatcher module gives control of the CPU to the process selected by scheduler

- switching context
- jumping to the proper location in the selected program
Dispatch latency -- time it takes for the dispatcher to stop one process and start another running

## Scheduling Algorithms

### Scheduling Criteria

- CPU utilization theoretically: $0\% ~ 100\%$, real systems : $40\% ~ 90\%$
- Throughput: number of completed processes per time unit
- Turnaround time (submission ~ completion)
- Waiting time (total waiting time in the ready queue)
- Response time (submission ~ the first response is produced (第一个CPU burst(执行)的时间))

### Algorithms

#### First-Come, First-served (FCFS) scheduling

- Process (Burst Time) in arriving order: P1(24), P2(3), P3(3)
- Gantt chart <img src="https://i.loli.net/2019/03/17/5c8e43e08ae78.png" width="400px"/>
- Waiting time P1 = 0, P2 =24, P3 =27
- Average waiting time (AWT) (0+24+27)/3 = 17
- Convoy effect: short process behind a long process

#### Shortest-Job-First (SJF) scheduling

- Associate with each process the length of its next CPU burst
- A process with shortest burst length  gets the CPU first
- SJF provides the minimum average waiting time (optimal)
- Two schemes
  - Non-preemptive -- once CPU given to a process, it cannot be preempted until its completion
  - Preemptive -- if a new process arrives with shorter burst length, preemption happens
- Non-preemptive SJF example  

| Process | Arrival Time | Burst Time |
| ------- | ------------ | ---------- |
| P1      | 0            | 7          |
| P2      | 2            | 4          |
| P3      | 4            | 1          |
| p4      | 5            | 4          |
  <img src="https://i.loli.net/2019/03/18/5c8ef25a8626f.png" width="400px"/>
- AWT = [(7-0-7) + (12 - 2 -3) + (8 -4-1) + (16-5-4)] /4 = 4
  - Response Time: p1 =0, p2 = 6, p3=3, p4 = 7
- Preemptive SJF example
   <img src="https://i.loli.net/2019/03/18/5c8ef3f119e66.png" width="400px"/>
   - AWT = 3
   - Response time P1 = 0, P2 = 0, P3 = 0, P4 = 2
<div class="note warning"><p>SJF difficulty: no way to know length of the next CPU burst</p></div>
- Approximate SJF: the next burst can be predicted as an exponentail average of the measured length of previous CPU bursts (set $\alpha = 1/2$)
  $$\tau_{n+1} = \alpha t_n + (1-\alpha) \tau_n = \frac12 t_n + \frac14 t_{n_1} + \frac18 t_{n-2}​$$
<img src="https://i.loli.net/2019/03/18/5c8ef586096a8.png" width="400px"/>

#### Priority Scheduling

- A priority number is associated with each process
- The CPU is allocated to the highest priority process
- SJF is a priority scheduling where priority is the predicted next CPU burst time
- Problem: Starvation (low priority process never execute)
- Solution: **aging**(as time progresses increase the priority of process)

#### Round-Robin(RR) Scheduling

- Each process gets a small unit of CPU time(time quantum)
- After TQ elapsed, process is preempted and added to the end of the ready queue
- TQ large $\rightarrow$ FIFO
- TQ small $\rightarrow$ (context switch) overhead increases

#### Multilevel Queue Scheduling

- Ready queue is partitioned ino separate queues
- Each queue has its own scheduling algortihm
- Scheduling must be done between queues
  - Fixed priority scheduling: prossibility of starvation
<img src="https://i.loli.net/2019/03/18/5c8efd2999070.png" width="400px"/>

#### Mutillevel Feedback Queue Schedule

- A process can move between the various queues; aging can be implemented
- Idea:  separate processes according to the characteristic of their CPU burst
  - I/O-bound and interactive processes in higher priority queue $\rightarrow$ short CPU burst
  - CPU-bound processes in lower priority queue long CPU burst
 <img src="https://i.loli.net/2019/03/18/5c8efe2739cf9.png" width="350px"/>
- multilevel feedback queue scheduler is defined by the following parameters:
  - Number of queues
  - Scheduling algorithm for each queue
  - Method used to determin when to upgrade/demote a process

#### Evaluation methods

- Deterministic Modeling — takes a particular predetermined workload and defines the performance of each algorithm for the workload
- Queueing Model — mathematical analysis
- Simulation — random-number generator or trace tapes for workload generation
- Implementation — the only completely accurate way for algorithm evaluation

## Mutli-Processor Scheduling & Multi-Core Processor Scheduling & Real-Time Scheduling

- Asymmetric multiprocessing
  - All system activities are handled by a processor (alleviationg the need for data sharing)
  - the other only execute user code (allocated by the master)
  - far simple than SMP
- Symmetric multiprocesiing (SMP):
  - each processor is self-scheduling
  - all processor in common ready queue, or each has its own private queue of ready processes
  - need synchronization mechanism
- Processor affinity
  A process has an affinity for the processor on which it is currently running
  - A process populates its recent used data in cache memory of its running processor
  - cache invalidation and repopulation has high cost
  - Soft affinity: possible to migrate between processors
  - hard affinity: not to migrate to other processor
- NUMA (non-uniform memory access):
  - Occurs in systems containing combines CPU and memory boards
  - CPU scheduler and memory-replacement works together
  - A process (assigned affinity to a CPU) can be allocated memory on the board where that CPU resides
  <img src="https://i.loli.net/2019/03/18/5c8f03b791a45.png" width="400px"/>
- Load-balancing
  - Keep the workload evently distributed across all processors, only necessary on systems where each processor has its own private queue of eligible processes to execute
  - Push migration : move(push) processes from overloaded to idle or less-busy processor
  - Pull migration: idle processors pulls a waiting task from a busy processor
  - Often implemented in parallel

### Multi-core Processor Scheduling

- Faster and consumer less power
- **memory stall**: When access memory, it spends a significant amount of time waiting for the data become available. (e.g. cache miss)
- Multi-threaded multi-core systems
  - Two (or more) hardware threads are assigned to each core (Intel Hyper-threading)
  - Takes advantage of memory stall to make progress on another thread while memory retrieve happens
    <img src="https://i.loli.net/2019/03/18/5c8f051f97e4d.png" width="400px"/>
- Two ways to multithread a processor:
  - coarse-grained: switch to another thread when a memory stall occurs. The cost is high as the instruction pipeline must be flushed
  - fine-grained(interleaved): switch between threads at the boundary of an instruction cycle. The architecture design includes logic for thread switching -- cost is low
- Scheduling for Multi-threaded multi-core systems
  - 1st level: Choose which software thread to run on each hardware thread(logical processor)
  - 2nd level: How each core decides which hardware thread to run

### Real-Time CPU Scheduling

Real-time does not mean speed, but keeping deadlines

- Soft real-time requirements, missing the deadline is unwanted , but is not immediately critical (multimedia streaming)
- Hard real-time requirements, Missing the deadline results in a fundamental failure (nuclear power plant controller)

### Read-Time Scheduling Algorithms

Evaluation: Ready, Execution, Deadline

#### Rate-Monotoinc (RM) algorithm

- short period, higher priority (fixed-priority RTS scheduling algorithm)
- Ex: T1 = (4,1) (red), T2 = (5,2) (orange), T3 = (20,5)(green) (Period, Execution)
- priority: T_1 > T_2 > T_3
<img src="https://i.loli.net/2019/03/18/5c8f086daa22a.png" width="450px"/>

#### Earliest-Deadline-First(EDF) algorithm

- Earlier deadline, higher priority (dynamic priority algorithm)
- Ex: T1 = (2, 0.9), T2 = (5, 2.3)
  <img src="https://i.loli.net/2019/03/18/5c8f09455c102.png" width="400px"/>

## Operating System Examples

- Solaris Scheduler
- Windows XP Scheduler
- Linux Scheduler