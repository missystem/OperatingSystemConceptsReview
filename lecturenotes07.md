## [Lecutre 07: Concurrency and Synchronization (Chapter 6/7)](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecture-7-synchronization.pdf)

## Outline
* Background
* [Critical-Section Problem](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes07.md#critical-section-problem-dijkstra-1965)
* [Peterson’s Solution](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes07.md#petersons-solution)
* [Synchronization Hardware](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes07.md#hardware-support-for-synchronization)
* [Mutual Exclusion (Mutex) Locks](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes07.md#mutex-locks)
* [Semaphores](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes07.md#semaphore-dijkstra)
* [Monitors](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes07.md#monitors)
* [Classic Problems of Synchronization](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes07.md#classical-problems-of-synchronization)
* [Chapter 6 Summary from OSC](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes07.md#chapter-6-summary-from-osc)
* [Chapter 7 Summary from OSC](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes07.md#chapter-7-summary-from-osc)

---

### Roadmap
<img width="408" height="250" src="https://github.com/missystem/cis415review/blob/master/roadmap.png">

### Concurrency and Synchronization
* Processes can execute concurrently (logically, physically)
	- May be interrupted at any time, partially completing execution
	- OS must concurrently execute its own software
* There are different kinds of resources that are shared between processes:
	- Physical (terminal, disk, network, ...)
	- Logical (files, sockets, memory, ...)
	- Memory
* Concurrent access to shared resources must be done in a consistent manner or else errors arise
* Maintaining data consistency requires mechanisms to ensure the orderly execution of cooperating processes
* This is the role of synchronization

### Resources
* Focus on “memory” as the shared resource for the purposes of this discussion
	- Processes can all read and write into memory 
	- Suppose multiple processes share memory
		- use IPC shared memory segments mechanisms
	- It might be easier to think of multiple threads 
		- sharing memory is natural

### Example Problem Due to Sharing
* Consider a shared printer queue
	- *```spoolQueue[n]```*
* 2 processes want to enqueue an element each to this queue
* *```tail```* points to the current end of the queue
	- It is also a shared variable
* Each process needs to do <br />
	```
	tail = tail + 1; 
	spoolQueue[tail] = "element";
	```

### What are trying to do?
* Want to have “consistent” and “correct” execution 
<br /><img width="355" height="125" src="https://github.com/missystem/cis415review/blob/master/spoolQueue.png"><br />
* What could go wrong?
	- ```tail = tail + 1``` is NOT a single machine instruction
		- So, what? Why do we care?
	- What assembly code does the compiler produce? <br />
	*Load tail, R1 <br />
	Add R1, 1, R2 <br />
	Store R2, tail <br />*
	- These 3 machine instructions might NOT be executed atomically ... Why not?
		- To *execute atomically* means to execute multiple instructions logically together as if they were a single instruction without being interrupted
	- What is the problem?

### Interleaving
* Each process executes this set of 3 instructions
* Interrupts might happen at any time
	- a context switch can happen at any time
* Suppose we have the following scenario:
<br /><img width="253" height="125" src="https://github.com/missystem/cis415review/blob/master/Interleaving.png"><br />
* Leading to incorrect execution:
<br /><img width="380" height="120" src="https://github.com/missystem/cis415review/blob/master/incorrectExecution.png"><br />
* ==> Race condition

### Race Conditions
* Several processes access and manipulate the same data concurrently and the outcome of the execution depends on the particular order in which the access takes place
* Race conditions are timing dependent
	- Errors can be non-repeatable
* Race conditions CAN cause the “state” of the execution to be inconsistent (incorrect)
	- It does not mean that because there is a race condition that the state WILL become inconsistent

### How to avoid race condition?
* Atomic: 
	- A set of instructions is **atomic** if it executed as if it was a single instruction (logically)
* What does this mean exactly?
	- Executing the instructions can not be interrupted?
	- It has more to do with the outcomes of executing the instructions with respect to other processes
* Suppose the 3 assembly instructions we were looking at were atomic
* Does this avoid the race condition?
* Critical Section: 
	- When executing a set of instructions is vulnerable to a race condition, that set of instructions are said to constitute a **critical section**
	- the process may be accessing — and updating — data that is shared with at least one other process

### Critical Section Problem (Dijkstra, 1965)
* Consider system of n processes {P<sub>0</sub>, P<sub>1</sub>, ... P<sub>n-1</sub>}
* Each process has critical segment of code
	- Process may be changing common variables, updating table, writing file, ... (in its critical section)
	- When one process is in its critical section, no other may be in its critical section (mutual exclusion)
* *Critical section problem* is to design a *protocol* between the processes to solve this
	- Each process must enter the critical section (entry section)
	- Each process then executes the critical section instructions 
	- Each process must exit the critical section (exit section)
	- Each process executes outside the critical section

### Critical Section
* General structure of process *P<sub>i</sub>*
<br /><img width="400" height="215" src="https://github.com/missystem/cis415review/blob/master/generalCriticalSectionStructure.png"><br />
* It is the entry and exit code that defines the critical section protocol
* The infinite do loop is just to suggest that a process will possibly want to enter its critical section multiple \#s of times, including never (0 times) or just once (1 time)

### Requirements for Solution to CS Problem
1. Mutual exclusion
	- If process *P<sub>i</sub>* is executing in its critical section, then no other processes can be executing in their critical sections
2. Progress
	- If no process is executing in its critical section and some processes wish to enter their critical sections, then only those processes that are not executing in their remainder sections can participate in deciding which will enter its critical section next, and this selection cannot be postponed indefinitely
3. Bounded waiting
	- There exists a bound, or limit, on the number of times that other processes are allowed to enter their critical sections after a process has made a request to enter its critical section and before that request is granted
	- Assume that each process executes at a nonzero speed
	- No assumption concerning relative speed of the *N* processes

### How to Implement Critical Sections
* Implementing critical section solutions follows 3 fundamental approaches
	1. Disable Interrupts
		- Effectively stops the scheduling of other processes
		- Does not allow another process to get the CPU
	2. Busy Waiting / Spinlock solutions
		- Pure software solutions
		- Integrated hardware-software solutions
	3. Blocking Solutions

### Disabling Interrupts
* We already know how to prevent another process from interrupting the current running process
	- Do not allow interrupts to occur by disabling them
* Advantages:
	- Simple to implement (single instruction)
* Disadvantages:
	- Do not want to give such power to user processes
	- Does not work on a multiprocessor ... Why?
		- Disabling interrupts on a multiprocessor can be time consuming, since the message is passed to all the processors
		- This message passing delays entry into each critical section, and system efficiency decreases
		- Also consider the effect on a system’s clock if the clock is kept updated by interrupts
	- Disables multiprogramming even if another process is NOT interested in critical section

### Busy Waiting (aka Spinning)
* Overall philosophy:
	- Keep checking some state (variables) until they indicate other process(es) are not in critical section
<br /><img width="453" height="227" src="https://github.com/missystem/cis415review/blob/master/busyWaiting.png"><br />
* Remember, P1 and P2 might do this multiple \#s of times, including 0.

### Reading, Writing, and Testing Locks
* Is the instruction below atomic?
	<br /> A: load locked, R1 
	<br />cmp R1, 1
	<br />beq A<br />
	```while (locked == TRUE)```  
* How about these instructions?
	```
	locked = TRUE;
	locked = FALSE;
	```
* Generally, if the high-level statement compiles to a single machine instruction, it is atomic
* Need reading, writing, testing of locks to be atomic

### Try Strict Alternation
* Consider this code (Left)
* Idea is to take turns using the critical section
	- Variable turn is used for this
* Does it work?
* What problems do you see? 
	- Is there mutual exclusion?
	- Is there progress?
	- Is there bounded waiting?  
* Remember, P1 and P2 might do this multiple #s of times, including 0

<br /><img width="275" height="412" src="https://github.com/missystem/cis415review/blob/master/strictAlternation.png"><img width="374" height="432" src="https://github.com/missystem/cis415review/blob/master/fixingProgress.png"><br />

### Fixing the “progress” requirement
* What about this code? (Right)
* Each process has a flag to say that they want to enter the critical section
* Got mutual exclusion
* Problems?
	- Deadlocked
* For this reason, it does NOT meet the progress or bounded waiting requirements either

### Peterson’s Solution
* Consider 2 processes
* Assume that the LOAD and STORE instructions are atomic and cannot be interrupted
* The two processes share two variables: 
	```
	int turn;
	boolean flag[2]
	```
* Variable turn indicates whose turn it is to enter the critical section
	- The ```flag``` array 
		- indicate if a process is ready to enter the critical section
		- ```flag[i] = true``` implies that process P<sub>i</sub> is ready
* Algorithm for Process Pi and Process Pj
<br /><img width="560" height="330" src="https://github.com/missystem/cis415review/blob/master/PetersonsSolution.png"><br />

### Does Peterson’s Solution work?
* Prove that the 3 CS requirements are met:
	- Mutual exclusion is preserved <br />
	P<sub>i</sub> enters CS only if:<br />
		- either ```flag[j] = false``` or ```turn = i```
		- if both processes are interested in entering, then 1 condition is false for one and true for the other process
	- Progress requirement is satisfied
		- a process wanting to enter will be able to do so at some point
		- Why?
	- Bounded-waiting requirement is met
		- eventually it will be P<sub>i</sub>’s turn if P<sub>i</sub> wants to enter
* Peterson’s solution **ONLY** works for 2 process solutions

### Multiple Processes
* How do we extend for multiple processes?
* Is there a way to modify Peterson’s approach?
* Consider the following enter /exit routines:
```
int turn;
int flag[N]; /* all set to FALSE initially */

enterCS(int myid) {
    otherid = (myid+1) % N;
    turn = otherid;
    flag[myid] = TRUE;

    while (turn == otherid && flag[otherid] == TRUE) ;
    /* proceed if turn == myid or flag[otherid] == FALSE */
}

leave_CS(int myid) {
	flag[myid] = FALSE;
}
```
* Remember, processes might do this multiple #s of times, including 0

### Bakery Algorithm (Leslie Lamport, 1974)
* We need to enforce a sequence in some manner that everyone will follow and contribute to making progress
* Think about a bakery
* See [A New Solution of Dijkstra's Concurrent Programming Problem](http://lamport.azurewebsites.net/pubs/bakery.pdf)
```
Notation: (a,b) < (c,d) if a<c  or  a=c and b<d

Every process has a unique id (integer) Pi

bool choosing[0..n-1]; /* all set to FALSE */ 
int number[0..n-1]; /

enter_CS(myid) { 
	choosing[myid] = TRUE; 
	number[myid] = max(number[0],number[1],...,number[n-1]) + 1;
	choosing[myid] = FALSE; 
	for (j=0 to n-1) {
	    while (choosing[j])
	      ;
	    while (number[j] != 0) && ((number[j],Pj)<(number[myid],myid))
	      ;
	}
}

leave_CS(myid) {
	number[myid] = 0;
}
```
* Evaluation of the Bakery Algorithm
	- Does it work?
	- What do you need to prove?
	- Show that it meets
		- Mutual exclusion
		- Progress
		- Bounded waiting requirements
	- Need to know that maximum \# processes

### Looking to the Hardware
* Complications arose because we had atomicity only at the granularity of a machine instruction
	- What a machine instruction could do is (was) limited
* Can we provide specialized instructions in hardware to provide additional functionality (with an instruction still being atomic)?
	- Looking again to hardware to help solve a OS problem q Many systems (now) provide hardware support for implementing the critical section code
* All solutions below are based on idea of locking
	- Protecting critical regions via locks
	- But without special instructions

### Hardware Support for Synchronization
* Uniprocessors – could disable interrupts
	- Currently running code executes without preemption
	- Generally too inefficient on multiprocessor systems 
		- OS using this not broadly scalable
* Modern CPUs provide atomic hardware instructions (2 general types)
	- **Test-and-Set**
		- test memory word and set value
	- **Compare-and-Swap**
		- swap contents of 2 memory words

### Critical-section Solution Using Locks
<br /><img width="170" height="165" src="https://github.com/missystem/cis415review/blob/master/lock.png"><br />
* The infinite while loop is just to suggest that a process will possibly want to enter its critical section multiple \#s of times, including 0 times and 1 time
* The problem before was that ```acquire``` and ```release``` could not be done in a single instruction
* Suppose it could with a single atomic instruction

### test_and_set instruction
* Definition
```
boolean test_and_set(boolean *target) { 
	boolean rv = *target;
	*target = true;
	return rv;
}
```
* Assume it executes atomically
* Returns the original value of passed parameter
* Set the new value of passed parameter to TRUE

### Solution using test_and_set()
* Shared boolean variable ```lock```, initialized to FALSE
* Mutual-exclusion implementation with ```test_and_set()```:
```
while (TRUE){ 
	...

	while (test_and_set(&lock) == TRUE) 
		;

		/* critical section */

	lock = FALSE;

		/* remainder section */
}
```
* if two test and set() instructions are executed simultaneously (each on a different core), they will be executed sequentially in some arbitrary order

### compare_and_swap Instruction
* Definition:
```
int compare_and_swap(int *value, int expected, int newvalue)
{
	int temp = *value;

	if (*value == expected)
		*value = newvalue;

	return temp;
}
```
* Assume it executes atomically
* Returns the original value of passed parameter value
* Set the variable ```value``` to the value of the passed parameter ```newvalue```
	- Only if ```value == expected```
* That is, the swap takes place only under this condition

### Solution using compare_and_swap
* Shared integer ```lock``` initialized to 0
```
while (TRUE) {
	...

	while(compare_and_swap(&lock, 0, 1) != 0)
		;	/* do nothing */

		/* critical section */

	lock = 0;

		/* remainder section */
}
```
* Does is work?
	- satisfies the mutual-exclusion requirement
	- does not satisfy the bounded-waiting requirement

### Bounded-waiting with test_and_set
* N processes: P<sub>0</sub>, P<sub>1</sub>, ..., P<sub>N-1</sub>
* This is the code for P<sub>i</sub> (i.e., each process is executing this code with *i* set to the process ID (*0 ≤ i ≤ N-1*)
* Bounded-waiting mutual exclusion with ```compare_and_swap()```
```
while (TRUE) {
	waiting[i] = TRUE;
	key = TRUE;
	while (waiting[i] && key == 1)
		key = compare and swap(&lock,0,1); 
	waiting[i] = false;

	    /* critical section */

	j = (i + 1) % n;
	while ((j != i) && !waiting[j])
		j = (j + 1) % n;
	
	if (j == i)
		lock = 0;
	else
		waiting[j] = false;
	
		/* remainder section */
}
```

### Spinning vs. Blocking
* In the previous solutions, we are spinning (busy-waiting) for some condition to change
	- This change should be effected by some other process
	- We are “presuming” that this other process will eventually get the CPU
		- is this a reasonable assumption?
		- needs a preemptive scheduler ... Why?
* This can be *inefficient* because:
	- wasting time quantum spinning
	- Sometimes, the programs may not work
		- suppose if the OS scheduler is not preemptive

### Blocking Approaches
* If instead of busy-waiting, the process relinquishes the CPU at the time when it cannot proceed, the process is said to **block**
	- It is still wanting to get into the critical section
	- It is put in the blocked queue
* It is the job of the process changing the *condition* to wake up a blocked process
	- Moves it from blocked back to ready queue
* Advantage:
	- Do not unnecessarily occupy CPU cycles 
* Disadvantages?

### Blocking Example
<br /><img width="390" height="165" src="https://github.com/missystem/cis415review/blob/master/BlockingExample.png"><br />
* Think of L as a lock
* NOTE: These must be OS system calls! Why?

### Mutex Locks
* Previous solutions are complicated and generally inaccessible to application programmers
* OS designers build software tools to solve critical section problem
* Simplest is a mutex lock (“mutual exclusion”)
* Protect a critical section by first ```acquire()``` a lock then
```release()``` the lock
	- Boolean variable indicating if lock is available or not
* Calls to ```acquire()``` and ```release()``` must be atomic
	- Usually implemented via hardware atomic instructions
* This solution generally requires busy waiting
	- This lock therefore is called a *spinlock*
* definition of ```aquire()```
```
acquire() {
	while (!available)
		; /* busy wait */
	available = false;
}
```
* definition of ```release```
```
release() { 
	available = true;
}
```
* Solution to the critical-section problem using mutex locks
	- Calls to either acquire() or release() must be performed atomically
```
while (TRUE) {
	acquire lock

		critical section

	release lock

		remainder section
}
```

### Synchronization Constructs
* Synchronization requires more than just exclusion
	- If printer queue is full, I need to wait until there is at least 1 empty slot
* Note that ```acquire()``` / ```release()``` are not very suitable to implement such synchronization ... Why?
* Main Disadvantage 
	- requires busy waiting
		- While a process is in its critical section, any other process that tries to enter its critical section must loop continuously in the call to acquire()
		- This continual looping is clearly a problem in a real multiprogramming system, where a single CPU core is shared among many processes
		- Busy waiting also wastes CPU cycles that some other process might be able to use productively
* We need constructs to enforce orderings
	- A should be done after B
* We need construction to help keep track of numbers of things

### Semaphore (Dijkstra)
* Synchronization tool that provides more sophisticated ways (than mutex locks) for processes to synchronize their activities
* A semaphore *S* is an integer variable
* Can only be accessed via two indivisible (atomic) operations
	- ```wait()``` and ```signal()```
	- Originally called P() and V() by Dijkstra <br />
	P = Probeer (‘try’ in Dutch)<br />
	V = Verhoog (‘increment’ in Dutch)<br />
* Definition of ```wait()```:
```
wait(S) {
	while (S <= 0)
		; // busy wait 
	S--;
}
```
* Definition of ```signal()```
```
signal(S) { 
	S++;
}
```
* [Semaphore (Wikipedia page)](https://en.wikipedia.org/wiki/Semaphore_(programming))

### Semaphore Usage
* *Binary semaphore*
	- Integer value can range only between 0 and 1
	- Same as a mutex lock
* *Counting semaphore*
	- Integer value can range over an unrestricted domain
* Can solve various synchronization problems with semaphores
* Consider P<sub>1</sub> and P<sub>2</sub> that require S<sub>1</sub> to happen before S<sub>2</sub>
	- Create a semaphore ```synch``` initialized to 0
	```
	P1:
		S1;
		signal(synch);
	P2:
		wait(synch);
		S2;
	```
* Can implement a counting semaphore *S* be a binary semaphore?
	- yes

### Semaphore Implementation
* Must guarantee that no two processes can execute the ```wait()``` and ```signal()``` ...
	- ... on the same semaphore ...
	- ... at the same time ...
* Thus, the implementation becomes the critical section problem where the ```wait``` and ```signal``` code are placed in the critical section
	- Could have busy waiting in critical section implementation
		- but implementation code is short
		- little busy waiting if critical section rarely occupied
* Note that applications may spend lots of time in critical sections and therefore this is not a particularly good solution

### Semaphores without Busy waiting
* With each semaphore there is an associated waiting queue
* Each entry in a waiting queue has two data items:
	- value (of type integer)
	- pointer to next record in the list
* Two operations:
	- ```block()```
		- place the process invoking the operation on the appropriate waiting queue
	- ```wakeup()```
		- remove one of processes in the waiting queue and place it in the ready queue
```
typedef struct{ 
	int value;
	struct process *list; 
} semaphore;

wait(semaphore *S) {
	S->value--;
	if (S->value < 0) {
		add this process to S->list;
		block(); 	// or sleep();
	}
}

signal(semaphore *S) {
	S->value++;
	if (S->value <= 0) {
		remove a process P from S->list;
		wakeup(P); 
	}
}
```
* NOTE: These are involving OS system calls, and there is no atomicity lost during the execution of these routines because interrupts are being disabled.

### Problems with Semaphores
* Incorrect use of semaphore operations: 
	- ```signal (mutex) .... wait (mutex)```
	- ```wait (mutex) ... wait (mutex)```
	- Omitting of ```wait (mutex)``` or ```signal (mutex)``` (or both)
* Deadlock and Starvation are possible
 
### Deadlock and Starvation
* Deadlock:
	- occurs if 2 or more processes are waiting indefinitely for an event that only one of the waiting processes can cause
	- Let S and Q be two semaphores initialized to 1
	<br /><img width="370" height="174" src="https://github.com/missystem/cis415review/blob/master/deadlock.png"><br />
* Starvation (indefinite blocking)
	- A process may never be removed from the semaphore queue in which it is suspended
* Priority inversion
	- Scheduling problem when lower-priority process holds a lock needed by higher-priority process
	- Solved via priority-inheritance protocol

### Monitors
* A high-level abstraction that provides a convenient and effective mechanism for process synchronization
* Abstract Data Type (ADT)
	- Internal variables only accessible by code within the procedure
	- Routines (operations) that operate on the internal variables (shared)
	- External world only sees these operations (not the shared data or how the operations and synchronization are implemented)
* Only one process may be active within the monitor at a time
	- All the processes that are executing monitor code, there can be at most 1 process in ready queue (rest are either blocked or not in monitor!)
* Pseudocode syntax of a monitor
```
monitor monitor name 
{
	/* shared variable declarations */

	function P1 ( . . . ) { 
		...
	}
	function P2 ( . . . ) { 
		...
	}
	function Pn ( . . . ) { 
		...
	}

		.
		.
		.
	initialization code ( . . . ) { 
		...
	} 
}
```

### Schematic view of a Monitor
<br /><img width="316" height="400" src="https://github.com/missystem/cis415review/blob/master/SchematicViewofMonitor.png"><br />

### Condition Variables
* ```condition x, y;```
* Two operations are allowed on a condition variable:
	- ```x.wait()```
		- a process that invokes the operation is suspended until ```x.signal()```
	- ```x.signal()```
		- resumes one of the processes (if any) that invoked ```x.wait()```
		- if no ```x.wait()``` on the variable, then it has no effect on the variable
* NOTE: If the ```signal``` comes before the ```wait```, the signal gets lost!!!
	- You need to be careful since signals are not stored unlike semaphores

### Monitor with Condition Variables
<br /><img width="430" height="303" src="https://github.com/missystem/cis415review/blob/master/MonitorwithConditionVariables.png"><br />

### Condition Variables Choices
* If process *P* invokes ```x.signal()```, and process *Q* is suspended in ```x.wait()```, what should happen next?
	- Both *Q* and *P* cannot execute in parallel
	- If *Q* is resumed, then *P* must wait
* Options include
	- Signal and wait
		- P waits until Q either leaves the monitor or it waits for another condition
	- Signal and continue
		- Q waits until P either leaves the monitor or it waits for another condition
	- Both have pros and cons – language implementer can decide
	- Monitors implemented in Jave, C#, Concurrent Pascal, ...

### Mesa versus Hoare Semantics
* Can implement signal in two ways
* *Mesa* (signal and continue)
	- Signal puts waiter on the ready list
	- Signaller keeps lock and the processor
	- Q either waits until P leaves the monitor or waits for another condition
* *Hoare* (signal and wait)
	- Signal gives processor and lock to waiter
	- Waiter gives processor and lock back to the signaller when it finishes
	- It is possible to support nested signalling
	- P either waits until Q leaves the monitor or waits for another condition

---

## Chapter 6 Summary from OSC
* **Race Condition**
	- occurs when processes have concurrent access to shared data 
	- the final result depends on the particular order in which concurrent accesses occur
	- can result in corrupted values of shared data
* **Critical Section**
	- a section of code where shared data may be manipulated 
	- a possible race condition may occur
	- The critical-section problem is to design a protocol whereby processes can synchronize their activity to cooperatively share data
* **Solution** to the critical-section problem must satisfy the following 3 requirements: 
	1. mutual exclusion
		- ensures that only one process at a time is active in its crit- ical section
	2. progress
		- ensures that programs will cooperatively determine what process will next enter its critical section
	3. bounded waiting
		- limits how much time a program will wait before it can enter its critical section
* Software solutions to the critical-section problem
	- Peterson’s solution
	- do not work well on modern computer architectures
* **Hardware support** for the critical-section problem includes:
	- memory barriers
	- hardware instructions
		- such as the compare-and-swap instruction
	- atomic variables.
* **Mutex Lock**
	- provides mutual exclusion by requiring that a process acquire a lock before entering a critical section and release the lock on exiting the critical section
* **Semaphores**
	- like mutex locks
	- can be used to provide mutual exclusion
	- However, whereas a mutex lock has a binary value that indicates if the lock is available or not, a semaphore has an integer value and can therefore be used to solve a variety of synchronization problems
* **Monitor**
	- an Abstract Data Type 
		- provides a high-level form of process synchronization
	- A monitor uses condition variables that allow processes to wait for certain conditions to become true and to signal one another when conditions have been set to true
* **Liveness Problem**
	- Solutions to the critical-section problem may suffer from liveness problems, including
		- deadlock
		- starvation
* The various tools that can be used to solve the critical-section problem as well as to synchronize the activity of processes can be evaluated under varying levels of contention. 
	- Some tools work better under certain contention loads than others

---

### Classical Problems of Synchronization
* Classical problems used to test newly-proposed synchronization schemes
	- Bounded-Buffer Problem
	- Readers and Writers Problem 
	- Dining-Philosophers Problem

### Bounded-Buffer Problem
* n buffers, each can hold one item
```int n```
* Semaphore ```mutex``` initialized to the value 1 
```semaphore mutex = 1;```
* Semaphore ```full``` initialized to the value 0
```semaphore full = 0```
* Semaphore ```empty``` initialized to the value n
```semaphore empty = n;```
<br /><br />
* The structure of the **producer** process 
```
while (TRUE) {
		...
	/* produce an item in next produced */ 
		...
	wait(empty);	/* what is known afterwards? */
	wait(mutex);

	/* --- start of critical section --- */
		...
	/* add next produced to the buffer */	
		...
	/* --- end of critical section --- */

	signal(mutex);	/* release CS access */
	signal(full);	/* informs the consumer */
}
```
<br /><br />
* The structure of the **consumer** process 
```
while (TRUE) {
		...
	/* produce an item in next produced */ 
		...
	wait(full);	/* what is known afterwards? */
	wait(mutex);

	/* --- start of critical section --- */
		...
	/* remove an item from buffer next_consumed */	
		...
	/* --- end of critical section --- */

	signal(mutex);	/* release CS access */
	signal(empty);	/* informs the consumer */
		...
	/* consume the item in next consumed */ ...
		...
}
```

### Readers-Writers Problem
* A data set is shared among a number of concurrent processes
	- Readers 
		- only read the data set (does not perform any updates)
	- Writers 
		- can both read and write
* Problem 
	- Allow multiple readers to read at the same time
	- Only one writer can access the shared data at the same time
* Several variations of how readers and writers are considered 
	- All involve some form of priorities
* Shared data
	- Dataset
	- Semaphore ```rw_mutex``` initialized to 1
	```semaphore rw mutex = 1;```
	- Semaphore ```mutex initialized``` to 1
	```semaphore mutex = 1;```
	- Integer ```read_count``` initialized to 0
	```int read_count = 0;```
<br /><br />
* The structure of a **writer** process
```
while (TRUE){ 
	wait(rw_mutex);

	/* --- start of critical section --- */
		...
		/* writing is performed */ 
		...
	/* --- end of critical section --- */

	signal(rw_mutex);
}
```
<br /><br />
* The structure of a **writer** process
```
while (TRUE){
	wait(mutex);
	read_count++;
	if (read_count == 1) 	/* what is happening here? */
		wait(rw_mutex); 
	signal(mutex);

	/* --- start of critical section --- */
		...
		/* reading is performed */ 
		...
	/* --- end of critical section --- */

	wait(mutex);
	read_count--;

	if (read_count == 0)
		signal(rw_mutex); 
	signal(mutex);
}
```
	- Can multiple readers be in critical section?
		- yes
	- Can writer be in critical section?
		- only one writer can be in critical section

### Readers-Writers Problem Variations
* Do you see any problems? 
* First variation (above)
	- No reader kept waiting unless writer has permission to use shared object
* Second variation
	- Once writer is ready, it performs the write ASAP 
	- How would you implement this?
* Both may have starvation leading to even more variations
* Problem is solved on some systems by kernel providing reader-writer locks

### Dining Philosophers Problem (Dijkstra)
* Philosophers spend their lives alternating thinking and eating
* When a philosopher thinks, she does not interact with her colleagues.
* When a philosopher gets hungry and tries to pick up the two chopsticks that are closest to her (the chopsticks that are between her and her left and right neighbors)
* A philosopher may pick up only one chopstick at a time
	- Need both to eat, then release both when done
* In the case of 5 philosophers
	- Shared data
		- bowl of rice (data set)
		- ```semaphore chopstick[5]``` initialized to 1
* [Wikipedia page for Dining Philosophers Problem](https://en.wikipedia.org/wiki/Dining_philosophers_problem)

### Dining Philosophers Problem Algorithm
* The structure of philosopher *i*
```
while (true) {
	wait(chopstick[i]); 
	wait(chopstick[(i + 1) % 5]);
		...
	/* eat for a while */ 		<-- critical section
		...
	signal(chopstick[i]);
	signal(chopstick[(i + 1) % 5]);
		...
	/* think for awhile */ 
		...
}
```

### Preventing Starving Philosophers
* Deadlock handling
	- Allow at most 4 philosophers to be sitting simultaneously at the table
		- cheating
	- Allow a philosopher to pick up the forks only if both are available
		- picking must be done in a critical section
	- Use an asymmetric solution
		- an odd-numbered philosopher picks up first the left fork and then the right fork
		- even-numbered philosopher picks up first the right fork and then the left fork

### Monitor Solution to Dining Philosophers
* A monitor solution to the dining-philosophers problem
```
monitor DiningPhilosophers
{
	enum {THINKING, HUNGRY, EATING} state[5];
	condition self[5];

	void pickup(int i) { 
		state[i] = HUNGRY; 
		test(i);
		if (state[i] != EATING)
		self[i].wait();
	}

	void putdown(int i) { 
		state[i] = THINKING; 
		test((i + 4) % 5); 
		test((i + 1) % 5);
	}

	void test(int i) {
		if ((state[(i + 4) % 5] != EATING)
			&& (state[i] == HUNGRY) 
			&&(state[(i + 1) % 5] != EATING)) 
		{
	    	state[i] = EATING;
			self[i].signal();
		} 
	}

	initialization code() {
		for (int i = 0; i < 5; i++)
			state[i] = THINKING;
	} 
}
```

### What does each Philosopher do?
* Each philosopher *i* invokes the operations ```pickup()``` and ```putdown()``` in the following sequence: <br />
	```
	DiningPhilosophers.pickup(i);
	EAT
	DiningPhilosophers.putdown(i);
	```
* No deadlock
* Starvation is possible

### Resuming Processes within a Monitor
* If several processes queued on condition x, and ```x.signal()``` executed, which should be resumed?
* FCFS frequently not adequate
* conditional-wait construct of the form ```x.wait(c)```
	- c is priority number
	- Process with lowest number (highest priority) is scheduled next

---

## Chapter 7 Summary from OSC
* Classic problems of process synchronization include 
	- bounded-buffer
	- readers–writers
	- dining-philosophers problems
	- Solutions to these problems can be developed using the tools presented in Chapter 6
		- mutex locks
		- semaphores
		- monitors
		- condition variables.
* Linux uses a variety of approaches to protect against race conditions, including atomic variables, spinlocks, and mutex locks.
* The POSIX API provides mutex locks, semaphores, and condition variables. POSIX provides two forms of semaphores: named and unnamed. Several unrelated processes can easily access the same named semaphore by simply referring to its name. Unnamed semaphores cannot be shared as easily, and require placing the semaphore in a region of shared memory.
* Alternative approaches to solving the critical-section problem include transactional memory, OpenMP, and functional languages. Functional languages are particularly intriguing, as they offer a different programming paradigm from procedural languages. Unlike procedural languages, functional languages do not maintain state and therefore are generally immune from race conditions and critical sections.








