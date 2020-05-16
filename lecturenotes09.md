

## Outline
* Background
* Swapping
* Contiguous Memory Allocation q Segmentation
* Paging
* Structure of the Page Table
* Examples
	- The Intel 32 and 64-bit Architectures 
	- ARM Architecture

---

### Objectives
* To provide a detailed description of various ways of organizing memory hardware
* To discuss various memory-management techniques, including paging and segmentation
* To give a detailed example pure segmentation and segmentation with paging

### Background
* Program must be brought (from external storage) into memory and “loaded” in a process for it to be run
	- A program goes through several steps before producing an executable code image that the OS uses to create a process
* A program starts with “logical” addresses that are in the “logical” address space its process
	- Instructions produce addresses with respect to logical addresses (also called “virtual” addresses)
	- Need to bind (map between) “logical” addresses and “physical” addresses in memory
* All this depends on both software and hardware
	- Software includes compilers, loaders, and OS
	- Hardware include memory system, address translation, ...

### Address Binding
* Addresses represented in different ways at different stages of a program’s life
	- Need to bind one address to another
	- Source code addresses usually symbolic (i.e., names)
	- Compiled code has addresses that need to bind to relocatable addresses
	- Linker or loader will bind relocatable addresses to absolute addresses
	- Each binding maps one address space to another
* All of these addresses are “logical” adddresses

### Memory and Address Binding
* Main memory and CPU registers are the only storage directly addressable by a program
	- CPU registers are accessed in one CPU clock (or less)
	- Memory system sees addresses from read/write operations
* Addresses used in CPU instructions are logical addresses
	- Reference the logical address space of a process
	- These logical addresses must be “translated” to physical addresses
* What does address translation (address binding)?
	- Compile time: If a memory location is known a priori , code can be generated with absolute addresses, but needs to be recompiled if the location changes
	- Load time: If a memory location is known relative to an address, code can be generated with relocatable addresses
	- Execution time: addresses are mapped dynamically during execution using memory management hardware






















