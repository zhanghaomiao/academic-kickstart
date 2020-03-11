---
title: Multithread Programming
#linktitle: Tips 1-2
toc: true
type: docs
date: "2019-05-05T00:00:00+01:00"
draft: false

menu:
  operating_system:
   parent: Concepts
   weight: 5
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 4
---

## Threads

- Threads is a light process, basic unit of CPU utilization
- All threads belonging to the same process share **code section**, **data section**, and **OS resources** (e.g. open files and signals)
- Each thread has its own (thread control block) **thread ID, program counter, register set, and a stack** <img src="https://i.loli.net/2019/03/17/5c8e26e6c2312.png" width="400px"/>
- Example
  - web browser, once thread displays contents while the other thread receives data from network
  - web server, one request(thread), better performance as code and resource sharing
  - RPC server
  <img src="https://i.loli.net/2019/03/17/5c8e27ee06cfa.png" width="400px"/>
- Benefits of Multithreading
  - **Responsiveness**: allow a program to continue running even if part of it is blocked or is performing a lengthy operation
  - **Resource sharing**: several different threads of activity all within the same address space
  - **Utilization of MP arch**: Several thread may be running in parallel on different processors
  - **Economy**: Allocating memory and resources for process creation is costly.

## Multicore Programming

- multithread programming provides a mechanism for more efficient use of multiple cores and improved concurrency (threads can run in parallel)
- Multicore systems putting pressure on system designers and application program (scheduling algorithms use cores to allow the parallel execution
- challenges in Multicore Programming
  - **Dividing activities**: divide program into concurrent tasks
  - **Data splitting**: divide data accessed and manipulated by the tasks
  - **Data dependency**: synchronize data access
  - **Balance**: evenly distribute tasks to cores
  - **Testing and debugging**

### User vs. Kernel Threads

- User thread -- thread management done by user level threads library (Pthreads, Java threads, Win32 threads)
- Kernel threads -- supported by the kernel(OS) directly (Windows 2000, Linux)
- User threads
  - thread library provides support for thread creation, scheduling and deletion
  - Generally fast to create and manage
  - If the kernel is single-threaded, a user-thread blocks $\rightarrow$ entire process blocks even if other threads are ready to run
- Kernel threads
  - The kernel performs thread creation, scheduling, etc.
  - Generally slower to create and manage
  - If a thread is blocked, the kernel can schedule another thread for execution
- Models (user threads to kernel threads)
  Many-to-one, one-to-one(kernel threads 有限制，大部分系统）, Many-to-Many

## Shared-Memory Programming

- Definition: processes communicate or work together with each other through a shared memory space which can be accessed by all processes (Faster & more efficient than message passing)
- Many issues as well, **Synchronization**, **Deadlock**, **Cache coherence**
- Programming techniques (Parallelizing compiler, Unix processes, Threads(Pthread, Java))

### Pthread

- Pthread is the implementation of POSIX standard for thread
- `pthread_create(thread, attr, routine, arg)`
  - `thread`: An unique identifier (token) for the new thread
  - `attr`: it is used to set thread attributes. NULL for the default values
  - routine: The routine that the thread will execute once it is created
  - arg: A single argument that may be passed to routine <img src="https://i.loli.net/2019/03/17/5c8e3392a48d7.png" width="400px"/>
- pthread_join(threadId, status)
  - Blocks until the specified threadId thread terminates
  - One way to accomplish synchronization between threads
- pthread_detach(threadId)
  - Once a thread is detached, it can never be joined
  - Detach a thread could free some system resources

<img src="https://i.loli.net/2019/03/21/5c933ded79175.png" width="400px"/>

Example

```C
  #include <pthread.h>
  #include <stdio.h> #define NUM_THREADS 5
  void *PrintHello(void *threadId) {
  long* data = static_cast <long*> threadId;
    printf("Hello World! It's me, thread #%ld!\n", *data);
    pthread_exit(NULL);
  }
  int main (int argc, char *argv[]) {
  pthread_t threads[NUM_THREADS];
  for(long tid=0; tid<NUM_THREADS; tid++){
      pthread_create(&threads[tid], NULL, PrintHello, (void *)&tid);
  }
  /* Last thing that main() should do */
    thread_exit(NULL);
  }
```

### Java Threads

- Thread is created by Extending Thread class, Implementing the Runnable interface
- Java threads are implemented using a thread library on the host System
- Thread mapping depends on the implementation of JVM

### Linux threads

- Linux does not support multithreading
- Various Pthreads implementation are available for user-level
- The `fork` system call -- create a new process and a copy of the associated data of the parent process
- The `clone` system call -- create a new process and a link that points to the associated data of the parent process
- A set of flags is used in the clone call for indication of the level of the sharing
  - None of the flag is set $\rightarrow$ clone = fork
  - All flags are set $\rightarrow$ parent and child share everything
<img src="https://i.loli.net/2019/03/17/5c8e3635b4594.png" width="400px"/>

## Threading Issues

- Does `fork()` duplicate only the calling thread or all threads? Some UNIX system support two versions of `fork()`
  - `execlp()` works the same, replace the entire process, if `exec()` is called immediately after forking, then duplicating all threads is unnecessary
    <img src="https://i.loli.net/2019/03/17/5c8e37759091b.png" width="400px"/>
- Thread Cancellation
  - Asynchronous cancellation (one thread terminates the target thread immediately) 等 main thread 有空
  - Deferred cancellation (default option) The target thread periodically checks whether it should be terminated, allowing it an opportunity to terminate itself in an orderly fashion (canceled safety)
- Signal Handling
  - Signals (synchronous or asynchronous) are used in UNIX systems to notify a process that an event has occurred
  - A signal handler is used to process signals
    1. Signal is generated by particular event
    2. Signal is delivered to a process
    3. Signal is handled
- Thread Pools
  - Create a number of threads in a pool where they await work
  - Advantages
    - Usually slightly faster to service a request with an existing thread than create a new thread
    - Allows the number of threads in the application(s) to be bound the size of the pool
  - \#  Of threads: \# of CPUs, expected \# of requests, amount of physical memory