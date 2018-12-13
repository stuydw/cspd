## Monday, 1/8 Stop. Collaborate, and listen by Henry Zheng

**Tech News:** [Senate bill to reverse net neutrality repeal gains 30th co-sponsor, ensuring floor vote](http://thehill.com/policy/technology/367929-senate-bill-to-reverse-net-neutrality-repeal-wins-30th-co-sponsor-ensuring)

**Bonus:** [Nebraska Introduces Law to Reinstate Net Neutrality](https://www.inverse.com/article/39994-nebraska-proposes-reinstate-net-neutrality)

## Server Only Functions
### listen - `<sys/socket.h>`
- Set a socket to passively await a connection.
- Needed for stream sockets.
- Does not block.
#### listen(socket descriptor, backlog)
- socket descriptor
	- return value of socket
- backlog
	- number of connections that can be queued up
	- depending on the protocol, this may not do much

### accept - `<sys/socket.h>`
- Accept the next client in the queue of a socket in the listen state.
- Used for stream sockets.
- Performs the server side of the 3 way handshake.
- Creates a new socket for communicating with the client, the listening socket is not modified.
- Returns a descriptor to the new socket.
#### `accept(socket descriptor, address, address length)`
- socket descriptor
	- descriptor for listening socket
- address
	- pointer to a `struct sockaddr_storage` that will contain information about the new socket after accept succeeds
- address length
	- pointer to the variable that will contain the size of the new socket address after accept succeeds

Using listen and accept:
```c
//create socket
int sd;
sd = socket(AF_INET, SOCK_STREAM, 0);

//use getaddrinfo and bind

listen(sd, 10);

int client_socket;
socklen_t sock_size;
struct sockaddr_storage client_address;

client_socket = accept(sd, (struct sockaddr *)&client_address, &sock_size);
```

## Client Only Functions
### connect - `<sys/socket.h>`, `<sys/types.h>`
- Connect to a socket currently in the listening state.
- Used for stream sockets.
- Performs the client side of the 3 way handshake.
- Binds the socket to an address and port.
- Blocks until a connection is made (or fails).

#### `connect(socket descriptor, address, address length)`
- address
	- pointer to a `struct sockaddr` representing the address
- address length
	- size of the address, in bytes
- address and address length can be retrieved from `getaddrinfo()`
- Notice that the arguments mirror those of `bind()`.

Using connect:
```c
//create socket
int sd;
sd = socket(AF_INET, SOCK_STREAM, 0);

struct addrinfo * hints, * results;
// use getaddrinfo (not shown)

connect(sd, results->ai_addr, results->ai_addrlen);
```

---
## Friday, 1/5 Stop. Collaborate, and listen by Mansour Elsharawy

**Tech News:** [Spectre and Meltdown Security Bugs Discovered Simultaneously in our Processor Chips](https://www.wired.com/story/meltdown-spectre-bug-collision-intel-chip-flaw-discovery/)

_Finally, how to do networking in C!_

#### Extra Libraries That You Haven't Seen Before But Will Need Now
```
<sys/socket.h>
<netdb.h>
```

### socket()
* Creates a socket!
* Returns a socket descriptor (an `int` that works like a file descriptor)
* Syntax: `socket(DOMAIN,TYPE,PROTOCOL)`
* DOMAIN is the type of address to be used, use `AF_INET` or `AF_INET6` for IPv4 and IPv6 addresses, respectively.
* TYPE is the type of socket you plan to use, use `SOCK_STREAM` for stream socket and `SOCK_DGRAM` for datagram socket.
* PROTOCOL is the combination of domain and type - If set to 0, the OS will set it to the correct type (TCP or UDP) (Best to just set it to 0)

**Example:**
```c
int sd = socket(AF_INET,SOCK_STREAM,0);
```

### struct addrinfo

The system library uses `struct addrinfo` to represent network address information (like the IP address, port, and protocol)
Here are some of the fields we will need to use and the values we can set them to:

#### .ai_family
* `AF_INET` for an IPv4 address
* `AF_INET6` for an IPv6 address
* `AF_UNSPEC` for an unspecified type of address

#### .ai_socktype
* `SOCK_STREAM` as specified above
* `SOCK_DGRAM` as specified above

#### .ai_flags
* `AI_PASSIVE` Automatically sets to any incoming IP address (good for servers!)

#### .ai_flags
* A pointer to a `struct sockaddr,` which contains the IP address

#### .ai_addrlen
* Size of the IP address


### getaddrinfo()
* This method is used to lookup information about the desired network address and get one or more matching `struct addrinfo` entries.
* Syntax: `getaddrinfo(NODE,SERVICE,HINTS,RESULTS)`
* NODE is a string containing IP address or hostname to lookup - If set to `NULL`, it will use the loopback address.
* SERVICE is a string with a port number or service name (if the service is in `/etc/services`) (And yes, the port number is in a string.)
* HINTS is a pointer to a `struct addrinfo` used to provide the settings for the lookup.
* RESULTS is a pointer toa `struct addrinfo` that will be a linked list containing entries for each matching address (memory allocation handled automagically!).

### bind() (used by servers only)
* Binds a socket to an address and port
* Returns 0 on success, -1 on failure (and sets our good old friend errno)
* Syntax: `bind(SOCKET_DESCRIPTOR,ADDRESS,ADDRESS_LENGTH)`
* `SOCKET_DESCRIPTOR` is the return value of `socket()`
* The `ADDRESS` and `ADDRESS_LENGTH` fields can be retrieved from the results of `getaddrinfo().`

### listen() (used by servers only)
* Set a socket to passively wait for a connection.
* Necessary for stream sockets
* DOES NOT BLOCK! THIS IS NOT READ()!
* Syntax: `listen(SOCKET_DESCRIPTOR,BACKLOG)`
* `BACKLOG` is the maximum number of connections that can be queued up at a time (may not do much depending on protocol)

...And that's all folks! (For now)

---

## Wednesday, 1/3 Socket to Me by Sonal Parab

**Tech News:** [Amazon Patents Blended Reality Mirror for Virtual Dress-Up](https://www.geekwire.com/2018/amazon-patents-blended-reality-mirror-shows-wearing-virtual-clothes-virtual-locales/)

### Network Ports  
Allow a single computer to run multiple services.  
* A socket combines an IP address and port.  
  
Each computer has 2^16 (65,536) ports.   
Some ports are reserved for specific services.
* 80: http  
* 22: ssh  
* 443: ssl

You can select any port, as long as it won't conflict with a service running on the desired computer.   
* ports < 1024 are reserved and should generally not be used
* `/etc/services` will have a list of registered ports for your local system

### Network Connection Types

#### Stream Sockets  
* Reliable 2 way communication.  
* Must be connected on both ends.  
* Data is received in the order it is sent. (not as easily done as it sounds)  
* Most use the Transmission Control Protocol (TCP).  

#### Datagram Sockets  
* "Connectionless" - an established connection is not required.  
* Data sent may be received out of order (or not at all).  
* Uses the User Datagram Protocol.    

---
## Tuesday, 1.2.18: Socket to Me, by Ida Wang

**Tech News:** [Gaming addiction classified as disorder by WHO](http://www.bbc.com/news/technology-42541404)

### Socket
A connection between 2 programs over a *network*.  
A socket corresponds to an IP (internet protocol) Address / Port pair.

#### To use a socket
1. create the socket (kind of like creating the WKP)
2. bind it to an address and port
3. listen (creator) / initiate a connection (client)
4. send / receive data

### IP Addresses
*All* devices connected to the internet have an IP address.  
IP addresses come in 2 flavors, IPv4 (the main standard) and IPv6 (what should be the main standard).  
Addresses are allocated in blocks to make routing easier.

**IPv4:** 2 byte addresses of the form `[0-255].[0-255].[0-255].[0-255]`  
- Each group is called an *octet*.  
- At most there are 2^32, or ~4.3 billion IPv4 addresses.

**IPv6:** 16 byte addresses of the form: `[0-ffff]:[0-ffff]:[0-ffff]:[0-ffff]:[0-ffff]:[0-ffff]:[0-ffff]:[0-ffff]`  
- Each group is known as a *hextet* (although not as standard as the octet).  
- Leading 0s are ignored.  

Any number of consecutive all 0 hextets can be replaced with `::`

For example:  
`0000 : 0000 : 0000 : 0000 : 004f : 13c2 : 0009 : a2d2`  
can also be written as  
`:: 4f : 13c2 : 9 : a2d2`

---
## Monday, 12/18 Always tip your servers by Jerome Freudenberg

**Tech news:** [Twitter cracks down on hate](https://www.washingtonpost.com/news/the-switch/wp/2017/12/18/twitter-purge-suspends-account-of-far-right-leader-who-was-retweeted-by-trump/?utm_term=.56ee6be9cd9b)

#### Homework Tips
* You can pass variable (ACK) to confirm connections between server and client
  * a more robust method would be to pass an integer and perform an operation on it
* Put the server in a forever loop and then place a while loop inside that runs until the client exits
  * Since the pipe is not removed, use sighandler to remove it then exits

#### Forking Server/Client Design Pattern
* Handshake
  1) Client connects to server and sends the private FIFO name. Client waits for a response from the server.
  2) Server receives client’s message and forks off a subserver
  3) Subserver connects to client FIFO, sending an initial acknowledgement message
  4) Client receives subserver’s message, removes its private FIFO

* Operation
  1) Server removes WKP and closes any connections to the client
  2) Server recreates WKP and waits for a new connection
  3) Subserver (wkp) and client (pp) send information back and forth

