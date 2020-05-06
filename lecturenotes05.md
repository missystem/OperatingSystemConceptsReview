## [Lecture 05: Threads (Chapter 4)](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecture-5-threads.pdf)

## Outline
* [Why thread? - Advantages of threads](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes05.md#advantages-of-threads)
* [Why thread? - Problem solved with threads](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes05.md#problems-solved-with-threads)
* [Why not thread?](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes05.md#why-not-threads)
* [Threads vs. Processes](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes05.md#threads-vs-processes-difference)
* [Thread Models](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes05.md#many-to-one-thread-model)
* [Pthreads](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes05.md#posix-threads)
* [Summary](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes05.md#summary)

### Program with Multiple Processes
<img width="811" height="487" src="https://github.com/missystem/cis415review/blob/master/prog_with_multiple_processes.png">

### [Program Creation System Calls](https://github.com/missystem/cis415review/blob/master/lecturenotes03.md#program-creation-system-calls)
* fork()
	- Copy address space of parent and all threads
* vfork()
	- Do not copy the parent’s address space
	- Share address space between parent and child
	- While parent blocks, child must exit or call exec
* exec()
	- Load new program and replace address space
	- Some resources may be transferred (open file descriptors)
	- Specified by arguments
* clone()
	- Like fork() but child shares some process context
	- More explicit control over what is shared
	- Process address space can be shared!
	- Calls a function pointed to by argument

### Process Model
*  Much of the OS’s job is keeping processes from interfering with each other
* Each process has its OWN resources to use
	- Program code to execute, address space, files, ...
* Processes are good for isolation (protection)
	- Prevent one process from affecting another process
* Processes are *heavyweight*
	- Pay a price for isolation
	- A full “process swap” is required for multiprocessing 
	- There is lots of process state to save and restore
	- OS must context switch between them 
		- intervene to save/restore all process state
* Is there an alternative?

### Why Threads?
* A process is “a program in execution”
	- Memory address space containing code and data
	- Other resources (e.g., open file descriptors)
	- State information (PC, register, SP) => PCB details
* Consider a process in 2 respects (categories) 
	1. Collection of resources required for execution
		- code, address space, open files, ...
	2. A “thread of execution”
		- current state of execution (CPU state) 
		- where the execution is now (PC)
* Suppose we think about these separately!
	- Resources
	- Execution state

### Terminology
* Multiprogramming
	- Running multiple programs on a computer concurrently
	- Each program is one process or a set of processes
	- Processes of different programs are independent
* Multiprocessing
	- Running multiple processes on a computer concurrently 
	- OS manages mapping of processes to processor(s)
* Multithreading
	- Define multiple execution contexts (threads of execution) in a single address space (of a process)
	- OS manages mapping of threads to an address space
	- OS manages mapping of threads to processor(s)

### What’s a Thread?
* A basic unit of CPU utilization; 
* It comprises 
	- a thread ID
	- a program counter (PC)
	- a register set
	- a stack
* Thread of execution through a program on a CPU
	- Program counter					(*per thread*)
	- Registers							(*per thread*)
* Memory
	- Address space						(*process*)
		- address space is *shared*
	- Stack 							(*per thread*)
		- each thread has its own stack pointer
	- Heap <br />						(*process*)
		\+ private dynamic memory 		(*per thread*)
* I/O
	- Share files, sockets, ...			(*process*)

### Why Multithreaded Applications?
* Multiple threads sharing a common address space
	- More correctly, they share process resources, including memory
* Why would you want to do that?
	- Some applications could be written to support concurrency
	- How do you get concurrency?
		- One way is to create multiple processes
		- Use IPC to support process-level concurrency
* Some applications want to share data structures among concurrently executing parts of the computation
	- Is this possible with processes? => shared segments
	- Is it difficult with processes? => well, it is not that easy
	- Again, use IPC for process-level data sharing
* What is the problem? What is the solution?

### Advantages of Threads
* Threads could be used if there is no need for the OS to enforce resource separation
	- This is a trust issue, in part (back off on isolation / protection)
	- However, introduces more issues with respect to concurrency
* Benefits
	* Responsiveness (improved)
		- Possible to have a thread of execution that never blocks
	* Resource sharing (is facilitated)
		- All threads in a process have equal access to resources
	* Economy (of resources)
		- Thread-level resources are “cheaper” than process resources!
		- Threads are “lighter weight”
	* Utilization of multiprocessors (Scalability)
		- Run multiple threads on multiple processes without the overhead of running multiple processes

### Single-Threaded vs. Multithreaded
* Regular UNIX process can be considered as a special case of a multithreaded process
	- A process that contains just one thread
* Multithreaded process has multiple threads
<img width="1328" height="604" src="https://github.com/missystem/cis415review/blob/master/singleThreaded_multithreaded.png">

### Working with Threads
*  In a C program
	- *main()* procedure defines the first thread 
	- C programs always start at *main()*
* Now create a second thread
	- Allocate resources to maintain a second execution context in same address space
		- think about what state will be needed for a thread
	- Want something similar to *fork()* but simplere
		- supply a procedure name when start the new thread
	- Remember this creates another thread of execution

### Threads vs. Processes (difference)
* Easier to create than a new process
* Less time to terminate a thread than a process
* Less time to switch between two threads 
	- within the same process
* Less communication overheads
	- communicating between the threads of one process is simple because the threads share everything 
	- address space is shared
	- thus memory is shared

### Which is Cheaper?
* Create new process or create new thread (in existing process)?
* Context switch between processes or threads? 
* Interprocess or Interthread communication?
* Sharing memory between processes or threads?
* Terminating a process or terminating a thread (not the last one)?
* Time to create 100,000 processes (Linux 2.6 kernel, x86-32 system) <br />
(vfork(): faster fork) <br />
clone() creates a lightweight Linux process (thread) <br />

| Process creation method | Time (sec), elapsed (real) |
| -----------------------:|:--------------------------:|
| fork()                  | 22.27 (7.99)               |
| vfork()                 | 3.52 (2.49)                |
| clone()                 | 2.97 (2.14)                |

### Implications?
* Consider a web server on a Linux platform
* Measure 0.22 ms per *fork()*
	- Maximum of (1000 / 0.22) = 4545.5 connections/sec
	- 0.45 billion connections per day per machine
		- fine for most servers
		- too slow for a few super-high-traffic front-line web services
* Facebook serves O(750 billion) page views per day
	- Guess \~1-20 HTTP connections per page
	- Would need 3,000 -- 60,000 machines just to handle fork(), without doing any work for each connection!
* What is the problem here?

### Thread Attributes

| Global to process     | Local to specific thread    |
|:--------------------- |:--------------------------- |
| - memory              | - thread ID                 |
| - PID, PPID, GID, SID | - stack                     |
| - controlling term    | - signal mask               |
| - process credentials | - thread-specific date      |
| - record locks        | - alternate signal stack    |
| - FS information      | - error return value        |
| - timers              | - scheduling policy/priority|
| - resource limits     | - Linux-specific (e.g., CPU affinity)|

### Threading Models
* *Programming* - library or system call interface
	- Kernel threading (most common)
		- thread management support in the kernel
		- invoked via system call
		- NOTE: CPU only runs kernel threads
	- User-space threading
		- thread management support in user-space library
		- threads are "thread switched" by a user-level library
		- linked into you program
* *Scheduling* - application or kernel scheduling
	- May create user-level or kernel-level threads

### Kernel Threads
* Thread management support in kernel
	- Sets of system calls for creating, invoking, and switching among threads
* Supported and managed directly by the OS
	- Thread objects in the kernel
* Nearly all OSes support a notion of threads
	- Linux -- thread and process abstractions are mixed
	- Solaris
	- Mac OS X
	- Windows XP
	- ...

### User-space Threads
* Thread management support in user-space library
	- Sets of functions for creating, invoking, and switching among threads (all in user mode)
	- Need to switch threads stacks in user space ...
* Linked into your program
	- Thread libraries
* Examples:
	- [Qthreads](http://www.cs.sandia.gov/qthreads/)
	- [GNU Pth](http://www.gnu.org/software/pth/)
	- [Cilk](https://en.wikipedia.org/wiki/Cilk)

### Implementing User-space Threading
* Threads can perform operations in user mode that are usually handled by the OS
	- Assumes cooperating threads so hardware enforcement of separation not required
* Idea:
	- Think of a “dispatcher” subroutine in the process that is called when a thread is ready to relinquish control to another thread
	- Manages stack pointer, program counter
	- Switches process’s internal state among threads

### Many-to-One Thread Model
* Many user-level threads correspond to a single kernel thread
	- Kernel is not aware of the mapping
	- Handled by a thread library <br />
	<img width="570" height="330" src="https://github.com/missystem/cis415review/blob/master/manytoone.png"> <br />
* How does it work?
	- Create and execute a new thread
	- Upon yield, switch to another user thread in the same process
		- kernel is unaware
	- Upon wait, all threads are blocked 
		- kernel is unaware there are other options
		- can not wait and run at the same time

### One-to-One Thread Model
* One user-level thread per kernel thread
	- A kernel thread is allocated for every user-level thread
	- Must get the kernel to allocate resources for each new user- level thread <br />
	<img width="570" height="330" src="https://github.com/missystem/cis415review/blob/master/onetoone.png"> <br />
* How does it work?
	- Create new thread
		- system call to kernel
	- Upon yield, switch to another kernel thread in system
		- kernel is aware
	- Upon wait, another thread in the process may run
		- only the single kernel thread is blocked
		- kernel is aware there are other options in this process

### Many-to-Many Thread Model
* A pool of user-level threads maps to a pool of kernel threads
	- Pool sizes can be different (kernel pool is no larger)
	-  A kernel thread is pool is allocated for every user-level thread
	- No need for the kernel to allocate resources for each new user-level thread <br />
	<img width="570" height="330" src="https://github.com/missystem/cis415review/blob/master/manytomany.png"> <br />
* How does it work?
	- Create new thread
		- may map to kernel thread dynamically
	- Upon yield, switch to another thread 
		- kernel is aware
	- Upon wait, another thread in the process may run
		- if a kernel thread is available to be scheduled to that process
		- kernel is aware of the mapping between user and kernel threads

### Two-Level Model
<img width="580" height="320" src="https://github.com/missystem/cis415review/blob/master/twolevel.png"> <br />

### Problems Solved with Threads
* Imagine you are building a web server
* Problem:
	- A web server accepts client requests for web pages, images, sound, and so forth. A busy web server may have several (perhaps thousands of) clients concurrently accessing it. If the web server ran as a traditional single-threaded process, it would be able to service only one client at a time, and a client might have to wait a very long time for its request to be serviced.
* Solution: 
	- You could allocate a pool of threads, one for each client
		- thread would wait for a request, get content file, return it 
	- How would the different thread models impact this?
* Imagine you are building a web browser
	- You could allocate a pool of threads 
		- some for user interface
		- some for retrieving content
		- some for rendering content
	- What happens if the user decided to stop the request? ◆
		- mouse click on the stop button

### Multithreaded Server Architecture
<img width="1010" height="370" src="https://github.com/missystem/cis415review/blob/master/MultithreadedServerArchitecture.png"> <br />

### Linux Threads
* Linux uses one-to-one thread model
	- Threads are called tasks
* Linux views threads as “contexts of execution”
	- Threads are defined separately from processes
	- There is flexibility in what is private and shared
* Linux system call
	- *clone(int (\*fn)(), void \*\*stack, int flags, int argc, ...)*
	- Create a new thread (Linux task)
* May be created in the same address space or not
	- Flags (on means “share”): clone VM, clone filesystem, clone files, clone signal handlers
	- If all these flags off, what system call is clone equal to?

### POSIX Threads
* POSIX Threads is a thread API specification
	- Does not define directly the implementation
	- Could be implemented differently
* A POSIX standard (IEEE 1003.1c) API for thread creation and synchronization
	- Specification, not implementation
	- Provided either as user-level or kernel-level (\*)
	- API specifies behavior of the thread library
	- Implementation is up to development of the library
* Common in UNIX operating systems
	- Solaris, Linux, Mac OS X
* POSIX Threads is also known as *Pthreads*

### POSIX Threads Functions
* start the thread
	```pthread_create()```
* return thread ID
	```pthread_self()```
* for comparisons of thread ID's
	```pthread_equal()```
* return from the start function
	```pthread_exit()```
* wait for another thread to terminate & retrieve value from pthread_exit()
	```thread_join()```
* terminate a thread, by TID
	```pthread_cancel()```
* thread is immune to join or cancel & runs independently until it terminates
	```pthread_detach()```
* thread attribute modifiers
	```pthread_attr_init()```

### POSIX Threads FAQ
* How to pass multiple arguments to start a thread? 
	- Build a struct and pass a pointer to it
* Is the pthreads ID unique to the system?
	- No, just process – Linux task ids are system-wide
* After pthread_create(), which thread is running? 
	- It acts like fork in that both threads are running
* How many threads terminate when ...
	- exit() is called? – all in the process
	- pthread_exit() is called? – only the calling thread
* How are variables shared by threads? 
	- Globals, local static, dynamic data (heap)

### Figure 4.11 Multithreaded C program using the Pthreads API.
```
#include <pthread.h>
#include <stdio.h> #include <stdlib.h>

int sum; 	/* this data is shared by the thread(s) */
void *runner(void *param); /* threads call this function */

int main(int argc, char *argv[])
{
	pthread_t tid; 		/* the thread identifier */
	pthread_attr_t attr; 	/* set of thread attributes */

	/* set the default attributes of the thread */ 
	pthread_attr_init(&attr);

	/* create the thread */
	pthread_create(&tid, &attr, runner, argv[1]);

	/* wait for the thread to exit */ 
	pthread_join(tid,NULL);

	printf("sum = %d∖n",sum); 
}


/* The thread will execute in this function */ 
void *runner(void *param)
{
	int i, upper = atoi(param);
	sum = 0;

    for (i = 1; i <= upper; i++)
    	sum += i;

    pthread_exit(0);
}
```

### Figure 4.12 Pthread code for joining ten threads.
```
#define NUM_THREADS 10

/* an array of threads to be joined upon */ 
pthread_t workers[NUM THREADS];

for (int i = 0; i < NUM THREADS; i++) 
	pthread_join(workers[i], NULL);
```

### Concurrency with Threads
* Consider the client-server example again
* Now want to run with just a single process
	- Need to use threads to get concurrency
* Process "main" (parent) thread waits for clients
	- Parent thread forks (or dispatches) a new thread to handle each new client connection
	- Responsibilities of the child thread:
		- handles the new connection
		- exits when the connection terminates

### Client-Server with Pthreads
<img width="330" height="390" src="https://github.com/missystem/cis415review/blob/master/client_server_pthreads_01.png">
<img width="580" height="390" src="https://github.com/missystem/cis415review/blob/master/client_server_pthreads_02.png">
<img width="580" height="390" src="https://github.com/missystem/cis415review/blob/master/client_server_pthreads_03.png">
<img width="580" height="390" src="https://github.com/missystem/cis415review/blob/master/client_server_pthreads_04.png">
<img width="580" height="390" src="https://github.com/missystem/cis415review/blob/master/client_server_pthreads_05.png">
<img width="580" height="390" src="https://github.com/missystem/cis415review/blob/master/client_server_pthreads_06.png">

### Implications?
* Consider a web server on a Linux platform
* 0.0297 ms per thread create (time for clone()) 
	- 10x faster than process forking
	- Maximum of (1000 / 0.0297) = \~33,670 connections/sec
	- 3 billion connections per day per machine
		- much, much better
* Facebook would need only 500 machines 
* So, why do we need processes at all?<br />
Just write everything using threads<br />
	- Writing safe multithreaded code can be complicated 
	- Why? What are the issues?

### Why not threads?
* Threads can interfere with one another
	- Impact of more threads on caches
	- Impact of more threads on TLB
	- Bug in one thread can lead to problems in others
* Executing multiple threads may slow them down 
	- Impact of single thread vs. switching among threads
* Harder to program a multithreaded program 
	- Multitasking hides context switching
	- Multithreading introduces concurrency issues


### Concurrent Threads
* Benefits
	- All threads are running the same code
		- still the case that much of the code is identical!
	- Shared-memory communication is possible
	- Good CPU and network utilization 
		- lower overhead than processes
* Disadvantages
	- Synchronization is complicated
	- Shared fate within a process
		- one rogue thread can hurt you badly

### Inter-Thread Communication
* Can you use shared memory?
	- YES
	- just need to allocate memory in the address space
		- No need for fancy IPC shared memory
* Can you use message passing
	- YES
	- would have to build infrastructure
* Can threads utilize IPC mechanisms
	- would need to make sure only 1 thread uses at a time

### Fork/Exec Issues
* Semantics are ambiguous for multithreaded processes
* fork()
	- How does it interact with threads?
* exec()
	- What happens to the other threads?
* fork, then exec
	- Should all threads be copied?

### Thread Pools
* General Idea:
	1. Create a number of threads at start-up 
	2. Place them into a pool
		- where they sit and wait for work
	3. When a server receives a request
		- it submits the request to the thread pool and resumes waiting for additional requests
	4. If there is an available thread in the pool
		- it is awakened, and the request is serviced immediately
	5. If the pool contains no available thread
		- the task is queued until one becomes free
	6. Once a thread completes its service
		- it returns to the pool and awaits more work. 
	7. Thread pools work well when the tasks submitted to the pool can be executed asynchronously
* Benefits:
	1. Servicing a request with an existing thread is often faster than waiting to create a thread
	2. A thread pool limits the number of threads that exist at any one point. 
		- important on systems that cannot support a large number of concurrent threads
	3. Separating the task to be performed from the mechanics of creating the task allows us to use different strategies for running the task
		- For example, the task could be scheduled to execute after a time delay or to execute periodically.
* Pool of threads
	- Create (all) at initialization time
	- Assign task to a waiting thread
		- it is already made so it should be fast
	- Use all available threads
* If the task is done
	- Suppose another request is in the queue
	- should we use running thread or another thread?
* Concern for the setup time cost
* Faster than setting up a process, but what is necessary?
	- How do we improve performance?

### Signal Handling
* What’s a signal?
	- A form of IPC
	- Send a particular signal to another process
* Receiver’s signal handler processes signal on receipt
* Example
	- Tell the Internet daemon (*inetd*) to reread its config file
	- Send signal to inetd: *kill -SIGHUP \<\pid>*
	- inetd’s signal handler for the SIGHUP signal re-reads the config file
* Note: some signals cannot be handled by the receiving process, so they cause default action (kill the process)
* Synchronous signals
	- Generated by the kernel for the process
	- Due to an exception -- divide by 0
		- events caused by the thread receiving the signal
* Asynchronous signals
	- Generated by another process
* Asynchronous signals are more difficult for multithreading

### Signal Handling and Threads
* Send a signal to a process <br />
Which thread should it be delivered to? -- It depends...
* Choices
	- Thread to which the signal applies 
	- Every thread in the process
	- Certain threads in the process
	- A specific signal receiving thread
* UNIX signal model created decades before Pthreads
	- Conflicts arise as a defult
* Synchronous vs. asynchronous cases
* Synchronous
	- Signal is delivered to the same process that caused the signal
	- Which thread(s) would you deliver the signal to?
* Asynchronous
	- Signal generated by another process
	- Which thread(s) in this case?


### Thread Cancellation
* 2 choices to stop a thread from executing
	- Synchronous cancellation
		- wait for the thread to reach a point where cancellation is permitted
		- no such operation in Pthreads, but can create your own
	- Asynchronous cancellation
		- terminate it now
		- *pthread_cancel(thread_id)*

### Terms
* *Reentrant* code
	- Code that can be run by multiple threads concurrently
* *Thread-safe* libraries
	- Library code that permits multiple threads to invoke the safe function
	- Mainly concerned with variables that should be private to individual threads
	- Requires moving some global variables to local variables
* Requirements
	- Rely only on input data
		- Or thread-specific data
	- Must be careful about locking (later)

### Thread Assignment (Scheduling)
* How many kernel threads to create for a process? 
	- In an M:N model there can be significantly more user-level threads (M) than kernel-level threads (N)
	- Kernel threads are the “real” threads that can be allocated to a CPU (core) and run
	- Want to keep enough kernel threads to satisfy the desired level of concurrency in a program and activity in the system
* Suppose that all kernel threads except 1 are blocked
	- What happens if the last kernel thread blocks?
	- Does it matter if there are more user threads “ready” to run?
* In multiprocessing, thread affinity is the notion of assigning a thread to run on a particular CPU
	- Help to improve the thread’s performance by keeping its execution resources (CPU, cache, memory) local to CPU

### Scheduler Activation
* It would be nice if the kernel told the application that a kernel thread was blocking
	- Scheduler activation
		- at thread block, the kernel tells the application via an upcall
		- an upcall is a general term for an invocation of application function from the kernel
	- User-level thread scheduler can then get a new user-level thread created
* Way of conveying information between the kernel and the user-level thread scheduler regarding the disposition of:
	- \# user-level threads (increase or decrease)
	- User-level thread state
		- running to waiting, waiting to ready

---

## Summary
* Threads
	- A mechanism to improve performance and CPU utilization
* Kernel-space and user-space threads
	- Kernel threads are real, schedulable threads
	- User-space may define its own threads (but not real)
* Threading models and implications
	- Programming systems
	- Multi-threaded design issues
* Useful, but not a panacea
	- Slow down system in some cases 
	- Can be difficult to program
* Multiprogramming and multithreading are important concepts in modern operating systems

## Summary from book
* **thread** 
	- a basic unit of CPU utilization
	- threads belonging to the same process share many of the process resources
		- including code and data
* There are 4 primary benefits to multithreaded applications: 
	1. responsiveness
	2. resource sharing
	3. economy
	4. scalability
* **Concurrency** 
	- multiple threads are making progress, whereas 
* **Parallelism** 
	- multiple threads are making progress simultaneously. 
* On a system with a single CPU, only concurrency is possible; parallelism requires a multicore system that provides multiple CPUs.
* challenges in designing multithreaded applications:
	- dividing and balancing the work
	- dividing the data between the different threads
	- identifying any data dependencies
	- test and debug (especially challenging)
* **Data parallelism** 
	- distributes subsets of the same data across different computing cores and performs the same operation on each core
* **Task parallelism** 
	- distributes tasks across multiple cores
	- each task is running a unique operation
* User applications create **user-level threads**
	- must ultimately be mapped to kernel threads to execute on a CPU
	- approaches:
		- many-to-one: maps many user-level threads to one kernel thread
		- one-to-one 
		- many-to-many
• **thread library** 
	- provides an API for creating and managing threads
	- 3 common thread libraries: Windows, Pthreads, Java threading
	- Windows
		- for the Windows system only
	- Pthreads 
		- available for POSIX-compatible systems such as UNIX, Linux, and macOS
	- Java threads 
		- run on any system that supports a Java virtual machine (JVM)
• **Implicit threading**
	- involves identifying tasks — not threads
	- allowing languages or API frameworks to create and manage threads
	- Approaches
		- thread pools
		- fork-join frameworks
		- Grand Central Dispatch
	- an increasingly common technique for programmers to use in developing concurrent and parallel applications.
• **Threads Termination**
	- Asynchronous cancellation
		- stops a thread immediately, even if it is in the middle of performing an update
	- Deferred cancellation
		- informs a thread that it should terminate but allows the thread to terminate in an orderly fashion
	- In most circumstances, deferred cancellation is preferred to asynchronous termination.
• Linux system
	- does not distinguish between processes and threads;
	- it refers to each as a task
	- *clone()* system call used to create tasks 
		- behave either more like processes or more like threads

