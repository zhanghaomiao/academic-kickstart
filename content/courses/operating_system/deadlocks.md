---
title: DeadLocks
toc: true
type: docs
date: "2019-05-05T00:00:00+01:00"
draft: false

menu:
  operating_system:
   parent: Concepts
   weight: 8
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 7
---

## Introduction

- A set of blocked process each holding some resources and waiting to acquire a resource held by another process in the set
- Ex1: 2 processes and 2 tape dirves, each process holds a tape drive, each process requests another tape drive
- Ex2: 2 processes, and semaphores A & B, P1(hold B, wait A): wait(A), signal(B), P2(hold A, wait B): wait(B), signal(A)

### Necessary Conditions

- Mutual exclustion (only 1 process at a time can use a resource)
- Hold & Wait: a process holding some resources and is waiting for another resource
- No preemption: a resource can be only released by a process voluntarily
- Circular wait: there exist a set {$P_0, P_1, ..., P_n$} of waiting process such that $P_0 \rightarrow P_1 \rightarrow P_2, ... , P_0$
- All four conditions must hold for possible deadlock
<img src="https://i.loli.net/2019/03/20/5c9194d343ffc.png" width="400px"/>

## System Model

- Resources type $R_1, R_2, ... , R_m$ E.g. CPU, memory pages, I/O devices
- Each resource type $R_i$ has $W_i$ instances, E.g. a computer has 2 CPUs
- Each process utilizes a resouce as follows:
  Request $\rightarrow$ use $\rightarrow$ release

### Resource-Alocation Graph

- 3 processes, P1-P3
- 4 resources, R1-R4 (the black dot represent the number of instance)
- Request edges: P1$\rightarrow$ R1: P1 requests R1
- Assignment edges: R2$\rightarrow$ P1: one instance of R2 is allocated to P1
- P1 is hold on an instance of R2 and waiting for an instance of R1
  <img src="https://i.loli.net/2019/03/20/5c9196704420b.png" width="200px"/>
- if the graph consists a cycle, a deadlock may  exist
- deadlock
  <img src="https://i.loli.net/2019/03/20/5c919780cc361.png" width="400px"/>
- no deadlock
  <img src="https://i.loli.net/2019/03/20/5c9197d43b55e.png" width="400px"/>
- If graph contains no cycle $\rightarrow$ no deadlock
- If graph contains a cype:
  - if one instance per resource type $\rightarrow$ deadlock
  - if multiple instances per resource type $\rightarrow$ possibility of deadlock

## Handing Deadlocks

- Ensure the system will never a deadlock state
  - deadlock preventation: ensure that at least one of the 4 necessary conditions cannot hold
  - deadlock avoidance: dynamically examines the resource-allocation state before allocation
- Allow to enter a deadlock state and then recover
  - deadlock detection
  - dedlock recovery
- Ignore the problem and pretend that deadlocks never occur in the system (used by most operating systems, including UNIX)

### Deadlock Prevention

- Mutual Exclustion(ME): do not require ME on sharable resources
  - e.g. there is no needto ensure ME on read-only files
  - Some resources are not shareable, however (e.g. printer)
- Hold & Wait:
  - When a process requests a resource, it does not hold any resource
  - Pre-allocate all resources before executing (resource utilization is low, starvation is possible)
- No preemption
  - When a process is waiting on a resource, all its holding resources are preempted (e.g. P1 request R1, which is allocated P2, which in turn is waiting on R2, R1 can be preempted and reallocated to P1)
  - Applied to resources whose states can be easily saved and restored later (e.g. CPU registers & memory)
  - It cannot easily be applied to other resources(e.g. printers & tape drives)
- Circular wait
  - impose a total ordering of all resources types
  - A process requests resources in an increasing order
    - Let $R = {R_0, R_1, ... , R_N}$ be the set of resource types
    - When request $R_k$, should release all $R_i$, $i \geq k$

### Avoidance Algortihm

- safe state: a system is in a safe state if there exists a sequence of allocations to satisfy requests by all processes (this sequence of allocations is called safe sequence)

<img src="https://i.loli.net/2019/03/20/5c91a399d222f.png" width="400px"/>

- Single instance of resource type (resource-allocation graph (RAG) algorithm based on circle detection)
- Multiple instance of resource type
  banker's algorithm based on safe sequence detection

#### Resource-Allocation Graph (RAG) Algorithm

- Request edge
- Assignment edge
- Claim  edge: $P_i \rightarrow R_j$, process $P_i$ may request $R_j$ in the future
- Clain edge converts to request edge (when a resource is requested by process)
- Assignment edge converts back to claim edge (when a resource is released by a process)
- Resources must be claimed a priori in the system
- Grant a request only if no cycle created
- Check for safety using a **cycle-detection algorithm, $O(n^2)$**
  <img src="https://i.loli.net/2019/03/22/5c94686f2a7e4.png" width="250px"/>
- R2 cannot be allocated to P2

#### Banker's algorithm

- Safe State with Safe Sequnce
- use for multiple instance of each resource type
- use a general safety algorithm to pre-determine if any safe sequence exists after allocation
- only proceed the allocation if safe sequence exists
- Procedures:
  1. Assume processes need maximum resources
  2. Find a process that can be satisfied by free resources
  3. Free the resource usage of the process
  4. repeat to step 2 until all processes are satisfied
- Example
  <img src="https://i.loli.net/2019/03/20/5c91a8e8527a0.png" width="200px"/>
  Safe sequence: $P_1, P_3, P_4, P_2, P_0$
  - if Request (P1) = (1,0,2): P1 allocation $\rightarrow$ 3, 0, 2 (Safe sequence: $P_1, P_3, P_4, P_0, P_2$)
  - if request (P4) = (3,3,0): P4 allocation  $\rightarrow$ 3,3,2 (no safe sequence can be found)

### Deadlock Detection

- Single instance of each resource type
  - convert request/assignment edges into wait-for graph
  - deadlock exists if there is a cycle in the wait-for graph
  <img src="https://i.loli.net/2019/03/20/5c91aa599dca0.png" width="400px"/>
- Multiple-Instance for each Resource type
Total instances: A(7), B(2), C(6) (Request 表示已经发生)
<img src="https://i.loli.net/2019/03/20/5c91aae734f68.png" width="400px"/>
  - The system is in a safe state $\rightarrow$ <P_0, P_2, P_3, P_1, P_4> (no deadlock)
  - If P2 request = <0, 0, 1> $\rightarrow$ no safe sequence can be found $\rightarrow$ the system is deadlocked

### Deadlock Recovery

- process termination
  - abort all deadlocked processes
  - abort 1 process at a time until the deadlock cyle is eliminated (which process should we abort first)
- Resource preemption
  - select a victim: which one to preempt
  - rollback: partial rollback or total rollback?
  - starvation: can the same process be preempted always?