* Pros and Cons
  * Simple way to handle multiple clients at once
  * Downside is a lack of communication among clients and subservers (which may not be necessary)


---


## Monday, 12/11 Creating a handshake agreement by William Hong

**Tech news:** [All abord the Tesla bandwagon](https://www.investopedia.com/news/teslas-selfdriving-truck-sees-early-success/)

Do-Now: Consider a program that uses pipes in order to communicate between 2 separate executable files. One file is a "server" that is always running. The other is a "client." Design a process by which both files can connect to each other and verify that each can send and receive data. Try to keep it as simple as possible

#### Handshake:
A procedure to ensure that a connection has been established between 2 programs. Both ends of the connection must verify that they can send and receive data to and from each other

#### 3 way handshake setup
This is useful for handling inter-process communications that keep sending data back and forth
1) Client sends a message to the server
2) Server sends a response to the client
3) Client sends a response back to the server

#### Basic server/client design pattern:
* A) Setup:
  * 1) Server creates a FIFO (Well Known Pipe) and waits for a connection
  * 2) Client creates a “private” FIFO, but it won’t starting listening just yet
* B) Handshake:
	* 1) Client connects to server (WKP) and sends the private FIFO name(tells server how to connect and verifies that 		client sent the data)
 	* 2) Client waits for a response from the server
  	* 3) Server receives client’s message and removes the WKP (everyone knows the name of the WKP, which if messed with 		 can disrupt flow of processes; WKP is security concern)
	* 4) Client receives server’s message, removes its private FIFO
	* 5) Client sends response to server
* C) Operation
	* 1) Server and client send info back and forth
* D) Reset
  * 1) Client exits, server closes any connections to the client
  * 2) Server recreates the WKP and waits for another client

Note that shared memory size is limited and what data we can put into shared memory is also limited; a pipe is more ideal for this kind of pattern

---

## Thursday, 12/07 What's A Semaphore? To Control Resources! by Jackie Xu

