## Monday, 1/8 Stop, Collaborate, and Listen by Daria Shifrina
**Tech news** [Physicists build muscle for shape-changing, cell-sized robots](https://www.sciencedaily.com/releases/2018/01/180103160115.htm)

We continued learning about how to create sockets.

* #### listen(server only) <sys/socket.h>
  ```listen(socket_descriptor, backlog);```

   * Socket descriptor: return value of socket
   * Backlog: number of connections that can be queued up 
	*Depending on the protocol this may not do much

listen uses:
* Set a socket to passively await a connection. 
* Needed for stream sockets. 
* Does not block.

* #### accept (server only) <sys/socket.h>

  ```accept (socket descriptor, address, address length);```

	* Socket descriptor: the listening socket descriptor
	* Address and address length intended for new socket
	  *Pointers that will be modified by accept
	  *Address: pointer to a struct sockaddr_storage that will contain information about the new socket after accept succeeds.
	  *Address length
	   *Pointer to a variable that will contain the size of the new socket address after accept succeeds

accept uses:
* Accept the next client in the queue of a socket in the listen state.
* Used for stream sockets
* Performs the server side of the 3 way handshake 
* Creates a new socket for communicating with the client, the listening socket is not modified (like WKP)
* Returns a descriptor to the new socket
* Blocks until a connection attempt is made

### Sample usage with listen and accept
```C
// create socket
Int sd; 
sd = socket(AD_INET, SOCK_STREAM, 0);
//use getaddrinfo and bind 
listen(sd, 10);
int client_socket; 
socklen_t sock_size;
struct sockaddr_storage client_address; 
client_socket = accept(sd,(struct sockaddr*)&client_address, &soc_size);
```

* #### connect (client only) <sys/socket.h> <sys/types.h>
  ```connect (socket descriptor, address, address length);```

	* Socket descriptor: descriptor for the socket
	* Address: pointer to a struct sockaddr representing the address
	* Address length: size of the address in bytes
	* Address and address length can be retrieved from getaddrinfo() 


connect uses:
* Connect to a socket that is currently in the listening state
* Used for stream sockets
* Performs the client side of the three way handshake
* Binds the socket to an address and port
* Blocks until a connection is made or the attempt fails

### Sample usage of connect
```C
// create socket
int sd;
sd = socket(AF_INET, SOCK_STREAM,0 );
struct addrinfo *hints, *results;
//use getaddrinfo( not shown)
connect(sd, results->ai_addr, results->ai_addrlen); 	
```


<hr>


## Friday, 1/5 Stop, Collaborate, and Listen by Jeffrey Luo
**Tech news** [65" Rollable TV Screen](https://www.theverge.com/2018/1/6/16859102/lg-display-rollable-oled-65-inch-ces-2018)

In previous classes, we learned that in order to use a socket, we must:

1. Create the socket
2. Bind it to an IP address and port
3. Listen and initiate a connection
4. Send and receive data

We were introduced to several method calls that can be used to accomplish some of these procedures.

### Method calls

* #### socket - <sys/socket.h>
  ```socket( <domain>, <type>, <protocol>);```
  
  Creates a socket. Returns a socket descriptor (int that works like a file descriptor).
  * domain: type of address (AF_INET or AF_INET6)
  * type: SOCK_STREAM or SOCK_DGRAM
  * protocol: combination of data and type settings. If set to 0, the OS will set to correct protocol (TCP or UDP)
  * NOTE: system library calls use a *struct addrinfo* to represent network addresses

* #### getaddrinfo - <sys/types.h> <sys/socket.h> <netdb.h>

  ```getaddrinfo( <node>, <service>, <hints>, <results>);```

  Lookup information about the desired network address and get one or more matching *struct addrinfo* entries.

  * node: string containing an IP address or hostname to lookup. If NULL, uses local machine's IP address.
  * service: string with a port number or service name (if in /etc/services)
  * hints: pointer to a *struct addrinfo* used to provide settings for the lookup (type of address, etc.)
  * results: pointer to a *struct addrinfo* that will be a linked list containing entries for each matching address. 
  
  getaddrinfo will allocate memory for these structs

* #### bind - <sys/socket.h>

  ```bind( <socket descriptor>, <address>, <address length>);```
  
  *Server only.* Binds the server to an address and port. Returns 0 on success, or 1 on failure. 
  
  * socket descriptor: return value of socket
  * address: can be retrieved from getaddrinfo
  * address length: size of the address in bytes


### struct addrinfo

These are the elements contained in addrinfo:

* .ai_family
  * AF_INET: IPv4
  * AF_INET6: IPv6
  * AF_UNSPEC: IPv4 or IPv6

* .ai_socktype
  * SOCK_STREAM
  * SOCK_DGRAM

* .ai_flags
  * AI_PASSIVE: auto set to incoming IP addresses
  * Many other flags exist, but may not be necessary for our purposes

* .ai_addr
  * Pointer to a *struct  sockaddr* containing the IP address

* .ai_addrlen
  * Size of the address in bytes

### Sample usage
```C
struct addrinfo *hints, *results;
hints = (struct addrinfo *)calloc(1, sizeof(struct addrinfo)); // note that you have to zero out struct addrinfo hints
hints->ai_family = AF_INET;
hints->ai_socktype = SOCK_STREAM; // TCP
hints->ai_flags = AI_PASSIVE; // only needed on server
getaddrinfo(NULL, "80", hints, &results); // server sets node to NULL. Note that you must pass fourth argument as a double pointer
// getaddrinfo("149.89.150.100", "9845", hints, &results); // client
// DO STUFF
free(hints);
freeaddrinfo(results);
```

<hr>

## Wednesday, 1/3 Socket To Me (continued) by Anish Shenoy
**Tech News** [Major Vulnerability Discovered in Processors](https://techcrunch.com/2018/01/03/a-major-kernel-vulnerability-is-going-to-slow-down-all-intel-processors-2/)

We picked up where we left off and continued to learn about networking. 

### Network Ports

* Allows a single computer to run multiple services
* Each computer has 2^16 (65,536) ports
* Some common reserved ports:
  * 80: http
  * 22: ssh
  * 443: ssl
* When slecting a port for your program, you can select any port as long as it doesn't conflict with a service running on the desired computer
* General Rules:
  * Ports under 1024 should'nt be used because they are reserved for the system
  * [This](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers) link shows a list of ports used by programs

### Network Connection Types

  * #### Stream Sockets:
    * Reliable 2-way communication
    * Must be connected on both ends
    * Data is recieved in the order it is sent (harder than it sounds)
    * Most use the Transmission Control Protocol (TCP)
	
  * #### Datagram Sockets: 	
    * "Connectionless" - an estblished connection is not required
    * Data sent may be recieved out of order (or not at all)
    * Most use the User Datagram Protocol (UDP)
    * Commonly used by video streaming services, video games, and other applications where speed is important.
  
<hr>

## Tuesday, 2018-01-02 Socket to Me by Gian Tricarico

I decided to mix it up a little bit and put the tech news at the bottom 'cause
that's what it says to do on the website anyway.

Today we began learning the basic concepts of networking. We will be moving on
to the coding part shortly and then we will go back to learn more about
conceptual networking stuff, as we young Padawans have much to learn.

### Socket

A socket is a connection between two programs over a network. Each socket corresponds to an IP* Address/Port pair (by that I mean a pair consisting of an IP
Address and a Port, not an IP Address or a Port pair, in case there is any
confusion with the slash I used in that sentence).

*IP stands for internet protocol.

To use a socket, you must

0. create the socket
1. bind it to an address and port
2. listen/initiate a connection (This is similar to how the clients and servers
behaved when we were working with pipes.)
3. send/receive data

### IP Addresses

All devices connected to the internet have an IP address. IP addresses come in
two flavors: IPv4 and IPv6. Addresses are allocated in blocks to make routing
easier.

IPv4 has 4-byte addresses of the form:

[0-255].[0-255].[0-255].[0-255]
(each group contains some integer from 0 to 255)

Each of these four groups is called an *octet*, because each group is 8 bits,
or one byte. At most there are 2<sup>32</sup>, or approximately 4.5 billion, IP
addresses. Teh problem is, that's not enough for the multitude of computers and
devices that are connected to the interwebs. We use network address translation
to deal with this, where there are multiple private addresses within a network
that has its own single public IP address. However, there is also IPv6, created
to make room for many more addresses (approximately 2<sup>128</sup>, which is a
very large number).

IPv6 addresses are in this format (using hexadecimal notation):
[0-ffff]:[0-ffff]:[0-ffff]:[0-ffff]:[0-ffff]:[0-ffff]:[0-ffff]:[0-ffff]

Each group in this system is known as a hextet (although this is not as standard
of a term as octet). In order to write it out more concisely, we ignore leading
0s within each hextet and replace any consecutive all-0 hextets with a
double colon (::). One can only use the double colon once in a single IP
address, however, as otherwise the address would be ambiguous.

For example, we can write

0000:0000:0000:004f:13c2:0009:a2d2

as

::4f:13c2:9:a2d2

IPv4 Mapped addresses have the following format:

0000:0000:0000:0000:0000:ffff:[IPv4 portion of the address], 

for example,

0000:0000:0000:0000:0000:ffff:12.155.166.101,

or simply ::ffff:12.155.166.101 for short.

This allows us to convert every possible IPv4 address to a corresponding IPv6
address, but not the other way around. This creates compatibility issues that
help explain why IPv4 is still the standard for the internet and the transition
to IPv6 is proceeding quite slowly.

**Tech News:** [Iran blocks messaging app Telegram used by protesters](https://www.usatoday.com/story/tech/2018/01/02/iran-blocks-messaging-app-used-protesters-share-info-demonstrations/998445001/)

<hr>

## Monday, 12/18 Always tip your servers by Ashneel Das
**Tech News** [Google uses light beams to connect rural India to the internet](https://techcrunch.com/2017/12/15/google-is-using-light-beam-tech-to-connect-rural-india-to-the-internet/)

In our most recent assignment, we created a server that could accommodate for one client. 
To have more than one client, we need to fork. 
* Handshake
  1. (Same as before) Client connects to server and sends the private FIFO name. Client waits for a response. 
  2. Server receives client's message and forks off a subserver. 
  3. Server closes and removes the WKP. 
  4. Subserver connects to client FIFO, sending an initial acknowledgment message. 

* Operation
  1. Server removes connections to WKP. 
  2. Server recreates WKP and waits for new connections. 

  Only the server needs to be modified! Not the client. 
## Monday 12/11 Creating a handshake agreement by Dmytro Hvirtsman
**Tech News**[Airbnb plans to start giving virtual tours of apartments](https://www.theverge.com/2017/12/11/16762046/airbnb-virtual-reality-previews-listings)

*Handshake: a procedure that establishes a connection between two programs
Basic Client/Server design pattern
* 1. Setup
	* Server creates a FIFO and waits for a connection (well known pipe)
	* Client creates a "Private FIFO"
* 2. Handshake
	* Client connects to server and sends a private FiFO name and waits for the servers response
	* Server recieves client message and removes Well Known Pipe
	* Server connects to client FIFO, sending an acknowledgement message
	* Client recieves message and removes private FIFO
	* Client sends response
* 3. Operation
	* Client and Server exchange data 
* 4. Reset
	* Client exits, server closes the connection
	* Server recreates Well Known Pipe


## Thursday 12/7 What's a semaphore? -- To Control Resources! by Haoyu Chen
**Tech News** [Next year’s Android phones will be able to capture 4K HDR video](https://www.theverge.com/2017/12/6/16742600/qualcomm-snapdragon-845-processor-4k-hdr-video-2019)

**semctl (Continued)**
* semctl(int semid, int semnum, in cmd, ...)
* This function takes in 3 or 4 arguments depending on the cmd
* The fourth argument is a type union semun
* The calling program must define this union as followed
```
/*union semun {
               int              val;    /* Value for SETVAL */
               struct semid_ds *buf;    /* Buffer for IPC_STAT, IPC_SET */
               unsigned short  *array;  /* Array for GETALL, SETALL */
               struct seminfo  *__buf;  /* Buffer for IPC_INFO
                                           (Linux-specific) */
           };
*/
```
* What is a union?
	* A c structure designed to hold only one value at a time from a group of potential values
	* the size of a union is just large enough to hold the largest piece of data it could potentially hold
**semop**
* semop ( <DESCRIPTOR> , <OPERATION> , <AMOUNT>)
	* Descriptor : the return value of semget
	* Operation : a pointer to a struct sembuf
		* Sembuf contains the following members
			* unsigned short sem_num ; // semaphore number
			* short sem_op; //semaphore operation
			* short sem_flg; // operation flags
				* SEM_UNDO : allow to OS to undo the given operation. Useful in the even that a program exits before it could release a semaphore 
				* IPC_NOWAIT : instead of waiting for the semaphore to be available return an error
	* Amount : how many semaphores is the operation being done on

<hr>

## Tuesday 12/5 How do we flag down a resource? by Judy Liu
**Tech News** [Are Exoskeletons the Future of Physical Labor?](https://www.theverge.com/2017/12/5/16726004/verge-next-level-season-two-industrial-exoskeletons-ford-ekso-suitx)

**Semaphores** (Edsger Dijkstra)
* IPC consrtuct used to control access to a shared resource (like a file or a shared memory)
* Most commonly, a semaphore is used as a counter representing how many processes can access a resource at a given time
	* If a semaphore has a value of 3, then it can have 3 active "users"
	* If a semaphore has a value of 0, then it is unavailable
* Most semaphore operations are "atomic", meaning they will not split up into multiple processor instructions
	
**Semaphore Operations**
* `<sys/types.h> <sys/ipc.h> <sys/sem.h>`
* Create a semaphore (not atomic)
* Set an intial value (not atomic)
* Remove a semaphore (not atomic)
1. `Up(s)/V(s)` - atomic
	- Release the semaphore to signal you are donw with its associate resource
	- Pseudocode: `s++`
2. `Down(s)/P(s)` - atomic
	- Attempt to take the semaphore 
	- If 0, wait for it to be available
	- Pseudocode: `while ( s == 0 ) { block } s--;`
3. semget: create/get access to semaphore
	- This is not the same as Up(s)/Down(s). It does not modify the semaphore
	- Returns semaphore descriptor -1 (errno)
* `semget( <key>, <amount>, <flags>)`
	- key: unqiue identifier
	- amount: semaphores are stored as sets of 4 or more, number of semaphores to create/get in a set
	- flags: includes permissions for the semaphore with bitwise
		- `IPC_CREAT`: create the semaphore and set value to 0
		- `IPC_EXCL`: fail if the semaphore already exists and IP_CREAT is on
		
## Monday 12/4, Memes by Eric Zhang
**Tech News** [Hackers can gain access to your computer monitor](http://www.businessinsider.com/how-hackers-can-compromise-your-computer-monitor-darkly-cybersecurity-ssl-mr-robot-red-balloon-security-2017-11)

**More Shared Memory Functions**

`shmdt( pointer )`
* Detach a variable from ashared memory segment
* Returns 0 upon success or -1 upon failure
  * pointer: the address used to access the segment

`shmctl( descriptor, command, buffer )`
* Perform operations on the shared memory segment
* Each shared memory segment has metadata that can be stored in a struct (`struct shmid_ds`)
* Some of that data stored: last access, size, pid of creator, pid of last modification
  * descriptor: return value of shmget
  * commands:
    * IPC_RMID: remove a shared memory segment
    * IPC_STAT: populate the buffer (`struct shmid_ds *`) with segment metadata
    * IPC_SET: set some of the segment metadata from buffer
  * buffer: can be `NULL` if using IPC_RMID

**Shell Commands**

`ipcs -m`: Lists shared memory

`ipcrm -m <shmid>`: clears the given shmid
<hr>

## Friday 12/1, Sharing is Caring by Ricky Chen
**Tech News** [Quantum Encryption Became Much Faster (and Practical)](https://www.technewsworld.com/story/Quantum-Key-Distribution-Gets-a-Speed-Boost-84983.html)

**Shared Memory - <sys/shm.h>, <sys/ipc.h>, <sys/types.h>**
- A segment of heap memory that can accessed by multiple processes.
- Shared memory is accessed via a key that is known by any process that needs to access it.
- Shared memory does not get released when a program exits.

**5 Shared Memory Operations**
* Create the segment (happens once)
* Access the segment (happens once per process)
* Attach the segment to a variable (once per process)
* Detach the segment from a variable (once per process)
* Remove the segment (happens once)

The commands for these begin with shm[.](https://www.youtube.com/watch?v=1pAMdn9oSPE)

`shmget( key, size, flags )`
* Create and access a shared memory segment
* Returns a shared memory descriptor (similar concept to a file descriptor), or -1 if it fails. (errno)
* *key*
  * Unique integer identifier for the shared memory segment (like a file name)
* *size*
  * How much memory to request (bytes)
* *flags*
  * Includes permissions for the segment, combined with bitwise or
    * IPC_CREAT: creates a segment
        * If segment is new, will set value to all 0’s.
    * IPC_EXCL: Fails if the segment already exists and IPC_CREAT is on
		
`shmat( descriptor, address, flags)`
* Attach a shared memory segment to a variable.
* Returns a pointer to the segment, or -1(errno)
  * *descriptor*
    * The return value of shmget
  * *address*
    * If 0, the OS will provide the appropriate address.
  * *flags*
    * Usually 0, there is one useful flag:
      * SHM_RDONLY: makes the memory read only
	
<hr>

## Tuesday 11/28, Aim: C, the ultimate hipster, using # decades before it was cool by Terry Guan
**Tech News** [Facebook releases AI for suicide prevention](https://www.cbsnews.com/news/facebook-artificial-intelligence-suicide-prevention/)

  * `#`: preprocessor instructions
    * These directives are handled by gcc first
    * Separate from actual C code. It's more like gcc code
    * `#include <LIB>` or `#include "LIB"`
  * `#define <NAME> <VALUE>`
    * replace all occurrances of NAME with VALUE
    * `#define TRUE 1`
    * macros: kind of like a function
      * `#define SQUARE(x) x * x`
      * `#define MIN(x, y) x < y ? x:y`
        * `?`: if boolean before it,  `:` else
  * conditional statement:
```C
#ifndef <IDENTIFIER> //ifndef: if not defined
<CODE>
#endif

//example
#ifndef PARSE_H
 #define PARSE_H
  void funct();
  //rest of C header code
#endif
```      
<hr>

## Monday 11/27, Redirection, how does it...SQUIRREL? by Cesar Mu
**Tech News** [Robotic falcons are being used to scare away birds from airports](https://goo.gl/DKUZbU)

**File Redirection**
- changing the usual input/output behavior of a program

**Command line redirection**
- ```>```
  - redirects stdout to a file by overwriting
  - ex: ```ps > ps_file```; ```COMMAND > FILE```
  - ex: ```cat file1 > file2``` will copy contents of file1 to file2 (same as cp)
  - ex: ```cat ``` will just open stdin and print to stdin; typing something will just print it again
    - type end of file character to exit
    - ```cat > file``` will write input to file
- ```>>```
  - redirects stdout to a file by appending, not overwriting
- ```2>```
  - redirects stderr to a file by overwriting
  - ```2>>``` to append
  - useful for log files to keep track of errors; ex: for web servers or database servers
- ```&>```
  - redirects both stdout and stderr to a file by overwriting
  - ```&>>``` to append
  
- ```<```
  - redirects stdin from a file
  - useful for testing programs; if your program reads input, you can just make it read from a file
- ```|``` (pipe)
  - redirects stdout of one command into stdin of the next
  - ex: ```ls | wc``` will take output of ls and put into wc
  
**Redirection in code**
- ```dup``` - <unistd.h>
  - duplicates an existing entry in the file table
  - returns a new file descriptor for the duplicate entry
  - ```dup(FD)```
    - returns the new file descriptor
- ```dup2``` - <unistd.h>
  - ```dup2(FD1, FD2)```
  - redirects FD2 to FD1
  - duplicates the behavior for FD1 at FD2
    - replaces FD2 with FD1 in the file table
  - problem: you lose what FD2 was
    - fix by first dup(FD2), then dup2(); this will keep FD2 in reserve so you can undo it later

## Wednesday 11/22 Aim: A Pipe by Any Other Name - Othman "Monty" Bichouna
**Tech News** [NASA's new titanium tires never flat](http://bgr.com/2017/11/26/nasa-airless-tire-no-flat/)

Today in class we discussed named pipes, often called FIFOs (First In First Out)

Similar functionality to unnamed pipes except FIFOs have a name that can be used to identify them via different programs. Like unnamed pipes, FIFOs are unidirectional

#### `mkfifo`

Shell command to make a FIFO:

```
$ mkfifo <NAME>
```

C command to make a FIFO:

#### `mkfifo - <sys/types.h> <sys/stat.h>`

Returns 0 on success and -1 on failure

### Interactions

* FIFOs will block on open until both ends of the pipe have a connection.
* Opening a pipe with `$ cat` will make it block for things to be written into it
* You can use another shell window and do `cat > pipename` to write user inputs to the pipe
* If you have a named pipe open on both ends, and you close one end, the other one also closes
* Pipes only exist in RAM, so their file size using `$ls -l` is 0 because that checks for disk space
* If 2 programs are reading from a pipe, it’s undefined which one will receive each line of data written
* Using `$ rm` a pipe is like turning a named pipe into an unnamed pipe

<hr>

## Friday 11/17, Aim: Ceci n'est pas une pipe, by Asim Kapparova
**Tech News** [Germany bans children's smartwatches, described as spying devices](http://www.bbc.com/news/technology-42030109)
* Pipe:
  * A conduit between 2 separate processes on the same computer
  * Pipes have 2 ends: rear + write ends
  * Pipes are unidirectional (a single pipe must be either read or write only in a process)
  * Pipes act just like files
  * You can transfer any data you like through a pipe using read/write
  * Unnamed pipes have no external identifier

* pipe -- in `<unistd.h>`
  * `pipe(int descriptors[2])`
  * Creates an unnamed pipe
  * Returns 0 if the pipe was created, -1 if not
  * Opens both ends of the pipe as files for reading and reading
  * Descriptors
    * Array that will contain the descriptors for each end of the pipe
  * Read is a blocking function: if there is nothing to read then it will put everything on hold until there is data.
  * The parent will wait for the child to write before it runs

* The pipe is basically like array: uses one side as read and one side as write


## Wednesday 11/15, Aim: Playing favorites
**Tech News** [Google writes Ai to write Ai](https://www.technologyreview.com/s/603381/ai-software-learns-to-make-ai-software/)
* Thread:
  * Similar to a child process
  * does not have its own memory
  * relies on parent instructions. Needs parent process to be running
  * This is NOT what `fork()` does
* void pointers is a generic pointer
* You can think of data type as number of bytes. int can be thought of as 4 bytes of data. These 4 bytes of data can hold different and significant data for each byte
* Ordering of bytes in a variable
  * In the examples below, there is a data type that has a certain amount of bytes. (I.e int is 4 bytes). Imagine a row of 4 bytes.
  * 1 byte is 8 bits. 8 bits has 2 possibilities (0 and 1), so each byte holds 2^8 bits of data.
  * Further data is incremented by powers of 2^8
  * endian-ness: Byte order of values
    * litle endian: least significant value would show up as first byte (1st byte is 2^8, 2nd byte is 2^16, 3rd byte is 2^8 etc)
    * big endian: most significant value would show up as first byte (1st byte is 2^32, 2nd byte is 2^16, etc)
* `wait( int *<STATUS>)`
  * In `<unistd.h>`
  * STATUS: used to store info about how the process exited
    * Holds return value of child program in second byte
    * first byte has a signal  

<hr>


## Tuesday 11/14, Wait for it — Yiduo Ke
**Tech News** [Scientists use a supercomputer to simulate one of history’s biggest quakes](https://www.digitaltrends.com/cool-tech/supercomputer-earthquake-simulation/)

We continued to talk about forking. We drew fork trees by analyzing outputs in the terminal.
```c
int childId = fork();
```
fork() returns the id of the child process when the current process forks. If it's 0, then that means the process has no children. If the parent is 1, then that means the parent quit but the child was still running when it printed that line. Such a process is referred to as an *orphan*.
```c
int main(){
    int i=0;
    printf("pre-fork\n");
    int f=fork();
    printf("me: %d, child: %d, parent: %d\n", getpid(), f, getppid());
    return 0;
 }

```
can produce the following output:
```
pre-fork
me: 130, child: 131, parent: 2
me: 131, child: 0, parent: 130
```
We can see that the parent process is 130, and its child is 131.

Forking n times produces 2^n processes.
Let's try forking twice, which should produce 4 processes:
```c
int main(){
    int i=0;
    printf("pre-fork\n");
    int f=fork();
    fork();
    printf("me: %d, child: %d, parent: %d\n", getpid(), f, getppid());
    return 0;
 }

```
can produce the following output:
```
pre-fork
me: 121, child: 122, parent: 2
me: 123, child: 122, parent: 121
me: 122, child: 0, parent: 1
me: 124, child: 0, parent: 122
```
We can see the ancestor is 121, it forked 122, which in turn forked 124. The branch that did not fork off of 121 is 123.

Then we learned about waiting.
```c
#include <unistd.h>
int pidOfChildThatExited = wait(int * status)
```
Wait stops a parent process from running until any child has provided status information to the parent (usually when the child has exited). The child sends a signal to the parents if it has exited. Wait returns the pid of the child that has exited, or -1 (errno). The parameter -- status -- is used to store information about how the child process exited.

Sample code with child sleeping for 2 seconds before execution:
```c
int main(){
    int i=0;
    printf("pre-fork\n");
    int f = fork();
    fork();

    if (f ==0 ){
        sleep(2);
        printf("I'm a child. post-fork: %d   f: %d   parent: %d\n", getpid(), f, getppid());
    }
    else{
        printf("I'm a parent. post-fork: %d   f: %d   parent: %d\n", getpid(), f, getppid());
    }
    return 0;
 }
 ```
 output:
 ```
 pre-fork
I'm a parent. post-fork: 137   f: 138   parent: 2
I'm a parent. post-fork: 139   f: 138   parent: 137
whateverPath/blah/folderName$ I'm a child. post-fork: 138   f: 0   parent: 1
I'm a child. post-fork: 140   f: 0   parent: 138
```
The command line shows up before one of the printed lines because the parent was done and exited so the terminal thinks the program was done but it actually wasn't and then the child ran.

## Monday 11/13 Fork by Daniel Pae
**TechNews:** [Want To Really Teach a Robot? Command It With VR](https://www.wired.com/story/embodied-intelligence-want-to-really-teach-a-robot-command-it-with-vr/)

We started by reviewing the previous 2 assignments

For assignment 10 a lot of people did not read the instructions properly.
 Just make sure to read everything over again before you submit.
Also test your makefiles.

We went over a solution to assignment 11
Here is mine:
```C
char ** parse_args ( char * line ){
  char ** args = (char **) malloc(sizeof(char *) * 6);
  int i = 0;
  while(line){
    args[i] = strsep(&line, " ");
    printf("str %d: %s,    line: %s\n", i, args[i], line);
    i++;
  }
  args[i] = NULL;
  return args;
}


int main(){
  char temp[100] = "ls -a -l";
  char** args = parse_args(temp);
  execvp("ls", args);
}
```
Just a few quick things about this:

* The char * line must point to a mutable string because strsep changes the string.
* The pointer line must also be mutable, it is in this case because arguments are passed by value.
* The last argument in args must be set to NULL.
* Strsep returns the place line originally points to, and moves the pointer to one past the char you make it look for. It also replaces that char with a NULL

###   Fork
Fork is in unistd.h

It creates a SEPARATE process. This new process is called a child and it is an exact copy of the parent. Essentially it continues the code from the fork.

The original, or parent, process still runs

The both have different PIDs

Do NOT
```C
while(1){
  fork()
}
```
The OS will not stop you like it will with segfaults or stack overflow errors. Bad things will happen.



## Thursday 11/09 Time to make an executive decision by Dmytro Hvirtsman
**Tech News:** [Amazon Cloud Cam Joins Burgeoning Smart-Home Ecosystem](https://www.technewsworld.com/story/Amazon-Cloud-Cam-Joins-Burgeoning-Smart-Home-Ecosystem-84942.html)

The executive family - <unistd.h>
	These functions are functions that run other programs from within
	When you do this the original process terminates

```C
execlp(<PROGRAM>, <arg1>, <arg2>, ...);
```
First argument will always be the name of the program
subsequent arguments are flags
always ends with NULL

to run ls -l -a :

```C
execlp("ls", "ls", "-l", "-a", NULL);
```

The difference between execlp and execvp is execvp uses an array for its arguments
	always has two variables
	the <FILE> can always be taken by doing arr[0] since the first argument is the name of the file

to run ls -l -a:

```C
char (*arr[4]);
arr[0] = "ls";
arr[1] = "-l";
arr[2] = "-a";
execvp(arr[0]. arr);
```

Wednesday, 2017-11-08 Sending mixed signals by Gian "Giant" Tricarico
====================================================================

**Tech News:** [Waymo's Autonomous Cars Cut Out Human Drivers in Road Tests](https://nyti.ms/2hP9QVh)

Signals are a limited way of sending information to a process in the form of an
integer value. `kill` is the command line utility to send a signal to a process:

	$ kill <PID>

The default action for `kill` is to send the signal SIGTERM (defined as the
integer 15) to PID. You can use flags to send different signals instead:

    $ kill [-<SIGNAL>] <PROCESS>

You can type the signal in the form of its name or its number.

e.g.

	$ kill -2 <PROCESS>

or

	$ kill -SIGINT <PROCESS>

Refer to the man page using `$ man 7 signal` to see a list of all the signals
(It will probably be a different man page actually, unless you're using the same
operating system as I am).

Signal handling in c programs <signal.h>
----------------------------------------

You can use `kill` in your code as well:

    kill(<PID>, <SIGNAL>)

-Returns 0 on success

-Returns -1 and sets errno on failure

### sighandler

To intercept signals in a c program you must create a signal handling function.
Convention is to use the following header:

```C
static void sighandler(int signo)
```

The function must be static and void, and it must take a single int parameter
(in C, `static` means that the function can only be called from within the file
in which it is defined, so if you include the file into another file, this
sighandler will not be used by the other file).

e.g.

```C
static void sighandler(int signo)
{
  if (signo == SIGINT) {
    printf("I've been interrupted, how rude!");
  }
}
```

This overrides the default action for that signal, meaning that it does not
exit the process anymore unless you specify as such by calling the exit
function within the if statement, e.g. `exit(1)`, which exits the program and
makes it return 1 rather than 0.

If you want to set the action for multiple signals, use additional if statements
within the same signal handling funtion rather than creating multiple functions.

Also, in order for the sighandler to take effect, you must call the signal()
function in your main method for each signal you want to intercept. The first
argument is the signal, and the second is your signal handling
function.

e.g.

```C
signal(SIGINT, sighandler);
signal(SIGUSR1, sighandler);
```

## Monday 11/06 Are your processes running? Then you should go out and catch them! By Inbar Pe’er
** Tech News: ** [SONY resurrects robot pet Aibo which it “put down” in 2006]
https://www.technewsworld.com/story/Sonys-Aibo-Resurrected-From-Robot-Pet-Cemetery-84929.html
fgets - <stdio.h>
	Read in from a file stream and store it in a string
	```C
	fgets(<DESTINATION>, <BYTES>, <FILE POINTER>)
	fgets(s, n, f);
	```
		Reads at most n-1 characters from filestream f and stores it in s.
		Automatically puts in a terminating null at the end.
	File pointer
		FILE * type, more complex than a file descriptor
		stdin is a FILE * variable
	Stops at newline, end of file, or the byte limit.
	If applicable, keeps the newline character as part of the string, appends NULL after.
fgets(s,256,stdin)
	read a line of text from standard in
```C
Example:
int main(){
	char s[200];
	int i;
	float f;
	printf(“enter data\n”);
	fgets(s, sizeof(s), stdin);
	printf(“s: %s \n”, s);
	return 0;
}https://github.com/mks65/dwsource.git
```
fgets call won't run until it knows that there is something in standard in to be read (BLOCKING)
new line is a signal to send whatever was in standard out, out. (FLUSHING THE BUFFER)
	easiest way to flush standard in buffer is to hit enter, but can do it manually
sscanf - <stdio.h>
	Scan a string and extract values based on a format string.
	scanf(<SOURCE STRING>, <FORMAT STRING>, <VAR1>, <VAR 2>…)

Processes
Every program that is running is a process. A process can create subprocesses, but these are no different from regular processes
A processor can handle one process per cycle (per core).
“multitasking” appears to happen because the processor switches all the active processes quickly.
operating system needs to be designed to schedule things effectively between multiple cores
Nowadays our OS’s are pretty good about this stuff (always getting better)
pid
	Every process has a unique identifier called the pid.
	pid 1 is the init processor
	ps - displays the processes currently running ( but only the ones you are the owner of and the ones attached to terminal sessions)
	ps -a (all the processes by everyone)
	ps -aux (processes not attached to terminal windows)
	first number is pid value

## Friday 11/03: Input? Fgets about it! by Md Abedin

**Tech News:** [MIT wrote an A.I. that generates horror stories](http://boston.cbslocal.com/2017/10/31/artificial-intelligence-horror-stories-mit/)

Today we went over the stat and dirinfo assignments and Mr. DW showed us how to code the extra features. We also learned some new functions like sprintf() and scanf(), and the syntax for command line arguments:

**human readable size:**

To print out the size of a file in human readable format, we learned how to use sprintf():

```C
sprintf(char * stringPointer, "Format string", value1, value2, ...)
```

This function takes a pointer to a string and fills it with a formatted string. The format string in the 2nd parameter can have formatting tags which are replaced by the values in the parameters following it.

To get a string of a filesize in human readable format, you could use

```C
char *getSizeString(off_t s){
    char *size = (char *)calloc(13, 1);
    if (s>= 1000000000){
        sprintf(size, "%0.2f GB", s/1000000000.0);
    }
    else if(s>=1000000){
        sprintf(size, "%0.2f MB", s/1000000.0);
    }
    else if(s>1000){
        sprintf(size, "%0.2f KB", s/1000.0);
    }
    else{
        sprintf(size, "%lld B", s);
    }
    return size;
 }
```

The %0.2f formatting code lets you print a float with 0 being the minimum characters displayed and 2 being the number of digits after the decimal point displayed.

**human readable permissions:**

To get the permissions of a file in a human readable format, you can use bitwise operators

```C
char * get_perm_string(mode_t mode){
    mode_t perms[3];
    int i;
    char *perm_string = (char *)malloc(10);
    perm_string[9]=0;

    perms[0] = (mode & 0b111000000) >> 6;
    perms[1] = (mode & 0b111000) >> 3;
    perms[2] = (mode & 0b111);

    for (i=0; i<3; i++){
        if (perms[i] & 0b100){
            perm_string[0+i*3]='r';
        }
        else{
            perm_string[0+i*3]='-';
        }

        if (perms[i] & 0b010){
            perm_string[1+i*3]='w';
        }
        else{
            perm_string[1+i*3]='-';
        }

        if (perms[i] & 0b1){
            perm_string[2+i*3]='x';
        }else{
            perm_string[2+i*3]='-';
        }
    }
    return perm_string;
}
```

This &'s the mode of a file with the binary representation of full rwx permissions for the each permission area, which results in the current active permissions for each area. It then bitshifts it to the right by a certain amount in order to get rid of the trailing 0s and just leave the 3 bits that represent the user's permissions.

**Command Line Arguments:**

```C
int main(int argc, char *argv[])
```

agrc: number of command line arguments

argv: array of command line arguments; the program name is the first element of the array

You can name these parameters whatever you want, but it's best to use argc and argv to be descriptive and follow the standard.

**scanf()**
```C
scanf( <FORMAT STRING>, <VAR 1>, <VAR 2> ... ): Included from <stdio.h>
```

Reads in data from stdin using the format string to determine types. Puts the data in each variable. Variables must be pointers	because C is a pass-by-value language
Example:

```C
int x; float f; double d;
scanf("%d %f %lf", &x, &f, &d);
```

---

## Wednesday, 11/01: Where do cs priests keep their files? (In directory!) by Nikita Borisov

**Tech News:** [Strechy silicone sensors developed with NO WIRES inside](https://www.sciencedaily.com/releases/2017/10/171023131932.htm)

### `*nix` directory files

Is a file containing the names of the files within the directory along with
basic information about the files(like file type etc.)

When files are moved in and out of a directory, no data is moved(only change in directory files)

Size of directory doesn't depend on size of files inside it(removing an empty file will change size of directory file).

### `opendir - <dirent.h>`:

Opens directory file w/o chaning cwd(current working directory)

`opendir(<PATH TO DIR FILE>);` returns a pointer to a directory stream (`dir *`)

### `closedir - <dirent.h>`:

Closes directory stream and frees `dir *`

`closedir(<DIRECTORY STREAM *>);` returns 0 on success and -1 on error(with `errno`)

### `readdir - <dirent.h>`:

Gives information about files inside directory file

`readdir(<DIRECTORY STREAM *>);` returns pointer to next entry in a directory or NULL(when at the end of dir file)

Can be used multiple times until desired file is reached

Return pointer points to `struct dirent`...

### `struct dirent - <sys/types.h>`

Contains info about files

* `d_name`: name of the file
* `d_type`: file type

---

## Monday, 10/30: Seek and ye shall find! by Othman "Monty" Bichouna

**Humorous Tech News:** [Microsoft Employee has to Install Google Chrome to Finish Presentation](https://thenextweb.com/microsoft/2017/10/30/microsoft-engineer-chrome-edge-google/)

**Not so Humorous Tech News:** [Is the FCC Purposefully Screwing up US School Broadband Projects?](https://www.theregister.co.uk/2017/10/27/fcc_school_broadband/)

### LSEEK

We learned about the *lseek* function:
*lseek* sets the current position in an open file.

**Returns:** the number of bytes from the current position to the beginning of the file, or -1 (errno)

```c
#include<unistd.h>
lseek(<file Descriptor>, <offset>, <whence>);
```

### Arguments:

`file Descriptor`: The file descriptor of the pointer that is going to be moved

`offset`: The number of bytes to move the position by. This can be negative

`whence`: Where to measure the offset from

* `SEEK_SET`: Offset from the start of the file
* `SEEK_CUR`: Offset from the current index of our bookmark
* `SEEK_END`: Offset from the end of the file

### STAT

The *stat* function will get information about a file, AKA its metadata

```c
#include<sys/stat.h>
stat(<path>, <stat buffer>);
```

### Arguments

`path`: The path of the file that we want the metadata of

`stat buffer`: A pointer to a struct stat

```c
struct stat sb;
stat("foo", &sb");
```

All the metadata gets put into the stat buffer

Some of the fields inside the stat struct include:

- `st_size`: File size in bytes
- `st_uid, st_gid`: User ID, Group ID
- `st_mode`: File permissions
- `st_atime, st_mtime`: Last access, Last modification

---

## Thursday, 10/26: Read your writes! by Matteo Wong

**Interesting Tech News:** [Twitter Bans Russian News Outlets](https://www.nytimes.com/2017/10/26/technology/twitter-russia-today-sputnik.html?rref=collection%2Fsectioncollection%2Ftechnology)

**Interesting Tech News 2, related(-ish) to first:** [Reddit bans Nazis](https://www.nytimes.com/2017/10/26/us/reddit-violence-policy.html?rref=collection%2Fsectioncollection%2Ftechnology)

I am including two pieces of tech news because they are both related to the regulation of what many perceive as dangerous content on the internet

**Not News but very interesting and related to news of a few days ago about racism in driverless cars:** [Radiolab podcast on ethics of driverless cars](http://www.radiolab.org/story/driverless-dilemma/)

### First we learned about the different *flags* for the `open()` function. They are:
* O_RDONLY
* O_WRONLY
* O_RDWR
* O_APPEND — start at the end of the file.
* O_TRUNC — start at the beginning of the file. If you open a file with the ability to write and with the O_TRUNC flag, it will effectively erase the rest of the contents (truncate them)
* O_CREAT
* O_EXCL — when combined with O_CREAT will return an error if the file exists

Each flag is a number and can be combined with bitwise OR:

O_WRONLY = 1 = 		00000001

O_APPEND = 8 = 		00001000

O_WRONLY | O_APPEND =	00001001


### errno — <errno.h>
errno is an integer that corresponds to an error message. If `open()`, `read()`, `write()`, or `close()` fails, it will update errno.

To access the error message, use `strerror(errno)`

strerror() is in <string.h>

For example, of errno==2, `strerror(errno)` returns "No such file or directory"


### umask — <sys/stat.h>

We noticed that when creating a file, the permissions don't always come out as we specified. For instance:

```c
int fd;
fd=open("foo",O_CREAT | O_EXCL, 0666);
```

We would expect the permissions of 0666 to be `rw-rw-rw-`

Instead, they are: `rw-rw-r--`, which corresponds to 0664, not 0666.

This is because by default files aren't give nthe exact permissions provided. There is a **mask** designed to make sure you don't give permissions that are safety concerns (ie. giving everyone writing permission).

The default value of the **mask** on linux/unix is 0002.

The mask is applied in the following way:

`new-permissions=~mask & mode`

Ex:
	mask = 0002 = 000 000 010

	mode = 0666 = 110 110 110

	~mask= n/a  = 111 111 101

	~mask & mode = 110 110 100

You can manually set the mask with
```c
umask(<MASK>);
```
which will define mask with a 3 digit octal number.


### close(int fd) — <unistd.h>
takes a file descriptor and removes fd from the file table

returns 0 if successful and -1 if it fails (as well as setting errno)


### read — <unistd.h>
Read in data from a file

read(<file descriptor>, <buffer>, <amount>)

read(fd,buff,n)

read n bytes from the file at fd in the file table and put that data in tbe buffer

buff must be a pointer

returns the number of bytes actually read. returns -1 and sets errno if unsuccessful


### write — <unistd.h>

write data to a file, reverse of read

write(<file descriptor>, <buffer>, <amount>)

write(fd,buff,n)

write n bytes from buff to file at fd

buff must be a pointer

returns the number of bytes written, or -1 and sets errno if unsuccessful

At the end of class we saw that you are not limited to writing strings, eg:

```c
	...
	int fd;
	fd=open("foo",O_WRONLY);
	char s[]="Hello there!"
	write(fd,s,sizeof(s));
	...
```
`cat foo` will give `hello there`

BUT

```C
	...
	int fd;
	fd=open("foo",O_WRONLY);
	int ia[]={1,2,3,4};
	write(fd,s,sizeof(ia));
	...
```

`cat foo` will print a bunch of strange characters because the shell is evaluating the ASCII values of the digits you are writing into foo.


---

## Wendnesday, 10/25: open/close read/write files by Edmond Wong
**Tech news:[robot bees can now break free of surface tension and get out of water](https://www.theverge.com/2017/10/25/16544996/robot-bees-harvard-fly-swim-water-rocket)

we learned 4 new functions today

```c
#include <fcntl.h>
int open( char * filename, int access_mode, int permission );
```
this opens a file and returns a number to referecne it in the file table
most computers have file tables the size of 256

access modes:

O_RDONLY - read only
O_WRONLY - write only
O_RDWR   - read or write
O_APPEND - append to end of file

permissions: (this parameter is only nessary when a file is being created) so open(<filename>,<accessmode>) works too

we did not go over these permissions in class yet

S_IWRITE
S_IREAD
S_IWRITE | S_IREAD

```c
#include <fcntl.h>
int read( int handle, void * buffer, int nbyte);
```

read reads nbytes from the file and places the read data into buffer.
read returns the number of bytes read and -1 in the case of an error.
handle is the number that open() returns

```c
#include <fcntl.h>
int write( int handle, void * buffer, int nbyte);
```

writes writes from buffer to the file referenced with handle
write returns the number of bytes written and returns -1 if there is an error

```c
#include <fcntl.h>
int close(int handle);
```
close closes the file. it returns 0 if successful and -1 if there was an error.

______________________________________________________________________________

## Tuesday, 10/24: File This Under Useful Information by Anish Shenoy

**Tech News** [Google Is Creating its Own Neighborhood](http://www.independent.co.uk/life-style/gadgets-and-tech/news/google-quayside-toronto-plans-sidewalk-labs-canada-eastern-waterfront-weather-control-loft-buildings-a8015866.html)

### Three Kinds of File Permissions
* Read [r]: File is readable
* Write [w]: File is writeable
* Executable [x]: File is executable
* These three permissions can be represented as 3-digit binary numbers or 1-digit octals (0b denotes binary 0 denotes octal)
	- 0b100 <==> 04 => Read-Only
	- 0b111 <==> 07 => Read, Write, Execute
	- 0b101 <==> 05 => Read, Execute

### Three Permission 'Areas'
There are three permission "areas"
* User, Group, Others
* Others does NOT refer to everyone, it refers to everyone besides User and Group
* Each area can have its own permissions
	- ex: 0644 <=> 101100100 => User can read/write, group can only read, others can only read.

### Viewing and Changing Permissions
* Permissions can be viewed by using the `ls -l` command
* Permissions can be changed with the `chmod` command
	- ex: `chmod 664 broken.c` sets the permissions of broken.c to 664
	- This is a more useful alternative to the `chmod +` command we were used to using
* The Owner can always change the permissions of a file

### Metadata
* The Metadata of a file stores information about the file
	- This information can include things like file size, and permissions


--------------------------------------------------------------------------------

## Monday, 10/23 : The Missing Bits and Pieces by Ashneel Das
**Interesting Tech News:** [AI exhibits bias towards men in many cases, including autonomous vehicles](https://www.technologyreview.com/s/609129/the-dangers-of-tech-bro-ai/)

Integers can be represented in base 2, 8, and 16 by using the following prefixes:
- Binary: 0b
- Octal: 0
- Hexadecimal: 0x

There is NO difference between the values of these integers.
```c
0b1011 == 013 == 0xB == 11
```
Arithmetic in different bases also works.
There are certain operators that can be used on values that are evaluated on each individual bit, known as bitwise operators.

```c
unsigned char c = 0b10110011;
```
Right shift moves all the bits to the right and adds 0s to the front.
```c
c >> 1
```
This makes the value of c 01011001.
Left shift moves all bits to the left and adds 0s to the end.
```c
c << 1
```
This makes the value of c (after the right shift AND left shift) 10110010.
Other bitwise operators are negation, or, and, xor.
- ~: Negation flips every bit. 0s become 1s, and 1s become 0s.
- |: Performs the or operator on every pair of bits in the two numbers.
- &: Performs the and operator on every pair of bits in the two numbers.
- ^: Performs the exclusive or operator on every pair of bits in the two numbers.

--------------------------------------------------------------------------------

## Thursday, 10/19 : Get Dem Bugs by John Lin
**Interesting Tech News:** [Intel says Nervana computer chips will accelerate AI Revolution](https://www.cnet.com/news/intel-says-its-computer-chips-will-accelerate-ai-revolution/)

GDB
- Stands for GNU Debugger
- Tool for debugging a C program
- First compile the program, then call the gdb on the executable
```
gcc broken.c
gdb ./a.out
```
- In the GDB, you can start running the program by calling
```
run
```
- You can place a break point in the program by calling
```
break line_number
```
- You can print the value of a variable by calling
```
print variable_name
```
- When a program stops at a break point, you can continue executing the program until the next break point by calling
```
continue
```

--------------------------------------------------------------------------------

## Wednesday, 10/18: back to the grind by Jasper Cheung
**Interesting Tech News:** [giant robot battle!!! WOW!](https://www.cnet.com/news/robots-megabots-twitch-suidobashi-battle-japan-usa/) [(vid)](https://www.youtube.com/watch?v=Z-ouLX8Q9UM)

Valgrind
- tool for debugging memory issues in C
- (e.g. memory leaks, accessing freed memory, use of intialized memory)
- You must complie with -g to use valgrind (and other similar tools)
```
gcc -g foo.c
```
- then you call valgrind when running the executable
```
valgrind --leak-check=yes ./a.out
```
- you may see different results when running valgrind on different OSs

---

## Friday, 10/13: Calloc and Realloc by Judy Liu

**Interesting Tech News:**[VR could trick stoke victims' brains toward recovery](https://www.cnet.com/news/vr-could-trick-stroke-victims-brains-toward-recovery/)

Calloc - clear allocation
```c
calloc (size_t n, size_t x);
```
- Allocates n*x bytes of memory
- Ensures that each bit is set to 0
- Works like malloc in all other ways

```c
int *p;
p = (int*) calloc (5, sizeof(int));
```

Realloc
```c
realloc(void *p, size_t x);
```

- Changes the amount of memory allocated in a given block
- Like free, p must point to the beginning of the block
- If x is smaller than the original size of the allocation, the extra bytes will be released
- If x is larger than the original size of size then:
 <p> 1. If there is enough space at the end of the original allocation, the original allocation will be updated </p>
OR
<p> 2. If there is not enough space, a new allocation will be created, containing the orginal values (copied over). The orginal allocation will be freed. </p>

```c
int *p;
p = (int *) malloc(10);
p = (int *) realloc(p,40);
```

--------------------------------------------------------------------------------


## Thursday, 10/12: "Malloc & Free: The dynamic duo" by Alexander Shevchenko

**Interesting Tech News:** [Facebook has launcehd a service through which its users can order food online](https://www.reuters.com/article/us-facebook-food/facebook-launches-u-s-food-order-and-delivery-service-idUSKBN1CI1UU)

Dynamic memory allocation:

```c
malloc(size_t x)
```
- Allocates x bytes of memory (from the heap)
- Returns the adress at the beginning of the allocation
- Returns a void * , always typecast to the correct pointer type


```c
int *p;
p = (int *)malloc(5 * sizeof(int));
```

```c
free(void *ptr)
```

- Releases dynamically allocated memory
- Takes one parameter, a pointer to the beginning of a dynamically allocated block of memory
- Every call to malloc/calloc should have a corresponding call to free

```c
int *p;
p = (int *)malloc(20);
free(p);
```

--------------------------------------------------------------------------------


## Wednesday, 10/11: "If you don't pay attention you'll get into a heap of trouble" by Jeffrey Lin

**Interesting Tech News:** [Google Will Hit 100% Renewable Energy this Year](https://web.archive.org/web/20171011014233/https://www.inverse.com/article/37308-google-renewable-energy-goal)

Recall the "stack trace diagrams" used for debugging to make sense of stack
memory.

We can attempt to make a linked list structure like so:

```c
#include <stddef.h> // declares NULL

struct node {
  int data;
  struct node *next;
};

struct node *insert_front(struct node *front, int d) {
  struct node new;
  new.data = d;
  new.next = front;
  return &new;
}

int main() {
  struct node *list = NULL;
  list = insert_front(list, 12);
  return 0;
}
```

Note that the compiler generates a warning:

```
tmp $ clang x.c                                                      
x.c:12:11: warning: address of stack memory associated with local variable 'new'
      returned [-Wreturn-stack-address]   
  return &new;       
          ^~~        
1 warning generated.
```

The node is a piece of stack memory. When `insert_front` ends, *new* is popped off
the stack.
The value might still be there, but "might does not make right".

You need to use dynamically allocated memory to create a linked list.
Heap memory stores dynamically allocated memory.
Data will remain in the heap until it is released (or the program terminates).
Heap memory can be accessed through pointers, and can be accessed across many
functions.

--------------------------------------------------------------------------------

## Tuesday, 10/10: "If you don't pay attention you'll get into a heap of trouble" by Jeffrey Lin

**Interesting Tech News:** [Hacking is inevitable, so it's time to assume our data will be stolen](https://archive.is/EVOUi)

We use the `.` operator to access a value inside a struct.

```c
struct {int a; char x;} s;
s.a = 10;
s.x = '@';
```

You may not put a struct within itself, as that creates a compile-time error:

```c
int main() {
  struct foo {int a; char x; struct foo f;};
}
```

```
tmp $ clang x.c
x.c:2:39: error: field has incomplete type 'struct foo'
struct foo {int a; char x; struct foo f;};
                                      ^
x.c:2:8: note: definition of 'struct foo' is not complete until the closing '}'
struct foo {int a; char x; struct foo f;};
       ^
1 error generated.
1 tmp $
```

However, you may put a pointer to that struct within itself:

```c
int main() {
  struct foo {int a; char x; struct foo *f;};
}
```

This allows us to create a data structure similar to nodes and linked lists.

The `.` operation binds before the `*` operation.

```c
struct foo *p;
p = &s; // access the value of p.x using (*p).x
```

To access data from a struct pointer, you may also use `->`:

```c
p->x // same as (*p).x
```

You can typedef structs, but you must be careful as this will hide the fact that
you are working with a struct.

Moving on to stack memory versus heap memory:

Computer programs separate memory usage into two parts: the stack and the heap.
Every program can have its own stack and heap.

Stack memory stores all normally declared variables (including pointers and
structs), arrays, and function calls.
Functions are pushed onto the stack in the order they are called, and popped off
when completed.
When a function is popped off the stack, the stack memory associated with it is
released.

--------------------------------------------------------------------------------

## Friday, 10/3 | Finding your type | Arif Roktim

**Tech News:** [Google Fiber Scales Back TV Service To Focus Solely On High-Speed Internet](https://hothardware.com/news/google-fiber-scales-back-tv-service-to-focus-solely-on-gigabit-internet)

On Friday, we went over some commands involving types.

`typedef`: Provides a new name for an existing type  
Syntax: `typedef <real type> <new name>;`  
Reasons for using typedef include compatibility and readability.  
For example, strlen() returns type size\_t, which is typedef'd to `unsigned long`. It's clear that the function is returning a size.  
Usage:
```c
typedef long lolcat;
lolcat rainbow = 5;
```

We also went over `struct`. Structs are used to create a new type that is a collection of values.
Syntax:
```c
struct{ int a; char x; } var0;
struct{ int a; char x; } var1;
struct{ char *s1; char *s2; } var2;
printf("sizeof var0: %lu", sizeof(var0));
```
The type of variable `var0` is `struct{ int a; char x; }`.

It's annoying to have to type `struct{ int a; char x; }` over and over again so there is prototyping. Prototyping allows you to name a struct type.  
Syntax:
```c
struct foo {int a; char x;};
// struct foo is a prototype to be used later
struct foo s;
printf("sizeof s: %lu", sizeof(s))
```
Notice how the prototype has to be preceded by `struct`.

You can also make a prototype and instantiate a new variable at the same time:
```c
struct bar {int a; char x;} t;
struct bar u;
```

We didn't go over how to access/modify the collection of values in a struct.

---

## Tuesday, 10/3 | Make it So | Asim Kapparova

**Tech News:** [drone deliveries using codes flashed from your phone’s LED](https://www.theverge.com/2017/10/4/16417986/delivair-drone-concept-led-flash-coded-cambridge-consultants)

Multiple File Compilation
- You can combine multiple C files into a C program by including them all in one gcc command.
-`gcc test.c string.c foo.c woohoo.c`
- You cannot have duplicate functions or global variable names across these files.
- Be careful: only one of these files should have a main

Using `gcc -c`
- By using `gcc -c cfile.c` a file is created with the name cfile.o
- It is not a functional program, but it is compiled code.
- This is primarily used to compile c files without a main function.
- .o files can be linked with each other or with other c fiels by compiling them all at once.
- `gcc a.o b.c c.c`

Make
- Create compiling instructions and setup dependencies.
- Standard name used for make is makefile.

Syntax::
```make
<TARGET>: <DEPENDENCIES>
	<RULES>
```

For Example:
```make
all: dwstring.o main.o
	gcc -o strtest dwstring.o main.o

dwstring.o: dwstring.c dwstring.h
	gcc -c dwstring.c

main.o: main.c dwstring.h
	gcc -c main.c

clean:
	rm *.o
	rm *~

run: all
	./strtest
```

- Running a makefile: `make`
  - This will automatically make the first target. Which will recursively check if any dependencies need to be updated and then update them.
  - You can also include a target `make <TARGET>` and this will make the target specified recursively.
- The first target is typically a file that doesn't exist such that it is always updated.
- We also usually include a clean target to remove junk files and the like.
- Similarly we include a run target, that runs the final product.

Make is a good way to manage large processing of compiling/running code.

---
## Friday, 9/29 | Functions that Deal with Strings | Haoyu Chen

**Interesting Tech News:** [Belkin - Apple Dongle has lighthing port and headphone jack](https://www.theverge.com/circuitbreaker/2017/10/1/16393078/apple-belkin-rockstar-iphone-adapter-headphone-lightning)

**DO NOW**: Discuss the functions that you researched for homework with classmates
```C
#include <string.h>
char *strcpy(char *dest, char *src);
char *strncpy(char *dest, char *src, size_t n);
```
#### strcpy(), strncpy() : ####
These functions copy a string from one address to another, stopping at the NUL terminator on the srcstring.
strncpy() is just like strcpy(), except only the first n characters are actually copied. Beware that if you hit the limit, n before you get a NUL terminator on the src string, your dest string won't be NUL-terminated.
Both functions return dest for your convenience, at no extra charge.
```C
#include <string.h>
int strcat(const char *dest, const char *src);
int strncat(const char *dest, const char *src, size_t n);
```
#### strcat(), strncat() ####
These functions take two strings, and stick them together, storing the result in the first string.
These functions don't take the size of the first string into account when it does the concatenation.
Both functions return a pointer to the destination string, like most of the string-oriented functions.
```C
#include <string.h>
int strcmp(const char *s1, const char *s2);
int strncmp(const char *s1, const char *s2, size_t n);
```
#### strcmp(), strncmp() ####
strcmp() compares the entire string down to the end, while strncmp() only compares the first n characters of the strings.
Returns zero if the strings are the same, less-than zero if the first different character in s1 is less than that in s2, or greater-than zero if the first difference character in s1 is greater than than in s2.
```C
#include <string.h>
char *strchr(char *str, int c);
char *strstr(const char *str, const char *substr);
```
#### strchr(), strstr() ####
The functions strchr() finds the first occurance of a letter in a string
strchr() returns a pointer to the occurance of the letter in the string, or NULL if the letter is not found.
Let's say you have a big long string, and you want to find a word, or whatever substring strikes your fancy, inside the first string. Then strstr() is for you! It'll return a pointer to the substr within the str!
strstr() returns a pointer to the occurance of the substr inside the str, or NULL if the substring can't be found.

---

## Thursday, 9/28 | A string of functions | by Angelica Zverovich
**Interesting Tech News:** [Liquid Nitrogen Drives 18-Core Core i9-7980XE Above 6GHz](https://www.extremetech.com/computing/256623-liquid-nitrogen-drives-18-core-core-i9-7980xe-6ghz)

**DO NOW**: Write a function to find length of strings.

There are many possible ways to do this.
Here is one example using bracket notation:
```C
int length(char *s) {
  int i = 0;
  while (s[i]) { //not necessary to write s[i] != 0 as the conditional because 0 is false in C
    i ++;
  }
  return i;
}
```
Here is another example using pointers:
```C
int length (char *s) {
    int i = 0;
    while (*s++) { //you can imcrement the pointer in the while loop statement
        i ++;
    }
    return i;

}
```

Note that while it is possible to make the argument a char[] instead of a char * , any array is automatically passed as a pointer to the function, so there is no purpose in doing so.

#### Declaring Strings ####
* `char *s2 = "Mankind"`  

   Creates immutable string "mankind" and returns a pointer to that string.You cannot modify strings created this way because the memory the pointer points to is immutable.

* You can only assign char arrays with = at declaration

   `char s[] = "hello"; //ok!`             
   `s[] = "seven"; //NOT ok`

   Array are immutable pointers, you can't reassign an array pointer.

* Char pointers can assigned using = at any time

    `char * s = "hello"; //ok!`                  
    `s = "seven"; //also ok!`            

---


## Wednesday, 9/27 cstrings by Eric Zhang

**Interesting Tech News:** [Some websites are using visitors' computers to mine cryptocurrency as an alternative to running ads](https://www.theguardian.com/technology/2017/sep/27/pirate-bay-showtime-ads-websites-electricity-pay-bills-cryptocurrency-bitcoin?CMP=twt_gu)

Today we used our knowledge of arrays and pointers to understand how strings work in c.

### Strings in c
Strings as character arrays
* Strings in c are simply character arrays.
* By convention, the last character in a string is a NULL character, denoted by `0`, or `'\0'`.
* This allows the end of a string to be found, as otherwise there is no way to find the length of an array.
* When printing a string, for example, everything from starting from the pointer location to the next null character is printed.

String literals
* When creating a string literal using `""`, an immutable string is created in memory, and is automatically null terminated.
* Consequently, all references to the same string literal reference the same immutable string in memory.

Declaring strings
* `char s[256];` is a normal array declaration, allocating 256 bytes to character array s.
* `char s[256] = "Imagine";` allocates the memory to s, then creates and copies the immutable string "Imagine", including the terminating null character.
* `char s[] = "Imagine";` creates the immutable string "Imagine", then allocates the appropriate amount of memory to s and copies it.

---

## Tuesday, 9/26 Let's Head to Function Town by Cesar Mu

**Interesting Tech News:** [Twitter just doubled the character limit for tweets to 280](https://www.theverge.com/2017/9/26/16363912/twitter-character-limit-increase-280-test)

Today we continued our discussion on passing by value and array variables, and learned of new ways to fix function declaration orders.

### Passing Arrays Into Functions
* Functions that take arrays as arguments can use pointer variables as their parameters.
```C
float arr[5];
foo(arr);

void foo(float *x){...} // *x will refer to the memory address of the array "arr"
```
* This works because array variables are inherently immutable pointers.

### Declaring Functions
* C is a procedural language, meaning that the order of the functions matters.
* Previously, we would just place any helper functions we needed at the top of the code and call it a day, but this will run into issues when we have cyclical functions that call on themselves.

* #### We have 2 better ways to fix this issue:
    1. Declare the function header at the beginning of the code, then define later.
        * You don't need to provide names for the parameters, you just need the data types.
        ```C
        void print_array(int, int); // beginning of the code
        ...
        void print_array(int x, int y){...} // later on in the code
        ```
    2. Put the function headers in a separate file, then `#include` the header file.
        * When `#include`ing, the header file name must be in quotes.
        ```C
        // in "my_header.h"
        void print_array(int, int);

        // in "program.c"
        #include "my_header.h"
        ```

### Other Interesting Things
* An array variable points to a memory address, but that variable itself also has the same memory address.
* There is no way to determine what data type an array holds, because the collection of 0s and 1s in memory can be interpreted in any way, as int, float, double, etc.

---

## Monday, 9/25 How to Write Functioning Code, by Michael Lee

**Interesting Tech News:** [Poor coding limits IS hackers' cyber-capabilities, says researcher](http://www.bbc.com/news/technology-41385619)

We started class with Alex presenting his homework code. Many of us had code that looked something like this:
```C
int *p;
p = &array1[9-i];
array2[i] = *p;
```

While this code is correct, `&array1[9-i]` is redundant. If we do `array[index]`, that is shorthand for `*(array + index)`. A smarter way to do this would be `p = array + 9 - i`, which uses addition of pointers.

Then we looked at an extreme case of C funkiness. Because array indexing is simply just pointer addition, the commutative property applies. Therefore:

```C
array[25]
```
and
```C
25[array]
```
yield the same result. However, good programmers would never do this because it is a headache to read.



#### Functions
C functions are structured similar to their java counterparts:
* returntype name(parameters){...}
Arguments are pass by values, meaning a copy of them is stored and used in the body of the function. This means if you have code like this:
```C
void foo(int a){
    a++;
}

int main(){
    int b = 3;
    foo(b);
    printf("%d\n", b);
    return 0;
}
```
the output value of b would still be `3`.

---















## Wednesday, 09/20 Try Not to Hurt Yourself, the Point is Sharp by Jeffrey Luo

**Interesting Tech News**: [Researchers are using machine learning to predict earthquakes](https://phys.org/news/2017-08-machine-learning-earthquake-lab.html)

Today, we were introduced to pointers and how they work alongside memory addresses. Pointers are a relatively new concept as they aren't explicitly featured in Java. Below are the main points we covered in class.

### Pointers and their syntax
* Pointers simply are variables that store a memory address.
* Because they're used to store addresses, all pointers are the same size, regardless of type. Typically, they are either 4 or 8 bytes large depending on architecture (32 vs 64 bit).
* Pointers are denoted by using the asterisk (`*`).
* Memory addresses can be obtained by using the ampersand (`&`) before the variable. Ex: `&a` returns the address of variable `a`.
* Addresses stored in pointers can be printed using the `%p` format modifier. Ex: `printf("%p\n", p)` prints the memory address of pointer `p`.

Example:
```C
int x = 4;
int *ip; // creates an integer pointer
printf("Address of x: %d\n", &x); // prints "Address of x: 15288" (for the sake of this example)
ip = &x; // initializes pointer 'ip' by assigning it the address of variable 'x'
```
We can the visualize the above as:

|         |    x    |       ip     |
|--------:|:-------:|:------------:|
|  Value  |      4  |     15288    |
| Address |  15288  | (not needed) |

* Pointers can be dereferenced  also by using the asterisk (`*`). This returns the value of the data which the pointer points to. Continuing from above:
```C
printf("Value of x: %d\n", *ip); // prints "Value of x: 4"
```
* Finally, remember that pointers themselves have memory addresses! After all, they are variables themselves.

### Pointer Arithmetic

* As pointers store memory addresses, they can be incremented at a size corresponding to the data type. Because of this, we can access different memory addresses relative to the initial memory address.
Example:
```C
int a = 10;
int *ip = &a;
char b = 'a';
char *cp = &b;
ip++; // increments ip by 4 bytes (size of int)
cp++; // increments cp by 1 byte (size of char)
```


---

## Monday 09/18 Data Types and Variables By Daniel Regassa

**Interesting Tech News**: [Intel is working with Waymo to build fully self-driving cars](https://www.theverge.com/2017/9/18/16328284/intel-waymo-partnership-self-driving-car-chrysler)

### Character and String Literals
* Just to refresh our memories, in programming languages a literal is a value that can never be changed after
    it is declared, while on the contrary, a variable represents a value that can be redeclared indefinitely.
* In C, there exist both character and string literals (even though there is no "string" data type)
* Character Literals are delimited by single quotes like so: `'a'`
* String Literals are delimited by double quotes like so: `"hello, world!"`

### Unsigned Integers
* All variables of the primitive data types can be initialized with the `unsigned` keyword at the front
* Ex: `unsigned int foo = 42;`
* Unsigned variables can only take up non-nengative values
* They can hold up to twice as many positive numbers as their "signed" counterparts
* For example, a char can hold the values from -128 to 127, while an unsigned char can hold the values 0 to 255

### Booleans
* Unlike other programming languages like Java or Python, there is no boolean data type.
* Instead, in C we use integer values for booleans
* `0` corresponds to false, and any other integer corresponds to true

**WARNING**: Don't do this unintentionally:

 `if(x = 6){...}`

When you mean to do this:

 `if (x == 6){...}`

As in the former the assignment operator will return the value of the integer literal, in this case 6, which will always
be interpreted as true.

### Memory Management (to be continued in the next lesson)
* Memory in C is allocated either at compile time (static memory) or run time (dynamic memory)

#### Compiler Allocated Memory
* Packaged with the compiled binary file
* The variables and arrays we have been using so far use this type of mamory
* Variables not have a standard default value, like they would in other languages like Java (the default being `0` for many
data types)
- Instead a variable that has not been declared yet will use whatever value already resides in that spot in memory
- This is because C's philosophy is to do only exactly what we tell it to, and auto-declaring every variable to a default
value would be a wasteful operation. C assumes that if we wanted to set something, we would on our own

---

## Thursday 09/14 Always Read the Fine Print By Ricky Chen

**Interesting Tech News:** [OpenAI bot beats professional DoTA2 players.](https://blog.openai.com/dota-2/) [(Further reading)](https://blog.openai.com/more-on-dota-2/)

Do Now: List the Java primitives.
* int
* double
* float
* char
* long
* short
* byte
* boolean

### All About C Primitives
In C, there are no byte or boolean primitives. All C primitives are numerical, which means that char is expressed using ASCII.

These primitives differ in both their sizes and the content which they can store. int, short, and long are used for integers while double and float are for floating point numbers. The following table should give insight into the the size of each primitive:

| Primitive | Size (Bytes)| Range |
| ------------- |:-------------:| -----:|
| int | 4 | ~ -2 billion to 2 billion |
| char | 1 |    |
| double | 8 | |
| float | 4 |  |
| long | 8 | 14 digits of percision |
| short | 2 | 7 digits of percision |

### All About Printf
The function `printf(<string literal>, var1, var2...)` is essential to the C library. It is found in the stdio.h library and can be used by adding `#include <stdio.h>` to your code.
The function itself can take one argument that must be a string, **not a variable.** If you wish to append a variable to an existing string, you may do something like the following:
```C
int i = 29;
printf("The number is: %d\n", i);
```
Notice the use of `%d`. This marks the spot for which the variable will be placed into the string. `%d` is not universal formatting for all data types; thus that the following table should be of use:

| Type | Formatting|
| ------------- |-------------:|
| int | %d |
| char | %c |
| double | %lf |
| float | %f |
| long | %ld |
| pointer | %p |

As a side note, int is paired with `%d` due to it being a *d*ecimal number.

Following this, we were asked to create a C program that would first print a variable correctly and then incorrectly. This was my code for correctly:
```C
#include <stdio.h>

int x = 42;
double y = 28.6668;
char z = 's';

int main(){
    printf("Int is: %d\n", x);
    printf("Double is: %lf\n", y);
    printf("Char is: %c\n", z);
    return 0;
}
```
This was what was shown in the terminal:
>Int is: 42

>Double is: 28.666800

>Char is: s

Now, for doing it incorrectly, I modified my main class to be:
```C
int main(){
    printf("Int is: %lf\n", x);
    printf("Double is: %c\n", y);
    printf("Char is: %d\n", z);
    return 0;
}
```
Which resulted in:
>Int is: -0.026042

>Double is:

>Char is: 115

Interesting to note is that I did not get a warning for the third printf call.

---

## Wednesday, 9/13 If Variables are the Spice of Life, Then are They SpiceC? By Queenie Xiang

**Interesting Tech News:** [Facebook's Artificial Intelligence Robots Get "Shut Down"](http://www.independent.co.uk/life-style/gadgets-and-tech/news/facebook-artificial-intelligence-ai-chatbot-new-language-research-openai-google-a7869706.html)

The aim for today's class was "Variables are the spice of life." Our do now was to run <pre><code>$man stdio</code></pre> and share what we learned.

We found out that this displayed the manual and documentation for stdio. It shows the name, library, synopsis, description, standards, list of functions, and bugs.

Generally, <code>man</code> represents the manual where you can look up the details of a command or program you're not familiar with. So for example, if you wanted to look up the documentation on <code>ls</code>, you would type in <pre><code>$man ls </code></pre> Each command/program has a section of the manual it belongs to. The number of the section appears in the top right corner.

To look up something in a particular section:
<pre><code> man section# command/program_name </code></pre>
For example, <code>printf</code> exists in many sections. If you wanted to look at the <code>printf</code> in section 3 instead of section 1, you would type:
<pre><code>$man 3 printf</code></pre>

In the manual, if <code>...</code> is in a function headline, it means the function can take an arbitrary number of arguments.

To include librarys and use functions defined in other files:
* (1) Check that you're using the function correctly: the arguments and return type must match what the function defined.
* (2) Link the code for the external function to your executeable code: GCC can automatically link functions in standard libraries and it grabs ONLY what is needed.

Use: <pre><code> #include </code></pre> (used during step 1 above)
* Used to include function headers with your code: this tells the compiler to look and check for these functions.
* It is necessary to match arguments and return type.
* It's not necessary for linking(step 2 above).
* If it's a standard C library, use angle brackets: <pre><code> <standard_c_library_name> </code></pre> (otherwise, don't).

An example of this would be:
```C
#include <stdio.h>
```

By the way, the # in <code> #include </code> allows the include to be interpreted before all other code.

---  

## Tuesday, 9/12 The ABCs of C by Michael Cheng

**Interesting Tech News:** [Meet the iPhone X, Apple's New High-End Handset](https://www.wired.com/story/apple-iphone-x-iphone-8/)

We started class by creating the classic "Hello, World!" program in C:
<pre><code>int main(){
   printf("Hello, World!\n");
   return 0;
}</pre></code>

After compiling with <code>$ gcc hello.c</code> and executing with <code>$ ./a.out</code>, the resulting output was <code>Hello, World!</code> with a newline at the end. There is no <code>println()</code> in C, so a newline must be added manually, if desired.

We noticed that there are striking similarities to Java, such as in syntax, with the are curly brackets and semi-colons. C also requires a main function, similar to Java, but note that it is not a facsimile of Java's, as shown above. The main function must return a value. Furthermore, C is a procedural language, not an object-oriented language.

We then went over the reasons why <code>./</code> is used when executing C programs. For one, it is used for security, and because it is shorthand for the current directory. An error will occur if <code>./</code> is not used, as bash will look for the command in locations described in the <code>$PATH</code> environmental variable only.

Then we had went over briefly on the history of C:
* C was developed at Bell Labs by Dennis Ritchie in the early 70s while he was working on UNIX. C was then used to rewrite UNIX.
* Dennis Ritchie and Brian Kernighan published *The C Programming Language* in 1978, a reference book for the C programming language.

---  

## Monday, 9/11 Jumping into the C by Daria "Dasha" Shifrina

**Interesting Tech News:** [Robot Learns to Follow Orders like Alexa](http://news.mit.edu/2017/robot-learns-to-follow-orders-like-alexa-0830)

Today, we were introduced to C for the first time. We started off by contrasting Java and C. Unlike C, Java provides a JVM(java virtual machine) that compiles your code to run in different computing environments. In C, you do not have a virtual machine so you must manually recompile when switching between operating systems.

Then, we learned that there are several conventions when it comes to C:
* Put in .c files
* snake_case
Ex: black_mamba.c

We also learned how to compile a C file. To compile, you must use the gcc(gnu c compiler). One "inconvenience" of C is that it doesn't recognize your file name when you compile the code, instead creating an output file called "a.out", which stands for "assembler output." There are ways to change that output file name. Here's an example of what you'd write in your terminal if you were trying to compile black_mamba.c:
<blockquote>

    $ gcc black_mamba.c #Creates a.out files  

    $ ./a.out #output of black_mamba.c  

    $ gcc -o snake black_mamba.c #rename a.out to snake  

    $ ./snake #output of black_mamba.c
</blockquote>

---

## Wednesday, 9/6 Blissfull respite, I hardly knew ye by Clyde "Thluffy" Sinclair

**Interesting Tech News:** [Do You Even Olin?](https://blog.ledwards.com/the-college-that-produces-founders-at-3-times-the-rate-of-stanford-2c53ea44f91e)

Don't forget, each new post should go above the one before. If you are late in posting, please put yours in the correct position.

#### Look, it's a list! ####
* Blah
* Blah
* Blah

Below you'll find a horizonal bar, please put one at the end of your post.

---
