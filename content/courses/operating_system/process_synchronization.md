---
title: Process Synchronization
toc: true
type: docs
date: "2019-05-05T00:00:00+01:00"
draft: false

menu:
  operating_system:
   parent: Concepts
   weight: 7
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 6
---


## Background

- Concurrent access to shared data may result in data inconsistency
- Maintaining data consistency requires mechanism to ensure the orderly execution of cooperating processes
- Consumer & Producer Problem
- Race condition: the situation where several processes access and manipulate shared data concurrenlty. The final value of the shared data depends upon which process finishes last, commonly described as **critical sectio**n problem
- To prevent race condition, concurrent processes must be synchronized, on a single-process machine, we could disable interrupt or use non-preemptive CPU scheduling

### Crtical Section

- Purpose: a protocal for processes to cooperate
- Probelm description:
  - N process are competing use some shared data
  - Each process has a code segment, called critical selection, in which the shared data is accessed
  - Ensure that when one process is executing in its critical section, no other process is allowed to execute in its critical selection  $\rightarrow$ **mutually exclusive**
- Critical Section Requirements
  1. **Mutual Exclusion:** if process P in executing in its CS, no other processes can be executing in their CS
  2. **Progress**: if no process is executing in its CS and there exist some processes that wish to enter their CS, these processes cannot be postponed indefinitely
  3. **Bounded Waiting**: A bound must exist on the number of times that other processes are allowed to enter their CS after a process has made a request to enter its CS

## Solution

### Algorithm for two Processes

- only 2 processes, $P_0$ and $P_1$
- Shared variables
  - int turn; // initially turn = 0
  - turn = i $\rightarrow$ $P_i$ can enter its critical section
<img src="https://i.loli.net/2019/03/18/5c8f267ace3ce.png" width="400px"/>
  - Mutual exclustion (yes); Progress (No); Bounded-Wait(Yes)

### Peterson's solution for Two processes

- shared variables
  - int turn, initially turn =0
  - turn = i $\rightarrow$ $P_i$ can enter its critical section
  - `boolean flag[2]` // initially flag[0] = flag[1] = false
  - `flag[I] = true` $\rightarrow$ $P_i$ ready to enter its critical section
  - Mutual Selection, progress, bounded waiting proof
- Example

  ```C
  do {
      flag[i] = TRUE; // Pi 是否想要进去
      turn = j; // 先把key交给对方
      while (flag[j] && turn == j);
      // critical section
      flag[i] = FALSE;
      remainder section
  } while(1);
  ```

### Bakery Algorithm (n processes)

- Before enter its CS, each process receives a number
- Holder of the smallest number enters CS
- The numbering scheme always generates number in non-decreasing order; i.e. 1,2,3,3,4,5,5,5
- if processes $P_i$ and $P_j$ receive the same numbe, if $i<j$ then $P_i$ is served first
  Bounded-waiting because processes enter CS on a First-come, First Served basis

    ```C
    // process i:
    do {
      choosing[i] = TRUE;
      num[i] = max(num[0], num[1], ..., num[n-1]) + 1;
      choosing[i] = FALSE;
      for(j =0; j<n; j++){
        while(choosing[j]); // cannot compare when num is being modified
        while((num[j]!=0) && ((num[j], j)) < (num[i],i))); // FCFS
      }
      // critical section
      num[i] = 0;
      // reminder section
    }while(1);
    ```

### Pthread Lock/Mutex Routines

- To use mutex, it must be declared as type `pthread_mutex_t` and initialized with `pthread()_mutex_init()`
- A mutex is destoryed with `pthread_mutex_destory()`
- A critical selection can then be protected using `pthread_mutex_lock()` and `pthread_mutex_unlock()`

### Condition Variables (CV)

- CV represent some condition that a thread can:
  - Wait on, until the condition occurs; or
  - Notify other waiting threads that the condition has occured
- Three operations on condition variables:
  - `wait()` — Block until another thread calls `signal()` or `broadcast()` on the CV
  - `signal()` — Wake up one thread waiting on the CV
  - `broadcast()` — Wake up all threads waiting on the CV