**Interesting Tech News:** [Facebook Usage Among Teens Set to Drop in U.S.](https://www.bloomberg.com/news/articles/2017-08-21/facebook-usage-among-teens-set-to-drop-in-u-s-emarketer-says)

Don't forget, each new post should go above the one before. If you are late in posting, please put yours in the correct position.

### More Semaphore Methods!

#### semctl
* function description on website under [Work 14](http://www.stuycs.org/courses/mks65/dw/work/work14getsemcontrol)
* `<DATA>` argument
    * variable for setting/storing semaphore metadata
    * type union semun
        * you have to declare this union in your main c file on linux machines
        * union: structure designed to hold only one value at a time from a group of potential values
			* unlike a struct in which all values are held at the same time
			* i.e. (for the homework assignment due today) it only holds the int value

#### union semun
```C
union semun {
    int val;
    struct semid_ds *bug;
    unsigned short *array;
    struct seminfo *__buf;
};
```

#### semop
* perform an atomic semaphore operation
* you can up/down a semaphore by any integer value, not just 1
* `semop( <DESCRIPTOR>, <OPERATION>, <AMOUNT>)`
* `<DESCRIPTOR>`: what's returned from semget
* `<AMOUNT>`: the amount of semaphores you want to operate on in the semaphore set
* `<OPERATION>`: a pointer to a struct sembuf


#### struct sembuf
```C
struct sembuf {
    short sem_op;
    short sem_num;
    short sem_flag;
}
```

* sem_num: index of semaphor you want to work on
* sem_op:
    * down(s): any negative number
    * up(s): any positive number
    * 0: block until semaphore reaches 0
* sem_flag:
    * SEM_UNDO: allow the OS to undo the given operation. userful in the event that a program exits before it could release a semaphore (basically you should always include this)
    * IPC_NOWAIT: instead of waiting for the seamphore to be available, return to err


---

## Tuesday, 12/05 Semaphores by Max Chan

**Tech News**:  [How the Bot Stole Christmas](https://www.nytimes.com/2017/12/06/business/bots-shopping-christmas-holidays.html?rref=collection%2Fsectioncollection%2Ftechnology&action=click&contentCollection=technology&region=stream&module=stream_unit&version=latest&contentPlacement=2&pgtype=sectionfront)

We learned what semaphores are, and how to deal with them.

### Semaphores
* IPC construct used to control access to a shared resource (like a file, shared memory, etc.)
* Commonly used as counter representing how many processes can access at a time
* Counter of 0 means unavailable
* Operations are atomic, meaning it **WILL NOT** split into multiple processor instructions

### Semaphore operations
The following operations are non-atomic:
* Create a semaphore
* Setting init value
* Remove a semaphore

The initial value of a semaphore is 0

Up(s) / V(s)
* Atomic
* Release the semaphore to signal you are done with the resource

Down(s) / P(s)
* Atomic
* Attempt to take the semaphore
* If semaphore is 0, will **wait till available**

### Semaphores in C
Need:
* <sys/types.h>
* <sys/ipc.h>
* <sys/sem.h>

`semget`
* Create or get access to semaphore
* Does not modify semaphore
* Returnes semaphore descriptor or -1 if error

semget(KEY, AMOUNT, FLAG)
* KEY: Unique semaphore identifier
* AMOUNT: How many semaphores
* FLAGS: INcludes permissions, combine bitwise

## Monday, 12/04 Memes by Anthony Hom
**Tech News**: [Go Oreo ... Towards the Androids](https://www.digit.in/mobile-phones/android-oreo-go-announced-will-come-with-lightweight-apps-and-data-saving-features-for-low-end-phone-38482.html)

**Yes, another tech news**: [Texting 911 Efficiency for Minnesota](http://m.startribune.com/text-to-911-is-minnesota-s-new-emergency-texting-service/462075943/?section=local)

We continued talking about how we can manipulate shared memory for class.

### shmdt
* Detach a variable from a shared memory segment
* Returns 0 upon succes or -1 upon failure

```C
shmdt (pointer)
```
* The `pointer` is the address used to access the segment

### shmctl
* Performs operations on the shared memory segment

* Each shared memory segment has metadata that can be stored in a struct (struct shmid_ds)

* Some of the data stored includes the last access, the size, the PID of the creator, or the PID of the last modification

```C
shmctl (descriptor, command, buffer)
```

* The descriptor returns the value of shmget
* The commands:

`IPC-RMID`: Removes a shared memory segment

`IPC-STAT`: Populates the buffer (struct shmid_ds*) with segment metadata

`IPC-SET`: Sets some of the segment metadata from the buffer.

---

## Friday, 12/01: Sharing is Caring by Shakil Rafi
**Interesting tech news**: [MacOSX High Sierra had a major security hole](https://www.cnet.com/how-to/how-to-fix-the-macos-high-sierra-password-bug/)

We learned about how to use shared memory

### What is shared memory?
Shared memory is a segment of memory that can be access by any process using a key. This memory is not freed when all processes exit.

### What can you do with shared memory?
You can do five different operations using shared memory using <sys/shm.h>, <sys/ipc.h>,and  <sys/types.h>:
* create the segment (happens once)
* access the segment (happens once per process)
* attach segment to a variable (once per process)
* detach the segment from the variable (once per process)
* remove the segment (happens once)

#### shmget
* usage: shmget(key, size, flags)
* create or access a shared memory segment
* returns shared memory descriptor or -1 if fails
* key is unique int id for segment
* size is amount of memory to request
* flags is permissions for the segment
    * IPC_CREAT: creates the segment
        * if segment is new, sets all values to 0
    * IPC_EXCL: fail if segment already exists and IPC_CREAT is on

#### shmat
* usage: shmat(descriptor, address, flags)
* attach a segment to a variable
* returns pointer to segment or -1 if fails
* descriptor is just return value from shmget
* address is 0 (can mess up easily if you use anything else)
* flags are permissions:
    * usually it's 0
    * SHM_RDONLY: makes it all read-only

---

# 11.28.17 - C, the ultimate hipster, using # decades before it was cool.


**Interesting Tech News:** [Useless $2,100 techno-contraption](https://www.digitaltrends.com/cool-tech/bionic-double-hand-prosthesis/)

### \#
Used to provide preprocessor instructions.  
These directives are handled by gcc first.

**`#include <LIBRARY> | "<LIBRARY>"`**
- link libraries to your code
- when you use `include`, the compiler finds the file and replaces include with what is inside (i.e. replace the include with all of the headers inside a header file)

**`#define <NAME> <VALUE>`**
- replace all occurences of NAME with VALUE
- `#define TRUE 1`

For example, for the pipe assignment you could put the following in a header file and include it in the actual file:
```c
#define READ 0
#define WRITE 1
```

**macros:**
- `#define SQUARE(x) x * x`  
- SQUARE(x) is the NAME, x * x is the VALUE
...  
`int y = SQUARE(9);` ==> `int y = 9 * 9;`

- `#define MINE(x,y) x < y ? x : y`

**conditional statement:**
Useful when using header files and there are multiple functions with the same name.
```c
#ifndef <IDENTIFIER> //if not defined
#define <IDENTIFIER>
<CODE>
#endif
```
If the identifier has been defined, ignore all the code up until the endif statement.

For example:
```c
#ifndef PARSE_H // parse.h would break
#define PARSE_H
int count_tokens ...
...
#endif
```
---

## Monday, 11/27: Redirection, how does it... SQUIRREL by Jeremy Sharapov

**Interesting Tech News:** [Bitcoin Approches $10,000](https://www.theguardian.com/technology/2017/nov/27/bitcoin-nears-10000-dollar-mark-as-hedge-funds-plough-in)

### File Redirection

* Changing the usual input/output behavior of a program

### Command Line Redirection

* `>` - Redirects stdout to a file
	* Overwrites the contents of the file
	* `<COMMAND> > <FILE>`
	* Ex.
		```
		ps > ps_file
		cat ps_file
		```
		Now ps is in ps_file
	* Ex2. `ls > file_list`
	* `cat` - Take this and send it to stdout
		* "cat" without the filename prints stdin to stdout
			* When you type something and hit enter, the next line will "echo" what you just typed
		* Reminder that ctrl-d ends the file and closes the connection
		* `cat > new_file` creates a crappy text editor where you can't make mistakes
			* `cat new_file > newer_file`
				* Copies contents of new_file to newer_file (basically cp)
* `>>` - Redirects stdout to a file by appending
* `2>` - Redirects stderr to a file
	* Overwrites the file like `>`
		* Use `2>>` to append
* `&>` - Redirects both stdout and stderr to a file by overwriting
	* `&>>` appends
* `<` - Redirects stdin from a file
* `|` (Pipe)
	* Redirects stdout from one command to stdin of the next
	* Ex. `ls | wc`
		* Returns the word count of ls
* `bash < commands.txt` - runs all commands in the file using bash
	* Use this to test your shell
		* Don't worry about the random prompts (bash handles these automatically)

### Redirection in C Programs

* `dup` - <unistd.h>
	* `dup(fd)`
	* Duplicates an existing entry in the file table
	* Returns a new file descriptor for the duplicated entry
* `dup2` - <unistd.h> (very original naming here)
	* `dup2(fd1, fd2)` - Redirects fd2 to fd1
	* Ex. `dup2(3, 1)` - Replaces stdout with "foo.txt" (that we created earlier)
	* Duplicates the behavior for fd1 at fd2
	* You will lose any reference to the original fd2; that file will be closed
	* Any print always goes into the file directory of 1, regardless of whether or not it points to stdout
	* Remember that `fork()` gives the child the parent's file table
	* Ex2. (Use this in your project)
		```c
		int x = dup(1);
		dup2(3, 1); //assume the fd of 3 is a file we created to log stuff
		//do stuff here
		dup2(x, 1);
		close(x);
		```
---

## Tuesday, 11/21: A pipe by any other name... by Helen Ye

**Interesting Tech News:** [Android phones 'betray' user location to Google](http://www.bbc.com/news/technology-42079858)

### Named Pipe

Also known as FIFOs.

Same as unnamed pipes except FIFOs have a name that can be used to identify
them via different programs.

Like named pipes, FIFOS are unidirectional.

#### `mkfifo`

Shell command to make a FIFO.

```
$ mkfifo <NAME>
```

#### `mkfifo - <sys/types.h> <sys/stat.h>`

c function to create a FIFO

Returns 0 on success, and -1 on failure.

Once created, the FIFO acts like a regular file, and we can use open, read,
write, and close on it.

```c
mkfifo( <name>, <permissions>)
```

FIFOs will block on open until both ends of the pipe have a connection.

#### Some BAD Behavior using Named Pipes

* Two write ends, one read end open
    * Read will display information sent by both writing ends.
      The pipe will stay open if one of the writes is closed.
* Two read ends, one write
    * Information will be read by either pipe, in no particular order.
      Closing the write end will close the pipe.
* Deleting the pipe file while it is open
    * The pipe can still be used as it exists in memory and deleting the file
      in storage will not affect this.

---

## Friday, 11/17: Ceci n'est pas une pipe by James Ko

**Interesting Tech News:** [SpaceX's mysterious Zuma launch is postponed indefinitely](https://www.theverge.com/2017/11/15/16649440/spacex-falcon-9-launch-zuma-mission-us-government-live-stream)

### Pipes

- A pipe is a conduit between two processes on the same computer.
- Pipes have two ends: a *read end* and a *write end*.
  - There is a very strong similarity between them and files; they are also interacted with via the `read`/`write` functions.
- Pipes are unidirectional: one may be only read from or written to in a given process.
- There are two kinds of pipes: *unnamed pipes* and *named pipes*. Unnamed pipes have no external identifier; named pipes do.

- `pipe( int descriptors[2] )` - `unistd.h`
  - Creates an unnamed pipe
  - Returns 0 if the pipe was created, -1 if it was unsuccessful.
  - Opens both ends of the pipe as files
    - There is an entry in the file table for both ends
  - `descriptors`: buffer that the pipe stores the descriptors in.
  - `pipe` is typically used in conjunction with `fork` so that the child process knows the descriptors, even though it's not the process that called `pipe`. (important: `pipe()` call must come before `fork()`)
  - When a pipe is initially created, both ends are open for reading and writing.

### Examples

```c
int fds[2];
pipe(fds);

if (fork() == 0) {
  // Child process
  close(fds[READ]);
  char s[10] = "helli!";
  write(fds[WRITE], s, sizeof(s));
} else {
  // Parent process
  close(fds[WRITE]);
  char s[10];
  read(fds[READ], s, sizeof(s));
  printf("parent received: [%s]\n", s);
}
```

---

## Wednesday, 11/15: Playing Favorites by Herman Lin

**Interesting Tech News:** [New Closest Temperate Exoplanet](http://www.eso.org/public/news/eso1736/)

### endianness
* the way the bytes of a number can be referred to its endianness
* categorized into 'big-endian' and 'little-endian'
* big-endian: most significant bytes are at the end


	int i = 302; //in big-endian, bytes are represented as[0|0|1|46]

* little-endian: least significant bytes are at the end


	int i = 302; //in little-endian, bytes are represented as [46|1|0|0]

### waitpid - <unistd.h>
* waits for a specific child


	waitpid (pid, status, options)

* pid - give the PID of the specific child to look for, or -1 to wait for any child
* options - give options that can set other behaviors for wait, or 0 to work normally

---

## Tuesday, 11/14 Wait For It... By Jackie Xu

**Interesting Tech News:** [FDA Approved First Digital Pill](https://www.theverge.com/2017/11/14/16648166/fda-digital-pill-abilify-otsuka-proteus)

### fork() and getppid() continued
* calling fork twice results in 4 child processes
* return value of fork() to the parent is child process's ID or -1 if errno
* return value of fork() to the child is 0
* return value of getppid() for the parent is the BASH session
* return value of getppid() for the child is 1 once the parent process ends
* parent and child processes run concurrently, but if you can call sleep(1) to one process to change the order

#### example
```c
int main() {

    pid_t  pid;

	pid = fork();
    fork();

	printf("pid: %d\t f: %d\t  parent: %d\n", getpid(), pid, getppid());
    return 0;

}
```
#### output (without sleep)
```
pid: 35157	 f: 0	  parent: 35156
pid: 35159	 f: 0	  parent: 1
pid: 35158	 f: 35157	  parent: 35156
pid: 35156	 f: 35157	  parent: 34152
```

#### example
```c
int main() {

    pid_t  pid;

	pid = fork();
    fork();

    if (pid != 0) {
        sleep(1);
    }

	printf("pid: %d\t f: %d\t  parent: %d\n", getpid(), pid, getppid());
    return 0;

}
```
#### output (with sleep)
```
pid: 35179	 f: 0	  parent: 35178
pid: 35181	 f: 0	  parent: 35179
pid: 35180	 f: 35179	  parent: 35178
pid: 35178	 f: 35179	  parent: 34152
```

### wait() - <unistd.h>
* stops a parent process from running until any child has provided status info to the parent via a signal (usually the child has exited)
* returns the pid of the child that exited or -1 (errno)
* wait (int * status)
    * the parameter (status is used to store info about how the process exited)
* if multiple children, then the first child that exits will trigger wait

### threads
* similar to processes, but more complicated to interact with
* shared memory pool
* started from some other process, but other one must still be running in order for thread to exist

---

## Monday 13/11 What the Fork? By Haiyao Liu

**COOL and GOOD tech news:** [DeepMind's AlphaGo Zero 18 Oct 2017 - nonsupervised AI masters Go tabula rasa](https://deepmind.com/documents/119/agz_unformatted_nature.pdf)

In today's class we learned:
* *homework directions are to be read and followed*
* *strsep() can be used to create an array of strings (char\*\*) in-place*
* *fork() creates a duplicate process (child) when called within an existing process (parent)*

### strsep() <string.h>
`strsep( <POINTER TO STRING> , <DELIMINATOR STRING> )`
* finds the first occurence of any character within the deliminator string or the terminating null and replaces it with a terminating null.
* changes the provided string pointer to the address of the character directly following the replaced character - or NULL if the end of the string has been reached.
* returns the original string pointer.

#### example
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main() {

    char *str;
    char *tmp;

    strcpy(str, "these are words,\tnot whitespace");
    printf("original:\t%s\nparts:\n", str);

    while ((tmp = strsep(&str, " \t"))) { // note 1-2
      printf("> %s\n", tmp);
    }

    return 0;

}
```
#### output
```
original:	these are words,	not whitespace
parts:
> these
> are
> words,
> not
> whitespace
```

### fork() <unistd.h>
* Creates a copy, or child process, of the current process, the parent.
* The stack memory, heap memory, and file table of the child process upon creation are identical to the parent's at the point fork() is executed.
* PID is changed as the child is a distinct process.

#### example
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {

    pid_t  pid;

    printf("I'm going to be the PARENT!! My <PID> is: %d\n", getpid());

    pid = fork(); //note 3

    if (pid == 0) {
        printf("I'm the CHILD and my <PID> is: %d\n", getpid());
    }

    else {
        printf("I'm now the PARENT and my <PID> is still: %d\n", getpid());
    }

    return 0;
}
```
#### output
```
I'm going to be the PARENT!! My <PID> is: 29090
I'm now the PARENT and my <PID> is still: 29090
I'm the CHILD and my <PID> is: 29091
```
Note that 29090 and 29091 are simply PID's and will differ.

### notes
1. strsep replaces ALL characters within the deliminator string, it isn't limited to one character.
2. wrapping an assignment in parentheses will silence the "assignment in conditional" or "= vs ==" warning.
3. concurrent process running prevents simply sending PID's to stdout (print order is unpredictable).
4. avoid f-bombs!

---

## Thursday, 11/09: Execlp and Execvp, by Joshua Turcotti
**Interesting tech news**: [Setback for SpaceX as its next-gen rocket explodes during testing](https://www.digitaltrends.com/cool-tech/spacex-rocket-explodes-during-testing/)

We learned about executing processes from within a C program.

#### What happens when you execute a process from within a C program?
Your original processes (a.out or however you named it) is replaced by a new one that you specificy.

#### How do you do this?
`int execlp(const char *file, const char *arg, ...);` is a function that terminates the current process and spawns a new one. The name of this new one is sprovided by the frist arugment <file>, which is searched for in the path as if typed into a terminal. The remaining arugments, fed as a list, specificy the arguments (argv) passed to that new process, with argv[0] always set to the name of the process.
`int execvp(const char *file, char *const argv[]);` is an alternative function in which the arguments are passed in a vector (array) as opposed to a list; it is often the more convenient notation.
Both are found in `<unistd.h>`.

#### Examples
```c
#include <unistd.h>

int main() {
	execlp("ls", "ls", "-a", "/", NULL);
}

ALTERNATIVELY:

int main() {
	const char *args[8];
	args[0] = "ls";
	args[1] = "-a";
	args[2] = "/";
	args[3] = NULL;

	execvp(args[0], args);
}

```
Note that in the both cases, the args must be terminated by NULL.

#### A Few Final Notes
* the 'p' stand for path, because the $PATH is searched for the names of the executables
* it could be replaces by 'e', and you could provide your own environment
* it could be ommitted entirely, and you would have to specify an explicit path

---
## Wednesday, 11/08: Sending Mixed Signals, by Aryan Bhatt
**Interesting tech news**: [Apple is Reportedly Launching New iPad](https://www.theverge.com/circuitbreaker/2017/11/8/16624842/apple-2018-ipad-face-id-no-home-button-rumor)

We learned about signals today.

#### So what is a signal in the first place?
A signal is a limited way to send information to a process.
Each of the 32 signals is represented by an int from 1-32

#### Sending signals from the terminal
`$ kill -<signal_int> <PID>` is a command line utility that allows you to sends the signal with flag <signal_int> to the process with the given PID. If you don't include an argument for the flag of the signal you want to send, the command will send a SIGTERM signal by default.

`$ killall [-<signal int>] <program_name>` sends the signal with flag <signal_int> to all programs with the name <program_name>

#### Catching and Sending Signals from a C function
`kill(PID, signal_int)` is the function form of the terminal kill command. Returns 0 when it's successful and -1 (errno) when it fails.
To catch signals in your code, you write the sighandler function.
```c
static void sighandler(int signo){
    if (signo == 11) {
        printf("SEGFAULT received\n");
    }
    if (signo == 7) {
    	printf("oh no another signal!\n");
    }
}
```
The name of the function and the argument can be changed, but conventionally, sighandler and signo are used.
However, the function must be static and void and have a single int argument.
In your main function, you need to include a line to check for each signal you want to catch -
`signal(SIGNUMBER, sighandler);`
This attaches sighandler to the function in the file.
The argument sighandler is the name of the function that you defined to catch signals.

#### A Few Final Notes
* use `$ fg` to bring a process to the foreground
* static functions can only be used in the file in which they're written, and they're not stored in stack memory
* getpid() returns the PID of the current process
* remember to `#include <signal.h>` at the beginning of your code
* some signals, like SIGKILL and SIGSTOP can't be caught
* you can add in `exit(1)` in the sighandler function to force an exit

---

## Monday, 11/06: Are your processes running? Then you should go out and catch them! by Jan Kowalski
**Actual tech news**: [Paradise Papers show Apple used tax havens](http://www.businessinsider.com/paradise-papers-apple-found-new-tax-haven-after-us-senate-tax-crackdown-2017-11)<br>
**Interesting tech news**: [Star Citizen to release persistent universe](https://www.neowin.net/news/star-citizens-new-planet-sized-cities-unveiled-at-citizencon)<br>
**Old tech news**: [Reflections on Trusting Trust](http://vxer.org/lib/pdf/Reflections%20on%20Trusting%20Trust.pdf)

We completed the lesson on user input and began learning about processes.


## **Input**
### **fgets()** - `<stdio.h>`

Read in from a file stream and store it in a string

`fgets( <DESTINATION>, <BYTES>, <FILE POINTER>);`

`fgets(s, n, f);`
Reads at most n-1 characters from file stream f and stores it in s


**File Pointer**
- FILE * type, more complex than a file descriptor
- stdin is a FILE * variable
- fopen() returns a file pointer instead of a file descriptor, but does not handle sockets well (sockets not covered).
- fgets() stops at newline, end of file, or the byte limit.
- If applicable, keeps the newline character as part of the string, appends NULL after.

`fgets(s, 256/sizeof(), stdin)`
read a line of text from standard in

fgets() meant to be used for strings, use open() for raw numerical data
```C
    char s[200];

    printf("enter data: ");
    fgets(s, sizeof(s), stdin);
```



### **sscanf** - `<stdio.h>`

Scan a string and extract values based on a format string.

`sscanf(<SOURCE STRING>, <FORMAT STRING>, <VAR 1>, <VAR 2>, ...);`

```C
    int i;
    float f;
    sscanf(s, "%d %f", &i, &f);
```
Returns the number of successful 'conversions' (i.e. the number of correct reads).


## **Processes**
Every running program is a process. A process can create subprocesses, but these are no different from regular processes.

A processor can handle 1 process per cycle (per core)

"Multitasking" appears to happen because the processor switches between all the active processes quickly

**pid**
- Every process has a unique identifier called the pid.
- pid 1 is the init process
- Every entry in the /proc directory is a current pid

`$ ps`
lists running processes
(that you own, that are attached to terminal)

`$ ps -a`
lists running processes
(that are attached to terminal)

`$ ps -ax`
lists all running processes



### Useful Notes
- scanf() can result in overflows if the number of bytes passed is larger than destination.
- gets() [is insecure](http://man7.org/linux/man-pages/man3/gets.3.html#BUGS), don't use it.
- tty represents a terminal session that a process is attached to.

---

## Friday, 11/03: Input? Fgets about it! by Yedoh Kang
**Interesting tech news**: [Logitech is experimenting with a keyboard built for VR](https://techcrunch.com/2017/11/02/logitech-is-experimenting-with-a-keyboard-built-for-virtual-reality/)

We started off class by going over the two previous homeworks, stat and dirinfo. Mr. DW then went over his solutions to the homeworks, and showed us different ways to print out the permissions in ls -l form, including bitwise operators such as bit shifting and &(and).

We then discussed different ways of getting input:

**Command Line Argument**
* `int main (int argc, char *argv[])`
* program name is considered the first command line argument: `argv[0]`
* argc: # of command line arguments
* argv: array of command line arguments

**scanf - <stdio.h>**
* `scanf(<FORMAT STRING>, <VAR 1>, <VAR 2>, ...);`
* reads in data from stdin using the format string to determine types
* puts the data in each variable
* variable must be a pointer
* Example:
  * Expects user to input in the variables in the EXACT format, including spaces
	* If not followed, it might lead to incorrect answers and possibly, errors
```
 int x;
 float f;
 double d;
 scanf("%d %f %lf", &x, &f, &d);
```

---

## Wednesday, 11/01: Where do doctors keep their files? by Shakil Rafi
**Interesting tech news**: [Intel 8th generation processors are out with ~60% performance boost](http://www.trustedreviews.com/news/intel-8th-gen-cpus-release-date-specs-price-2952599)

We learned about interacting with directory files using `dirent.h` functions

### What is a directory actually?
* A directory is a list of filenames for all files that reside in the directory
* It also holds some basic info about the file like filetype and size
* Moving a file just moves the filename to another directory file (doesn't copy bytes)
* Linux directories are 4096 bytes (or 4 kilobytes)

### opendir - <dirent.h>
* usage: `opendir(<path>)`
* opens a directory file
* does not change current working directory
* does not create a directory
* cannot write to directory (only read)
* returns pointer to directory stream (`DIR*`)
* directory streams pass info from given directory
* does not require a call to `malloc()`
    * also don't `free()` this

### closedir - <dirent.h>
* usage: `closedir(<dir stream>)`
* closes directory stream

### readdir - <dirent.h>
* usage: `readdir(<dir stream>)`
* returns next file in dir as a `struct dirent` or `NULL` if nothing left

### struct dirent - <sys/types.h>
* contains info about directory file
* `d_name`: name of file
* `d_type`: filetype as int

---

## Monday, 10/30: Seek and ye shall find, by Ida Wang

**Interesting tech news**: [YouTube tweaks advertising algorithm](http://www.bbc.com/news/technology-41801705) (again).

We learned about `lseek` and `stat`.

### lseek - `<unistd.h>`  
- set the current position in an open file
- returns the number of bytes the current position is from the beginning of the file, or -1 (errno).

#### `lseek(<FILE DESCRIPTOR>, <OFFSET>, <WHENCE>)`  
OFFSET: The number of bytes to move the position by (can be negative)  
WHENCE: where to measure the offset from
- `SEEK_SET`: offset is evaluated from the beginning of the file
- `SEEK_CUR`: offset is relative to the current position in the file
- `SEEK_END`: offset is evaluated from the end of the file

Application in our last assignment (dev-random): instead of closing the file and reopening to move from write to read, use `lseek` to set the position to the beginning of the file again.

### stat - `<sys/stat.h>`
- get information about a file (metadata)
- `stat(<PATH>, <STAT BUFFER>)`
```c
struct stat sb;
stat("foo", &sb);
```
PATH: file path (no file descriptor required because you can stat a file without opening it)

STAT BUFFER: must be a pointer to a struct stat
- All the file information gets put into the stat buffer.

Some of the fields in struct stat:
- `st_size`: file size in bytes
- `st_uid, st_gid`: user ID, group ID
- `st_mode`: file permissions
- `st_atime, st_mtime`: last access, last modification

---

## Thursday, 10/26 Read your writes! by Henry Zheng
**Tech News:** [Amazon Key is a new service that lets couriers unlock your front door](https://www.theverge.com/2017/10/25/16538834/amazon-key-in-home-delivery-unlock-door-prime-cloud-cam-smart-lock)

Today, we learned about umask and read/write file commands.

**umask - <sys/stat.h>**
* Sets the file creation permission mask
* By default, created files are not given the exact permissions provided in the mode argument to open. Some permissions are automatically shut off.
* Umask is applied in the following way:
	- new_permissions = ~mask & mode;
* The default mask is 0002.
* `umask( <MASK> );`
	- You can define the umask using a 3 digit octal #
	- `umask(0000);`

**read - <unistd.h>**
* Read in data from a file
* `read( <FILE DESCRIPTOR>, <BUFFER>, <AMOUNT> );`
* `read( fd, buff, n );`
	- Read n bytes from the fd's file and put that data into buff
* Returns the number of bytes actually read. Returns -1 and sets errno if unsuccessful
* BUFFER must be a pointer

**write - <unistd.h>**
* Write data to a file
* `write( <FILE DESCRIPTOR>, <BUFFER>, <AMOUNT> );`
* `write( fd, buff, n );`
	- Write n bytes from buff into fd's file
* Returns the number of bytes actually written. Returns -1 and sets errno if unsuccessful

---

## Wednesday, 10/25 Opening up a world of possibilities by Sonal Parab
**Tech News:** [High-tech Mirror for cancer patients only works if you smile](http://money.cnn.com/2017/10/24/technology/smile-mirror-cancer-patients/index.html)

We continued our discussion about files.  

**open**  
`#include <fcntl.h>`  
Adds a file to the file table and returns its file descriptor  
If open fails, -1 is returned, extra error information can be found in errno.  
* `errno` is an int variable that can be found in `<errno.h>`, using `strerror` (in `string.h`) or errno will return a string description of the error  
```C
open(<PATH>,<FLAGS>,<MODE>)
```

mode  
* Only used when creating a file. Set the file's permissions using a 3 digit octal #  

flags  
* Determine what you plan to do with the file.  
* Use the following constants:  
	- `0_RDONLY`: read only  
	- `0_WRONLY`: write only  
	- `0_RDWR`: read and write  
	- `0_APPEND`: start at the end  
	- `0_TRUNC`: start at the beginning (if combined with write would overwrite the file)  
	- `0_CREAT`: creates the file, must be used if file does not exist, opens file if it exists  
	- `0_EXCL`: must be combined with `0_CREAT`, will return an error if the file exists  
* Each flag is a number, to combine flags we use bitwise OR
    ```
    O_WRONLY = 1            00000001
    O_APPEND = 8            00001000
    O_WRONLY | O_APPEND =   00001001
    ```
---

## Tuesday, 10/24 File This Under Useful Information by Charles Weng
**Tech News:** [Does the 3DS have a Future?](https://arstechnica.com/gaming/2017/04/after-nintendo-switch-does-the-3ds-have-a-future/)

Today was mostly a review of file permissions that we covered back in intro.

We started off by linking octal to the file permissions (read, write and execute, abbreviated to rwx respectively) and went over the following:
* there is 8 total combinations for those permissions
* permissions could be represented by a single digit in octal
  - this stems from a 3 digit binary representation in which a 1 represents permission given and 0 represents permission denied
  - 6 will get converted to 110 which cross referenced with rwx means read and write permissions
* there are 3 sets of permissions; one for user, one for group, and one for others
  - others does NOT refer to everyone, but everyone not included by user and group
  - you can view file permissions in the terminal with the 'ls -l' or 'll' command (long format listing), which shows metadata
* this is used by the terminal command chmod to more easily modify file permissions for a file
  - 644 is default
  - `chmod 644 name.c` as opposed to `chmod u=rw,g=r,o=r name.c` or using separate flags

We then went on to talk about the first character that we saw in `drw-r--r--` which denoted the file as a file descriptor or a directory. This was just a file that listed out the pointer to and names of other files and is used in GUIs for organization.

We also talked about how this relates to c with file interactions through stdio.h and how when you shouldn't be complaining about the limit on the amount of files you can have open (256). A relevant function for this is `getdtablesize()` which returns the first power of two greater than your table size. There will always be 3 files in the table: STDIN_FILENO: stdin, STDOUT_FILENO: stdout, and STDERR_FILENO: stderr

## Monday, 10/23 A bit of wisdom by Kenny Chen
**Tech News:** [Should We Fear the Rise of Intelligent Robots?](https://www.livescience.com/59802-should-we-fear-intelligent-robots.html)

Today we discussed a few more `printf` formatting options for printing integers in different bases, ways to represent an integer in different bases, and bitwise operators.

**Different Bases**

`"%o"` prints an integer in **octal**, and `"%x"` prints an integer in **hexadecimal**. (Correspondingly, `"%d"` prints an integer in **decimal**. There is no formatting character for binary.)  
So, for example, printing 13 with:  
`"%d"` results in `13`  
`"%o"` results in `15`  
`"%x"` results in `d`  

We can also represent integers in different bases:  
In **binary**, we precede the digits with `0b`.  
In **octal**, we precede the digits with `0`.  
In **hexadecimal**, we precede the digits with `0x`.  
```C
int b = 0b1011; //11
int o = 0122; //82
int x = 0xa3; //163
```
These integers are all stored the same way; that is,  
`0b1011`, `013`, `0xb`, and `11` are all the same.  

**Bitwise Operators**

`>>` is the right shift. It moves all the bits once to the right, and adds 0s to the front.  
`<<` is the left shift. It moves all the bits once to the left, and adds 0s to the end.  
Right shift and left shift do not affect memory outside of the integer.  

So, if `char b = 0b1011;` (that is, `b`'s bits are `00001011`, then  
`b = b >> 1;` would have `b` be the same as `00000101`. Then,  
`b = b << 1;` would have `b` be the same as `00001010`.  

`~` is negation. It flips every bit.  
`|` is or. It performs or for each pair of bits.  
`&` is and. It performs and for each pair of bits.  
Lastly, `^` is xor. It performs xor for each pair of bits.  

Now, suppose we have  
`unsigned char a = 0b10110111;` and  
`unsigned char b = 0b01011100;`. Then:  
`~a` would be `01001000`.  
`a | b` would be `11111111`.  
`a & b` would be `00010100`.  
`a ^ b` would be `11101011`.  


At the end of class, a swap algorithm using xor was given.  
Given the integers `a` and `b`:
```C
a = a ^ b;
b = a ^ b;
a = a ^ b;
```
swaps the values of `a` and `b`.

---

## Thursday, 10/19 Get Dem Bugs by Xing Tao Shi
**Tech News:** [Samsung to Give Linux Desktop Experience to Smartphone Users](https://www.technewsworld.com/story/Samsung-to-Give-Linux-Desktop-Experience-to-Smartphone-Users-84896.html)

**GDB**
* stands for GNU debugger
* You must compile with -g in order to use GDB
```C
gcc -g foo.c
```
* Usage:
```C
$ gdb ./a.out
```
There are many things you can do with GDB such as:
Run the program
```C
run
```
Place a break point at a line, this essential makes it so that when you run the program it stops here
```C
break line_number
```
Place a break point at a function, this essential makes it so that when you run the program it stops here
```C
break func_name
```
To continue from a break point until the next break point you can
```C
continue
```
To continue from a break point and execute the next line you can
```C
next
```
To continue from a break point and execute the next line or the next line of the function located in the next line you can
```C
step
```
Print a variable
```C
print variable
```
Show the broken code
```C
list
```
Quit the GDB shell
```C
quit
```
---

## Wednesday, 10/18 valgrind by Herman Lin
**Tech News:** [LIGO Detects Fierce Collision of Neutron Stars for the First Time](https://www.nytimes.com/2017/10/16/science/ligo-neutron-stars-collision.html)

**valgrind**
* tool for debugging memory issues in C programs
* You must compile with -g in order to use valgrind (and other similar tools)
```C
gcc -g foo.c
```
* Usage:
```C
$ valgrind --leak-check=yes <program>
```

---

## Friday, 10/13 calloc & realloc by Fabiola Radosav
**Tech News:** [Can we teach robots ethics?](http://www.bbc.com/news/magazine-41504285)

**calloc**
```C
void *calloc(size_t n, size_t x);
```
* like malloc but initializes memory to 0 (clears allocation)
* allocates n * x bytes of memory
* Example code:
```C
int *p;
p = (int *)calloc(5, sizeof(int));
```

**realloc**
```C
void *realloc(void *p, size_t x);
```
* changes the amount of memory allocated to a given block (x is the new size)
* p must be a pointer to the beginning of an allocated block of memory, but it does not have to be the original pointer
* if x is smaller than the original size of the allocation, the extra bytes will be released
* Example code:
```C
int *p;
p = (int *)malloc(20);
p = (int *)realloc(p, 40);
```

---

## Thursday, 10/12 malloc & free: the dynamic duo! by Penn Wu
**Tech News:** [Bitcoin rockets above $5,000 to all-time high](https://www.reuters.com/article/us-global-markets-bitcoin/bitcoin-rockets-above-5000-to-all-time-high-idUSKBN1CH0YQ)

We spent most of the class writing code demonstrating either the functions `malloc` and `free` or `calloc` and `realloc`.

**malloc**
```C
void *malloc(size_t x);
```
* allocates x bytes of memory dynamically (i.e., from the heap), and returns the address to the beginning of the allocated memory.
* returns a void pointer, which you should typecast to the correct type like so:
```C
int *p;
p = (int *)malloc(sizeof(int) * 5);
```

**free**
```C
void free(void *p);
```
* frees dynamically allocated memory starting from the pointer parameter
* doesn't immediately mean that whatever data was stored there is gone, it just means that it isn't guaranteed to be there anymore
* in other words, **don't use memory that you freed**

---

## Tuesday, 10/10 and Wednesday 10/11 If you don't pay attention you'll get into a heap of trouble! by William Hong
**Tech News:** [More Governments Test Out Cryptocurrency](http://www.investopedia.com/news/more-governments-test-out-cryptocurrencies/)

### AIM: If you don't pay attention you'll get into a heap of trouble!

We briefly discussed structs, below is prototyping a struct
#### struct
```C
struct foo {
    int a;
    char x;
};
```
Using the code above, we can declare variables as struct foo
```C
struct foo s;
struct foo s2; //this is a different variable of type foo
```
Prototypes are different than directly declaring a struct such as below
```C
struct {int a; char x;} s;
```
Prototypes are meant to be used later, like a template

IMPORTANT: We use the . operator to access a value inside a struct, see example below
```C
s.a = 10;
s.x = '@';
```
Structs can contain variables and even more structs. You can also have a pointer to a struct within the struct itself, thus making a linked list data structure
```C
struct foo{
    //variables, etc
    struct foo *next;
};
```

Note: even if we use a struct in the parameter of a function, it is stilled passed by value, which means a copy of the argument is made. Do this instead...
```C
void foo_stuff(struct foo *s); //as opposed to void foo_stuff(struct foo s)
```

Important note on the dot operator: . binds before *
```C
struct foo *p;
p = &s;
(*p).x; //this is accessing the data in a pointer to a struct
```
Additional note: anything you do to p will modify whatever's going on in x

IMPORTANT: to access data from a struct pointer, you can also use ->
```C
p->x; //this is the same as (*p).x
```
If there's a pointer inside a struct with another pointer to another struct, you can chain the arrows

!!! Caution: you can typedef structs, but this will hide the fact that you are working with a struct

We also discussed memory, but didn't have time to talk about heap memory yet. Computer programs separate memory usage into two parts: stack and heap. Every program can have its own stack and heap

Notes on stack memory
- Stack memory stores all normally declared variables (including pointers and structs), arrays and function calls
- Functions are pushed into the stack in the order they are called, and popped off when completed
- When a function is popped off the stack, the stack memory associated with it (like an array for instance) is released. You can't use that memory later because it's gone
-The stack contains all the memory we learned so far in C, including arguments to functions. The copy of the argument(s) associated with that function is put onto the stack

```C
struct node{
	int data;
	struct node *next;
}

struct node *insert_front(struct node *front, int d){
	struct node new;
	new.data = d;
	new.next = front;
	return &new;
}
```
- On the above example, this looks good on paper. This should implement a linked list data structure
- However, this is BAD. The struct node new would cease to exist because it's stack memory
- Declaring a variable in a function puts it into the stack and it subsequently gets popped off



Notes on Heap memory
- Stores dynamically allocated memory
- IMPORTANT DIFFERENCE FROM STACK MEMORY: data will remain in the heap until it is released manually by you or when the program terminates
- Can be accessed through pointers
- Can be accessed across many functions
- Persists throughout the duration of the program's execution, until it terminates of course
- A memory leak occurs when a program uses lots of heap memory

---

## Friday, 10/6 Finding your type by Jerome Freudenberg
**Tech News:** [YouTube Changes Search Algorithms](https://www.theguardian.com/us-news/2017/oct/06/youtube-alters-search-algorithm-over-fake-las-vegas-conspiracy-videos)

### AIM: Finding your type

On Friday, we discussed useful keywords in C:

#### typedef
-Usage:
```C
typedef <real type> <new name>;
```
-provides a new name for an existing data type
-Ex:
```C
typedef unsigned long size_t;
size_t x = 139; //x is really an unsigned long
```
-Can be used to improve portability

#### struct
-Usage:
```C
struct {int a; char x;} s;
```
-creates a new type that is a collection of values
-The example above is technically 5 bytes, but has extra padding so that the memory allocated is a power of 2

-You can also assign it to a variable:
```C
struct foo {int a; char x;};
```
-This means that foo is a prototype for this kind of struct, to be used later
		(e.g.)
```C
struct foo s1;
```
-Note the semicolon after the curly brace as people tend to forget to place it there

###### We also discussed what this dude, Linus, wrote about coding style:
[His thoughts](https://www.kernel.org/doc/Documentation/process/coding-style.rst)

-Noteworthy things included that Hungarian notation is for the brain damaged

-He also mentioned typedefs are not for readability and should not be used for structures and pointers

---

## Tuesday, 10/3 Make It So by Eric Chen
**Tech News:** [General Motors to go Electric](https://www.washingtonpost.com/news/innovations/wp/2017/10/02/death-of-diesel-begins-as-gm-announces-plans-for-all-electric-future/?utm_term=.7a940daf5ee4)

AIM: Make it so

Separate Compilation
- You can combine multiple C files into a C program by including them all in one gcc command.
- i.e. `gcc test.c string.c foo.c woohoo.c`
- One and only one of these files must have a main function.
- You cannot have duplicate functions or global variable names across these files.

`gcc -c`
- Compiles a C file into a .o file, it is not a fully functional program, but it is compiled code.  Do this to compile a .c file that does not contain a main() function.
- .o files can be liked together with .c files through gcc.

Make
- Create compiling instructions and setup dependencies.
- Standard name for the file is makefile.
- Syntax:
	```<TARGET>: <DEPENDENCIES>
	[TAB]<RULES>```
- `<TARGET>` can be either a file or just a name.
- i.e.

```
strtest: dwstring.o main.o
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
- In order to run a makefile, the syntax is `make <TARGET>`
	It will only make that specific target.
	If `<TARGET>` is left blank, it will make the first target and all its dependencies only.
- The clean target will run using `make clean`
- The run target will compile all its dependencies and run automatically with `make run`

---

## Friday, 9/29 String Functions by Connie Lei
**Tech News** [Microsofts's Nadella Wants To Help Coders Take A Quantum Leap](https://www.wired.com/story/microsofts-nadella-wants-to-help-coders-take-a-quantum-leap/)

We started class in our groups and explained our homework, showing each function and running our test code. We then went over the functions together in class so we could have a class discussion about it and ask questions.

You need to use `#include <string.h>` at the top of your file

* **strcpy/strncpy**
   * `strcpy( char * dest, char * src );`
   * `strncpy( char * dest, char * src, int n);`
   * These two functions copy strings from the source to the destination. The latter one allows you to choose the length of the source string. If you choose to copy a string of a length longer than the destination memory allocation, the null terminating is overwritten. You need to make sure to copy over the null terminating as well.
* **strcat/strncat**
   * `strcat( char * dest, char * src );`
   * `strncat( char * dest, char * src, int n);`
   * The two functions allow you to concatenate strings from the source to the destination. The destination string has to have enough space for both functions. The null termination from the destination string is overwritten with the first character in the source string.
* **strcmp/strncmp**
   * `strcmp( char * s1, char * s2 );`
   * `strncmp( char * s1, char * s2, int n );`
   * These function allows you to compare two strings, it returns s1 - s2 so if s1 is less than s2, a negative number is returned, a positive if s1 is greater and 0 if the two are equal. The latter function will only compare the first n characters of the string.
* **strchr/strstr**
   * `strchr( char * s1, char c );`
   * `strstr( char * s1, char * s);`
   * These function return a pointer to the first instance of the character or the index for the string. A null pointer will be returned if the string/character does not exist.
---

## Thursday, 9/28 Functions with Strings by Gilvir Gill
**Tech Noos** [San Francisco Sues Equifax over data breach](https://techcrunch.com/2017/09/27/san-francisco-sues-equifax-on-behalf-of-15-million-californians-affected-by-the-breach/)

We started clas by writing code that will return the length of the string, abusing the fact that String literals are automatically terminated with '\0's (null, or 0). We had the idea of 0 being false reinforced by realizing that we could just terminate with the boolean `(*(str+ i))`, which will return 0 when it lands on the null.


```C
#include <stdio.h>
int str_len(char * str) {
    int i = -1;
    while ((str[++i]));
    return i;
}

void main() {
    char * s = "hello";
    char * s0 = "";
    printf("Length of \"%s\": %d\n", s, str_len(s));
    printf("Length of \"%s\": %d\n", s0, str_len(s0));
}
```
We also reviewed the difference between different expressions, particularly mutable pointers, immutable arrays, immutable String literals, and their mutable copies held in array locations.

For example:

* `char *s3 = "Mankind"` just points to the immutable piece of memory that holds "Mankind"

In addition, you can only assign char arrays with = at declaration. If you declare it with an = it first finds the literal that was created in the beginning in memory and then clones it in an O(n) procedure. On the other hand, if you declare a char pointer, it simply points to the literal. If you want to ensure that you have a mutable copy of a string, you want to declare it with an array. You can of course assign a pointer to it afterwards if needed.

We were later told to examine one of four functions:
* 0 : strcpy/strncpy
* 1 : strcat/strncat
* 2 : strcmp/strncmp
* 3 : strchr/strstr

for homework.

---
## Wednesday, 9/27 Strings in C by Anthony Hom

**Tech News** [A smaller Echo and thee new Echo Plus from Amazon](https://www.theverge.com/2017/9/27/16375984/amazon-echo-plus-hands-on-impressions-photos)

We started class off by writing a block of code that had a char-typed pointer pointing to the string 'hello' and a char-typed array containing 'hello'. At the end the code contained a print statement that referred to the address of 'hello'. Afterwards, the class went over why the print statement with the 'hello' address was the same as the pointer's address as opposed to the array's address. Then we went to talk more about strings and memory addresses.

```C
#include <stdio.h>
#include <stdlib.h>

int main(){
char *s = "hello";
char s1[] = "hello";
printf("s points to: %p\n", s);
printf("s1 points to %p\n", s1);
printf("address of \"hello\": %p\n", &"hello");

// -----------------------------------------------

char s2[5];
printf("s: %s\n", s);
s2[0]= 'h';
s2[1] = 'i';
s2[2] = 0;
printf("s2: %s\n", s2);

return 0;
}
```

### About that address...
* We determined that in the first part of the block of code above, &hello refers to an address of the literal string 'hello' which array s1 does not do.

### What are strings?
* Strings are char arrays, or an array of char.
* By convention, strings end with a null character.
* Examples include either `''` or `C 0 ` or ` '\0' ` (Escape 0). We do this for the escape 0 because it is an ascii value.
* IMPORTANT: The null character is useful for using while loops instead of length. Why? Because we do not know the length of the string in C and we cannot return the size back.

### More on the null
* When you make a literal string using ` "" `, an immutable string is created in the memory. These strings are automatically null terminated.

### How does C know our address?
* When using multiple literals, C does not make a copy of them and C will refer to the same string.
* All references to the same literal string refers to the same immutable string in the memory.

### Declaring Strings
* `char s[256]` is a normal array declaration. This array allocates 256 bytes. We could use all 256 bytes, but some things could break .
* It is important that we use 255 bytes of this array and leave the last byte null.

### Examples:
```C
char s[256] = "Imagine"; // Allocates 256 bytes and creates an immutable string "Imagine"
// It then copies (including the terminating null) into the array
```
```C
char s[] = "Tuesday"; //Array s allocates 8 bytes.
// It creates an immutable string "Tuesday" and then copies the string and the null into the array.
```

### The last part of the block of code in the Do Now!
* Given our array s2 with 5 bytes, we fill in index 0 to 1 with 'hi', followed by the terminating null.
* IMPORTANT: After the null, anything that we put in the array afterwards will not be displayed nor is used by the array.

---
## Tuesday, 9/26 Let's Head to Function Town, by Jonathan Wong

**Tech News** [Twitter is experimenting with 280-char tweets](https://www.theverge.com/2017/9/26/16363912/twitter-character-limit-increase-280-test)

Once we came into class, we were asked to bring up a copy of HW#3, the array_swap hw using pointers. Upon the completion of this, we went over how to create functions again.  

### Passing Arrays

* Arrays are pointers, and thus only store a memory address
*This means that if you pass an array to a function, you're simply passing the memory address, NOT a copy.

```C
int arr[2];
arr[0] = 0;

void changeArr(int arr[]){
  arr[0] = 5;
}

changeArr(arr);

printf("arr 0 is %d\n", arr[0]); // will return 5
```

### Getting around pass by value

* If we want to use a function to change a variable, we can pass a pointer as a parameter to the function.

### Using Functions in a Procedural-based language
* Since C is a procedural based language, using functions can get a little tricky, especially when calling functions within other functions. This leaves three options.
  1. Declare all the functions in such an order such that the functions that other functions rely on are declared first. This can get messy and very confusing.
  2. C has something called function headers. They look like this.
  ```C
  void changeArr(int arr[]);
  ```
  Or this.
  ```C
  void changeArr(int* );
  ```
  A function header is simply the first line of the function. It does not require parameter names, as it only type checks.

  3. Option 2, but outsourcing the function headers to another file. In C, these are header files, and they typically end in `.h`. To do this, you need to let the compiler know where your header file is.
  ```C
  #include “PATH/filename.h”
  ```
  Note that angle brackets are not included, being replaced by quotes instead. This is because the angle brackets are for special locations like `usr/bin` where standard libraries are stored.

### Misc.

* You can use pointers and arrays in very similar manners.
---
## Thursday, 9/25 How to Write Functioning Code, by alex lu

**Tech News** [In flight Netflix will be available on more airlines in 2018](https://www.engadget.com/2017/09/25/in-flight-netflix-comes-to-more-airlines/)

We kicked off class with a quick presentation of Charles' homework, followed by some relevant questions pertaining to pointers.
### Pointers and Arrays
* Array variables are immutable pointers
* Pointers can be assigned to array variables

We learned that `array[17]` is simply shorthand for `*(array + 3)`. This can get us into some fun and interesting trouble.
```
int array[100];
array[17] = *(array + 17)
==> *(17 + array)
==> 17[array]
```
Hm... This doesn't seem too logical. Don't do it! But this emphasises that `array[17]` is nothing more than shorthand, so the communative property gives us the strange expression `17[array]`. Let's talk about functions.

### Function Stuff
* Header format: <return type>, <name>, (<arguments>){...}
* ALL functions are "pass by value"
* This means that if a fxn `foo(int a)` takes in some int k, the fxn will create a copy of k, named a: `foo(k) ==> a = k`
* The arguments are just variables containing the value passed to them
---
## Wednesday, 9/20 Try not to hurt yourself, the point is Sharp by Max Chan

**Tech News** [Despite all odds, Hyperloop One just raised another $85 million](https://techcrunch.com/2017/09/22/despite-all-odds-hyperloop-one-just-raised-another-85-million/)

We started off class today by finishing up arrays from the previous day.  The array variable only tracks where the memory allocated for the array is, which is why we can go outside the bounds of the array, and won't get an error.  Then we moved onto pointers.

### Pointers
* Pointers are variable designed to store memory addresses.
* `*` is used to denote a pointer variable (`int *p` `double *q` `char *r`)
* We discussed different type of pointer variables, including int, double, and char
* Because pointer variables, regardless of type, all point to a memory location, they're all the same size.  Today, most machines will allocate 8 bytes of memory to a pointer variable, while previously it was 4 bytes (Which meant the maximum memory your program could have was 4GB)
* Pointer variables story the addresses in hexidecimal format
* When printing pointers, use the `%p` formatting character
* `&` is used to get the address of a variable:
```C
int i;
int *p;
p = &i; //p now stores the address of the int i
```
* `*` is also the dereference operator, which accesses the value at a memory location:

```C
int i = 42;
int *p = &i;
printf("i = %d\n", *p); // prints "i = 42"
```

### Pointer Arithmetic
C will automatically add the right increment of memory to a pointer variable, based on its type:
```C
int *p = &i;
char *c = &l;
p++; // will add 4 to p
c++; // will add 1 to c
```

The reason 4 is added to p is because p points to an int, which has 4 bytes of memory.  Likewise, 1 is added to c because a char is allocated only 1 byte.


## Tuesday, 9/19 Arrays and Memory by Winnie Chen
**Tech News:** [When Driver and Car Share the Same Brain](http://nautil.us/issue/51/limits/when-driver-and-car-share-the-same-brain)
---
**Aim:** What’s the point of it all?

### Arrays

Declaring an Array

`<elements' type> <array name>[<array size>]`
* E.g. ` int b[5]`
* An array must have a fixed size set at declaration as a literal (not a variable name nor a function).
* When declaring an array, the program is only requesting memory (generally thought to be consecutive) from the OS. The data currently in the memory is not changed.
* In this case, the program is requesting 20 bytes (5 * 4 bytes) of memory from the OS.
* This memory is allocated at compile time, so one cannot use expressions, which would be evaluated at runtime, as the array size.

    E.g. ` int b[5 + 1]` will not work because the 5 + 1 part is evaluated at runtime, not compile time.
* There is something different called dynamic memory allocation, which is done at runtime.

There is no length function, but the ` sizeof(<data type or expression>)` function can be used to calculate the length of the array in this way: ` sizeof(<array name>) / sizeof(elements’ type>)`

* E.g. `sizeof(b) / sizeof(int)`

There is no boundary checking.

* When you reference something using an out-of-bounds index, there is no warning nor compiler error.

    E.g. `printf(“%d\n”, b[5]);`

    This acts normally, printing the current value in the memory there (around the memory specified for the array).

    Imagine the memory:

    |Memory                   |            |        |        |        |        |        |           |           |
    |:------------------------|:-----------|:-------|:-------|:-------|:-------|:-------|:----------|:----------|
    |Address (4 bytes/element)|996         |1000    |1004    |1008    |1012    |1016    |1020       |1024       |

    If the array starts at 1000, the memory specified for the array spans from the block with the address 1000 to the block with the address 1016, but there still exists memory before and after that, which can be referenced with out of bounds indices.

    One can also modify/reference variables using out of bounds arrays, although it is not a good idea to do so.

    Different compilers allocate memory differently. For example, some compilers pad memory, so there are blocks of memory between each variable.

    Nowadays, we have protected memory, preventing our programs from referencing memory that other programs are using. Doing so will result in a segmentation fault because the OS will not allow the protected memory to be referenced.



---
## Monday, 9/18 More on Variables + Arrays by Mansour Elsharawy

**Tech News!:** [ANDROID OREO FEATURES Y'ALL](http://www.androidauthority.com/android-8-0-review-758783/)

Aim: A vast array of possibilities

In the Do Now, we discussed with our partners any interesting, weird, or confusing things/issues we found while working on our Project Euler problems. Then we delved a more into variables in C!

### More on Variables!

First off, character literals can be denoted by single quotes instead of their numerical denomination.
Example: `char c = 'A';` and `char c = 65;` effectively do the same thing, but one is much more obvious as representing the character A.

String literals also exist, even though there is no String data type in C.
Example: `"Hello!"`

When declaring variables, the *order* in which you declare variables (and by extension, functions) matters.
Example...

```C

int i = 7;
int j = i + 1;
```
...is an entirely viable piece of code. *However...*

```C
int j = i + 1;
int i = 7;
```
...this will not work.

Variables also cannot be initialized in for loops in C, unlike Java, but can be initialized beforehand.

Example of proper for loopage:

```C

int i;
for(i = 0; i < 10; i++){
printf("i is %d\n",i);
}
```

Variables can also be declared as "unsigned," prohibiting the variable from taking on a negative variable.
This expands the range of values a variable can have, with 0 as the lower bound and a higher upper bound than an unsigned variable.
Example: `unsigned char x;` 0 <= x <= 255, where just `char x;` has -128 <= x <= 127

Booleans don't exist in C, but any variable with a value of 0 is regarded as false. *Any* other value evaluates to true.

Example of an infinite loop:
```C
while(1){
printf("To infinity and beyond!");
}

```

**IMPORTANT**
```C
int x=5;
if(x=6){
printf("foo");"
}
```
Even though the intended purpose of this code may be to skip the block containing the print statement, "foo" will be printed, as C will interpret the assignment of x to 6, which returns what x was assigned to (6), and that will evaluate as true.

Sometimes we may actually want this effect in the future!

### Memory Management

Memory allocation can happen at either compile or run time (as it is dynamic).
When the compiler sees you declaring a character variable for example, it will reserve 8 bits in memory for it, and is packaged with the binary of the program.

There is *no* default value for any variable. You get assigned a piece of memory which *might* be zero, but no guarantees!

We also briefly touched arrays, but the bell rang before we got anywhere meaningful besides the initialization:

```C
int l[5];
```
Cheers!



---
## Thursday, 9/14 Primitives by Jeremy Sharapov

**Interesting News:** [US bans Kaspersky software from government agencies](https://www.cnet.com/news/us-bans-kaspersky-software-from-government-agencies-trump-dhs-russia/)

**Bonus (I'm sick of Apple so here's some Google):** [Pixel 2 Launches on October 4th](https://arstechnica.com/gadgets/2017/09/google-teaser-site-promises-a-pixel-2-launch-on-october-4/)

Aim: Always read the fine print!

For our Do Now, we listed all of the primitives in Java, which are:
* int
* char
* bool
* float
* double
* byte
* short
* long

In C we have all of them except bool and byte, and all of them are numeric, with the only differences between them being if they include floating points and the size of the variable in memory.
However, do note that the size can be platform dependent; use `sizeOf(<Type>)` to find the size in bytes.

|  Type  | Standard Size (in bytes) |                          Range                         |
|:------:|:------------------------:|:------------------------------------------------------:|
|  char  |             1            |                       -128 - 127                       |
|   int  |             4            |             -2,147,483,648 - 2,147,483,647             |
|  short |             2            |                    -32,768 - 32,767                    |
|  long  |             8            | -9,223,372,036,854,775,808 - 9,223,372,036,854,775,807 |
|  float |             4            |                    7 digit precision                   |
| double |             8            |                   14 digit presicion                   |

`printf()` - `stdio.h`
  * The most important C function
  * Prints formatted Strings
  * `printf(<String literal>, <var 1>, <var 2>, ...); `
  * Simple printf does not need the <var> arguments
  * printf does not add a newline at the end! Use `\n`
  * If you want to print variables, you must include formatting placeholders in the String argument
  * ex.
    ```C
    int i = 5;
    int main(){
      printf("i is %d", i);
      return 0;
    }
    ```
    print outs `"i is 5"`

|         Type        | Formatting Character |
|:-------------------:|:--------------------:|
|         int         |          %d          |
|         long        |          %ld         |
|        float        |          %f          |
|        double       |          %lf         |
|         char        |          %c          |
| char array (String) |          %s          |
|       Pointer       |          %p          |

---
## Wednesday, 9/13 Variables are the Spice of Life by Jennifer Zhang

**Interesting News:** [Apple Face ID](http://www.popsci.com/apple-face-ID)

* `man` is a command that will display a user manual
    * The manual is divided into sections (we talked about the first three):
        1. executable programs or shell commands
        2. system calls
        3. library routines
    * If you wanted to look up something from the manual, the command would be `man **keyword**`
        * For example:
            * `man ls`
            * `man man`
            * `man stdio`
    * Some functions may have the same name, and you might want to be able to specify which section the function you want is in.
        * For example, `man printf` will automatically give you the page for a printf in the 1st section. If you wanted to look up a printf from stdio, however, (stdio is in the 3rd section) you would instead type `man 3 printf`. (see below for more details)

If you recall from yesterday, when we ran our Hello World code, we were given a warning when using the function `printf()`:
* `printf()` is part of stdio, which is the standard input/output library.

When compiling your C code, `gcc` will go through two steps:
1. Checks if everything matches (syntax, function names, arguments and return type, etc.)
   * If it finds a function it does not see defined in the code, it assumes that you know what you're talking about, and will give you a warning.
2. Links the code for the external function
    * `gcc` can automatically link functions in standard libraries

Although warnings will still allow you to compile and run your code, it is ideal if your code has no warnings!

* `#include`
    * Include it in the header of your code
    * Put system libraries inside <>
    * Two major libraries we will be using:
        1. `#include <stdio.h>`
        2. `#include <stdlib.h>`

---
## Tuesday, 9/12 Hello, World! by Jake Goldman

**Interesting News:** [Can AI recreate a game engine?](https://www.digitaltrends.com/computing/ai-super-mario-bros-game-engine/)

**Bonus:** [Obligatory iPhone X hype](https://www.wired.com/story/apple-iphone-x-iphone-8/)

We began class by looking at a basic hello world program in C:

```C
int main(){
  printf("Hello Everybody!\n");
  return 0;
}
```
After running the code, we discussed several observations:
* The `main()` method in C doesn't have arguments
* `printf()` is used to print to the command line (it doesn't add a newline at the end)
* The `main()` method can have a return type (usually this will be an int and can indicate how a program terminated)

We then moved on to discuss why we use `./filename` to run C executables. In general, when one uses a command, the machine will search through certain directories stored in a variable known as PATH. As an example, when someone uses `ls`, the computer will go through each of the directories stored in PATH to find a file named `ls`, which will then be run. The computer only searches in these predetermined directories as a means of security, as certain programs could theoretically run themselves, leading to all kinds of potential issues. By specifying the directory of the executable we are trying to run, we are telling the machine that this is a safe file, and showing where to run it.

History of C:

The C language was developed by Denis Ritchie in the early 70s while working on UNIX at Bell Labs. The language ended up being the basis of the UNIX kernel, and in 1978, Ritchie along with Brian Kernighan published "The C Programming Language," which is considered to be one of the best books ever written about a programming language.

C Design Choices:
* C is designed to match very closely with machine level instructions (this creates leaner and faster programs)
* C is a procedural language (NOT object oriented)
* C syntax is very similar to Java syntax (Java syntax was based on C)

---
## Monday, 9/11 Basic C Code by Helen Ye

**Interesting News:** [Pluto's features are being named!](http://www.skyandtelescope.com/astronomy-news/first-pluto-features-officially-named/)

#### Aim: Let's C what you can do

Basic C code
* Source Code:
	* Put in .c files (convention)
	* Naming convention: snake_case
* Compiling (with command `gcc`)
	* Compiles to architecture specific (OS + Processor) executable files
	* Default name of compiled file is a.out
	* gcc option: -o \<file\> compiles to the given file name
	* Run file with ./\<file\>
		* C checks the $PATH for the file if you just type in the file name, unless you specify  it's directory

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
