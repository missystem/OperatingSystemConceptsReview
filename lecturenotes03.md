## [Lecture 03: Processes (Chapter 3)](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecture-3-processes.pdf)

## Outline
* [Process Concept](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes03.md#process-concept)
* [Process Operation](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes03.md#process-creation)
* [System Calls to Create Processes](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes03.md#program-creation-system-calls)
* [Process Scheduling](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes03.md#process-scheduling)
* [Summary](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes03.md#summary)

### Overview of Processes
* How are processes created?
	- from binary program to executing process
* How is a process represented and managed?
	- process creation, process control block
* How does the OS manage multiple processes?
	- process state, ownership, scheduling
* How can processes communicate?
	- interprocess communication, concurrency, deadlock

### Supervisor and User Modes
* OS runs in “supervisor” mode
	- has access to protected (privileged) instructions only
available in that mode (kernel executes in ring 0)
	- allows it to manage the entire system
* OS “loads” programs into processes
	- run in user mode
	- many processes can run in user mode at the same time

### Process Concept
* An operating system executes a variety of programs:
	- Batch system 
		- jobs
	- Time-shared systems
		- user programs or tasks
	- job & process used almost interchangeably
* A process is a **program in execution**
	- Process execution can result in more processes being created
* Multiple parts of a process
	- Program code 
		- image, text
	- Execution state
		- program counter, processor registers, ...
	- Stack 
		- containing temporary data (e.g., call stack frames) 
			- function parameters, return addresses, local variables, ...
	- Data section
		- containing global variables
	- Heap
		- containing memory dynamically allocated during run time

### A Process in Memory
* A process has to reference memory for different purposes
	- Instructions
	- Stack
		- subroutine “frames”, local variables
	- Data
		- static and dynamic (heap)
* Logical memory
	- is what can be referenced by an address
	- \# address bits in instruction address determine logical memory size
	- a “logical address” is from 0 to the size of logical memory (max)
* Compiler and OS determine where things get placed in logical memory
<img width="290" height="474" src="https://github.com/missystem/cis415review/blob/master/process_in_memory.png">

### Process Address Space
* All locations addressable by process
	- Also called logical address space
	- Every process has one
* Restrict addresses to different areas
	- Restrictions enforced by OS
	- Text segment is where read only program instructions are stored
	- Data segment hold the data for the running process (read/write)
		- heap allows for dynamic data expansion
	- Stack segment is where the stack lives
* Process (logical) address space starts at 0 and runs to a high address
<img width="370" height="450" src="https://github.com/missystem/cis415review/blob/master/address_space.png">

### Process Address Space
* Program (Text)
* Global Data (Data)
* Dynamic Data (Heap)
	- grows up
* Thread-local Data (Stack)
	- grows down
* Each thread has its own stack
* \# address bits determine the addressing range <br />
<img width="363" height="493" src="https://github.com/missystem/cis415review/blob/master/process_add_space01.png">

```
int value = 5;					Global

int main() {
	int *p;					Stack

	p = (int *)malloc(sizeof(int));		Heap

	if (p == 0) {
		printf("ERROR: Out of memory\n”);
		return 1; 
	}

	*p = value; 
	printf("%d\n", *p); 

	free(p);
	return 0;
}
```

### [Process Address Space in Action](https://github.com/missystem/cis415review/blob/master/ProcessAddressSpaceInAction.md)

### Process Creation
* What happens?
	- New process object in the OS kernel is created
		- build process data structures
	- Allocate address space (abstract resource)
		- later, allocate actual memory (physical resource)
	- Add to execution (ready) queue
		- make it runnable
* Is the OS a process?
	- Yes, it is the first process when system is booted

### Process Creation Options (Parent and Child)
* Process hierarchy options
	- Parent process (very first process) creates Children processes
	- Child processes can create other child processes
	- Tree of processes
	<img src="https://github.com/missystem/cis415review/blob/master/process_tree.png">
* Resource sharing options
	- Parent and children share all resources
	- Children share subset of parent’s resources
	- Parent and child share no resources
* Execution options
	- Parent and children execute concurrently
	- Parent waits until children terminate
* Address space
	- Child duplicate of parent
	- Child has a program loaded into it

### Executing a Process
* What is required?
* Registers store state of execution in CPU
	- Stack pointer
	- Data registers
* Program count to indicated what to execute?
	- CPU register holding address of next instruction
* A “thread of execution”
	- able to execute instructions
	- has its own stack
	- each process has at least 1 thread of execution
* A process thread executes instructions found in the process’s address space ...
	- usually the text segment
* ... until a trap or interrupt
	- Time slice expires (timer interrupt)
	- Another event (e.g., interrupt from other device)
	- Exception (program error)
	- System call (switch to kernel mode)

### Program Creation System Calls
* *fork()*
	- Copy address space of parent and all threads
* *forkl()*
	- Copy address space of parent and only calling thread
* *vfork()*
	- Do not copy the parent’s address space
	- Share address space between parent and child
* *exec()*
	- Load new program and replace address space
	- Some resources may be transferred (open file descriptors)
	- Specified by arguments

### Process Creation with New Program
<img src="https://github.com/missystem/cis415review/blob/master/fork.png"><br />
* **Parent process** calls *fork()* to spawn child process
	- Both parent and child return from fork()
	- Continue to execute the same program
* **Child process** calls *exec()* to load a new program

### C Program Forking Separate Process
```
int main( )
{
	pid_t pid;
	// fork another process
	pid = fork( );
	if (pid < 0) { // error occurred
		fprintf(stderr, "Fork Failed"); exit(-1);
	} 

	else if (pid == 0) { /* child process */
		execlp("/bin/ls", "ls", NULL); // exec a file
	} 

	else { 		/* parent process */
		// parent will wait for the child to complete
		wait(NULL);
		printf ("Child Complete"); 
		exit(0);
	} 	
}
```
```
execl, execlp, execle, execv, execvp, execvpe
```
all execute a file and are frontends to execve <br />

### Process Layout
<img align="left" width="350" height="350" src="https://github.com/missystem/cis415review/blob/master/process_layout.png"> <br /><br />

1. PCB with new PID created 
2. Memory allocated for child initialized by copying over from the parent
3. If parent had called wait(), it is moved to a waiting queue 
4. If child had called exec(), its memory is overwritten with new code and data
5. Child added to ready queue and is
all set to go now
<br /><br />Effects in memory after parent calls fork()
<br /><br /><br /><br /><br />

### Analogy
* Doing a fork() is analogous to cloning
	- Copy is exactly the same
	- Completely able to live independently
	- Each has its own ability to execute
	- Each has its own resources
		- gets a copy of the parent’s process address space (unless you use another variant of fork())
		- gets a copy of the parent’s PCB (includes its memory management information)
	- Still executes the same code
* What if you want to change the clone’s behavior
* Doing an exec() is analogous to replacing the brain
	- Executing new program code

### Process Termination
* Process executes last statement and asks the OS to delete it (via *exit()*)
	- Output data from child to parent (via *wait()*)
	- Process' resources are deallocated by OS
* Parent may terminate execution of children processes (via *abort()*) if:
	- Child has exceeded allocated resources
	- Task assigned to child is no longer required
	- Parent is exiting
		- some OS do not allow child to continue if parent terminates
		- all children terminated - cascading termination

### Relocatable Memory
* Program instructions use addresses that logically start at 0
* Cannot place all programs in memory at physical address 0
* Relocation is the mechanism needed that enables the OS to place a program in an arbitrary location in memory
	- Gives the programmer the impression that they own the processor and the memory
* Program is loaded into memory at specific locations
	- Need some form of address translation (relocation) to do this
		- base-limit, segmentation, paging, virtual memory
* Also may need to share program code across processes

### Process State
* What do we need to track about a process?
	- How many processes?
	- What’s the state of each of them?
* How to track them?
	- Process table
		- Kernel data structure tracking processes on system
	- Process control block (PCB)
		- Structure for tracking process context

### Process Execution State
* As a process executes, it changes state
	- New:		  The process is being created (starting state)
	- Running:	  Instructions are being executed
	- Waiting:	  The process is waiting for some event to occur
	- Ready:      The process is waiting to run
	- Terminated: The process has finished execution (end state)
<img width="621" height="261" src="https://github.com/missystem/cis415review/blob/master/process_state.png">

### Process State
* consists of:
	- [Address space](https://github.com/missystem/cis415review/blob/master/lecturenotes03.md#process-address-space) (what can be addressed by a process)
	- Execution state (what is need to execute on the CPU)
	- Resources being used (by the process to execute)
* Address space contains code and data of a process
* Processes are individual execution contexts
	- Threads are also include here
* Resources are physical support necessary to execute
	- Memory: physical memory, address translation support
	- Storage: disk, files, ...
	- Processor: CPU (at least 1)

<img width="621" height="261" src="https://github.com/missystem/cis415review/blob/master/process_state.png">

### Process State 
* Running
	- Process is executing in the processor and in memory with all resources to run
* Ready
	- Process in memory with all resources to run, but is waiting for dispatch onto a processor
* Waiting
	- Process is not running, instead waiting for some event to occur

### State Transitions
* New Process ==> Ready
	- Allocate resources necessary to run
	- Place process on process queue (usually at end)
* Ready ==> Running
	- Process is at the head of process queue
	- Process is scheduled onto an available processor
* Running ==> Ready
	- Process is interrupted
		- usually by a timer interrupt
	- Process could still run, in that it is not waiting something
	- Placed back on the process queue

### State Transitions: Page Fault Handling
* Running ==> Waiting
	- Either something exceptional happened that caused an interrupt to occur (e.g., page fault exception) <br /> 
	or the process needs to wait on some action (e.g., it made a system call or requested I/O)
	- Process must wait for whatever event happened to be serviced
* Waiting ==> Ready
	- Event has been satisfied so that the process can return to run
	- Put it on the process (ready) queue
* Ready ==> Running

### State Transitions: Other Issues
* Priorities
	- Can provide policy indicating which process should run next
* Yield
	- System call to give up processor voluntarily
	- For a specific amount of time (sleep)
* Exit
	- Terminating signal (Ctrl-C)

### Process Control Block (PCB)
* also called the *task control block*
* contains information (metadata) associated with a specific process kept by the OS (kernel)
* including:
	- Process state:
		- new, ready, running, waiting, halted, etc
	- Program counter
		- indicates the address of the next executed process's instruction
	- CPU registers
		- for process thread
		- vary in number and type, depending on the computer architecture
	- CPU-scheduling information
		- includes a process priority, pointers to scheduling queues, and any other scheduling parameters
	- Memory-management information
		- memory allocated to the process
	- Accounting information
		- the amount of CPU and real time used, time limits, account numbers, job or process numbers, etc
	- I/O status information
		- the list of I/O devices allocated to the process
		- a list of open files, etc
* OS maintains a list of PCBs for all processes
* Process state lists link in the PCBs

<img width="254" height="392" src="https://github.com/missystem/cis415review/blob/master/PCB.png"> <img width="325" height="150" src="https://github.com/missystem/cis415review/blob/master/PCB_list.png">

### Per Process Control Information
* Process state
	- Ready, running, waiting (momentarily)
* Links to other processes
	- Children
* Memory Management
	- Segments and page tables
* Resources
	- Open files
* And much more...

### [*/proc*](https://github.com/missystem/cis415review/blob/master/lecturenotes02.md#sysfs-file-and-proc-files) File System
* Linux and Solaris
	- *ls /proc*
	- Process information pseudo-file system
	- Does not contain “real” files, but runtime system information
		- system memory
		- devices mounted
		- hardware configuration
	- A directory for each process
* A directory for each process
	- */proc/<pid>/io* <br />
		I/O statistics
	- */proc/<pid>/environ* <br />
		Environment variables (in binary)
	- */proc/<pid>/stat* <br />
		Process status and info

### Context Switch
* OS switches from one execution context to another
	- One process to another process
	- Interrupt handling
	- Process to kernel (mode transition, not context switch)
* Current process to new process
	- Save the state of the current process
		- PCB: describes the state of the process in the CPU
	- Load the saved context for the new process
		- load the new process’s process control block into OS and registers
	- Start the new process
* Does this differ if we are running an interrupt handler?
<img width="600" height="494" src="https://github.com/missystem/cis415review/blob/master/context_switch_process_to_process.png">

### Context Switch Performance
* No useful work is being done during a context switch <br />
	(i.e., no user process is running)
	- Want to speed up context switch processing
	- If a system call can be done in user mode, then the OS does not have to context switch
* Hardware support helps in context switching
	- Multiple hardware register sets
	- Be able to quickly set up the processor
* However, hardware optimization may conflict
	- Managing address translation tables is difficult
	- Different virtual to physical mappings on different processes

### Process Scheduling
* Maximize CPU use, quickly switch processes onto CPU for time sharing
* Process scheduler selects among available processes for next execution on CPU
* Maintains scheduling queues of processes
	- Job queue 
		- set of all processes in the system
	- Ready queue
		- set of all processes residing in main
	- Device queues
		- set of processes waiting for an I/O device
	- Processes migrate among the various queues

### Representation of Process Scheduling
* Process scheduling queueing diagram represents:
	- queues, resources, flows
* Processes move through the queues
<img width="534" height="304" src="https://github.com/missystem/cis415review/blob/master/process_scheduling.png">

### Ready Queue And Various I/O Device Queues
<img width="497" height="432" src="https://github.com/missystem/cis415review/blob/master/ready_IO_queues.png">

### Schedulers
* **Short-term scheduler** (process scheduler)
	- Selects which process should be executed next
	- Allocates CPU to running process
* **Medium-term scheduler** (multiprogram scheduler)
	- Manages process (jobs) in execution
	- Moves partially executed jobs to/from disk storage
	- Adjusts the degree of multiprogramming
* **Long-term scheduler** (job scheduler)
	- Selects which jobs should be allowed to run
	- Loads process and makes it ready to run

### Process Actions in Client-Server
* Example of forking to create a new process 
* Consider a web server <br />
<img width="116" height="60" src="https://github.com/missystem/cis415review/blob/master/process_actions_in_Client_Server_01.png"> <br />
* A remote “client” wants to connect and look at a webpage hosted by the web server <br />
<img width="300" height="150" src="https://github.com/missystem/cis415review/blob/master/process_actions_in_Client_Server_02.png"> <br />
* An interprocess communication is made and a connection is established <br />
<img width="300" height="150" src="https://github.com/missystem/cis415review/blob/master/process_actions_in_Client_Server_03.png"> <br />
* What if the web server only served this client until it was done looking at web pages?
* How can the web server support multiple “concurrent” clients?
* Create more “concurrency” by creating more processes! <br />
<img width="450" height="145" src="https://github.com/missystem/cis415review/blob/master/process_actions_in_Client_Server_04.png"> <br />
* A fork() copies the parent PCB and establish a child process initialized with that PCB
* All of the parent’s state is inherited by the child, including the connection to the client
* Responsibility for “servicing” the client is handed off to the “child” server process <br />
<img width="630" height="145" src="https://github.com/missystem/cis415review/blob/master/process_actions_in_Client_Server_05.png"> <br />
* The “parent” server process goes back to waiting for new clients
* The “parent” server should disengage from the client by releasing its (duplicate) connection <br />
<img width="450" height="150" src="https://github.com/missystem/cis415review/blob/master/process_actions_in_Client_Server_06.png"> <br />
* The “child” server is now completely responsible for the client connection
* There is still a logical relationship between the child / parent server processes
* Communication takes places between the client and “child” server process <br />
<img width="300" height="150" src="https://github.com/missystem/cis415review/blob/master/process_actions_in_Client_Server_07.png"> <br />
* The “parent” server process is not involved
* Now, this can procedure can continue as new client connections come in <br />
<img width="450" height="240" src="https://github.com/missystem/cis415review/blob/master/process_actions_in_Client_Server_08.png"> <br />
* In this way, the amount of “concurrency” increases with more server processes <br />
<img width="620" height="270" src="https://github.com/missystem/cis415review/blob/master/process_actions_in_Client_Server_09.png"> <br />
* It gives both logical isolation between the servers, as well as simplifies the design and functionality
* Many types of servers operate this way
* A objective is to avoid blocking in the server
	- If one server is stalled, others can run <br />
<img width="630" height="330" src="https://github.com/missystem/cis415review/blob/master/process_actions_in_Client_Server_10.png">

## Summary
* Process
	- Execution state of a program
* Process Creation 
	- fork and exec
	- From binary representation
* Process Description
	- Necessary to manage resources and context switch
* Process Scheduling
	- Process states and transitions among them 
* Interprocess Communication
	- Ways for processes to interact (other than normal files)

## Summary from book
* A **process** is a program in execution
	- the status of the current activity of a process is represented by the program counter, as well as other registers.
* The **layout of a process in memory** is represented by 4  sections:
	1. text
	2. data
	3. heap
	4. stack
* As a process executes, it changes **state** 
	- 4 general states of a process: 
		1. ready
		2. running
		3. waiting
		4. terminated
* **Process Control Block (PCB)**
	- the kernel data structure that represents a process in an OS
* **Process scheduler**
	- select an available process to run on a CPU
* An operating system performs a **context switch** when it switches from running one process to running another
* **Create processes**
	- fork() is used in Linux system
