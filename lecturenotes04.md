## [Lecture 04: Interprocess Communication (Chapter 3)](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecture-4-ipc.pdf)

## Outline
* [Interprocesses Communication (IPC)](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes04.md#interprocess-communication-ipc)
* [Sockets](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes04.md#sockets)
* [Remote Procedure Calls (RPC)](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes04.md#remote-procedure-calls-rpc)
* [Summary](https://github.com/missystem/OperatingSystemConceptsReview/blob/master/lecturenotes04.md#summary)


### Process Communication
* Process cooperation is a fundamental aspect of an OS and of the computing environment it is maintaining on behalf of executing applications
* Processes need to interact and share information
* Process model is a useful way to isolate running programs <br />
(separate resources, state, and so on)
	- It can simply programs (no need to worry about other processes)
	- But processes do not always work in isolation, nor do we want them to
	- Sometimes it is easier to design programs if there are multiple processes working together
* Why provide environment that allows process cooperation?
	- Information sharing
	- Computation speedup
	- Modularity

### Process Communication (Interoperation)
* When is communication necessary?
* Examples in OS:
	- Kernel/OS access to user process data
	- Processes sharing data via shared memory
	- Processes sharing data via system calls
	- Processes sharing data via file system
	- Threads with access to same data structures
* In general, there are numerous examples in computer science where interoperation is important
	- DB transactions, distributed computing, parallelism


### Interprocess Communication (IPC)
* Mechanism for processes to communication and synchronize
* Logically, we want some sort of messaging system
* Logically, an IPC facility would provide 2 operations for processes to communicate:
	- Send(message) (message size fixed or variable)
	- Receive(message)

* However, first there needs to be a communication channel
	- If processes P and Q wish to communicate they open a channel
	- Then exchanging messages can proceed

* How is the communication channel (link) realized?
	- Physically, there are different alternatives
	- Logically, need abstract interfaces, protocols, and properties

### IPC Mechanisms
* Allow cooperating processes to exchange data
* 2 fundamental methods (OS supported):
	- Shared memory
		- Pipes, Shared buffer
		- A region of memory that is shared by the cooperating processes is established
		- Processes can then exchange information by reading and writing data to the shared region
		- Faster than message passing
	- Message passing
		- Mailboxes, Sockets
		- Communication takes place by means of messages exchanged between the cooperating processes
		- Useful for exchanging smaller amounts of data (no conflicts need be avoided)
		- Easier to implement in a distributed system than shared memory
* Choosing depends on the application
<img width="553" height="353" src="https://github.com/missystem/cis415review/blob/master/communication_models.png">

### IPC in Shared-Memory Systems
* Producer-Consumer Problem
	- Producer writes
	- Consumer reads <br />
<img width="345" height="90" src="https://github.com/missystem/cis415review/blob/master/producer_consumer.png"> <br />
* Producer action
	- While the buffer not full ... stuff can be written (added) to the buffer
* Consumer actions
	- When stuff is in the buffer ... it can be read (removed)
* Must manage where new stuff is in the buffer
* 2 types of buffers
	- Unbounded buffer (Realistic?)
		- Variable in size (no practical limit on the size of the buffer)
		- The consumer may have to wait for new items
		- The producer can always produce new items
	- Bounded buffers (Problems?)
		- Fixed in size
		- The consumer must wait if the buffer is empty
		- The producer must wait if the buffer is full

### Shared Memory -- Producer
```
item next_produced;			
							// Circular buffer
while (true) {
	/* produce an item in next produced */

	while (((in + 1) % BUFFER_SIZE) == out) 
		; /* do nothing ... buffer is full */

	buffer[in] = next_produced;
	in = (in + 1) % BUFFER_SIZE;
}
```

### Shared Memory -- Consumer
```
item next_consumed;

while (true) {
	while (in == out)
        ; /* do nothing ... buffer is empty */
	
	next_consumed = buffer[out]; 
	out = (out + 1) % BUFFER_SIZE;

	/* consume the item in next_consumed */
}
```

### IPC with Shared Memory
* Communicate by reading/writing from a specific memory location (in logical memory)
	- Setup a shared memory region (segment) in your process
	- Permit others to attach to the shared memory region
* [*shmget(2)*](http://man7.org/linux/man-pages/man2/shmget.2.html) -- create shared memory segment 
	- Permissions (read and write)
	- Size
	- Returns an identifier for the segment
* [*shmat(2)*](http://man7.org/linux/man-pages/man2/shmat.2.html) -- attach to existing shared memory segment
	- Specify identifier
	- Location in local address space 
	- Permissions (read and write)
* Also, operations for detach and control

### IPC with Pipes
* Producer-Consumer mechanism for data exchange
	- prog1 | prog2 (shell notation for pipe)
	- Output of prog1 becomes the input to prog2
	- Precisely, a connection is made so that the stdout of prog1 is connected to stdin of prog2
* OS sets up a fixed-size buffer (OS manages it)
	- System calls: [*pipe(2)*](http://man7.org/linux/man-pages/man2/pipe.2.html), [*dup(2)*](http://man7.org/linux/man-pages/man2/dup.2.html), [*popen(2)*](http://man7.org/linux/man-pages/man3/popen.3.html)
* Producer
	- Write to buffer, if space available
* Consumer
	- Read from buffer if data available

### Management of Pipes
* Buffer management
	- A finite region of memory (array or linked-list)
	- Wait to produce if no room
	- Wait to consume if empty
	- Produce and consume complete items
* Access to buffer
	- Write adds to buffer (updates end of buffer)
	- Reader removes stuff from buffer (updates start of buffer) 
	- Both are updating buffer state
* Issues
	- What happens when end is reached (e.g., in finite array)? 
	- What happens if reading and writing are concurrent?
	- Who is managing the pipe?

### IPC in Message-Passing Systems
* Mechanisms for processes to communicate and to synchronize actions
* Messaging system
	- Processes communicate without shared variables
	- Use messages instead
* Establish communication link
	- Producer sends on link
	- Consumer receives on link
* IPC Operations
	- *send(P, message)*: send a message to process P
	- *receive(Q, message)*: receive a message from process Q
* Communication link logical implementations
	- Direct or indirect communication
	- Synchronous or asynchronous communication 
	- Automatic or explicit buffering
* Issues
	- What if a process wants to receive from any proces?
	- What if communicating processes are not ready at same time?
	- What size message can a process receive?
	- Can other processes receive the same message from one process?

### Synchronous Messaging
* Direct communication from one process to another
* Synchronous send
	- *send(P, message)*
	- Producer must wait for the consumer to be ready to receive the message
* Synchronous receive
	- *receive(ID, message)*
	- ID could be any process
	- Wait for someone to deliver a message
	- Allocate enough space to receive message
* Synchronous means that both have to be ready
	- Otherwise, process (sender or receiver) is blocked

### Properties of Direct Communication Links
* Links are established automatically
* A link is associated with exactly one pair of communicating processes
* Between each pair there exists exactly one link
* The link may be unidirectional, but is usually bi-directional
* Messaging
	- Processes must name each other explicity (Naming)

### Asynchronous Messaging
* Indirect communication from one process to another
* Asynchronous send
	- *send(M, message)*
	- Producer sends message to a buffer M (like a mailbox)
	- No waiting needed (modulo busy mailbox)
* Asynchronous receive
	- *receive(M, message)*
	- Receive a message from a specific buffer (get your mail)
	- No waiting (modulo busy mailbox)
	- Allocate enough spacee to receive message
* Asynchronous means that can send/receive when it's ready
* What are some issues with the buffer?

### Properties of Indirect Communication Link
* Link established only if processes share a mailbox
* A link may be associated with many processes
* Each pair of processes may share several communication links
* Link may be unidirectional or bi-directional
* Messages
	- Messages are directed and received from mailboxes (also referred to as ports)
	- Each mailbox has a unique ID
	- Processes can communicate only if they share a mailbox

### Synchronization
* Message passing may be either *blocking* or *non-blocking*
* **Blocking** is considered synchronous
	- Blocking send
		- sender is blocked until the message is received
	- Blocking receive
		- receiver is blocked until a message is available
* **Non-blocking** is considered asynchronous
	- Non-blocking send
		- sender sends the message and continue
	- Non-blocking receive
		- receiver receives: a valid message or a null message
* Different combinations possible
	- If both send and receive are blocking, we have a rendezvous

### Communication in Client–Server Systems
* Sockets
* Remote procedure calls (RPC)
* Remote method invocation (a la Java)

### Sockets
* An endpoint for communication
* Sockets are named by
	- IP address (roughly, machine)
	- Port number (service: ssh, http, ...)
	- Concatenate the two (161.25.19.8:1625)
* A pair of processes communicating over a network employs a pair of sockets (one for each process)
	- Connect one socket to another (TCP/IP)
	- Send/receive message to/from another socket (UDP/IP)
* Semantics
	- Bidirectional link between a pair of sockets
	- Messages: unstructured stream of bytes
* Connection between
	- Processes on same machine (UNIX domain sockets)
	- Processes on different machines (TCP or UDP sockets)
	- User process and kernel (netlink sockets)
<img width="640" height="440" src="https://github.com/missystem/cis415review/blob/master/communication_models.png"> <br />

### Files and File Descriptors (POSIX)
* POSIX system calls for interacting with files
	- *open(), read(), write(), close()*
	- *open()* returns a *file descriptor*
		- an integer that represents an open file
		- inside the OS, it's an index into a table that keeps track of any state associated with you interactions, such as the file position
	- you pass the file descriptor into *read, write*, and *close*
	- File descriptors are kept as part of the process information in the process control block

### Networks and Sockets
* UNIX likes to make all I/O look like file I/O
	- Can use *read()* and *write()* to interact with remote computers over a network
* File descriptors are used for network communications
	- The socket is the file descriptor
* Just like with files
	- Your program can have multiple network channels (sockets) open at once
	- You need to pass *read()* and *write()* the socket file descriptor to let the OS know which network channel you want to use
* Examples of Sockets
	- HTTP / SSL
	- email (POP/IMAP)
	- ssh
	- telnet

### IPC and Sockets
<img width="675" height="430" src="https://github.com/missystem/cis415review/blob/master/IPCandSockets.png">

### File Descriptors
<img width="775" height="480" src="https://github.com/missystem/cis415review/blob/master/fileDescriptor.png">

### Type of Sockets
* Stream sockets
	- For connection-oriented, point-to-point, reliable bytestreams
		- uses TCP, SCTP, or other stream transports 
* Datagram sockets
	-  For connection-less, one-to-many, unreliable packets
		- uses UDP or other packet transports
* Raw sockets
	- For layer-3 communication
		- raw IP packet manipulation

### Stream sockets
* Typically used for client / server communications 
	- also for other architectures (peer-to-peer)
* Client
	- An application that establishes a connection to a server
* Server 
	- An application that receives connections from clients <br />
<img width="295" height="85" src="https://github.com/missystem/cis415review/blob/master/streamSocket01.png"><br /><img width="295" height="85" src="https://github.com/missystem/cis415review/blob/master/streamSocket02.png"><br /><img width="290" height="75" src="https://github.com/missystem/cis415review/blob/master/streamSocket03.png"><br />

### Datagram Sockets
* Used less frequently than stream sockets
	- They provide no flow control, ordering, or reliability
* Often used as a building block
	- Streaming media applications
	- Sometimes, DNS lookups
<br /><img width="320" height="200" src="https://github.com/missystem/cis415review/blob/master/datagramSockets01.png"><br /><img width="335" height="185" src="https://github.com/missystem/cis415review/blob/master/datagramSockets02.png"><br />

### Issues using Sockets
* Communication semantics
	- Reliable or not
* Naming
	- How do we know a machine’s IP address? DNS
	- How do we know a service’s port number?
* Protection
	- Which ports can a process use?
	- Who should you receive a message from?
		- Services are often open -- listen for any connection
* Performance
	- How many copies are necessary?
	- Data must be converted between various data types

### Remote Procedure Calls (RPC)
<br /><img width="513" height="617" src="https://github.com/missystem/cis415review/blob/master/RPC.png"><br />
* message-based communication scheme
* Each message is addressed to an RPC daemon listening to a **port** on the remote system, and each contains an identifier specifying the function to execute and the parameters to pass to that function
	- port: a \# included at the start of a msg packet
* The function is then executed as requested, and any output is sent back to the requester in a separate message
* Supported by systems
	- CORBA
	- Java RMI
* Issues
	- Support to build client/server stubs
	- Marshaling arguments and code
	- Layer on existing mechanism (e.g., sockets) 
	- Remote server crashes ... then what?
* Performance vs. abstractions
	- What if the two processes are on the same machine?
* Marshaling
<br /><img width="660" height="365" src="https://github.com/missystem/cis415review/blob/master/marshaling.png"><br />

### Remote Method Invocation (RMI)
* RMI is a Java mechanism similar to RPCs
* RMI allows a Java program on one machine to invoke a method on a remote object
<br /><img width="735" height="295" src="https://github.com/missystem/cis415review/blob/master/RMI.png"><br />

### IPC Summary
* Lots of mechanisms
	- Pipes
	- Shared memory 
	- Sockets
	- RPC
* Trade-offs
	- Ease of use, functionality, flexibility, performance
Implementation must maximize these
	- Minimize copies (performance)
	- Synchronous vs Asynchronous (ease of use, flexibility)
	- Local vs Remote (functionality)

## Summary
* **Shared memory**
	- Two (or more) processes share the same region of memory
* **Message passing**
	- Two processes communicate by exchanging messages with one another
* **Pipe**
	- provides a conduit for two processes to communicate
	- 2 forms of pipes:
		- Ordinary pipes
		- Named pipes
* 2 common tyeps of **client–server communication**
	- **sockets**
		- allow two processes on different machines to communicate over a network
	- **remote procedure calls (RPCs)**
		- abstract the concept of function (procedure) calls in such a way that a function can be invoked on another process that may reside on a separate computer