- All condition variable operation must be performed while a mutex is locked
- In Pthread, CV type is pthread_cond_t
  - Use `pthread_cond_init()` to initalize
  - `pthread_cond_wait(&theCV, &somelock)`
  - `pthread_cond_signal(&theCV)`
  - `pthread_cond_broadcast(&theCV)`
- Example:
  - A thread is designed to take action when x = 0
  - Another thread is responsible for decrementing the counter
  - All condition variable operation must be performed while a mutex is locked
  <img src="https://i.loli.net/2019/03/22/5c94451f9362f.png" width="400px"/>
- Procedure
  - left thread
    1. Lock mutex
    2. Wait()
        1. Put the thread into sleep and releases the lock
        2. Waked up, but the thread is locked
        3. Re-acquire lock and resume execution
    3. Release the lock
  - right thread
      1. Lock mutex
      2. Signal()
      3. Release the lock()

### ThreadPool Implementation

```C
  struct threadpool_t {
    pthread_mutex_t lock;
    pthread_cond_t notify;
    pthread_t *treahd;
    threadpool_task_t *queue;
    int thread_count;
    int queue_size;
    int head;
    int tail;
    int count;
    int shutdown;
    int started;
  };

  typedef struct {
    void (*function) (void*);
    void *argument;
  } threadpool_task_t;

// allocate thread and task queue
pool->threads = (pthread_t *) malloc(sizeof(pthread_t) * thread_count);
pool->queue = (threadpool_task_t *) malloc(sizeof(threadpool_task_t) * queue_size);

// threadpool implementation
static void *threadpool_thread(void *threadpool)
{
  threadpool_t *pool = (threadpool_t *) threadpool;
  threadpool_task_t task;

  for(;;){
    // lock must be taken to wait on conditional varaibl
    pthread_mutex_lock(&(pool->lock));
    while((pool->count=0) && (!pool->shutdown)) {
      pthread_cond_wait(&(pool->notify), &(pool->lock));
      task.function = pool->queue[pool->head].function;
      task.argument = poll->queue[pool->head].argument;
      pool->head +=1;
      pool->head = (pool->head == poll->queue_size) ? 0 : pool->head;
      pool->count -=1;

      pthread_mutex_unlock(&(pool->lock));
      (*(task.function))(task.argument);
    }
  }
}

```

### Hardware Support

- The CS problem occurs because the modification of a shared variable may be interrupted
- If disable interrupts when in CS,  not feasible in multiprocessor machine, clock interrupts cannot fire in any machine
- HW support solution: atomic instruction
  - atomic: as one uninterruptible unit
  - Examples: TestAndSet(var), Swap(a, b)

#### Atomic TestAndSet()

  ```C
  booelean TestAndSet(bool &lock) {
    bool value = lock;
    lock = TRUE; // return the value of "lock" and set "lock" to ture
    return value;
  }
  ```

  <img src="https://i.loli.net/2019/03/18/5c8f386e68d10.png" width="400px"/>
 Mutual Exclusion (Yes), Progress(Yes), Bounded-Wait(No)

#### Atomic Swap()

- Idea: enter CS if `lock=false`
- Shared data: boolean lock; //initially `lock=FALSE`
 <img src="https://i.loli.net/2019/03/18/5c8f398d3e9f6.png" width="400px"/>

## Semaphores

- A tool to generalize the synchronization problem (easier to solve, but no guarantee for correctness)
- A record of how many units of a particular resource are available
  - if #record = 1 $\rightarrow$ binary semaphore, mutex lock
  - if #record > 1 $\rightarrow$ counting semaphore
- Accessed only through 2 **atomic** ops: `wait` & `signal`
- **Spinlock** implementation (浪费CPU资源)

    ```C
    wait(S) {
      while (S<=0);
      S--;
    }
    signal(S){
      S++;
    }
    ```

- POSIX Semaphore (OS support)
  Semaphore is part of POSIX standard but it is not belong to Pthread, it can be used with or without thread
