## [Lecture 06: CPU Scheduling (Chapter 5)](https://github.com/missystem/cis415review/blob/master/lecture-6-scheduling.pdf)

## Outline
* [Basic Scheduling Concepts](https://github.com/missystem/cis415review/blob/master/lecturenotes06.md#basic-concepts--cpu-io-bursts)
* [Scheduling Criteria](https://github.com/missystem/cis415review/blob/master/lecturenotes06.md#scheduling-criteria) 
* [Scheduling Algorithms](https://github.com/missystem/cis415review/blob/master/lecturenotes06.md#scheduling-algorithms)
* [Thread Scheduling](https://github.com/missystem/cis415review/blob/master/lecturenotes06.md#thread-scheduling)
* [Multiple processor Scheduling](https://github.com/missystem/cis415review/blob/master/lecturenotes06.md#multiple-processor-scheduling)
* [Real-Time CPU Scheduling](https://github.com/missystem/cis415review/blob/master/lecturenotes06.md#real-time-scheduling)
* [Algorithm Evaluation](https://github.com/missystem/cis415review/blob/master/lecturenotes06.md#algorithm-evaluation)
* [Summary](https://github.com/missystem/cis415review/blob/master/lecturenotes06.md#summary)
---

### Resource Allocation
* In a multiprogramming / multiprocessing system, OS shares resources among running processes
	- There are different types of OS resources?
* Which process gets access to which resources and why?
	- To maximize performance
	- To increase utilization, throughput, responsiveness, ... 
	- Enforce priorities

### Resource Types
* Memory
	- Allocate portion of finite resource
	- Physical resources are limited
	- Virtual memory tries to make this appear infinite
* I/O
	- Allocate portion of finite resource and time spent with the resource
	- Store information on disk
	- A time slot to store that information
* CPU
	- Allocate time slot with resource
	- A time slot to run instructions

### Types of CPU Scheduling
* CPU resource allocation => **Scheduling**
* *Long-term scheduling* (admission)
	- determining whether to add to the set of processes to be executed
* *Medium-term scheduling*
	- determining whether to add to the number of processes partially or fully in memory (degree of multiprogramming)
* *Short-term scheduling*
	- determining which process will be executed by the processor
* *I/O scheduling*
	- determining which process’s pending I/O request will be handled by an available I/O device

### CPU Scheduling Views
* Single process view
	- GUI (graphical user interface) request
		- click on the mouse (*responsiveness*) 
	- Scientific computation
		- long-running, but want to complete ASAP (*time to solution*)
* System view (objectives)
	- Get as many tasks done as quickly as possible
		- *throughput* objective
	- Minimize waiting time for processes
		- *response time* objective
	- Get full utilization from the CPU
		- *utilization* objective

### Process Scheduling
* Process transition diagram
* OS perspective
<img width="707" height="353" src="https://github.com/missystem/cis415review/blob/master/processscheduling.png">

### When does scheduling occur?
* CPU scheduling decisions may take place when:
	1. A process switches from running -> waiting
	2. A process switches from running -> ready
	3. A process switches from waiting -> ready
	4. A process terminates
* Process voluntarily gives up (yields) the CPU
	- CPU scheduler kicks in an decides who to go next
	- Process gets put on the ready queue
	- It could get immediately re-scheduled to run
 
### Scheduling Problem
* Choose the ready/running process to run at any time
	- Maximize “performance”
* Model (estimate) “performance” as a function 
	- System performance of scheduling each process
		- *f(process) = y*
	- What are some choices for *f(process)*?
* Choose the process with the best y
	- Estimating overall performance is intractable
	- Scheduling so all tasks are completed ASAP is a NP-complete problem
	- Adding in preemption does not help

### Preemption
* Can we reschedule a process that is actively running (i.e., preempt its execution)?
	- If so, we have a preemptive scheduler
	- If not, we have a non-preemptive scheduler
* Suppose a process becomes ready
	- A new process is created or it is no longer waiting
* There is a currently running process
* However, it may be “better” (whatever this means) to schedule the process just put on the ready queue
	- So, we have to preempt the running process
* In what ways could the new process be better?

### Basic Concepts – CPU-I/O Bursts
* Maximum CPU utilization is obtained with multiprocessing ... Why?
* CPU–I/O burst cycle
	- Process execution consists of cycles of CPU execution and I/O wait
		- run instructions (CPU burst)
		- wait for I/O (I/O burst)
* CPU burst distribution
	- How much a CPU is used during a burst?
	- This is a main concern ... Why?
* Scheduling is aided by knowing the length of these bursts
<img width="240" height="430" src="https://github.com/missystem/cis415review/blob/master/cpu_IO_burst.png">

### Histogram of CPU Burst Times
* Profile for a particular process
<img width="650" height="425" src="https://github.com/missystem/cis415review/blob/master/histogram_of_cpu_burst.png">

### Dispatcher
* Dispatcher module gives control of the CPU to the process selected by the *short-term* scheduler
* This involves:
	- Switching context
		- save context of running process
		- loading process context of selected process to run
	- Switching to user mode
	- Jumping to the proper location in the user program to continue execution of the program
* *Dispatch latency*
	- time it takes for the dispatcher to stop one process and start another running
	- Context switch time

### Scheduling Criteria
* *Utilization / Efficiency*
	- Keep the CPU busy 100% of the time with useful work
* *Throughput*
	- Maximize the number of jobs processed per hour.
* *Turnaround time (latency)*
	- From the time of submission to the time of completion.
* *Waiting time*
	- Sum of time spent (in ready queue) waiting to be scheduled on the CPU
* *Response time*
	- Time from submission until the first response is produced (mainly for interactive jobs)
* *Fairness*
	- Make sure each process gets a fair share of the CPU

### Scheduling Algorithm Optimization Criteria
* Max CPU utilization 
* Max throughput
* Min turnaround time 
* Min waiting time
* Min response time

### Scheduling Algorithms
* Some may seem intuitively better than others
* But a lot has to do with the type of offered *workload* to the processor
* Best scheduling comes with best context of the tasks to be completed
	- Knowing something about the workload behavior is important
* Distinguish between non-preemptive and preemptive cases
	- This has to do with whether a running process can be stopped during its execution and put back on the ready queue so that another process can acquire the CPU and run

### Scheduling Algorithms
* First-come, First-serve (FCFS)
	- Non-preemptive
	- Does not account for waiting time (or much else) 
		- Convoy problem
* Shortest Job First (SJF)
	- May be preemptive
	- Optimal for minimizing waiting time (how?)
* Round Robin
* Priority Scheduling
* Multilevel Queue Scheduling

### First-Come, First-Served (FCFS)
* Serve the jobs in the order they arrive
* Managed with a FIFO queue
* Nonpreemptive
	- Process is run until it has to wait or terminates
	- OS can not stop the process and put it on ready queue
	- *Once the CPU has been allocated to a process, that process keeps the CPU until it releases the CPU, either by terminating or by requesting I/O*
* Simple and easy to implement
	- When a process is ready, add it to tail of ready queue, and serve the ready queue in FCFS order
* Fair
	- No process is starved out
	- Service order is immune to job size (does not depend)
	- It depends only on *time of arrival* <br />

* Example: <br />

| Process | Burst Time |
|:-------:|:----------:| 
|    P<sub>1</sub>   |     24     | 
|    P<sub>2</sub>   |      3     |
|    P<sub>3</sub>   |      3     |

* Burst time here represents the job’s entire execution time <br />
	- Suppose that the processes arrive in the order: P1, P2, P3
		- Assume processes arrive at the same time (e.g., time 0)
	- The Gantt chart for the schedule is: <br />
	<img width="500" height="75" src="https://github.com/missystem/cis415review/blob/master/FCFSex1.png"> <br />
		- Waiting time for P1 = 0; P2 = 24; P3 = 27
		- Average waiting time: (0 + 24 + 27) / 3 = 17
* Reduce waiting time
	- Suppose that the processes arrive in the order: P2, P3, P1
		- Assume that they arrive at the same time
	- The Gantt chart for the schedule is: <br />
	<img width="500" height="75" src="https://github.com/missystem/cis415review/blob/master/FCFSex2.png"> <br />
		- Waiting time for P1 = 6; P2 = 0; P3 = 3
		- Average waiting time: (6 + 0 + 3)/3 = 3
		- Much better than previous case ... Why?
		- *Convoy effect*: short processes gets placed behind long process in the scheduling order (all the other processes wait for the one big process to get off the CPU)

### Shortest-Job-First (SJF)
* Suppose we know length of *next* CPU burst
* Then us these lengths to schedule the process
	- Process with the shortest next CPU burst time goes first
* Two schemes:
	1. Nonpreemptive 
		- once CPU given to the process it cannot bepreempted until it completes its CPU burst
	2. Preemptive 
		- if a new process arrives with CPU burst length less than remaining time of current executing process, then preempt
		- known as the Shortest-Remaining-Time-First (SRTF)
* SJF is optimal
	- Gives minimum average waiting time for a set of processes
	- So we should always use it, right?
* It cannot be implemented at the level of CPU scheduling, as there is no way to know the length of the next CPU burst
* Example <br />

| Process | Arrival Time | Burst Time |
|:-------:|:------------:|:----------:|
|    P<sub>1</sub>   |      0.0     |      7     |
|    P<sub>2</sub>   |      2.0     |      4     |
|    P<sub>3</sub>   |      4.0     |      1     |
|    P<sub>4</sub>   |      5.0     |      4     |

* Nonpreemptive SJF
	- Scheduler makes a decision at the time when the next job is to be scheduled <br />
	<img width="500" height="115" src="https://github.com/missystem/cis415review/blob/master/SJFex1.png"> <br />
	- Average waiting time = (0 + 6 + 3 + 7) / 4 = 4
* Preemptive SJF <br />
	- Scheduler makes a decision at any time using preemption to stop the currently running process <br />
	<img width="520" height="105" src="https://github.com/missystem/cis415review/blob/master/SJFex2.png"> <br />
	- Average waiting time = (9 + 1 + 0 +2) / 4 = 3

### Determining Length of Next CPU Burst
* can only predict the length (do not know for sure)
	- We expect that the next CPU burst will be similar in length to the previous ones
	- By computing an approximation of the length of the next CPU burst, we can pick the process with the shortest predicted CPU burst
* The next CPU burst is generally predicted as an ***exponential average*** of the measured lengths of previous CPU bursts
	- *Given:* <br />
		*t<sub>n</sub> - actual length of the n-th CPU burst* <br />
		*τ<sub>n</sub> - predicted value for the n-th CPU burst* <br />
		*τ<sub>n+1</sub> - predicted value for the next CPU burst* <br />
	 	*α such that 0 ≤ α ≤ 1*
	- *Define:* <br />
		τ<sub>n+1</sub> =αt<sub>n</sub> +(1−α)t<sub>n</sub>
* What to set α to?
	- α = 0? Recent history does not count
	- α = 1? Only the last CPU burst counts
	- Commonly, α set to 1⁄2
* Since both α and (1 - α) are ≤ 1, each successive term has less weight than its predecessor
* SRTF is the preemptive version

### CPU Burst Prediction
* α = 1/2 and τ0 = 10 <br />
<img width="600" height="433" src="https://github.com/missystem/cis415review/blob/master/nextCPUburst.png"> <br />

### Example of Shortest-remaining-time-first
* Now we add the concepts of varying arrival times and preemption to the analysis

| Process | Arrival Time | Burst Time |
|:-------:|:------------:|:----------:|
|    P<sub>1</sub>   |      0     |      8     |
|    P<sub>2</sub>   |      1     |      4     |
|    P<sub>3</sub>   |      2     |      9     |
|    P<sub>4</sub>   |      3     |      5     |

* Preemptive SJF Gantt Chart <br />
<img width="500" height="75" src="https://github.com/missystem/cis415review/blob/master/SRTFex.png"> <br />
* Average waiting time <br />
= [(10 - 1) + (1 - 1) + (17 - 2) + (5 - 3)] / 4 <br />
= 26 / 4 <br />
= 6.5 msec <br />


### Round Robin (RR)
* Each process gets a small unit of CPU time (time quantum / time slice)
	- Usually 10-100 milliseconds
	- After this time has elapsed, the process is *preempted* and added to the end of the ready queue
* Approach
	- Consider *n* processes in the ready queue
	- Consider time quantum is *q*
	- Then each process gets *1/n* of the CPU time
	- In chunks of at most *q* time units at once
	- No process waits more than *(n - 1)q* time units
* Example of RR with Time Quantum = 4 <br />

| Process | Burst Time |
|:-------:|:----------:|
| P<sub>1</sub> | 24 |
| P<sub>2</sub> |  3 |
| P<sub>3</sub> |  3 |

* The Gantt chart <br />
<img width="500" height="75" src="https://github.com/missystem/cis415review/blob/master/RRex.png"> <br />
	- Typically, higher average turnaround than SJF, but better response times
	- *q* should be large compared to context switch time 
		- Usually *q* is between 10ms to 100ms
		- Context switch < 10 uses
* RR Time Quantum
	- Round robin allows the CPU to be virtually shared between the processes
		- Each process has the illusion that it is running in isolation (at *1/n*-th the CPU speed)
	- Smaller time quantums make this illusion more realistic, but there are problems
		- What is the main problem? <br />
		If context-switch time is added in, the average turnaround time increases even more for a smaller time quantum, since more context switches are required.
	- Larger time quantums will give more preference to processes with larger burst times
		- What scheduling algorithm is approximated when quantums are very large? <br />
		If the time quantum is too large, RR scheduling degenerates to an FCFS policy

* How a smaller time quantum increases context switches: <br />
<img width="480" height="210" src="https://github.com/missystem/cis415review/blob/master/RRcontextswitch.png"> <br />
* How turnaround time varies with the time quantum: <br />
<img width="410" height="340" src="https://github.com/missystem/cis415review/blob/master/RRturnaround.png"> <br />
* ==> 80 percent of the CPU bursts should be shorter than the time quantum
* If time quantum size decreases, what happens to the context switches?
	- Context switches are not free!
		- Saving/restoring registers
		- Switching address spaces
		- Indirect costs (cache pollution)

### Priority Scheduling
* Each process is given a certain priority “value”
* Always schedule the process with highest priority
	- Preemptive
	- Non-preemptive
* Problems:
	- Starvation (indefinit blocking)
		- Low priority processes may never execute 
* Solution:
	- Aging
		- gradually increasing the priority of processes that wait in the system for a long time
* FCFS and SJF are specialized versions of Priority Scheduling
	- Assigning priorities to the processes in a certain way
		- What would the priority function be for FCFS? 
		- What would the priority function be for SJF?

* Example of Priority Scheduling <br />

|      Process 		 | Burst Time |  Priority  |
|:------------------:|:----------:|:----------:|
|    P<sub>1</sub>   |     10     |      3     |
|    P<sub>2</sub>   |      1     |      1     |
|    P<sub>3</sub>   |      2     |      4     |
|    P<sub>4</sub>   |      1     |      5     |
|    P<sub>5</sub>   |      5     |      2     |

* Priority scheduling Gantt Chart (non-preemptive)<br />
<img width="495" height="75" src="https://github.com/missystem/cis415review/blob/master/PSex.png"> <br />
	- Average waiting time = 8.2 msec

### Scheduling Desirables
* SJF
	- Minimize waiting time
		- requires estimate of CPU bursts 
* Round robin
	- Share CPU via time quanta
		- if burst turns out to be “too long”
* Priorities
	- Some processes are more important
	- Priorities enable composition of “importance” factors
* No single best approach -> combinations

### Round Robin with Priority
* Have a ready queue for each priority level
* Always service the non-null queue at the highest priority level
* Within each queue, you perform round-robin scheduling between those processes
* Problems
	- With fixed priorities
		- Lower priority processes can get starved out
	- In general, you employ a mechanism to “age” the priority of processes
<img width="356" height="246" src="https://github.com/missystem/cis415review/blob/master/RRwithP.png"> <br />

### Multilevel Queue Scheduling
* Ready queue is partitioned into separate queues: 
	- foreground (interactive)
	- background (batch)
* These two types of processes have different response-time requirements and so may have different scheduling needs
* Each queue has its own scheduling algorithm: 
	- foreground – RR
	- background – FCFS
* Scheduling must be done between the queues 
	- Fixed priority scheduling
		- possibility of starvation 
	- Time slice
		- each queue gets a certain amount of CPU time which it can schedule amongst its processes
<img width="450" height="250" src="https://github.com/missystem/cis415review/blob/master/MQscheduling.png"> <br />

### Multilevel Feedback Queue
* Allow processes move between queues <br />
	*Aging* can be implemented this way
* A process uses too much CPU time will be moved to a lower-priority queue
* A process that waits too long in a lower-priority queue may be moved to a higher-priority queue
* Multilevel-feedback-queue scheduler defined by the following parameters:
	- Number of queues
	- Scheduling algorithms for each queue
	- Method used to determine when to upgrade a process
	- Method used to determine when to demote a process
	- Method used to determine which queue a process will enter when that process needs service
* Example <br />
<img width="350" height="210" src="https://github.com/missystem/cis415review/blob/master/MFQ.png"> <br />
* Three queues:
	- Q<sub>0</sub> – RR with time quantum 8 milliseconds
	- Q<sub>1</sub> – RR time quantum 16 milliseconds
	- Q<sub>2</sub> – FCFS
* Scheduling
	- A new job enters queue Q<sub>0</sub> which served FCFS
		- when it gains CPU, job receives 8 milliseconds
		- if it does not finish in 8 milliseconds, job is moved to queue Q<sub>1</sub>
	- At Q<sub>1</sub> job is again served FCFS and receives 16 additional milliseconds
		- if it still does not complete, it is preempted and moved to queue Q<sub>2</sub>

### Traditional UNIX Scheduling
* Multilevel Feedback Queues
* 128 priorities possible (-64 to +63)
* 1 Round Robin Queue per priority
* Every scheduling event the scheduler picks the highest priority (lowest number) non-empty queue and runs jobs in Round Robin

### UNIX Process Scheduling
* Negative numbers reserved for processes waiting in kernel mode
	- just woken up by interrupt handlers
	- why do they have a higher priority?
* Time quantum = 1/10 sec 
	- empirically found to be the longest quantum that could be used without loss of the desired response for interactive jobs such as editors
	- Short time quantum means better interactive response
	- Long time quantum means higher oeverall system throughput since less context switch overhead and less processor cache flush
* Priority dynamically adjusted to reflect
	- Resource requirement (e.g., blocked awaiting an event)
	- Resource consumption (e.g., CPU time)

### Linux Scheduler
* Kernel 2.4 and earlier: essentially the same as the traditional UNIX scheduler
* Kernel 2.6: O(1) scheduler
	- Time to select process is constant regardless of system load or the number of processors
	- Separate queue for each priority level
	- CPU affinity (keeps processes on same CPU)
* More recently (kernel 2.6.23 and up): CFS
	- Completely Fair Scheduler (runs O(log N))
		- uses red-black trees rather than runqueues

### Linux Scheduling
* Two algorithms:
	- time-sharing 
	- real-time
* Time-sharing (still abstracted)
	- Two queues: 
		- active 
		- expired
	- In active, until you use your entire time slice (quantum), then expired
		- once in expired, wait for all others to finish (fairness)
	- Priority recalculation -- based on waiting vs. running time 
		- from 0-10 milliseconds
		- add waiting time to value, subtract running time
		- adjust the static priority
* Real-time
	- Soft real-time
	- Posix.1b compliant – two classes
		- FCFS and RR (highest priority process always runs first)

### Priorities and Time-Slice Length
<img width="378" height="218" src="https://github.com/missystem/cis415review/blob/master/PandTSL.png"> <br />

### Thread Scheduling
* When threads are supported, it is threads that are scheduled, not processes
	- Distinction between user-level and kernel-level threads
* Many-to-one and Many-to-many models, thread library schedules user-level threads to run on LWP
	-  Known as process-contention scope (PCS) since scheduling competition is within the process
	- Typically done via priority set by programmer
* Kernel thread scheduled onto available CPU is system-contention scope (SCS) - competition among all threads in sytem

### Multiple-Processor Scheduling
* CPU scheduling is more complex with multiple CPUs
* Consider homogeneous processors within a multiprocessor
* *Asymmetric* multiprocessing
	- Only one processor accesses the system data structures, reducing the need for data sharing
	- Some parts of the OS only runs on this processor
* *Symmetric* multiprocessing (SMP)
	- Each processor is self-scheduling
		1. All threads may be in a common ready queue.
		2. Each processor may have its own private queue of threads
	- Currently, most common approach <br />
	<img width="405" height="210" src="https://github.com/missystem/cis415review/blob/master/multiprocessor.png"> <br />

### Multicore Processors
* Recent trend to place multiple processor cores on same physical chip
* Faster and consumes less power
* Multiple threads per core also growing 
	- Takes advantage of memory stall to make progress on another thread while memory retrieve happens

### Multithreaded Multicore System
* when a processor accesses memory, it spends a significant amount of time waiting for the data to become available
	- Memory stall occur
		- primarily because modern processors operate at much faster speeds than memory
		- can also because of a cache miss (accessing data that are not in cache memory)
<br /> <img width="455" height="120" src="https://github.com/missystem/cis415review/blob/master/singlecore.png"> <br />
* To remedy this situation, many recent hardware designs have imple- mented multithreaded processing cores in which two (or more) hardware threads are assigned to each core
* Multithreaded multicore system
<br /> <img width="470" height="125" src="https://github.com/missystem/cis415review/blob/master/mtmc.png"> <br />

### Load Balancing
* Need to keep all CPUs loaded for efficiency
* Load balancing keep the workload evenly distributed across all processors in an SMP system
* Push migration
	- periodically checks the load on each processor
		- if it finds an imbalance
		- evenly distributes the load by moving (or pushing) threads from overloaded to idle or less-busy processors
	- Pushes task from overloaded CPU to other CPUs
* Pull migration
	- Idle processors pulls waiting task from busy processor

### Processor affinity
* Process favors the processor on which it is currently running
	- soft affinity
		- OS has a policy of attempting to keep a process running on the same processor
		- but not guaranteeing that it will
	- hard affinity
		- allowing a process to specify a subset of processors on which it can run

### Real-time Scheduling
* *Hard real-time* systems
	- required to complete a critical task within a guaranteed amount of time
	- A task must be serviced by its deadline; service after the deadline has expired is the same as no service at all
* *Soft real-time* computing
	- requires that critical threads receive priority over less critical ones
	- provide no guarantee as to when a critical real-time process will be scheduled

* Minimizing Latency
* Priority-Based Scheduling
* Rate-Monotonic Scheduling
* Earliest-Deadline-First Scheduling
* Proportional Share Scheduling
* POSIX Real-Time Scheduling

### Algorithm Evaluation
* Maximizing CPU utilization under the constraint that the maximum response time is 300 milliseconds
* Maximizing throughput such that turnaround time is (on average) linearly proportional to total execution time

---

## Summary
* CPU Scheduling
	- Algorithms
	- Combination of algorithms
		- Multi-level Feedback Queues
* Scheduling Systems
	- UNIX
	- Linux

## Summary from book
* **CPU scheduling**
	- The task of selecting a waiting process from the ready queue 
	- Allocating the CPU to it
	- The CPU is allocated to the selected process by the dispatcher
* **Scheduling algorithms**
	- preemptive
		- where the CPU can be taken away from a process 
	- nonpreemptive
		- where a process must voluntarily relinquish control of the CPU
	- Almost all modern operating systems are preemptive
* Scheduling algorithms can be **evaluated** by 5 criteria: 
	1. CPU utilization
	2. throughput
	3. turnaround time
	4. waiting time
	5. response time.
* **First-come, first-served (FCFS) scheduling**
	- The simplest scheduling algorithm
	- May cause short processes to wait for very long processes
* **Shortest-job-first (SJF) scheduling**
	- Provably optimal
	- Providing the shortest average waiting time
	- Implementing is difficult because predicting the length of the next CPU burst is difficult
* **Round-robin (RR) scheduling**
	- Allocates the CPU to each process for a time quantum
		- If the process does not relinquish the CPU before its time quantum expires, the process is preempted, another process is scheduled to run for a time quantum
* **Priority scheduling**
	- Assigns each process a priority
	- The CPU is allocated to the process with the highest priority
	- Processes with the same priority can be scheduled in FCFS order or using RR scheduling
* **Multilevel queue scheduling**
	- Partitions processes into several separate queues arranged by priority
	- The scheduler executes the processes in the highest-priority queue
	- Different scheduling algorithms may be used in each queue.
* **Multilevel Feedback Queues**
	- similar to multilevel queues
	- except that a process may migrate between different queues
* **Multicore processors** 
	- place one or more CPUs on the same physical chip
	- each CPU may have more than one hardware thread
	- From the perspective of the operating system
		- each hardware thread appears to be a logical CPU
* **Load balancing** on multicore systems 
	- equalizes loads between CPU cores
	- migrating threads between cores to balance loads may invalidate cache contents
		- may increase memory access times
* **Soft real-time scheduling**
	- gives priority to real-time tasks over non-real-time tasks
* **Hard real-time scheduling**
	- provides timing guarantees for real-time tasks
* **Rate-monotonic real-time scheduling**
	- schedules periodic tasks using a static priority policy with preemption
* **Earliest-deadline-first (EDF) scheduling**
	- Assigns priorities according to deadline
	- The earlier the deadline the higher the priority
	- the later the deadline, the lower the priority.
* **Proportional share scheduling**
	- allocates *T* shares among all applications
	- If an application is allocated *N* shares of time, it is ensured of having *N∕T* of the total processor time
* **Completely fair scheduler (CFS)**
	- Linux uses it
	- Assigns a proportion of CPU processing time to each task
	- The proportion is based on the virtual runtime (```vruntime```) value associated with each task
* **Modeling and Simulations**
	- can be used to evaluate a CPU scheduling algorithm
























