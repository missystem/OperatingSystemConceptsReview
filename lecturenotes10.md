# Virtual Memory

## Outline
* [Background](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes10.md#background)
* [Demand Paging]()
* [Copy-on-Write]()
* [Page Replacement]()
* [Allocation of Frames]()
* [Thrashing]()
* [Memory-Mapped Files]()
* [Kernel Memory Allocation]()
* [Other Considerations]()
* [Operating-System Examples]()

---

### Background
* Code needs to be in memory to execute, but entire program rarely used
	-  Error code, unusual routines, large data structures
* Entire program code not needed at same time
* Consider ability to execute partially-loaded program 
	- Program no longer constrained by physical memory limits
	- Each program takes less memory while running
		- more programs run at the same time
		- increased CPU utilization and throughput with no increase in response time or turnaround time
	- Less I/O needed to load or swap programs into memory results in each user program running faster


### Virtual Memory
* Separation of logical memory from physical memory
	-  If you can figure out how to have only part of the program needs to be in memory for execution ...
	- ... then can separate the logical address space from physical address space and it can be larger
* Benefits
	- Can have programs with memory requirements much larger than physical memory
	- Allows address spaces to be shared by several processes
	- Allows for more efficient process creation
	- More programs running concurrently
	- Less I/O needed to load or swap processes
* We call this “memory” virtual memory to distinguish from physical memory


### Virtual Address Space (Logical Address Space)
* Each process has a logical address space
	- Usually start at address 0, contiguous addresses until end of space defined by # bits in the address
* *virtual address space* of a process == *logical address space*
* Memory management problem is the same
	- Memory management unit (MMU) must map logical address space to physical address space
	- Look at our memory management mechanisms
* Difference from our discussion before
	- To meet the needs of memory requested by processes
	- Allow the amount of logical memory space used to be greater than the size of the physical memory
	- Allow a process to run without all its memory in physical memory
* Usually design logical address space for stack to start at Max logical address and grows “down” while heap grows “up”
	- Maximizes address space use
	- 
	<img width="206" height="262" src="https://github.com/missystem/cis415review/blob/master/figure3.1_layout_of_a_process_in_memory.png">


### Demand Paging 
* Could bring entire process into memory at load time
* Or bring a page into memory only when it is needed
	- Less I/O needed, no unnecessary I/O
	- Less memory needed
	- Faster response
	- More users
* Similar to paging system with swapping (diagram on right)
* Page is needed (the page you're referencing) => reference to it
	- invalid reference => abort
	- not-in-memory => bring to memory
* Lazy swapper
	- never swaps a page into memory unless page will be needed
	- Swapper that deals with pages is a pager

### Basic Concepts
* With swapping, pager guesses which pages will be used before swapping out again
* Instead, pager brings in only those pages ...




### (p13)

### When some pages are not in main memory (p15)


### Performance of Demand Paging
* 3 major activities
	1. Service the interrupt - careful coding means just several hundred instructions needed

* 
*  Effective Access Time (EAT)
	- EAT = (1 - p) * memory access + p * (page fault overhead + swap out + swap in + memory access)





### What happens if there is no free frame? (p27)
* Used up by process pages
* Also in demand from the kernel, I/O buffers, etc
* How much to allocate to each?
* *Page replacement* - find some page in memory, but not really in use, page it out
	- Algorithm - terminate? swap out? replace the page?
	- Performance - want an algorithm

* want a small page fault rate, 


### Page Replacement
* Prevent over-allocation of memory by modifying page-fault service routine to include page replacement
* Use modify (dirty) bit to reduce overhead of page transfers 
	- only modified pages are written to disk
* Page replacement completes separation between logical memory and physical memory 
	- large virtual memory can be provided on a smaller physical memory
* if the page we decided to replace is dirty -> we have to write to disk
* 

### Need for page replacement


### Basic Page Replacement
* Find the location of the desired page on diskqFind a free frame:
	- If there is a free frame, use it
	- If there is no free frame, use a page replacement algorithm to select a *victim frame*
	- Write victim frame to disk if dirty
* Bring  the desired page into the (newly) free frame; <br />update the page and frame tables
* Continue the process by restarting the instruction that caused the trap
* Note now potentially 2 page transfers for page fault –
	- increasing EAT


### Page Replacement
* identify victim -> 
<img src="">


### Page and Frame Replacement Algorithms
* How do we decide which page and frame to replace -> we need an algorithm
* Frame-allocation algorithm determines
	- How many frames to give each process
	- Which frames to replace
* Page-replacement algorithm
	- Want lowest page-fault rate on both first access and re-access
* 


### Page Faults Versus The Number of Frames
* Larger number of frame, less number of page faults


### First-In-First-Out (FIFO) Algorithm
* (first page that brought in is the first one kicked out)
* Reference string: 7,0,1,2,0,3,0,4,2,3,0,3,0,3,2,1,2,0,1,7,0,1 
* 3 frames (3 pages can be in memory at a time per process)
<img src="">
* Can vary by reference string: consider 1,2,3,4,1,2,5,1,2,3,4,5❍
	- Adding more frames can cause more page faults
		- Belady’s anomaly
* How to track ages of pages? 
	- Just use a FIFO queue


### FIFO Illustrating Belady’s Anomaly
* Increase \# of frames -> increase \# of page faults


### Optimal Algorithm
* Replace page that will not be used for longest period of time
	- 9 is optimal for the example
* How do you know this?
	- Can not read the future
* Used for measuring how well your algorithm performs
<img src="">


### Least Recently Used (LRU) Algorithm
* Use past knowledge rather than future
* Replace page that has not been used in the most amount of time
* Associate time of last use with each page
<img src="">
* 12 faults –better than FIFO but worse than OPT
* Generally good algorithm and frequently used
* But how to implement?
	- 


### LRU Algorithm
* Stack implementation
	- 

* Counter implementation
	- 

* LRU and OPT are cases of stack algorithms that don’t have Belady’s Anomaly


### 
* using stack to implement the LRU:
<img src="">


### LRU Approximation Algorithms
* LRU needs special hardware and still slow
* Reference bit
	- With each page associate a bit, initially = 0
	- When page is referenced bit set to 1
	- Replace any with reference bit = 0 (if one exists)
		- we do not know the order, however
* Second-chance algorithm
	- Generally FIFO, plus hardware-provided reference bit
	- Clock replacement
	- If page to be replaced has 
		- reference bit = 0 -> replace it
		- reference bit = 1 then:
			- set reference bit 0, leave page in memory
			- replace next page, subject to same rules


### Allocation of Frames
* Each process needs minimum number of frames
* Example:  IBM 370 –6 pages to handle SS MOVE instruction:
	- instruction is 6 bytes, might span 2 pages
	- 2 pages to handle from
	- 2 pages to handle to
* Maximum of course is total frames in the system
* Two major allocation schemes
	- fixed allocation
	- priority allocation
* Many variations


### Fixed Allocation
* Equal allocation 
	- For example, if there are 100 frames (after allocating frames for the OS) and 5 processes, give each process 20 frames
	- Keep some as free frame buffer pool

* Proportional allocation 
	- Allocate according to the size of process
	- Dynamic as degree of multiprogramming, process sizes change
* ...


### 






