- POSIX Semaphore routines
  - `sem_init(sem_t *sem, int pshared, unsigned int value)`
  - `sem_wait(sem_t *sem)`
  - `sem_post(sem_t *sem)`
  - `sem_getvalue(sem_t *sem, int *valptr)`
  - `sem_destory(sem_t *sem)`
- Example

```C
  #include<semaphore.h>
  sem_t sem;
  sem_init(&sem);
  sem_wait(&sem);
// critical section
  sem_post(&sem);
  sem_destory(&sem);
```

### n-Process Critical Section Problem

- shared data: semaphore mutex; // initially mutex = 1
- Process Pi

```C
  do {
    wait(mutex); // pthread_mutex_lock(&mutex)
      critical section
    signal(mutex); // pthread_mutex_unlock(&mutex)
      remainder section
  } while(1);
  Progress? Yes
  Bounded waiting? Depends on the implementation of `wait()`
```

### Non-busy waiting Implementation

- Semaphore is data struct with a queue, may be any queuing strategy

  ```C
    typedef struct {
      int value; // init to 0
      struct process *L // queue
    } semaphore
  ```

- wait() and signal()
  - use system calls: `block()` and `wakeup()`
  - must be executed atomically
  - Ensure atomic wait & signal ops?
    - Single-process: disable interrupts
    - Multi-processor: 1. HW support 2. SW solution(Peterson's solution, Bakery algorithm)

      ```C
      void wait(semaphore S)
      {
        S.value--;
        if(S.value < 0){
          add this process to S.L
          sleep();
        }
      }

      void signal(semaphore S) {
        S.value++;
        if(S.value<=0){
          remove a process P from S.L;
          wakeup(P);
        }
      }
      ```

### Semaphore with Critical Section

<img src="https://i.loli.net/2019/03/22/5c9460c709cdd.png" width="400px"/>

## Classical Synchronization Problems

- Purpose: Used for testing newly proposed synchronization scheme
- Bounded-Buffer (Producer-Consumer)
- Reader-Writer Problem
- Dining-Philosopher Problem

### Bounded-Buffer Problem

A poof of n buffers, each capable of holding one item

- Producer:
  - Grad an empty buffer
  - place an item into the buffer
  - waits if no empty buffer is available
- Consumer
  - grab a buffer and retracts the item
  - place the buffer back to the free poll
  - waits if all buffers are empty

### Readers-Writers Problem

- A set of shared data objects
- A group of processes (reader processes, writer processes, a writer process has exclusive access to a shared object)
- Different variations involving priority
  - **first RW problem**: no reader will be kept waiting unless a writer is updating a shared object (writer会starvation)
  - **second RW problem**: once a writer is ready, it performs the updates as soon as the shared object is released (writer has higher priority than reader; once a writer is ready, no new reader may start reading)
- First Reader-Writer Algorithm  

```C
  // mutual exclustion for write
  semaphore wrt = 1;
  semaphore mutex = 1;
  int readcount = 0;

  Writer() {
    while(TRUE) {
      wait(wrt);
      // Writer Code
      signal(wrt);
    }
  }

  Reader() {
    while(TRUE) {
      wait(mutex);
        readcount++;
        if(readcount==1) // 之后的 Reader 不需要拿lock
          wait(wrt);
      signal(mutex);
      // Reader Code
      wait(mutex);
        readcount--;
        if(readcount == 0)
          signal(wrt);
      signal(mutex);
    }
  }
```

### Dining-Philosopher Problem

- 5 person sitting on 5 chairs with 5 chopsticks
- A person is either thinking or eating
  - thinking: no interaction with the rest 4 persons
  - eating: need 2 chopsticks at hand
  - a person picks up 1 chopsticks at a time
  - done eating: put down both chopsticks
- deadlock problem
  - one chopstick as one semaphore
- starvation problem

## Monitors (A high-level language construct)

### Motivation

Although semaphores provide a convenient and effective synchronization mechanism, its correctness is depending on the programmer

- All processes access a shared data object must execute `wait()` and `signal()` in the right order and right place
- This may not be true because honest programming error or uncooperative programmer
- The representation of a monitor type consists of declarations of variables whose values define the state of an instance of the type and the functions(procedures) that implement operations on the type
- The monitor type is similar to a class in O.O. language
  - A procedure within a monitor can access only local variables and the formal parameters
  - The local variables of a monitor can be used only by the local procedures
- The monitor ensures that only one process at a time can be **active** within the monitor
  <div class="note info"><p> Similar idea is incorporated to many programming language (concurrent pascal, C#, and Java) </p></div>

### Monitor introduction

High-level synchronization construct that allows the safe sharing of an abstract data type among concurrent processes
<img src="https://i.loli.net/2019/03/19/5c90db44d3f69.png" width="400px"/>

- Monitor Condition Variables
  - To allow a process to wait within the monitor, a condition variable must be declared as `condtion x, y;`
  - Condition variable can only be used with the operations `wait()` and `signal()`
  - `wait()` means that the process invoking this operation is suspended until another process invokes
  - `signal()` resumes exactly one suspended process. If no process is suspended, the signal operation **has no effect** (in contrast, signal always change the state of semaphore)
  <img src="https://i.loli.net/2019/03/19/5c90dd71b3c01.png" width="400px"/>

### Dining-Philosophers Example

```C
monitor dp {
  enum {thinking, hungry, eating} state[5]; //current state
  condition self[5]; // delay eating if can't obtain chopsticks
  void pickup(int i) // pickup chopsticks
  void putdown (int i) // putdown chopsticks
  void test (int i) // try to eat
  void init(){
    for (int i =0; i<5; i++)
      state[i] = thinking;
  }
}

void pickup(int i) {
  state[i] = hungry;
  test(i);
  if (state[i] != eating)
    self[i].wait(); // wait to eat
}

void putdown(int i) {
  state[i] = thinking;
  // check if neighbors are waiting to eat
  test((i+4) % 5);
  test((i+1) % 5);
}

void test(int i) {
  if ((state[(i+4) % 5] != eating) && (state[(i+1) % 5] != eating) && (state[i] == hungry)) {
    state[i] = eating;
    self[i].signal(); // if Pi is suspended, resume it, if Pi is not suspended, no effect
  }
}
```

### Synchronized Tools in JAVA

- Synchronized Methods(Monitor)
  - Synchronized method uses the method receiver as a lock
  - Two invocations of synchronized methods cannot interleave on the same object
  - When one thread is executing a synchronized method for a object, all other threads that invoke synchronized methods the same object block until the first thread exist the object

    ```java
        public class SynchronizedCounter {
            private int c= 0;
            public synchronized void increment() {c++;}
            public synchronized void decrement() {c--;}
            public synchronized int value() {return c;}
        }
    ```

- Synchronized Statement (Mutex Lock)
  - Synchronized blocks uses the expression as a lock
  - A synchronized statement can be only be executed once the thread has obtained a lock for the object or the class that has been referred to in the statement
  - useful for improving concurrency with fine-grained

    ```java
    public void run(){
        synchronized(p1)
        {
            int i = 10; // statement without locking requirement
            p1.display(s1);
        }
    }
    ```

## Atomic Transactions

- Transactions: a collection of instructions (or instructions) that performs a single logic function
- Atomic Transactions: Operations happen as a single logical unit of work, in its entirely, or not at all
- Atomic translation is particular a concern for database system (Strong interest to use DB techniques in OS)

### File I/O example

- Transaction is a series of read and write operations
- Terminated by commit (transaction successful) or abort (transaction failed) operation
- Aborted transaction must be rolled back to undo any changes it performed (it is part of )

### Log-Based Recovery

- Record to stable storage information about all modifications by a transaction
- Write-ahead logging: Each log record describes single transaction write operation
  - Transaction time
  - Data item name
  - Old & new values
  - Special Events: <$T_i$, start>, <$T_i$, commmits>
- Log is used to reconstruct the state of the data items modified by the transactions
- Checkpoints
  - when faiulre occurs, must consult the log to determine which transactions must be re-done, searching process is time consuming and redone may not be necessary for all transactions
  - use checkpoints to reduce the above overhead, output all log records, modified data, log record <checkpoint> to stable storage