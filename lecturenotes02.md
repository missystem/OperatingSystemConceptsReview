## [Lecutre 02: OS Structure and System Calls (Chapter 2)](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecture-2-structure.pdf)

## Outline
* Hardware and OS relationship
* [Operating System Services](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes02.md#operating-system-services)
* [System Calls](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes02.md#systems-calls)
* [Types of System Calls](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes02.md#types-of-system-calls)
* [System Programs](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes02.md#system-programs-system-services)
* [Operating System Design and Implementation](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes02.md#os-design-and-implementation)
* [Operating System Structure](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes02.md#operating-system-structure)
* [Building and Booting an Operating System](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes02.md#building-and-booting-an-operating-system)
* [Operating System Debugging](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes02.md#operating-system-debugging)
* [Summary](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes02.md#summary)


### Objectives
* To describe computer system organization
* To describe the services an operating system provides to users, processes, and other systems
* To discuss the various ways of structuring an operating system
* To explain how operating systems are installed and customized and how they boot

### Canonical System Hardware
* CPU
	- processor to perform computations
* Memory
	- hold instructions and data
* I/O devices
	- disk, monitor, network, printer
* Bus
	- systems interconnection for communication
<img src="https://github.com/missystem/cis415review/blob/master/canonical_system_hardware.png">

### CPU Architecture
* CPUs are semiconductor device with digital logic
	- Arithmetic Logical Unit (ALU)
	- Program Counter (PC) and registers
	- Instruction Set Architecture (ISA)
* Registers
	- part of ISA
	- CPU's scratchpads for program execution
	- fastest memory available in a computer system
	- instruction, data, address (n-bit architecture, 32-bit or 64-bit)
* Cache
	- fast memory (close to CPU)
	- faster than main memory, but more expensive
	- managed by the hardware (not seen by the OS)
* Clock to synchronizes constituent circuits

### Memory
* Random Access Memory (RAM)
	- Semiconductor DIMMs on PCB
	- Volatile Dynamic RAM (DRAM)
* Use as "main" memory for instruction and data
* OS manages main memory
	- It is a resource necessary for any code execution
* CPU fetches instruction and data from memory
	- into its instruction processing
	- into its cache / registers
* Memory controller implements logic for:
	- reading / writing to DRAM
	- refreshing DRAM to maintain contents
	- address translation circuitry for virtual memory

<img src="https://github.com/missystem/cis415review/blob/master/pc_motherboard.png">

### I/O Devices, Hard Disks, and SSD
*  Large variety, varying speeds, lot of diversity
	- Disk, tape, monitor, mouse, keyboard, NIC, ...
	- Serial or parallel interfaces
* Each device has a **controller**
	- Hides low-level details from OS (hardware interface)
	- Manages data flow between device and CPU/memory
* Hard disks are *secondary storage devices*
	- Mechanically operated with sequential access
	- Cheap (bytes / $), but slow
	- Orders of magnitude slower than main memory
* Solid state devices (SSD) are increasingly common

### Interconnects
* **bus** - hardware interconnect for supporting the exchange of data, control, signals, ...
	- Physicalspecification
	- Defined by a protocol
	- Data and control arbitration
* System bus - for CPU connection to memory primarily
	- Main bus going to CPU, Memory, data
	- Also to bridge to device buses
* PCI bus - for devices
	- Connects CPU-memory subsystem to:
		- fast devices
		- expansion bus that connects slow devices
* Other device "bus" types
	- SCSI, IDE, USB, ...
<br /> PCI adaptor -> I/O controller
<br /> yellow part -> I/O bus <br />
<img src="https://github.com/missystem/cis415review/blob/master/system_bus.png">

### Operating System Services
* Provides an environment for program execution
* For helping the user
	- User Interface (UI)
		- A window system with a mouse that serves as a pointing device to direct I/O, choose from menus, and make selections and a keyboard to enter text
		- Commonly, graphical user interface (GUI) is used
		- Mobile systems such as phones and tablets provide a touch-screen interface
		- command-line interface (CLI)
			- uses text commands and a method for entering them
	- Program execution (load, run, terminate)
	- I/O operations (file or device)
	- File-system manipulation (file, directories, permission)
	- Communications (same computer or several over network) 
	- Error detection (hardware and software)
* For ensuring the efficient operation of the system
	- Resource allocation (across multiple, concurrent jobs)
	- Logging (or Accounting) (how much and what kind of resources)
	- Protection and security (resources, users)
![Figure 2.1 A view of operating system services](https://github.com/missystem/cis415review/blob/master/figure2.1_view_of_OS_services.png)

### OS Services and Hardware Support
* Protection
	- Kernel / User mode and protected instructions
	- Base / Limit registers
* Scheduling
	- Timer
* System Calls
	- Trap instructions
* Efficient I/O
	- Interrupts
	- Memory-mapping
* Synchronization
	- Stomic Instructions
* Virtual memory
	- Translation Lookaside Buffer (TLB)

### Kernel / User Mode
* A modern CPU has at least two execution modes
	- prevent user programs from interfering with the proper operation of the system
	- indicated by status bit in a protected CPU register
	- OS kernel runs in privileged mode (also called *kernel mode* or *supervisor mode*)
	- Applications run in normal mode (i.e., not privileged)
* OS can switch the processor to user mode
	- CPU then can only access user process address space
	- Can not do other things as well, such as talk to devices
* Certain events need the OS to run
	- must switch the processor to privileged mode
	- e.g., I/O control, timer management, and interrupt management
* OS redefinition
	- Software that runs in privileged mode

### Protected Instructions
* Instructions that require privilege, such as:
	- Direct access to I/O
	- Modify page table pointers or the TLB
		- page: fixed sized block of data
	- Enable and disable interrupts
	- Halt the machine
* Access sensitive registers or perform sensitive
operations
* Only allow access in privilege modes
	- Otherwise, random programs may crash the machine

### Base and Limit Registers
* User processes must be protected from each other
* OS must be protected from user processes
* Memory referencing hardware to protect memory regions
	- Base register contains an address offset in physical memory (holds the smallest legal physical memory address)
	- Limit register is the maximum address that can be referenced (specifies the size of the range)
	- Both can be loaded only by the operating system before execution
* CPU checks each memory reference to make sure it is in proper range
* Both for instruction and data addresses
* Ensures references can not access other memory regions and corrupt memory

### Interrupts
* A key way in which hardware interacts with the operating system
* A hardware device triggers an interrupt by sending a signal to the CPU to alert the CPU that some event requires attention
* The interrupt is managed by the interrupt handler
* Interrupts make events that occur in the system visible for the OS needs to observe
* Interrupts can occur at any time
* OS polls for events (polling)
	- Inefficient use of resources
* OS is interrupted when events occur
	- I/O device has own logic (processor)
	- When device operation finishes, it pulls on interrupt bus
	- CPU “handles” interrupt

### Interrupt Vectoring
* Interrupts are asynchronous signals that indicate some need for attention by the OS
	- Replaces polling for events
* Represent:
	- Normal events to be noticed and acted upon
		- device notification
		- software system call
	- Abnormal conditions to be corrected (divide by 0)
	- Abnormal conditions that cannot be corrected (disk failure)
* Interrupt vectors are used to decide what to do with different interrupts
	- Address where interrupt routines live in the OS for different interrupt types
 <img src="https://github.com/missystem/cis415review/blob/master/figure1.5_Intel_processor_event_vector_table.png">

### Hardware Interrupts
* Signal from a device
	- Implemented by a controller (e.g., memory)
* Examples
	- Timer (clock)
	- keyboard, mouse
	- end of DMA transfer
* Response to processor request
* Unsolicited response
	- asynchronous
<img src="https://github.com/missystem/cis415review/blob/master/logical_diagram_of_interrupt_routing.png">

### Timer 
* OS needs timers for:
	- Time of day, CPU scheduling, ...
	- There can be multiple timers for different things
* Based on a hardware clock source (e.g., 1 Ghz)
	- Count-down timer (e.g., 1 Ghz to 1 Khz)
	- Generates an interrupt at 0 (e.g., every 1 msec)
* Use timers to ensure that certain future events occur
* Most importantly, it is a way for OS to regain control
* User programs can set up their own “software” timers

### Software Interrupts
* Software interrupts (traps)
	- Special interrupt instructions
		- int is the interrupt instruction
		- int 0x80 passes control to interrupt vector 0x80
	- Exceptions
		- can be fixed (e.g., page fault)
		- cannot be fixed (e.g., divide by zero)
* All invoke OS, just like a hardware interrupt
	- Trap starts running OS code in supervisor access space
	- This space can not be overwritten by the user program

### How Does a Process Run? (high level view)
* OS keeps track of which process is assigned to which sections in memory along with other details
* For a new process to run, memory is assigned by the OS, which puts the code in that location
	- switch to user mode
	- start running at first address of the program
* OS keeps record of every process
	- This is the *process context*
	- Assigned memory, current Program Counter, ...
	- Enough info to restart process where it left off

### Then along comes an interrupt ...
1. A hardware interrupt or a trap (software interrupt) happens
	- Example: received input from keyboard
2. OS records state of running process’s context
	- stored in a Process Control Block (PCB)
3. OS services the interrupt
	- Example: send something to the printer
4. OS picks process to restart
	- which process is picked depends on scheduling
	- moves back into user mode

### Interrupt Handling
* Each interrupt has an *interrupt handler*
* When an interrupt request (IRQ) is received
	- If interrupt mask allows interrupt, then ...
		1. save state of current process
			- at time of interrupt something else may be running
			- state: Registers (stack pointer), program counter, ...
		2. execute handler
		3. return to current process or another process

Interrupt (Trap) Handlers
<img src="https://github.com/missystem/cis415review/blob/master/interrupt_handler.png">

Multiple Interrupts
<img src="https://github.com/missystem/cis415review/blob/master/multiple_interrupts.png">

### Device Access
* Port I/O
	- uses special I/O instructions
	- uort number, device address (not process address)
* Memory-mapped I/O
	- uses memory instructions (load/store)
		- memory-mapped device registers
	- does not require special instructions <br />
[<img src="https://github.com/missystem/cis415review/blob/master/isolated_vs_memory_mapped_I:O.png">](http://ece-research.unm.edu/jimp/310/slides/8086_IO1.html)
IORC: I/O Read Control <br />
IOWC: I/O Write Control

### Direct Memory Access (DMA)
* Direct access to I/O controller through memory
* Reserve area of memory for communication with
the I/O device
	- Video RAM
		- CPU writes frame buffer
		- video card displays it
	- Network interfaces
* DMA is efficient and convenient and fast
	- Multiple components can talk to other components concurrently, rather than competing for cycles on a shared bus
	- After setting up buffers, pointers, and counters for the I/O device, the device controller transfers an entire block of data directly to or from the device and main memory, with no intervention by the CPU
	- Only one interrupt is generated per block, to tell the device driver that the operation has completed, rather than the one interrupt per byte generated for low-speed devices
	- While the device controller is performing these operations, the CPU is available to accomplish other work
<img src="https://github.com/missystem/cis415review/blob/master/figure1.7_how_modern_computers_work.png">

### Synchronization
* How can OS synchronize concurrent processes?
	- multiple threads, processes, interrupts, DMA
* CPU must provide mechanism for atomicity
	- Series of instructions that execute as one or not at all
* One approach:
	- Disable interrupts, perform action, enable interrupts
	- Advantages:
		- Requires no hardware support
		- Conceptually simple
	- Disadvantages:
		- Could cause starvation
* A Modern Synchronization Approach
	- Use hardware support for atomic instructions
		- Small set of instructions that cannot be interrupted
	- Examples:
		- **Test-and-set (TST)** <br />
		if word contains given value, set to new value
		- **Compare-and-swap (CAS)** <br />
		if word equals value, swap old value with new
		- Intel: LOCK prefix (XCHG, ADD, DEC, ...)
	- Used to implement locks 

### Process Address Space
* Process Address Space is all locations that are addressable by the process
* Every running program can have its own private address space
* Can restrict use of addresses so as to isolate different area
	- Restrictions are enforced by OS
	- Text section
		- read only program instructions are stored in
		- the executable code
	- Data section
		- hold the data for the running process (read/write)
		- global variables
	- Heap section
		- memory that is dynamically allocated during program run time
		- allows for dynamic data expansion
	- Stack section
		- temporary data storage when invoking functions <br />
		(such as function parameters, return addresses, and local variables)
	- the sizes of the text and data sections are fixed, as their sizes do not change during program run time <br />
	the stack and heap sections can shrink and grow dynamically during program execution
<img src="https://github.com/missystem/cis415review/blob/master/figure3.1_layout_of_a_process_in_memory.png">

### Virtual Memory
* Provide the illusion of infinite memory
* OS loads pages from disk as needed
	- page: fixed sized block of data
* Benefits
	- Allows the execution of programs that may not fit entirely in memory
* OS needs to maintain mapping between physical and virtual memory
	- Page tables stored in memory

### Address Translation Hardware
* Early virtual memory systems used to do translation in software
	- meaning the OS did it
	- an additional memory access for each memory access
* Address translation hardware solved this problem
	- Translation Look-aside Buffer (TLB)
* Modern CPUs contain TLB hardware
	- Fmall, fast-lookup hardware cache
	- Modern workloads are TLB-miss dominated (page number is not in the TLB)

#### Takeaway
* Modern architectures provide lots of features to help the OS do its job
	- Protection mechanisms (modes)
	- Interrupts
	- Device I/O
	- Synchronization
	- Virtual Memory (TLB)
* Otherwise impossible or impractical in software

### Operating System Layers
* Application
* Libraries (in application process)
* System Services
* OS API
* OS Kernel
* Hardware
<img src="https://github.com/missystem/cis415review/blob/master/unix_architecture.png"> 

### Applications to Libraries
* Application Programming Interface (API)
* Libraries (e.g., *libc*)
* Library routines (e.g., *printf()* of stdio.h)
* All within the process's address space
	- Statically linked
		- libraries are included as part of the application code
		- calls are resolved at compile time
	- Dynamically linked
		- libraries are loaded by the OS at execution time as needed
		- jump tables and pointers are resolved dynamically by linker

### Application to (System) Services
* Provide syntactic sugar for using resources
	- Printing
	- Program management
	- Network management
	- File management
* Provide special functions beyond OS
* [UNIX man pages](http://man7.org/linux/man-pages/man7/man-pages.7.html), sections 1 and 8
* Command line system programs

### Libraries to System Routines
* System call interface
	- [UNIX man pages](http://man7.org/linux/man-pages/man7/man-pages.7.html), sections 2
	- Examples:
		- open(), [read()](http://man7.org/linux/man-pages/man2/read.2.html), [write()](http://man7.org/linux/man-pages/man2/write.2.html) – defined in unistd.h
	- Call these via libraries? 
		- [fopen()](http://man7.org/linux/man-pages/man3/fopen.3.html) vs. [open()](http://man7.org/linux/man-pages/man2/open.2.html)
* Special files
	- Drives
	- [*/proc*](http://man7.org/linux/man-pages/man5/proc.5.html)
	- [*sysfs*](http://man7.org/linux/man-pages/man5/sysfs.5.html)

### System to Hardware
* Software-Hardware interface
* OS Kernel functions
	- Concepts		<=> Managers (hardware)
	- Files 		<=> File systems (drivers and devices)
	- Address space <=> Virtual memory (memory)
	- Programs 		<=> Process model (CPU, ISA)
* OS provides abstractions of devices and hardware objects
	- These abstractions are represented in software running in the OS and data structures that it maintains

### Systems Calls
* Programming interface to OS services via system libraries
* Typically written in a high-level language (C, C++) 
* Mostly accessed by programs via a high-level Application Programming Interface (API) (versus direct system call use)
	- Win32 (Windows), POSIX (Unix, Linux, MacOS), Java (JVM)
* Typically, a number is associated with each system call
	- System-call interface maintains a table indexed by call #
* System call interface invokes the intended system call in OS kernel and returns status of the system call and return values
* Caller just obeys API and understand what OS will do
	- Most details of OS interface hidden by API

### Types of System Calls
* **Process control**
	- create process, terminate process
	- load, execute
	- get process attributes, set process attributes
	- wait event, signal event
	- allocate and free memory
* **File management**
	- create file, delete file
	- open, close
	- read, write, reposition
	- get file attributes, set file attributes
* **Device management**
	- request device, release device
	- read, write, reposition
	- get device attributes, set device attributes 
	- logically attach or detach devices
* **Information maintenance**
	- get time or date, set time or date
	- get system data, set system data
	- get process, file, or device attributes
	- set process, file, or device attributes
* **Communications**
	- create, delete communication connection
	- send, receive messages
	- transfer status information
	- attach or detach remote devices
* **Protection**
	- get file permissions
	- set file permissions

#### The handling of a user application invoking the open() system call
<img src="https://github.com/missystem/cis415review/blob/master/figure2.6_handling_of_user application_invoking_open().png"> 

#### Standard C Library Example (printf())
<img src="https://github.com/missystem/cis415review/blob/master/printf().png"> 

#### System Call Example getpid()
<img src="https://github.com/missystem/cis415review/blob/master/getpid().png"> 

### System Call Handling
<img src="https://github.com/missystem/cis415review/blob/master/system_call_handling.png"> 

### System Call Process
1. Procedure call in user process
2. Initial work in user mode
3. Trap instruction to invoke kernel
4. Preparation
5. I/O command
6. Wait
7. Completion interrupt handling
8. Return-from-interrupt instruction
9. Final work in user mode
10. Ordinary return to user code

<ins>system operations</ins><br />
libc <br />
int 0x80 <br />
sys_read, mmap2 <br />
read from disk <br />
disk is slow <br /><br /><br />
libc

### Details on x86 / Linux
* A more accurate picture:
	- Consider a typical Linux process
	- Its “thread of execution” can be several places
		- in your program’s code
		- in glibc, a shared library containing the C standard library, POSIX support, and more
		- in the Linux architecture - independent code
		- in Linux x86-32/x86-64 code <br />
<img src="https://github.com/missystem/cis415review/blob/master/system_execution_detail_01.png"> <br />

* Some routines your program invokes may be **entirely handled by glibc**
	- Without involving the kernel
		- e.g., strcmp( ) from stdio.h
	- some initial overhead when invoking functions in dynamically linked libraries ...
	- ... but after symbols are resolved, invoking glibc routines is nearly as fast as a function call within your program itself <br />
<img src="https://github.com/missystem/cis415review/blob/master/system_execution_detail_02.png"> <br />

* Some routines may be **handled by glibc, but they in turn invoke Linux system calls**
	- Example: POSIX wrappers around Linux syscalls
		- POSIX *readdir()* --> invokes the underlying Linux *readdir()*
	- Example: C stdio functions that read and write from files
		- *fopen(), fclose(), fprintf()*, ... --> invoke underlying Linux *open(), read(), write(), close()*, ... <br />
<img src="https://github.com/missystem/cis415review/blob/master/system_execution_detail_03.png"> <br />

* Your program can choose to **directly invoke Linux system calls** as well
	- Nothing forces you to link with glibc and use it
	- But relying on directly invoked Linux system calls may make your program less portable across UNIX varieties <br />
 <img src="https://github.com/missystem/cis415review/blob/master/system_execution_detail_04.png"> <br />

### System Programs (System Services)
* System programs provide a convenient environment for program development and execution
* They can be divided into categories: 
	<!-- p75 on the book-->
	- File manipulation (management)
	- Status information
	- File modification
	- Programing-language support (program development)
	- Program loading and execution
	<!-- - Application programs -->
	- Communications
	- Background services
* Most user’s view of the operating system is defined by system programs, not the actual system calls

### File Interface
* Goal:
	- Provide a uniform abstraction for accessing the OS and its resources
* Abstraction
	- File
* Use file system calls to access OS services
	- Devices, sockets, pipes, etc
	- Also use in OS in general

### I/O with System Calls
* Much I/O is based on a streaming model (sequence of bytes)
	- *write()* sends a stream of bytes somewhere
	- *read()* blocks until a stream of input is ready
* Annoying details:
	- Might fail, can block for a while
	- Working with file descriptors
	- Arguments are pointers to character buffers
	- See [read()](http://man7.org/linux/man-pages/man2/read.2.html) and [write()](http://man7.org/linux/man-pages/man2/write.2.html)

### File Descriptors (int fd)
* A process might have several different I/O streams in use at any given time
	- These are specified by a *file descriptor* (a kernel data structure)
		- Each process has its own table of file descriptors
	- [*open()*](http://man7.org/linux/man-pages/man2/open.2.html) **associates** a file descriptor with a file
	- [*close()*](http://man7.org/linux/man-pages/man2/close.2.html) **destroys** a file descriptor
* stdin and stdout are usually associated with a terminal

### Regular File
* File has a *pathname* (e.g.: /tmp/foo)
* Can open the file
	- *int fd = open( “/tmp/foo”, O_RDWR )*
		- *O_RDWR* - flags for read/write access
	- for reading and writing
* Can read from and write to the file
	- *bytes = read( fd, buf, max ); /\* buf get output \*/*
	- *bytes = write( fd, buf, len ); /\* buf has input \*/*
		- *buf* - pointer to buffer

### Socket File
* File has a *pathname* (e.g.: /tmp/bar)
	- Files provide a persistence for a communication channel
	- Usually used for local communication (UNIX domain sockets)
* Open, Read, and Write via socket operations
	- *sockfd = socket( AF_UNIX, TCP_STREAM, 0 );*
	- *local.path* is set to */tmp/bar*
	- *bind ( sockfd, &local, len )*
	- Use sock operations to read and write

### Device File
* Files for interacting with physical devices
	- */dev/null* (do nothing)
	- */dev/cdrom* (CD-drive)
* Use file system operations, but are handled in device-specific ways
	- *open(), read(), write()* correspond to device-specific functions (act as function pointers!)
	- Also, use *ioctl* (I/O control) to interact

### *sysfs* File and */proc* Files
* These files enable reading from and writing to kernel
* [*/proc*](http://man7.org/linux/man-pages/man5/proc.5.html) files
	- Enable reading of kernel state for a process
	- Process information pseudo-file system
	- Does not contain "real" files, but runtime system information
		- system memory
		- devices mounted
		- hardware configuration
	- A lot of system utilities are simply calls to files in this directory
	- By altering files located here you can even read/change kernel parameters (sysctl) while the system is running
* [*sysfs*](http://man7.org/linux/man-pages/man5/sysfs.5.html) files
	- Provide functions that update kernel data
		- file's write function updates kernel based on input data

### Other System Calls
* Hook the output of one program into the input of another
	- *pipe()*
* Block until one of several file descriptor streams is ready
	- *select()*
* Special calls for dealing with network
	- AF_INET sockets, etc
* Send a message to other (or all) processes
	- *signal()*
* Most of these in [section 2](http://man7.org/linux/man-pages/dir_section_2.html) of Unix/Linux manual

### Syscall Functionality
* System calls are the main interface between processes and the OS
	- Like an extended “instruction set” for user programs that hide many details
	- First Unix system had a couple dozen system calls
	- Current systems have many more <br />
		(>300 in Linux and >500 in FreeBSD)

### OS Design and Implementation
* Design and implementation of OS is not “solvable”
	- There are different types for meeting different objectives
	- Some approaches have proven successful
* Internal structure of different OSes can vary widely
* Start the design by defining goals and specifications
* The design of the system will be affected by the choice of hardware and the type of system
* The requirements can be divided into two groups:
	- **user goals**
		- OS should be convenient to use, easy to learn, reliable, safe, and fast
	- **system goals**
		- OS should be easy to design, implement, and maintain, as well as flexible, reliable, error-free, and efficient

### OS Policy vs Mechanism
* Important principle to separate
	- Policy: 	 What will (should) be done?
		- e.g., deciding how long the timer is to be set for a particular user is a policy decision
	- Mechanism: How to do it?
		- e.g., the timer construct is a mechanism for ensuring CPU protection
* The separation of policy and mechanism is important
	- Allows maximum *flexibility* if policy decisions are to be changed later
	- Universal principle
* Specifying and designing an OS is a highly creative task of software engineering
* Policy decisions are important for all resource allocation. 
	- Whenever it is necessary to decide whether or not to allocate a resource, a policy decision must be made. 
	- Whenever the question is how rather than what, it is a mechanism that must be determined.

### Operating System Structure
* Common approach
	- partition the task into small components, or modules
* Monolithic Structure (approach)
	- a common technique for designing operating systems
	- no structure at all
		- place all of the functionality of the kernel into a single, static binary file that runs in a single address space 
	- Tightly coupled system
		- changes to one part of the system can have wide-ranging effects on other parts <br />
<img src="https://github.com/missystem/cis415review/blob/master/figure_2.12_traditional_UNIX_system_structure.png"> <br />
* Layered Approach 
	- Loosely coupled system
		- divided into separate, smaller components that have specific and limited functionality
	- Advantage
		- changes in one component affect only that component, and no others
		- allowing system implementers more freedom in creating and changing the inner workings of the system 
		- avoiding the problems of layer definition and interaction
	- Each layer is implemented only with operations provided by lower-level layers <br />
<img src="https://github.com/missystem/cis415review/blob/master/figure2.14_a_layered_OS.png"><br />
* Microkernels
	- removing all nonessential components from the kernel and implementing them as user-level programs that reside in separate address spaces
	- smaller kernel 
	- minimal process and memory management 
	- Benefits:
		- makes extending the operating system easier
		- easier to port from one hardware design to another
		- more security and reliability 
	- Disadvantages:
		-  the performance of microkernels can suffer due to increased system-function overhead
		- the OS may have to switch from one process to the next to exchange the messages <br />
<img src="https://github.com/missystem/cis415review/blob/master/architecture_of_typical_microkernel.png"><br />
* **Modules**
* Hybrid Systems

### Building and Booting an Operating System
* Operating-System Generation
	1. Write the operating system source code (or obtain previously written source code).
	2. Configure the operating system for the system on which it will run.
	3. Compile the operating system.
	4. Install the operating system.
	5. Boot the computer and its new operating system.
* System Boot
	1. A small piece of code known as the bootstrap program or boot loader locates the kernel.
	2. The kernel is loaded into memory and started.
	3. The kernel initializes hardware.
	4. The root file system is mounted.

### Operating-System Debugging
* Failure Analysis
* Performance Monitoring and Tuning
	- Counters
* Tracing
* BCC (BPF Compiler Collection)
	- a rich toolkit that provides tracing features for Linux systems


### Summary
* Operating systems must balance many needs
	- Job perspective: 
		- give the impression that each process has individual use of system
	- System perspective:
		- Comprehensive management of computer machine resources
* Operating system structures try to make use of system resources straightforward
	- Libraries
	- System services
	- System calls and other interfaces

* An operating system provides an environment for the execution of programs by providing services to users and programs.
* 3 primary approaches for interacting with an operating system
	1. command interpreters
	2. graphical user interfaces
	3. touch- screen interfaces.
* Systemcalls provide an interface to the services made available by an operating system. Programmers use a system call’s application programming interface (API) for accessing system-call services.
* System calls: 6 major categories:
	1. process control
	2. file management
	3. device management
	4. information maintenance
	5. communications
	6. protection.
* The standard C library provides the system-call interface for UNIX and Linux systems.
* OS include a collection of system programs that provide utilities to users
* Linker
	- combines several relocatable object modules into a single binary executable file. 
* Loader 
	- loads the executable file into memory, where it becomes eligible to run on an available CPU.
* Several reasons why applications are operating-system specific.
		- different binary formats for program executables
		- different instruction sets for different CPUs
		- system calls that vary from one operating system to another
* An operating system is designed with specific goals in mind 
	- These goals ultimately determine the operating system’s policies. 
	- An operating system implements these policies through specific mechanisms.
* Monolithic OS 
	- has no structure
	- all functionality is provided in a single, static binary file that runs in a single address space. 
	- disadvantage: difficult to modify
	- primary benefit: efficiency
* Layered OS
	- divided into a number of discrete layers, 
	- bottom layer - hardware interface 
	- highest layer - user interface
	- layered software systems have had some success, 
	- generally not ideal for designing operating systems
		- performance problems
* Microkernel approach
	- uses a minimal kernel
	- most services run as user-level applications
	- Communication takes place via message passing
* Modular approach
	- provides OS services through modules that can be loaded and removed during run time
* Hybrid
	- Many contemporary operating systems are constructed as hybrid systems using a combination of a monolithic kernel and modules.
* Boot loader
	- loads an OS into memory
	- performs initialization
	- begins system execution
* The performance of an OS can be monitored using either counters or tracing. 
	- Counters 
		- a collection of system-wide or per-process statistics
	- Tracing 
		- follows the execution of a program through the operating system










