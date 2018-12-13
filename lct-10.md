## Monday - January 08 | Stop. Collaborate and listen by James Smith

**Interesting tech news:** [SpaceX successfully sends secret 'Zuma' satellite to space](https://www.cnet.com/news/spacex-zuma-falcon-9-heavy-cape-canaveral-elon-musk/)

### accept (server only) `<sys/socket.h>`
* Accept the next client in the queue of a socket in the listen state.     
* Used for stream sockets.
* Perform the server side of the 3 way handshake
* Creates a new socket for communicating with the child, the listening
* Socket is not modified
* Returns a descriptor to the new socket
* Blocks until a connection attempt is made
* `accept(socket descriptor, address, address length)`
	* socket descriptor: Descriptor for the listening socket
	* address: Pointer to a struct sockaddr_storage that will contain information about the new socket after accept succeeds.

	* address length: Pointer to a variable that will contain the size of the new socket address after accept succeeds

**Using listen and accept**
```C
      // create socket
      int sd;
      sd = socket(_AF, SOCK_STREAM, 0};

      // use getaafirno
      listen(sd, 10);

      int client_socket;
      socklen_t sock_size;
      struct sockaddr_storage client_address;

      client_socket = accept(sd, (struct sockaddr *)&client_address, &sock_size);
```
### connect (client only) `<sys/socket.h> <sys/types.h>`
* Connect to a socket currently in the listening state.
* Used for stream sockets
* Performs the client side of the 3 way handshake
* Binds the socket to an address and port
* Blocks until a connection is made (or fails)
* `connect(socket descriptor, address, address length)`
	* socket descriptor: descriptor for the socket		   
	* address: Pointer to a struct sockaddr representing the address
	* address length: size of the address, in bytes
	* address and address length can be retrieved from getaddrinfo()

**Using connect** 
```C
      // create socket
      int sd;
      sd = socket(AF_INET, SOCK_STREAM, 0);

      struct addrinfo * hints, * results;
      // use getaddrinfo (not shown)

      connect(sd, results->ai_addr, results->ai_addrlen);
```
---

## Friday - January 05 | Stop. Collaborate and listen by Kyle Lin

**Interesting tech news:** [Intel, ARM and AMD chip scare: What you need to know](http://www.bbc.com/news/technology-42562303)

###### How to use a socket:

1. Create the socket
2. Bind it to an address and port (server)
3. Listen and accept (server) or connect (client)
4. Send/receive data

##### socket - <sys/socket.h>

* Creates a socket.
* Returns a socket descriptor (int that acts like a file descriptor).
* socket(***domain***, ***type***, ***protocol***)
  * ***domain***
    * Type of address
    * `AF_INET` or `AF_INET6` (*IPv4* or *IPv6*)
  * ***type***
    * `SOCK_STREAM` or `SOCK_DGRAM` (*Stream* socket or *Datagram* socket)
  * ***protocol***
    * Combination of domain and type settings.
    * If set to 0, the OS will set to the correct protocol (*TCP* or *UDP*)
  * ***example:***
    * `int sd = socket(AF_INET, SOCK_STREAM, 0);`

##### struct addrinfo

System library calls use a `struct addrinfo` to represent network addresses (containing information like IP address, port, protocol...)

* `struct addrinfo`
  * `.ai_family`
    * `AF_INET`: *IPv4*
    * `AF_INET6`: *IPv6*
    * `AF_UNSPEC`: *IPv4 or IPv6*
  * `.ai_socktype` (Warm and fuzzy or 100% cotton)
    * `SOCK_STREAM`: *Stream*
    * `SOCK_DGRAM`: *Datagram*
  * `.ai_flags` (Lots of them but this one is the most - most useful to us)
    * `AI_PASSIVE`: Automatically set to any incoming address
  * `ai_addr`
    * Pointer to a `struct sockaddr` containing the IP address.
    * Note: This struct does not actually add socks. If you need to stay warm, using this won't do anything.
  * `ai_addrlen`
    * Size of the address in bytes.

##### getaddrinfo - <sys/types.h> <sys/socket.h> <netdb.h>

* Lookup information about the desired network address and get one or more matching `struct addrinfo` entries.
* getaddrinfo(***node***, ***service***, ***hints***, ***results***)
  * ***node***
    * String containing an IP address or hostname to lookup
    * If `NULL`, uses the machine's local IP address.
  * ***service***
    * String with a port number or service name (if the service is in /etc/services)
  * ***hints***
    * Pointer to a `struct addrinfo` used to provide settings for the lookup (type of address, etc.)
  * ***results***
    * Pointer to a `struct addrinfo` that will be a linked list containing entries for each matching address.
    * `getaddrinfo()` will allocate memory for these structs.

##### Using getaddrinfo()

```C
struct addrinfo * hints, * results;
hints = (struct addrinfo *)calloc(1, sizeof(struct addrinfo));
hints->ai_family = AF_INET;
hints->ai_socktype = SOCK_STREAM; // TCP Socket
hints->ai_flags = AI_PASSIVE; // Only needed on server
getaddrinfo(NULL, "80", hints, &results); // Server sets node to NULL
//client: getaddrinfo("149.89.150.100", "9845", hints, &results);
//do stuff...
free(hints);
freeaddrinfo(results);
```

##### bind (server only) - <sys/socket.h>

* Binds the socket to an address and port.
* Returns `0` (success) or `-1` (failure)
* bind(***socket descriptor***, ***address***, ***address length***)
  * ***socket descriptor***
    * Return value of socket
  * ***address***
    * Pointer to a `struct sockaddr`
  * ***address length***
    * Size of the address in bytes
  * ***address*** and ***address length*** can be obtained from getaddrinfo

##### Using bind

```C
//Create socket
int sd;
sd = socket(AF_INET, SOCK_STREAM, 0);
struct addrinfo * hints, * results;
//use getaddrinfo, roast marshmellows, starting a campfire...
bind(sd, results->ai_addr, results->ai_addrles);
```

##### listen (server only) - <sys/socket.h>

* Sets a socket to passively await a connection.
* Needed for stream sockets.
* Does not **block**

---


## Wednesday, 01/03 | Socket it to me by Kristin Lin

**Interesting tech news:** [Fooling AIs with Stickers](https://www.theverge.com/2018/1/3/16844842/ai-computer-vision-trick-adversarial-patches-google)

***Why haven't we switched to IPv6?***             
The system for IPv4 is incompatible with IPv6. While many computers can support both IPv4 and IPv6, it's the Internet providers that prevent us from making the switch. They will have update their infrastructure, which takes both money and time. Companies like Google, who are interested in more IP addresses (more users), are urging current users to voice their concern to their providers.

*Sockets are analagous to named pipes. Instead of names, they require ports.*
### Network Ports ###
- If *sockets* were APARTMENT BUILDINGS,                          
the *IP address* would be the FRONT DOOR,                               
and the *port* would be each unique APARTMENT.                        

- There are 2^16 ports in your computer, or 65, 536 possible connections to programs.

- ONLY RULE: You can't use one port for two different programs. No conflicts.

###### RESERVED PORTS ######
Ports 1024 and under are officially reserved and should not be used.
- 80 : http                  
- 22 : ssh                     
- 443 : ssl                   
- [more](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers)

You can register your port officially with the IANA (Internet Assigned Numbers Authority).

### Network Connection Types ###
STREAM SOCKET                     
**TCP: Transmission Control Protocol**                
- reliable two-way communication                 
- must be connected on both ends                  
- data is received in the order it is sent (not as easy as it sounds)

DATAGRAM SOCKET              
**UDP: User Datagram Protocol**                
- "connectionless"--an established connnection is not required.                
- Data sent might be received out of order or not at all *(because data is sent in packets and may not take the same route; therefore arriving in a different order)*         
- Advantage of being much faster because no need to package everything together                
- Used in games and streaming media

---

## Tuesday, 1/2 How the networks by Brian Lim

**Interesting tech news:** [Big tech announcements expected in the upcoming Consumer Electronics Show](http://www.techradar.com/news/ces-2018)

### Sockets

A socket is a connection between 2 programs over a network. Each corresponds to an IP address / port pair. When using sockets, one program waits for a connection while the other initiates a connection.

How to use sockets:
* create the socket
* bind it to an IP / port pair
* wait for / initiate a connection, depending on which end the program is dealing with
* receive / send data

### IP Addresses

Whenever a device connects to the internet, it must get a unique IP address. Much like email addresses, the IP address is used to identify the device when it wants to communicate to the outside. When the device disconnects from the internet, the IP address is no longer needed and may be released for another device to use.

IP addresses are allocated in blocks to make routing easier.

Two types of addresses are used:

**IPv4**
* standard protocol
* 2^32 or ~4 billion addresses possible
* represented as 4 octets, each one byte represented as a number from 0 to 255, separated by dots
	* ex: 123.4.567.8

**IPv6**
* newer, fancier protocol
* 2^128 or ~10^38 addresses possible
* represented as 8 hextets, each 2 bytes represented as a hexadecimal from 0 to ffff with leading 0s ignored, separated by colons
	* ex: 1:123:90:ffa2:42:ffff:332:1
* a single string of consecutive zero hextets can be replaced by ::
	* ex: 1:80::2
* can represent IPv4 addresses as 5 zero hextets, 1 ffff hextet, followed by the IPv4 address representation
	* ex: ::ffff:149.89.150.100

**Special addresses**
127.0.0.1
* the localhost or loopback address
* refers to the local device
192.168.###.###
* refers to other devices in the private network
192.168.1.1
* usually refers to the router

---

## Monday, 12/18 | Fork off, subservers! by Peter Lee

**Interesting Tech News:** [US declares North Korea the culprit behind devastating WannaCry ransomware attack](https://www.theverge.com/2017/12/18/16793532/us-north-korea-wannacry-ransomware-cyberattack-trump-admin)

We already learned how to establish a connection between programs via the FIFO by completing Work 16. It's time to expand on our server-client design and allow for multiple clients to connect!

### Why?
A server where only one client can connect at a given time isn't too useful. 
We're going to have to implement a bit more code to enable multiple clients to connect to our server at once. 

### How?
We modify our handshake!
<ol>
	<li> Client connects to server and sends the private FIFO name. Client waits for a response from the server. </li>
	<li> Server receives clients message and <strong>forks off a subserver</strong>. The subserver will have a connection 		   to the WKP. </li> 
	<li> Server closes and removes the WKP. This does not affect the subserver's connection! </li>
	<li> Subserver connects to client FIFO, sending an initial acknowledgement message. </li>
	<li> Client receives subservers message, removes its private FIFO. </li>
	<li> Client sends response to subserver. </li>
	<li> Job done! </li>
</ol>
We Operate!
<ol>
	<li> Server recreates WKP and waits for a new connection. </li>
	<li> Subserver and client send information back and forth. </li> 
</ol>


## Monday, 12/11 | Server and Client - Handshaking by Adris Jautakas

**Interesting Tech News:** [Apple Said to Be Acquiring Shazam, the Song Identifying App](https://www.nytimes.com/2017/12/08/business/media/apple-shazam.html?rref=collection%2Fsectioncollection%2Ftechnology&action=click&contentCollection=technology&region=stream&module=stream_unit&version=latest&contentPlacement=5&pgtype=sectionfront)

Greetings humans.

I hope you're ready for this.

We're talking about servers and clients.

### BY ALL THAT IS HOLY, WHAT IS A SERVER AND A CLIENT

Oh young child, I shall show you the way.

A **Server** is a process that is constantly running.

A **Client** is a process that connects to a server.

Servers and clients send and receive information between each other.

You're probably all familiar with what a multiplayer game is.

If not, get out of my sight.

Each player has its own client that connects to a server. The server sends data back to the clients
to let the players see each other and the client sends info to the server so that the players
can actually move in the game.

In order to start a server-client connection, the server and client must verify that they can both
send and receive information properly

### INFORMATION YOU SAY??? HOW IN GODS NAME DO THEY SEND THAT

Hush, I'm internally falling asleep and require that you be quit.

There must be two FIFO pipes that allow a server to send info to the client and vice versa.

### HOW ARE PIPES SET UP WHAT IS ANYTHING HALP

Turn off your caps lock please.

It all starts with the server creating a public FIFO pipe. This pipe
can have any name, and can be accessed by anyone.

The server creates the pipe and waits for the client to connect.

Once the client connects, the client creates its own FIFO pipe with
a "private" (hard to guess) name, that can connect to the server.

You must try and make the client pipe have some kind of cryptic fancy name that no
hacker can guess, to try in vain to make it secure.

### BUT HOW OH HOW DO THEY VERIFY THAT THEY CAN SEND AND RECEIVE INFORMATION

Calm yourself. There is much to learn.

This is done using a fantabulous process kown as the ~ * **_SERVER-CLIENT H A N D S H A K E_** * ~ .

Both the server and the client must verify that they are able to safely send and receive information.
They must communicate with each other to make this work.

The H A N D S H A K E follows four simple steps:

1) Set up a server and client pipe
2) Client sends info to Server, including the name of the client's secret pipe
3) Server receives Client's FIFO pipe, and tries to send data back to the Client.
	This information should be the same information that the Client sent to the Server (the name of the secret pipe).
4) Client receives Server information, checks to see if the received data is correct
5) Client then sends the data back to the Server, so the Server can verify that it is able to send data successfully.

With these steps, the Client and Server can verify that they can both send and receive data correctly.

Isn't that absolutely the dandiest thing you've ever heard?

Now if you excuse me I'll try to find another way to escape from this grueling reality.


## Thursday, 12/07 | What's a semaphore? - To control resources by Marcus Ng

**Interesting Tech News:** [Instagram Standalone App For Direct Messaging](https://techcrunch.com/2017/12/07/instagram-is-testing-a-standalone-app-for-direct-messaging/)

## semctl

* DATA argument
	* Variable for setting/storing semaphore metadata
		* Type union semun
		 	* You have to declare this union in your main c file on linux machines.
```C
union semun {
	int val; // used for SETVAL (4 bytes)
	struct semid_ds *buf; // used for IPC_STAT and IPC_SET (8 bytes)
	unsigned short *array; // used for SETALL (8 bytes)
	struct sminfo *__buff; // (8 bytes)
};
```
			  
## What is a union?
* A c structure designed to hold only one value at a time from a group of potential values
* Difference between a union and struct
	* A union only contains one of those things at any given time
* Just large enough to hold the largest piece of data it could potentially contain
	* semop( DESCRIPTOR, OPERATION, AMOUNT )
		* Perform an atomic semaphore operations
		* You can Up/Down a semaphore by any integer value, not just 1
		* Descriptor: Returned from semget
		* Operation: A pointer to a struct sembuf
		* Amount: The amount of semaphore you want to operate on in the semaphore set.
		
```C
struct sembuf {
	short sem_op; // Down(S): Any negative number / Up(S): Any positive number
	short sem_num; // The index of the semaphore you want to work on
	short sem_flag; // SEM_UNDO, IPC_NOWAIT
}
```
* 0: Block until the semaphore reaches 0
* sem_flag
	* SEM_UNDO: Allow the OS to undo the given operation. Useful in the	event that a program exits before it could release a semaphore.
	* IPC_NOWAIT: Instead of waiting for the semaphore to be available, return an err

---

## Tuesday, 12/05 | How Do We Flag Down a Resource? by Jen Yu

**Interesting Tech News:**[A.I. Will Transform the Economy. But How Much, and How Soon?](https://www.nytimes.com/2017/11/30/technology/ai-will-transform-the-economy-but-how-much-and-how-soon.html?rref=collection%2Fsectioncollection%2Ftechnology)

## Semaphores ## 
  * An IPC construct created by Edsger Dijkstra used to __control access to a shared resource__ (like a file or shared memory)
  * Usually, a semaphore is used as a counter, representing how many processes can access a resource at a given time.
     * If a semaphore has a value of 3, then it can have 3 active users
     * *If it has a value of 0, then it is unavailable.*
  * Most operations are atomic (will not be split up into multiple processor instructions)
## Semaphore Operations ##
*These operations are used to:*
 1. Create a semaphore - default is 0
 2. Set an initial value
 3. Remove a semaphore
  
```Up(S) / V(S)``` - atomic
   * Releases the semaphore to signal you are done with its associated resource
   * Pseudocode: ```S++```
   
```Down(S) / P(S)``` - atomic
   * **Attempts** to take the semaphore
   * If the semaphore is 0, waits for it to be available
   * Pseudocode: ```while(s==0){block} s--```

**When using semaphores in a file, include: ```<sys/types.h>, <sys/ipc.h>, <sys/sem.h>```.**

```semget(<KEY>, <AMOUNT>, <FLAGS>)```
  * Create/get access to a semaphore, **does not modify!**
  * **Returns a semaphore descriptor or -1 (errno)**
    * ```KEY```: Unique semaphore identifier
    * ```AMOUNT```: # to be created/gotten in the set of sephamores (stored as one or more)
    * ```FLAGS```: Permissions for the semaphore, combine with bitwise or
      * ```IPC_CREAT```: create the semaphore and set the value to 0
      * ```IPC_EXCL```: Fail if the semaphore already exists and ```IPC_CREAT``` is on
---
## Monday, 12/04 Memes by Bayan Berri

**Interesting Tech News:** [Oracle High School](http://www.businessinsider.com/oracle-high-school-campus-2017-12)

## Memes and Memory ##
The aim meme refers to the history of the origin of the word meme from the book The Selfish Gene. This book was about how cultural information spreads and is shared.
Fun Fact: it's pronounced meme so that it sounds like gene 😆
## Some more Shared Memory Functions ##
### ```shmdt(pointer)``` ###
* Detach a variable from a shared memory segment
* Returns 0 upon success or -1 upon failure. 
* Pointer: address used to access the segment
### ``` shmctl(descriptor, command, buffer)``` ###
* Perform operations on the shared memory segment
* Each shared memory segment has metadata that can be stored in a struct
a) ```Struct shmid_ds```
* Some of that data stored:last access, size, pid of creator, pid of last modification.
* Descriptor: return value of shmget
* Commands:
1) ```IPC_RMID```: remove a shared memory segment
2) ```IPC_STAT```: populate the buffer with information (struct shmid_ds)
3) ```IPC_SET```: set some of the information for the segment to the info in buffer.

To see your shared memory, you can use ```$ ipcs -m ```

---

## Friday 12/01 | Sharing is Caring! by Grace Cuenca
**Tech News**: [Metascarcity and Bitcoin’s future](https://techcrunch.com/2017/12/03/metascarcity-and-bitcoins-future/)

## Shared Memory
- `<sys/shm.h>`, `<sys/ipc.h>`, `sys/types.h`
- Is a segment of heap memory that can be accessed by multiple processes
- Does not get released when a program exits. 
	- Thus, when you're done with a piece of shared memory, you must manually release it.

### 5 Shared memory observations
- Create the segment (happens once)
- Access segment (happens once per process)
- Attach the segment to a variable (once per process) 
	- ex. pointer to point to the shared memory
- Detach segment from a variable (once per process)
	- This doesn't change the value of the pointer, but it is detached so you don't have access to it anymore
- Remove the segment (happens once)

### `shmget`
- Create or access a shared memory segment
- Returns a shared memory descriptor (similar concept to file descriptor) or -1 if it fails (errno)
```c
shmget(key, size, flags)
```
key
- unique integer identifier for the shared memory segment (like a file name)
- is something you make up, not the same as memory descriptor

size
- how much memory to request

flags
- includes permissions for the segment (combined with bitwise or)
- `IPC_CREAT` creates the segment and sets all vals to 0
- `IPC_EXCL` fails if the segment already exists & `IPC_CREAT` is on

Example:
```c
#define KEY 100
int shmdt = shmget(KEY, sizeof(double), IPC_CREAT | 0600)
```

### `shmat`
- attach a shared memory segment to a variable
- returns a pointer to the segment or -1 (errno)
```c
shmat(descriptor, address, flags)
```

descriptor
- the return value of shmget

address
- if 0, OS will provide the appropriate memory address

flags
- usually 0, there is one useful flag: `SHM_RDONLY` which makes the memory readable

---

## Tuesday 11/28 | C, the ultimate hipster, using # decades before it was cool by Ryan Siu
**Tech News**: [MIT’s new desktop 3D printer technology increases speeds up to 10x](https://techcrunch.com/2017/11/28/mits-new-desktop-3d-printer-technology-increases-speeds-up-to-10x/)

## \#
- Used to provide preprocessor instructions.
- These direcives are handled by `gcc` first.

### #include 
`#include <LIBRARY> | "LIBRARY"`
- Link libraries to your code.

### #define 
`#define <NAME> <VALUE>`
- Replace all occurrences of NAME with VALUE

```c
#define TRUE 1
```

### Macros
- Macros act as functions
```c
#define SQUARE(x) x * x
y = SQUARE(3); // y = 3 * 3
#define MIN(x,y) x < y ? x : y
```

### #ifndef

```c
#ifndef <IDENTIFIER>
#define <IDENTIFIER>
<CODE>
#endif
```
- Code between the `#ifndef` and `#endif` only run if the IDENTIFIER has not been defined

---

## Monday 11/27 | File Redirection by Jensen Li
**Tech News** [Apple iPhone Foldable Patent](https://www.cnet.com/news/apple-iphone-foldable-patent/)

**Aim: Redirecting, how does it... SQUIRREL**

*What is file redirection?*
- File redirection is about changing the usual input/output behavior of a program

### BASH FUNCTIONS

```>```
- redirects stdout to file
  - ex. $ ls > output
  - *creates* output file (if it doesn’t exist) and *overwrites* contents of output

```>>```
- Redirects stdout to a file by appending

```2>```
- Redirects *stderr* to a file
- Overwrites the file (2>> appends)

```&>```
- Redirect *stdout and stderr* (&>> appends)

```<```
- Redirects *stdin* to a file

```| (pipe)```
- Redirects stdout from one command to stdin of the next
  - ex. ls | wc
  - will ls based off of wc (word count)

### COMMANDS IN C PROGRAMS

```dup( int <file descriptor> )```
- found in <unistd.h>
- Duplicates an existing entry in the file table
- Returns a new file descriptor of the duplicate entry

```dup2( int <fd1>, int <fd2> )```
- found in <unistd.h>
- Redirects fd2 to fd1
- Duplicates the behavior for fd1 at fd2
- You will lose any reference to the original fd2, that file will be closed
- If fd2 doesn’t exists, then return -1

### Things to note
- ```>``` overwrites but ```>>``` appends
- If you want to keep the fd2 then dup it and replace it later on with dup2

---

## Tuesday, 11/21 | A pipe by any other name | Ahbab Ashraf

**Tech News:** [Uber data breach](https://www.cnet.com/news/uber-hid-data-breach-paid-hackers-to-delete-user-and-driver-info/)

#### Named Pipes
* aka FIFOs (first in first out, like queues in Java)
* Interacted with in the same way that unnamed pipes are (reading, writing, unidirectionality)
* The key difference is the presence of a name, which lets it be accessed by other programs
* mkfifo
	* Shell command to make a named pipe
	* Usage: ```mkfifo <name>```
	* The pipe shows up as a special file when you ls, can be read and written to by multiple programs
	* Removing the pipe keeps the connection between programs, only it's now an unnamed pipe
* mkfifo (sys/types.h, sys/stats,h)
	* C function to create a named pipe
	* Usage: ```mkfifo(<name>, <permissions>)```
	* Returns 0 on success and -1 on failure
	* The pipe will block on open until there is a connection on both ends

---

## Wednesday, 11/15 | Playing Favorites By Cynthia Cheng
**Tech News:**  [Mozilla terminates its deal with Yahoo and makes Google the default in Firefox again](https://techcrunch.com/2017/11/14/mozilla-terminates-its-deal-with-yahoo-and-makes-google-the-default-in-firefox-again/)

Threads vs Forking
Fork: gives you a brand new process (child) which is a copy of the current process 
Threads: one process can have multiple threads (needs a "parent" process)
Threading allows easier access to memory and shared data vs child processes which do not share memory

### Example
```c
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
void print_byte(void *p, int size){
  int i =0
  unsigned char *cp = (unsigned char*) p;
  for (;i <size; i++){
    printf("byte %d: %u\n", i*cp);
    cp++;
  }
int main(){
int x = 32;
print_byte(&x, sizeof(x)) //print individually all the bytes
}
```
x has 32 in it

integer is 4 bytes

[32,0,0,0]

cp* points at 32 then goes through til after 0

result:

32

0

0

0


int x = 256;

print_byte(&x, sizeof(x));

result:

0

1

0

0



int x = 302;

print_byte(&x, sizeof(x))

result:

46

1 (!!)

0

0


46 in binary: 00101110

1 in binary: 0000001

0000001,00101110

^now the 1 is the 256

#### Endianness - byte order of numbers
* Little Endian: least significant byte to the most significant byte 
  46 = [01110100]
* Big Endian: most significant byte to the least significant byte 
  46 = [00101110]

#### waitpid(pid, status, options) - <unistd.h>
* waits for a specific child
* pid: the pid of the child process one is trying to find, or -1 for any child process
* status: Specifies the location to which the child process' exit status is stored.
* options: gives options that can set other behaviors for wait, or 0 for normal functions


## Tuesday, 11/14 | Wait for it By Peter Lee
**Tech News:**  [The FDA has approved the first digital pill](https://www.theverge.com/2017/11/14/16648166/fda-digital-pill-abilify-otsuka-proteus)

We learned in more detail about how we can determine which child belongs to which parent by using the return values of fork() and getppid(). We also learned how to be good parents by waiting for our slow children.

#### wait(int * status) - <unistd.h>
* Stops a parent process from running until any child has provided status information to the parent via a signal. (usually the child has exited).
* Returns the pid of the child that exited, or -1 (errno)
* The parameter (status) is used to store information about how the process exited.

#### getppid() - <unistd.h>
* Returns the process ID of the parent process of the current process.

#### Example
```c
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>

int main() {
  int f, t;
  printf("pre-fork\n");

  f = fork(); //fork() returns the PID of the child process
  t = 2;
  if (f == 0) {
    sleep(t); //child is asleep for 2 seconds!
    printf("I'm a child: %d\t f:%d\tparent: %d\n", getpid(), f, getppid());
  } else {
    printf("waiting for lazy child\n");
    wait(&t); //wait for child to sleep for 2 seconds!
    printf("I'm a parent: %d\t f:%d\tparent: %d\n", getpid(), f, getppid());
  }
  return 0;
}
```


---


## Friday, 11/13 | I Don’t Give a Fork! By Rihui Zheng
**Tech News:**  [Youtube Removes Extremist Videos](https://www.reuters.com/article/us-tech-hatespeech/google-broadens-takedown-of-extremist-youtube-videos-idUSKBN1DE05X)

We went over our signal assignment for the majority of the class. Most people did not realize that you were supposed to append a message to a file. Shame on you. We then went over how strsep() works.

#### Usage
`strsep( <string> , <separator> )`

Finds the first occurrence of separator in string and set the separator equal to null. It makes string point to the index after separator and returns the original string.

We then began learning about forks. We learned one function:
#### fork()- <unistd.h>
* Creates a separate process based on the current one. The new process is called a child, the original is the parent.
* The child process is a duplicate of the parent process. All parts of the parent process are copied, including stack and heap memory, and the file table.

#### Example
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(){
    printf(“Pre-fork: \n”);
    fork();
    printf(“Post-fork: %d\n”, getpid());
}
```
---


## Thursday, 11/9 | Executive Decision by Marcus Ng

**Tech news**: [Apple AR Headset](https://www.bloomberg.com/news/articles/2017-11-08/apple-is-said-to-ramp-up-work-on-augmented-reality-headset)

We learned about using execlp() and execvp() to execute processes in a C program.

#### int execlp(const char *file, const char *arg0...NULL);
The first argument of the function is the name of an executable file, which is searched for in the $PATH. The $PATH lists all of the directories where there are known executable programs. The rest of the arguments make up the command that is executed to create the new process. The arguments must end with NULL to signal that the arguments have ended. If the new process is successful, the old PID is used for the new process, replacing the old process. However, if the new process fails, the function returns -1, and the global variable `errno` is modified to reflect the error.

#### Example
```c
#include <unistd.h>

int main() {
    execlp("ls", "ls", "-a", "-l", NULL) // Remember to have NULL at the end of your args

    return 0;
}
```

#### int execvp(const char *file, char *const argv[]);
This function is similar to `execlp()`, except it takes an array of arguments following the name of the executable file instead of a list of individual arguments. The argument array must have NULL to signal that the arguments have ended.

#### Example
```c
#include <unistd.h>

int main() {
    char *arg[5];
    arg[0] = "ls";
    arg[1] = "-a";
    arg[2] = "-l";
    arg[3] = NULL; // NULL Terminated
  
    execvp(arg[0], arg);

    return 0;
}
```

---

## Wednesday, 11/8 | Mixed Signals by James Smith
**Tech news**: [World's Fastest Rocket Car's First Test](https://www.cnet.com/news/bloodhound-ssc-rocket-car-finally-makes-its-first-move/)

Today we discussed signals in programs.

#### Processes
`$ ps` Lists your current processes.
`$ ps -aux` Shows all processes with the username who started the process and the processes not associated with the terminal session. An example of the output can be seen below:

```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0  10436   104 ?        Ss   19:06   0:00 /init
Inquisi+     2  0.0  0.0  25688  2876 tty1     Ss   19:06   0:00 /bin/bash
Inquisi+    38  0.0  0.0  41356  1844 tty1     R    19:23   0:00 ps -aux
```

The important thing here is the PID which stands for Process ID. This is how the computer identifies your processes. It gets this information from the /proc/ directory.

What is /proc/ directory?
It is a directory that contains folders for every process and inside each folder it has the executed code and statistics on the process. My /proc/ directory is below as an example:
```
total 0
dr-xr-xr-x 7 root       root       0 Nov  8 19:22 1
dr-xr-xr-x 7 Inquisitor Inquisitor 0 Nov  8 19:22 2
dr-xr-xr-x 7 Inquisitor Inquisitor 0 Nov  8 19:27 56
dr-xr-xr-x 2 root       root       0 Nov  8 19:06 bus
-r--r--r-- 1 root       root       0 Nov  8 19:06 cgroups
-r--r--r-- 1 root       root       0 Nov  8 19:06 cmdline
-r--r--r-- 1 root       root       0 Nov  8 19:06 cpuinfo
-r--r--r-- 1 root       root       0 Nov  8 19:06 filesystems
-r--r--r-- 1 root       root       0 Nov  8 19:06 interrupts
-r--r--r-- 1 root       root       0 Nov  8 19:06 loadavg
-r--r--r-- 1 root       root       0 Nov  8 19:06 meminfo
lrwxrwxrwx 1 root       root       0 Nov  8 19:06 mounts -> self/mounts
lrwxrwxrwx 1 root       root       0 Nov  8 19:06 net -> self/net
lrwxrwxrwx 1 root       root       0 Nov  8 19:06 self -> 56
-r--r--r-- 1 root       root       0 Nov  8 19:06 stat
dr-xr-xr-x 6 root       root       0 Nov  8 19:06 sys
dr-xr-xr-x 2 root       root       0 Nov  8 19:06 tty
-r--r--r-- 1 root       root       0 Nov  8 19:06 uptime
-r--r--r-- 1 root       root       0 Nov  8 19:06 version
-r--r--r-- 1 root       root       0 Nov  8 19:06 version_signature
```

#### What is a signal?
A signal is a limited way to send information to a process. It sends an integer that tells the process to do something. Some important ones can be seen below, but all of them can be seen at `man 7 signal`

SIGINT - 2 - Process interupt from keyboard (aka ctrl-c)
SIGSTP - 20 - Stops program from keyboard (aka ctrl-z)
SIGSTOP - 19 - Stops process (used from a program)
SIGCONT - 18 - Continues the program if it is stopped
SIGSEGV - 11 - Invaliud memory address (aka segfault)
SIGTERM - 15 - Termination signal

#### Kill all the processes!
`$ kill <PID>` is a command line utility used to send a signal to the process associated with the PID
By default it sends a SIGTERM signal, but you can send any signal you want by supplying it the flag of the desired signal
`$ kill -<signal int> <PID>`
Additionally you can use `$ killall <program>` to kill an entire program using its name.

#### Using Signals in C
`kill(PID, signal)` : This is the terminal program kill, just in a function form. Returns 0 on success, -1 on failure.
`static void sighandler(int signo)` : A function YOU code in your C program to intercept signals for your program. Can handle all signals besides for SIGKILL and SIGSTOP. An example function could be:
```c
static void sighandler(int signo){
    if(signo == 11){
        printf("OH NOOOOO I HAVE A SEGFAULT!!!!!!!!");
    }
}
```
Your job is not over for handling signals. You have this function, but how is the program going to call it?
`signal(<signal>, sighandler)` This goes in your main method where it checks if the <signal> has been received and calles the sighandler function if it has.

---

## Monday, 11/6 | Are your processes running? Then you should go out and catch them! by Jawadul Kadir

**Tech news**: [Razer made a smartphone, and it’s an all-black version of the Nextbit Robin](https://arstechnica.com/gadgets/2017/11/razer-uses-nextbit-knowledge-to-craft-its-own-gamer-friendly-smartphone/)

We continued going over input functions, learning `fgets` and `sscanf`. We also went on to talk about what processors and processes are.

### fgets - `<stdio.h>`
Read in from a file stream and store it in a string

#### Usage:
`fgets(<DESTINATION>, <BYTES>, <FILE POINTER>)`

#### Given `fgets(s,n,f);`:

* Reads at most n-1 characters from file stream f and stores it in string s.
* Stops at newline, end of file, or the byte limit.
* If applicable, keeps the newline character as part of the string.
* A terminating null is appended.

##### File pointer

`FILE *` type, more complex than a file descriptor.

`stdin` is a `FILE *` variable that refers to standard in.

#### Example:
```c
#include <stdio.h>

int main(){
  char s[200];
  int i;
  float f;

  printf("enter data...");
  fgets(s, sizeof(s), stdin);

  printf("s: %s\n", s);
  return 0;
}
```
This example prompts the user to enter text, which `fgets` reads from `stdin` and copies it to s. Note that it will only read the first 199 characters, as `sizeof(s)` is 200, and there has to be room for the terminating null. It then prints out the value of s, which is the same as what the user inputted.

#### Other notes on `fgets`:

* NEVER USE `gets`! It does not have the byte limit that `fgets` has, which could lead to a buffer overflow.
* Blocking is when the program is waiting. `fgets` blocks while it is waiting to read something.
* Typing in the console does not get read until you hit enter, where the newline flushes the buffer.

### sscanf - `<stdio.h>`
Scan a string and extract values based on a format string. It returns the number of variables inputted.

#### Usage:
`sscanf(<SOURCE STRING>, <FORMAT STRING>, <VAR 1>, <VAR 2>�)`

#### Example:
```c
#include <stdio.h>

int main(){
  int i;
  float f;
  sscanf("4 1.3", "%d %f", &i, &f);

  printf("i: %d\n", i);
  printf("f: %f\n", f);
  return 0;
}
```
This example reads the integer 4, then changes the value at the address of i to 4, then a space, then the float 1.3 then changes the value at the address of f to 1.3. It then prints the values of i and f, which are 4 and 1.3 respectively.

### Processors
* A processor reads a certain number of bits at a time, usually 64.
* The processor does this several billions of times per second.
* A computer doesn't really multi-task, rather it cycles between processes very quickly.

### Processes
* Use `ps` in the console to list processes
* `ps` only displays processes that the user is the owner of and that are attached to a terminal session.


---

## Friday, 11/3 | Arguably, the main parts of the lesson by Brian Lim

**Tech news**: [Sony's bringing its robotic dog, Aibo, back to the market](https://www.cnet.com/news/sony-brings-back-aibo-teaches-old-dog-new-tricks/)

We spent a good amount of time reviewing the last assignments, dirinfo and stat. There were various approaches to those problems; make sure you have one that works.

### main
While we're used to writing `main` with no arguments, `main` can be defined alternatively as `int main(int argc, char *argv[]) {...}`, where the `argv` parameter itself can be written as `char **argv` or `char argv[][]`. This definition can be used to retrieve arguments from the command line.

`argc` gives the number of arguments that were input, while `argv` gives a list of the arguments. `argc` and `argv` both count the program name as an argument.

Ex:
```./a.out ab cd```
* `argc`     -> 3
* `argv[0]`  -> "./a.out"
* `argv[1]`  -> "ab"
* `argv[2]`  -> "cd"

---

## Wednesday, 11/1 | Where do compsci priests keep their files? by Kevin Li

**Tech news**: [Russia-Financed Ad Linked Clinton and Satan](https://www.nytimes.com/2017/11/01/us/politics/facebook-google-twitter-russian-interference-hearings.html?rref=collection%2Fsectioncollection%2Ftechnology&action=click&contentCollection=technology&region=rank&module=package&version=highlights&contentPlacement=1&pgtype=sectionfront)

### Directories
Linux and Unix directories are simply files that contain names of other files and basic information about them, like file type and size. This means that changing a directory, unless a file is being moved across drives, does not involve actually moving data-- rather, the directory files are just modified.

We learned three ways that we can work with directory files in C:
#### opendir - <dirent.h>
* ```opendir (<PATH>);```
* Opens a directory file without changing the current working directory (cwd)
* You don't need to specify any permissions because directory files are read only
* returns a pointer to directory stream (```DIR *```)

#### closedir - <dirent.h>
* ```closedir (<DIRECTORY STREAM *>);```
* Closes the directory stream and the pointer associated with it

#### readdir - <dirent.h>
* ```readdir (<DIRECTORY STREAM *>);```
* Returns a ```struct dirent``` to the next entry in a directory stream
* If all the entries have been returned, will return NULL

#### struct dirent - <sys/types.h>
* contains information stored in a directory file
* Some data members:
	* ```d_name```: Name of file
        * ```d_type```: File type as an int which matches a constant that you can find on the man page
* An example:
```c
DIR * d;
d = opendir (“somedir”);
struct dirent *entry;
entry = readdir (d);
closedir(d);
```

---

## Monday, 10/30 | lseek and ye shall receive | Adris Jautakas

**Tech news**: [What Virtual Reality Can Teach a Driverless Car](https://www.nytimes.com/2017/10/29/business/virtual-reality-driverless-cars.html?rref=collection%2Fsectioncollection%2Ftechnology)

Hello Mr. D.W. and that one person who was absent today! Here's what we learned while you were either teaching the class or were absent from it

### lseek and FIle bookmarks
When reading or writing to files, the file is iterated through via a "bookmark" index that is incremented to pass through each byte of data in the file. To read the entirety of a file, that bookmark must be set back to the start of the file. This bookmark can be adjusted manually with the `lseek` command.

#### Usage:
```c
#include<unistd.h>
lseek(File Descriptor, Offset, Whence);
```

#### Arguments:

`File Descriptor`: The file descriptor,

`Offset`: How many bytes to offset our bookmark. Negative values are acceptable.

`Whence`: Where to start the offset from.

Whence has three different parameters:

* `SEEK_SET`: Offset from the start of the file
* `SEEK_CUR`: Offset from the current index of our bookmark
* `SEEK_END`: Offset from the end of the file

For example, if you call lseek by passing `SEEK_CUR` as the Whence argument, you will move ahead in the file by the Offset argument's value, potentially skipping over a number of bytes of data.

#### Returns:
the absolute position of the file's bookmark relative to the start of the file, or -1 if there is an error. (If there is an error, errno is set)

### stat
stat allows us to access a file's metadata

#### Usage:
```c
#include<sys/stat.h>
stat(Path, Stat Buffer);
```

#### Arguments:
`Path`: The file path. Note, the file does not need to be opened to grab its metadata, hence why we're using its file path and not a file descriptor.

`Stat Buffer`: A pointer to an instance of a special buffer struct that can hold metadata.

#### Returns:
0 upon successful completion, -1 if unsuccessful and errno is set.

#### Example:
```c
xstruct stat yay_college_essays;
stat("foo_file", &yay_college_essays);
// Access the metadata somehow!
```

We didn't go over the variables that the stat struct holds, but here's a link that I found which shows you the contents of struct stat: [stat.h Programming Help](https://en.wikibooks.org/wiki/C_Programming/stat.h#Member_types)

Stay frosty.

---

## Wednesday, 10/25: Opening up a world of possibilities by Ryan Siu

**Tech News:** [Who gets to choose the type of ads we see online?](http://www.businessinsider.com/google-chrome-ad-blocking-forces-ad-tech-cos-to-abandon-business-2017-10)

### File Table
- A list of all files being used by a program while it is running.
- Contains basic information like the file’s location and size.
- The file table has a limited size, which is a power of 2 and commonly 256
- ```getdtablesize()``` will return the size

- Integer index of file tables is the file descriptor (fd)

- There are 3 files always open in the table:
  - 0 or ```STDIN_FILENO```: stdin
  - 1 or ```STDOUT_FILENO```: stdout
  - 2 or ```STDERR_FILENO```: stderr

### Open
- Contained in <fcntl.h>
- Add a file to the file table and returns its file descriptor
- If open fails, -1 is returned, extra error info can be found in errno
  - ```errno``` is an int variable that can be found in <errno.h>, using ```strerror()``` (in <string.h>) on ```errno``` will return a string description of the error

<strong>Usage</strong>:
```
open( <PATH>, <FLAGS>, <MODE> )
```
- mode: only user when creating a file, set the permissions using a 3 digit octal #
- flags: determine what you plan to do with the file
  - constants: O_RDONLY, O_WRONLY
  - Each flag is a number, to combine flags use bitwise OR like the following:
```		
O_WRONLY = 1		00000001
O_APPEND = 8		00001000
O_WRONLY | O_APPEND =	00001001
```
---

## Tuesday, 10/24: File this under useful information. (By Kyle Lin)

**Tech News** [Porsche built a Cayman electric car concept that's suprisingly quick](https://www.cnet.com/roadshow/news/porsche-built-an-electric-cayman-concept-thats-surprisingly-quick/)

&nbsp;

### File Permissions
There are three types of permissions
* Read [r]: File can be read
* Write [w]: File can be written to
* Executable [x]: File can be executed  

Permissions can be represented as 3 digit binary numbers or as a single digit octal.  
* 0b100 <=> 04 => Read-Only
* 0b111 <=> 07 => Read, Write, and Executable

### Permission 'Areas' (On Unix systems)
There are three permission "areas"  
* User: Determines what actions the 'owner' of the file can perform on a file.
* Group: Determines what actions a member of the 'group' the file belongs to can perform on a file.
* Other: Determines what actions all others (**excluding User and Group**) can perform on a file.

Each area can have its own permissions
* ex: 0644 <=> 0b101100100 => User can read/write, group can only read, other can only read.

### Looking at Permissions
The permissions can be modified easily through the terminal.
In an example, a `ls` list files command with the `-l` flag displays the `long format listing`.
In other words, it displays metadata information about the files.
* Metadata is data about the data of a file, hence the meta prefix.
* Metadata is not stored with the file, although it may be stored in adjacent memory.
* Metadata therefore doesn't add to the file size.
* Metadata is handled by the OS and contains the information given when running `ls` with the `-l` flag.

```bash
thisname@iscensored:~/Desktop/Stuy/Systems/removechar$ ls -l
total 16
-rw-rw-r-- 1 thisname thisname  986 Oct 23 14:16 main.c
-rwxrwxr-x 1 thisname thisname 8744 Oct 23 14:16 test
```
In this example, data is shown in the format `permissions hard-link-count owner owner-group filesize last-modified file/directory name`.  
* The hard-link-count of a file is typically 1 while a directory is typically > 1.

The permissions for `main.c` in this example are shown as `-rw-rw-r--`.
This means that the owner can read/write, the owner's group can read/write, but others can only read.

In the case of a directory being in the file list:
```bash
drwxrwxr-x 2 thisname thisname 4096 Oct 23 14:16 removechar
```
The initial character `d` indicates it is a directory. `rwxrwxr-x` indicates that the owner & owner group can read/write/execute the directory while others can only read or execute and not write. Note that the hard-link-count of the directory is 2 (and there are 2 files inside that directory as indicated in the code block located above).

### Modifying Permissions
To modify permissions, the terminal command `chmod` can be used.
Initial details:
```bash
-rw-rw-r-- 1 thisname thisname  986 Oct 23 14:16 main.c
```
After running `chmod 754 main.c`:
```bash
thisname@iscensored:~/Desktop/Stuy/Systems/removechar$ chmod 754 main.c
thisname@iscensored:~/Desktop/Stuy/Systems/removechar$ ls -l
total 16
-rwxr-xr-- 1 thisname thisname  986 Oct 23 14:16 main.c
-rwxrwxr-x 1 thisname thisname 8744 Oct 23 14:16 test
thisname@iscensored:~/Desktop/Stuy/Systems/removechar$
```
The permissions for main.c have been changed from 664 or `read-write-null-read-write-null-read-null-null` to 754 or `read-write-execute-read-null-execute-read-null-null`. (Note that each digit is an individual octal number).

---

## Monday, 10/23: A bit o' wisdom
**Tech News** [They're... NOT taking our jobs! (Robots and people working together at Boxed)](https://www.cnet.com/news/boxed-retailer-got-robots-automation-humans-jobs/)

### %o and %x ###
These are two integer specifiers, however they work a little differently than %d.
%d represents integers in **base 10**, while %o represents them in **base 8** and %x represents them in **base 16** (o is for Octal, x is for heXadecimal)

We can also work with these formats in our code, by prefixing our integer literals with :
- 0b for binary
- 0 for octal
- 0x for hexadecimal
Keep in mind, that no matter how a number is represented - it's still the same number. The base 10 number 11 is 1011 in binary, 13 in octal and B in hexadecimal. Therefore:
** 11 == 0b1011 == 013 == 0xB **

### Bitwise Operators!! ###
Bitwise operators are operations that get applied to the individual bits of a data type.

The first kind is bitwise shifting. What this does is move the bits over, either left or right. The operation is symbolized with either >> or << depending on the direction you're shifting.

x >> 1 - a right shift by 1.
	**ex** if a is 1 0 1 1 0 1, after the shift a is 0 1 0 1 1 0.
x << 1 - a left shift by 1.
	**ex** if a is 0 1 0 1 1 0 1, after the shift a is 1 0 1 1 0 1.

In general, a right/left bitshift moves all bits to the right or left (respectively) and adds 0s in the front or back (respectively). Any values that get "carried over the edge" are simply deleted. These operations always work, so no memory issues here!

Note that these operations are not inverses:
```
a = 0b10111;
a = a >> 1;
a = a << 1;
```
After running this code, a is now `10110`. The one at the end was lost by the left bitshift, and was replaced with a 0 by the right bitshift.

### Bit Logic! ###
Much like booleans, we have logic that works on bits (if you still haven't caught on, 1 is true and 0 is false!)
- Negation! The tilde (~) acts to negate a bit (0 to 1 or 1 to 0);
- Or! Much like the boolean or, this is done with the | symbol, but only one. a | b returns 1 when one of the bits is 1.
- And! Again, like the boolean and but only one &. a & b returns 1 when both bits are 1.
- Xor! A single carrot (^) represents the xor. a ^ b returns when only one of the two bits is 1.

Fun/Super useful fact: You can swap two variables without using a third with bit xor'ing!

```
b = a ^ b
a = a ^ b
```
---
## Thursday, 10/19: More ways to kill bugs by Jensen Li
**Tech News** [Google adds ‘try now’ button on Play Store listings to highlight Android Instant Apps](https://www.theverge.com/2017/10/19/16502198/google-android-instant-apps-play-store-try-it-now-button)

### GDB (Gnu DeBugger) ###

To use:
1. Compile the program
2. Call the gdb on the executable

**example**
```
gcc somefile.c
gdb ./a.out
```

The GDB provides an interactive shell

## Shell commands: ##
```
run
```
* runs the program until it ends/crashes
```
list
```
* lists the lines of code run around a crash
```
print <VAR>
```
* print a variable
```
backtrace
```
* show the current stack
```
break <line number>
```
* creates a breakpoint at the given line

## You can also run the program in pieces using the shell with commands like: ##
```
continue
```
* continues to run the program until the next breakpoint
```
next
```
* only runs the next line of the program
```
step
```
* runs next line of program
* will run function’s first line but that’s it

---

## Wednesday, 10/18: Getting Back on the Grind
**Tech News** [DeepMind’s Go-playing AI doesn’t need human help to beat us anymore](https://www.theverge.com/2017/10/18/16495548/deepmind-ai-go-alphago-zero-self-taught)

## valgrind
* Valgrind is a tool for debugging memory issues in C programs
* You must compile with -g in order to valgrind (and use similar tools)

**example**
```
$ gcc -g foo.c
$ valgrind --leak-check=yes ./a.out
```
This will give you a better description of the errors and memory management within your code.
Even if the error itself is indecipherable, valgrind gives you the line at which it occurs (not just SegFault!)

One final thing to mention is when running valgrind on machines in the csDojo, the memory management will only show
you the things that you malloc, calloc, realloc, and free. Macs run on a different OS, though, and show the memory used
in underlying processes that you may not have personally accessed. Some advice is to just make sure that you haven't
"definitely lost" memory in your program.

---
## Spooky Friday, 10/13: calloc & realloc by Brian Leung
###### **Tech News** [New Ion Thruster Prototype Breaks Records in Tests, Can Help in Journey to Mars](https://www.space.com/38444-mars-thruster-design-breaks-records.html)

## calloc
```c
void *calloc (size_t num, size_t size);
```
###### `calloc` dynamically allocates memory, like `malloc`. They're very similar except in 2 ways:
* All of the allocated memory is initialized as 0.
* It takes 2 arguments: num & size. It allocates a space of *num* * *size* bytes.

#### Example of use:
```c
p = (int *) calloc(5, sizeof(int));
```

## realloc
```c
void *realloc(void *ptr, size_t size)
```
###### `realloc` attempts to resize the memory block associated with a pointer.
If `size` is smaller than the original size, the extra bytes will be released.

If `size` is greater than the original size, the allocation will be increased, or a new allocation will be made to accomodate the larger block.

If you run out of memory, it returns NULL.

#### Example of use:
```c
ptr = (int *) realloc(ptr, sizeof(int)*3);
```

---
## Thursday, 10/12 malloc & free: The dynamic duo! by Victor Teoh
**Tech News:** [Google commits $1 billion in grants to train U.S. workers for high-tech jobs](https://techcrunch.com/2017/10/12/google-commits-1-billion-in-grants-to-train-u-s-workers-for-high-tech-jobs/)
#### malloc
```c
void *malloc(size_t size);
```
`malloc` allocates a requested amount of bytes of dynamic memory and returns a pointer to that memory.

```c
short* p;
p = (short *) malloc( 10 * sizeof(short));
```
* Void * as the return type is used so the function would work with any type of pointer.
* You can and should typecast the void pointer to the appropriate pointer type.

#### free
Unlike in java we must release our dynamicaly allocated memory when we are done with it.
```c
void free(void *ptr);
```
`free` takes a pointer to the beggining of dynamicaly allocated memory and releases it.

```c
short* p;
P = (short *) malloc( 10 * sizeof(short));
// code using p
free(p);
```

* Only one pointer pointing to the beginning must be used to free the memory.
* One should not work with the memory once it is freed.


## Tuesday and Wednesday, 10/10 and 10/11 If you don't pay attention, you'll get into a heap of trouble by Stanley Lin
**Tech News:** [The Quest for Production Quantum Computing Continues](https://techcrunch.com/2017/10/10/intel-moves-towards-production-quantum-computing-with-new-17-qubit-chip/)

Today, we continued examining structs and exploring how to use them.

Given this struct:
```c
struct boi {
	int x;
	char y;
} hai;
```

We can access the variables with the `.` operator:
```c
hai.x = 99;
hai.y = 'A';
```

### Quirky things to remember about structs!

* The type of the above struct is `struct boi`, not just `boi`. In fact, `boi` doesn't actually mean anything by itself, so we can do this:
```c
float boi = 5.6;
```
And it would work out fine (though it is highly unrecommended).
* Depending on your system (and potentially your OS version), the value returned when trying to get the `sizeof` a struct can vary. You may get a value that is the sum of the sizes of the variables contained in it (such as 9 if the members are an `int` and a `char`) or a potentially larger number. This is due to the padding in memory between the two members.
* You **cannot** place a variable with the same type as the struct within itself:
```c
struct boi {
	int x;
	char y;
	struct boi uh;
};
```
The compiler won't know how much memory to allocate to `uh`, because it doesn't know how much space it will take up: the definition of the struct hasn't even been complete yet!

### On `typedef`ing a struct
You may be tempted to `typedef` a struct to avoid having to type the entire type name. However, this will confuse people—it will no longer be clear if the variable they are using is a struct.
```c
struct boi yay;

typedef struct boi boi_t;
boi_t um;		//this could've been a typedef of some other primitive type, but no one would know!
```

### structs are passed by value!
If we declare a function like the one below:
```c
void lol(struct boi my_boi);
```
Every time we pass in a struct to it, the program will allocate memory every single time to make a copy of it. We can avoid this overhead by using pointers instead:
```c
void lol(struct boi *my_boi_on_point);
```

### Using struct pointers!
One way to use struct pointers is like this: `(*my_boi).x`
* Note the usage of the parentheses. The `.` operator has higher precedence than `*`, so we need it to first deference the pointer, then get the member variables from it.

This is too long for us. We like shorthands, so we can use the following:
```c
my_boi->x
```
The `->` operator isn't some special character on a keyboard; it's the hyphen (`-`) followed by an angle bracket (`>`) **without** a space in between. With it, we can access the members of a struct without having to deference the pointer with the `*` operator.

This is very nice, because now we can create cool things like linked lists. An example definition is below:
```c
struct node {
	int data;
	struct node *next;
};
```
Notice how we included `struct node *next`. This is valid because we are *pointing* to a struct, and a pointer always has a pre-defined size (dependant on your system of course).

### A brief explanation on stack memory
Soon we will be wanting to dynamically allocate memory for our variables. Right now, we have been using stack memory.

Whenever we define/intialize variables, or whenever we call a function, it is placed onto the stack, similar to the data structure of the same name. With the below program:
```c
void bar() {}

void foo() {
	bar();
}

int main() {
	foo();
}
```
A rough representation of the stack may look like this:

Stack |
----- |
`bar()`
`foo()`
`main()`
(bottom of stack)

Whenever a function returns, it is popped off the memory stack. So if `foo` finished, then the stack would look like this:

Stack |
----- |
`main()`
(bottom of stack)

### Local variables and the stack
Suppose we want to create the linked list we mentioned earlier. Let's say we create a function to add to the head of a linked list using the same definition of a node as the one above:

```c
struct node* add_to_front(struct node *head) {
	struct node new;	//(1)
	new.data = 42;
	new.next = head;

	return &new;		//(2)
}
```
There's actually a very big issue with this implementation. It's important to note that at `(1)`, we do indeed allocate memory for `new`; however, we are allocating it in the wrong place: the stack.

When we try to return the address of `new`, as we have done in `(2)`, that section of memory that was allocated for `new` gets released (or "popped off" the stack) the moment the function completes. Therefore, the program will see that section of memory as "open," and can put whatever it wants in there.

This can be worked around by requesting memory from the heap, aka dynamically allocating memory.

---

## Friday, 20/6 typedef and struct by Leo Liu
**Tech News:** [AIM Will Shut Down after 20 Years](https://www.theverge.com/2017/10/6/16435690/aim-shutting-down-after-20-years-aol-instant-messenger)

### typedef
Give a new name to an existing primative.
#### Syntax:
```c
typedef <primitive type> <name>
```
#### Example:
```c
typedef short compact
```
This will allow the declaration of variables of the type `compact`. Main purpose is for **portability**, as changing the primitive type in `typedef` will change it in all occurences of that type. Can also give extra information in the type name itself.

### struct
Create a new type that is a collection of other types.
#### Syntax:
```c
struct <type name>{
       type var1;
       type var2;
       //declare as many as necessary
} <var name>;
```
#### Example:
```c
struct foo{ int a; float b; char c};
struct foo s1;
struct foo s2;
```
- `<type name>` (optional) will save this structure so it may be used without repeated declaration of the specifications of its content.
- `<var name>` (optional) will declare a variable containing the declared collection of types.

**Note:**
`;` is present after the ending `}`

---

## Tuesday, 10/3 Make It So by Adeebur Rahman
**Tech News:** [NASA Black Marble Techology](https://www.cnbc.com/2017/10/03/nasa-uses-black-marble-tech-in-disaster-response-in-puerto-rico.html)

Multiple File Compilation
- You can combine multiple C files into a C program by including them all in one gcc command. `gcc a.c b.c c.c`
- You cannot have duplicate functions or global variable names across these files. (main counts as a function)

Compile into .o files.
- By using `gcc -c cfile.c` a file is created with the name cfile.o
- It is not a functional program, but it is compiled code.
- This is primarily used to compile c files without a main function.
- .o files can be linked with each other or with other c fiels by compiling them all at once. `gcc a.o b.c c.c`

Make
- Create compiling instructions and setup dependencies.
- Standard name used for make is makefile.

The syntax is as follows:
```make
<TARGET>: <DEPENDENCIES>
	<RULES>
```

Example makefile:
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

- Running a makefile is as simple as typing `make`
  - This will automatically make the first target. Which will recursively check if any dependencies need to be updated and then update them.
  - You can also include a target `make <TARGET>` and this will make the target specified recursively.
- The first target is typically a file that doesn't exist such that it is always updated.
- We also usually include a clean target to remove junk files and the like.
- Similarly we include a run target, that runs the final product.

Thus, using make simplifies the processing of compiling and running code.

---

## Friday, 9/29 | string.h functions | By Naotaka Kinoshita

**Interesting Tech News:** [Tattoos to Monitor Your Health](https://news.harvard.edu/gazette/story/2017/09/harvard-researchers-help-develop-smart-tattoos/)

We spent the class introducing the various string.h functions that we were assigned for homework.

#### `strcpy` and `strncpy`
##### `strcpy( char * dest, char * src );`
##### `strncpy( char * dest, char * src, int n);`

These two functions are made for copying strings. They both work by copying the `src` into `dest`. However, `strncpy` only copies the first `n` characters of `src`. Note that these functions change `dest`, so `dest` cannot be immutable, but `src` can.

```C
#include <string.h>
int main() {

  char str1[] = "hello";
  char str2[20];
  char str3[2];

  printf("%s\n", * strcpy( str2, str1 )); // will result in "hello" being printed
```
Note that for the first usage of `strcpy`, the first argument, "hello", is smaller in size than the second argument, which will allow for the copying to take place.

```C
  printf("%s\n", * strcpy( str3, str1 )); // will result in bad things (it could possibly work, but can also result in an Abort trap)
```
Don't copy strings that are larger than the destination string.

`strncpy` is like strcpy, but safer. You should probably use `strncpy` anytime you want to copy strings, regardless of how much of the string you want to copy over.

```C
  printf("%s\n", * strncpy( str3, str1, 1 )); // will result in "h" being printed
  printf("%s\n", * strncpy( str3, str1, 2 )); // terminating null won't be copied over - don't do this
```
Make sure that the terminating `0` will be copied over into the destination string.

```C
  printf("%s\n", strncpy( str2, str1, sizeof( str2 ) )); // will result in "hello" being printed
  printf("%s\n", strncpy( str3, str1, sizeof( str3 ) )); // will result in "he" being printed
```
By setting the maximum amount of chars equal to the size of `dest`, you can ensure that you won't end up with an error or unintended behavior as seen above with `strcpy`.

#### `strcat` and `strncat`
##### `strcat( char * dest, char * src );`
##### `strncat( char * dest, char * src, int n);`
These two functions are used to concatenate the source string to the destination string. As with `strcpy` and `strncpy`, `strncat` takes a number of chars in the source string that you want to concatenate over.

```C
#include <string.h>
int main() {

  char str1[16] = "h";
  char * str2 = "ello";

  printf("%s\n",  * strcat( str1, str2 )); // will result in "hello" being printed
```
Note that the size of `str1` has to be large enough for the size of `str1` + the size of `str2`. Like `strcpy` and `strncpy`, the destination string can't be immutable, but the source can.

```C
  printf("%s\n",  * strncat( str1, str2, 3 )); // will result in "hell" being printed
```
`strncat` will take the number of chars specificed, in this case 3, and concatenate to the destination string.

#### `strcmp` and `strncmp`
##### `strcmp( char * s1, char * s2 );`
##### `strncmp( char * s1, char * s2, int n ); `
These two functions compare the two strings. If `s1` is less than `s2`, a negative integer is returned, if `s1` is equal to `s2`, zero is returned, and if `s1` is greater than `s2`, a positive integer is returned. Note that the return value is not -1, 0, or 1, unlike other languages. `strncmp` will compare only the first `n` chars of the string, while `strcmp` will compare the entire string.

```C
#include <string.h>
int main() {

  printf("%d\n", strcmp("A", "a")); // will print -1
  printf("%d\n", strcmp("A", "A")); // will print 0
  printf("%d\n", strcmp("a", "A")); // will print 1

  printf("%d\n", strncmp("aA", "Aa", 1)); // compares "a" with "A" -- prints 1

  return 0;
}
```

#### `strchr` and `strstr`
##### `strchr( char * s1, char c );`
##### `strstr( char * s1, char * s);`
`strchr` returns a pointer to the first instance of the specified char `c`, while `strstr` returns a pointer to the index of the first char in the first occurence of the substring `s` in `s1`. Note that looking for a substring or char that doesn't occur in the string will result in a null pointer being returned.

```C
#include <string.h>
int main() {
  printf("%p\n", strchr("hello", 'e')); // will return pointer to 'e'
  printf("%c\n", * strchr("hello", 'e')); // will print 'e'
  printf("%p\n", strchr("hello", 'a')); // will return null pointer

  printf("%p\n", strstr("hello", "el")); // will return pointer to 'e'
  printf("%p\n", strstr("hello", "abc")); // will return null pointer

  return 0;
}
```

---

## Thursday, 9/28 | A string of functions | by Kristin Lin

**Interesting Tech News:** [Tech and Photography?](https://www.nytimes.com/2017/09/27/technology/personaltech/how-technology-has-changed-news-photography-over-40-years.html)

*DO NOW: Write a function to find length of strings.*

One possible way:
```C
int str_length (char *point) { //note, NOT char s[]
  int len = 0;
  while (*point) { //because 0 is FALSE in C
    count++;
    point++;
  }
  return len;
}
```
Another possible way using POINTERS:
```C
// gist: create a char pointer, increment until null, each char is 1 byte, so subtract
int str_length (char *point) {
  char *i;
  for (i = point; *i; i++); //note, empy FOR loops can be ended with a semicolon
  return i - point;
}
```
*INTERESTING NOTE: [What is "obfuscated code"?](http://www.ioccc.org/)*

#### Declaring Strings (CONT.) ####
* `char *s3 = "mankind"`          
creates immutable string "mankind" and returns a pointer to that string. You cannot modify strings created this way because the memory the pointer points to is immutable.

* `char s[] = "hello"; //ok!`             
`s[] = "seven"; //NOT ok`             
array pointers are immutable

* `char * s = "hello"; //ok!`                  
`s = "seven"; //also ok!`            
regular pointers are NOT immutable


###### THE TWO ESSENTIAL QUESTIONS: ######
1. *"Is my pointer immutable?"*
2. *"Is the data my pointer is pointing to immutable?"*

For homework, we split into groups of four, with the numbers 0, 1, 2, 3 being assigned by our seating arrangement.

---

## Wednesday, 9/27 | ctrings | by Khinshan "Shan" Khan [ :octocat: ](https://github.com/)

**Interesting Tech News:** [MIT CSAIL's origami-bot wears foldable exosuits](https://www.engadget.com/2017/09/27/mit-csail-transforming-robot-origami-exoskeleton/)

We started off with a do now involving us to see pointers and how they were related:

```C
#include <stdio.h>
#include <stdlib.h>

int main(){
  char *s = "hello";
  char s1[] = "hello";
  printf("s points to: %p\n", s);
  printf("s1 points to %p\n", s1);
  printf("address of \"hello\": %p\n", &"hello");

  return 0;
}
```

#### Strings
* Strings are character arrays
  * formatting character would be `%s`
* By convention, strings end with a null character, either:
  * `''`
  * `0`
  * `'\0'` --- This is because 0 has an ASCII value, so by placing an escape sequence, we place the character with code 0
* When you make a literal string using `""`, an immutable character array (string) is created in memory. The string will be automatically terminated (the last element is set to a null character)
* All references to same literal string refer to the same immutable string in memory
#### Declaring Strings

```C
// normal array declaration
// allocates 256 bytes
char s[256];

// allocates 256 bytes
// will create the immutable string "imagine", and then copies it into a character array along with null character
char s[256]= "imagine";

// allocates 8 bytes
// will create the immutable string "Tuesday", and then copies it into a character array along with null character
char s[]= "Tuesday";
```

Endowed with this knowledge, we proceeded to see the effects of self entered null characters:
<br>(following code was just added into main)

```C
char s2[4];

s2[0]= 'h';
s2[1] = 'i';
s2[2] = '!';
printf("s2: %s\n", s2);

s2[0]= 'h';
s2[1] = '0';
s2[2] = '!';
printf("s2: %s\n", s2);
```

Will be continued tomorrow.


---

## Tuesday, 9/26 | Headers| by Bayan Berri

**Interesting Tech News:** [Excel's getting smarter](https://techcrunch.com/2017/09/26/microsoft-excel-is-about-to-get-a-lot-smarter/)

###### Aim: Let's head to function town.

#### Continuing from Yesterday:
* We started class by continuing from the foo function that we spoke about at the end of Monday's class.
* We mentioned that foo works on an array as well.
We can do:

```C
void foo (float *x)
or
void foo(float *x[])
```

Object variables in Java are similar to pointers in c.
#### Void Pointers ####
* Used when a pointer is needed but you don't know or care about the type.
* Moves by __1 byte__
* Points to a chunk of memory.
* (More on this in the future)

As we already know- in C the order that identifiers are declared matters. Including functions, so they must be declared before use. There are a few ways to do this.

#### Declarations Possibilities ####
* Functions are declared before main
* You can declare a function header at the top and write a function header and body later on in the code (or not, you can declare it and not define it). If you decide to do this it solves the issue of function dependency in a weird circulation. You also won't need to include names for the variables (Only the type)
```C
void foo (int, int);
```
* The last possibility is to put all your headers in a file my_headers.h for example. At the top of your c program make sure to include my_headers.h (.h can be anything but .h is convention.) However it isn't included using angle brackets the usual way.


```C
#include "my_headers.h"
```
---
## Monday, 9/25 | Functioning Code | Piotr Milewski

**Interesting Tech News:**  [University of Tokyo pair invent loop-based quantum computing technique](https://www.japantimes.co.jp/news/2017/09/24/national/science-health/university-tokyo-pair-invent-loop-based-quantum-computing-technique/#.Wcmv-RdrxhF)

At the start of class we were told to write a c program with a simple function and to call the function in main(). Once we finished writing our basic programs we went over different ways of completing Homework 03. The remainder of the class was spent going over arrays, pointers, and certain unorthodox methods of utilizing arrays and pointers that are looked down upon by Mr. DW and the coding world. At the end of class we briefly discussed writing functions.

### Arrays and Pointers
* Array variables are immutable pointers.
* Pointers can be assigned to array variables.
	```C
	int ray[5]; //initialize array of size 5
	int *rp = ray; //set pointer to array
	ray[3]; <==> *(ray + 3); //access index 3 of array
	rp[3]; <==> *(rp + 3); //access index 3 of array (version 2)
	3[rp]; <==> *(3 + rp); //access index 3 of array (version 3)
	```
* From this code snippet we find that `ray[3]` simply translates to `*(ray + 3)` which means that whenever we try to index an array, all we are doing is dereferencing the address of `ray + 3` or in other words, accessing the memory at `ray + 3`. This let's us get away with calls such as `3[ray]` or `3[rp]`, even though they don't make logical sense (so just stick with the regular format of `array[index]`.
* This is also relevant to Homework 03 because rather than using `*pointer  = &array[i]` to set the address of the pointer, we can use `*pointer = array + i`.
	* Since we learned that a pointer of type `int` are treated as offsets of their own type and can be arithmetically manipulated, we can get away with calls such as `*pointer++` to traverse an array in a loop if `*pointer` points to an array.

### Writing Functions
* Function headers
	* `<return type> <name>(<parameters>)`
* All functions are pass by value
	* Function arguments are new variables that contain a copy of the data passed to them.
	* However, when we pass a pointer as an argument instead of a variable, then the address of the variable is passed instead of the value.
		* Any change made by the function using the pointer is permanently made at the address of the passed variable.
		```C
		void foo(int *a){ //pointer *a is an argument
			*a += 7; //add 7 to the value of pointer *a
		}
		int main() {
			int b = 7; //initialize int b and set it to 7
			foo(&b); //pass the address of b to foo()
			return 0;
		}
		```
		* After running this program, the values of `*a` and `b` will both be 14.

---

## Wednesday, 9/20 | Pointers | Khyber Sen

**Interesting Tech News:**  [Java 9 has just been released last Thursday, bringing a long-awaited module system](https://jaxenter.com/top-9-improvements-features-java-9-136081.html)

### Pointers
* Normally variables store values,
but pointers are special variables that store the memory address.
Java and Python don't explictly use pointers anywhere,
but in both languages, pointers are still used behind the scenes
for all non-primitives.  But using and manipulating pointers directly
is completely new.
* A `*` is used to declare a pointer:
    ```C
    int *pointer_to_int;
    char *pointer_to_char;
    ```
* Pointers are just an address, so they can be manipulated just like any other unsigned integer.
* `&` is the address-of operator.  It returns the address of a variable:
    ```C
    int i;
    int *pointer_to_i = &i;

    int a[10];
    int *pointer_to_beginning_of_a = &a[0];
    pointer_to_beginning_of_a += 5; // now points to 5th int in `a`
    ```
* Pointers are strongly-typed too, so `int *` is not the same as `char *`.
This means that when pointers are arithmetically manipulated,
they are treated as offsets into an array of their type.
Example:
    ```C
    char s[100];
    void *p = s;
    int *ip = p;
    assert(p + 10 * (sizeof(int) / sizeof(char)) == ip + 10)
    ```
    When you do `ip++`, the address is increased by sizeof(int).
* `*` is also the de-reference operator,
allowing you to access (read or write) the memory at that address:
    ```C
    int i = 0;
    int *ip = &i;
    *ip = 10;
    // i is now 10
    ```
* Arrays and pointers are slightly different,
but an array variable can also be assigned to a pointer
and treated identically.
* Pointers can be indexed like arrays, but it is just syntactic sugar.
`a[i]` is the same as `*(a + i)`.
* The actual address doesn't really matter and
will change every time you run the program.  
Only offsets from that address into allocated memory matter.
You could potentially assign any unsigned integer value to a pointer,
but just like accessing arrays with out of bound indices,
the results are undefined and will probably create a segfault.
Pointers should always be kept in allocated memory.
* Pointers can point to anything:  
    * normal types like `int` and `char`: `int *` or `char 8`
    * anything: `void *`
    * other pointers: `int **` (pointer to a pointer to an int)
    * even functions: `int (*f)(char)`
        (pointer named f to a function accepting a char and return an int)

I found this website really useful in understanding complex pointers:
https://cdecl.org/

---

## Tuesday, 9/19 | What's the point of it all? | Grace Cuenca

**Interesting Tech News:**  [Twitter Cracking Down on Accounts Promoting Terrorism](http://money.cnn.com/2017/09/19/technology/business/twitter-transparency-report/index.html)

## Memory Management
* Memory allocation either happens at compiler time or at run time (dynamic)
* Compiler allocated memory:
  * Packaged with the binary of the program
  * There is no standard default value (unlike Java). This is because C doesn't "zero out" anything. This saves processing time.
  * Variables and arrays are allocated here

Example:
```C
float a
int b[5] //when you do this, you ask the operating system to give you an array of 5 integers.
//The memory is allocated when it's compiled, not when it's run
```
## Arrays
* Must have a fixed size set at declaration
 * However, there is memory before and after the allocation
 * There is no error that appears if you go beyond the array size (either before or after)
 ```C
 b[6]
 b[-2]
 //both will not cause errors
```
 * **NOTE:** In accessing memory beyond the array, you may run into this error:
 ```C
segmentation fault (core dump) //the function is trying to access memory that it's not allowed to
```
* When you ask for an array in C, you're asking the operating system for a contiguous block of memory of a certain size
* There is no length function
* There is no boundary checking

---

## Monday, 9/18 | Data Types and Variables | Sabrina Wen

**Interesting Tech News:**  [Computers and Human Brains!](https://nyti.ms/2y5xcLS)

We spent the first half of class talking about how to submit homework assignments by using submodules (basically links to separate repositorys), and the second half going over data types and variables in C.

### How to use submodules to submit homework assignments:

1. First, make a *separate repository* for every homework assignment.
2. When you commit your repository (or any other repository for that manner), it's recommended that you use the flags `-a -m`. For example:
```
git commit -a -m "This is a commit msg!"
```
3. Clone the master repository (aka the class assignment respository) and make sure to use the HTTPS link.
4. Use the following to add your submodule to the master repository:
```
git submodule add *insert HTTPS link wow!*
```
5. Commit and push your file as your normally would. You can also remove the master repo from your computer if you want to.

### Data Types and Variables
* Character literals are single characters inside `''`. Ex: `'a'`, `'b'`
* String literals exist, even though there is no string data type. Ex: `"hello"`, `"hi there"`
* Note that `%s` is the placeholder for a string literal.

* The order that identifiers are declared is important. This includes functions, which must also be declared before they are used.
* Variables can't be declared inside for loop statements, but they can be initializeD. Ex:
```C
int a;
for (a = 0; a < 10; a++) {
    printf("hi\n");
}
```
* Any variable type can be declared as an "unsigned" variable. This means the variable will never be negative.
	1. The lower bound of any unsigned variable is equal to 0.
	2. The upper bound of any unsigned variable is greater than the unsigned version.
Ex: `unsigned char x;` where 0 <= x <= 255
* All boolean values are numbers.
	1. 0 --> false
	2. anything else --> true

**NOTE:** Since boolean values are represented by numbers, it's easy to trigger an infinite loop. For example:
```C
if (x = 6) {
    do stuff
}
```
Because x is assigned to the variable 6, the boolean value in the if statement will always be true.


---

## Thursday, 9/14 | Always read the fine print | Ahbab Ashraf

**Interesting Tech News:** [Iphone X? Never heard of it](https://www.theverge.com/2017/9/14/16306472/google-pixel-2-launch-date-october)

We spent most of the class discussing primitives in C, which largely are the same as in Java. However, both booleans and bytes are excluded in C. Below is a table of the 6 types of primitives, with other important info:

| Primitive | Size(bytes) | Range |
|:---:|:---:|:---:|
| char | 1 | -128 to 127 |
| int | 4 | ~-2 billion to ~2 billion |
| short | 2 |  |
| long | 8 |  |
| float | 4 | 7 digit precision |
| double | 8 | 14 digit precision |

#### More about primitives
* All primitives are numeric
* They differ in both their sizes and their type of numerical information (floating point vs integer)
* If you want to find a primitive's size on the fly, you can use ``sizeof(TYPE)`` (from stdlib.in)

We also talked about `printf` and formatting placeholders. This is a way of printing variables more easily than in Java. Here's an example:

```C
int i = 5;
printf("i is %d", i);
```

In this case, `%d` is the placeholder variable for the integer `i`. For every placeholder, there must be a corresponding value in the function call. Below are a number of placeholders and their corresponding data types:

| Type | Placeholder |
|:---:|:---:|
| int | %d |
| long | %ld |
| float | %f |
| double | %lf |
| char | %c |
| char array | %s |
| pointer | %p |

**NOTE:** For doubles, `%0.<x>` can be used as a placeholder as well, printing with x digits after the floating point.

---

## Wednesday 09/13 Variables are the spice of life by Jake Zaia

**Interesting Tech News:**  [Solar panels are getting cheaper faster than expected!](https://arstechnica.com/science/2017/09/solar-now-costs-6-per-kilowatt-hour-beating-government-goal-by-3-years/)

We spent the day discussing libraries in C, and how to include them in our programs, as well as how to utilize the `man` command to its full potential.

### Things to know about `man`
* `man` is short for "manual", the whole manual of how to use Linux (Or OS X) comes with the computer and is *in* the computer
* `man` also has pages for C, detailing libraries and commands
* The manual is split into 9 sections, and to locate each one, you can read the man page for man (`$ man man`)
* To view a specific section of the man pages, you can pass it the section as a parameter. ex. `$ man 3 printf` will check the 3rd section of the man pages for the page about "printf"

### Things to know about including libraries in C
* Upon compiling C does 2 things to use library functions
	1. Checks the function for syntactic use to make sure it is legal
	2. Links the code for the external function (and ONLY that function) to your executable
* To include a library in C, you write `#include <libraryname>` including the angle brackets/gang signs

---

## Tuesday, 09/12 Hello, world by Jeffrey Weng

**Interesting Tech News:** [Iphone X hype?](https://www.theverge.com/2017/9/12/16291244/new-iphone-x-photos-video-hands-on)

We were told to copy a simple Hello World program in C and compile it:

```C
int main(){
  printf("Hello Everybody!\n");
  return 0;
}
```

We also made some distinctions between C and Java:
* No public/private access modifiers
* No classes
* Not `System.out.println()` but `printf()` *(Note: `printf()` doesn't include a new line)*
* `main()` instead of `public static void main(String[] args)`
* We also touched on the idea of returning 0 instead of using a void function because it shows that the function has ran correctly


**IMPORTANT**: C is a imperative/procedural programming language while Java is a OOP language


*Calling our compiled file requires us to use `./<filename>` when we're in the same directory as the file instead of calling just the `<filename>`*
#### Why?

For example if we were to change our filename to a common command such as `ls`, which displays the files in your current directory, how do we determine the difference?

While calling `ls` we access an environmental variable called **PATH**, which tells the shell which directory to look for the ls program. Note this doesn't call our *ls program* but the
compiled C program in the system.

To call our *ls program* we would specify which directory it is in, to override our **PATH** environmental variable.

Doing this provides us extra security as it stops regular users from manipulating common commands such as `ls` and helps us differentiate similarly named programs.

#### Useful Commands

`which [-option]` - Searches the path of executable in system paths set in $PATH <br />
`echo` - you can use this to either *echo* a string you input or if you provide an environmental variable, show the values of the environmental variable

Examples of `echo`
1. `echo Hi`
Terminal spits out: Hi
2. `echo echo`
Terminal spits out: echo
3. `echo $PATH`
Terminal spits out: a series of colon-separated paths

OPTIONAL TEXTBOOK: The C Programming Language by Brian Kernighan and Dennis Ritchie


---

## Monday, 09/11 Venture into the C by Jen Yu

**Interesting Tech News:** ['The New Yorker' on Hyperloops](https://www.newyorker.com/tech/elements/hypnotized-by-elon-musks-hyperloop)

We learned the basics of C programming by creating a file and compiling it. Here are the steps to creating your very own executable C file:
1. Write source code in a text editor and save with the `.c` extension. <br>
	* **Note**: The naming convention for C is **snake_case**, _where all words are lowercase and distinct words are separated by underscores._

2. Open the command line and type in `gcc <filename>`. <br>
	* **gcc** stands for "GNU C Compiler", and compiles the file to the architecture specific (OS + Processor) executable file.

3. ``ls`` to list the files in your current directory.
   * You will see an ``a.out`` file, which is the default name of a compiled file in the C programming language.
4. Run the executable file: `./a.out`

OPTIONAL BUT **HIGHLY** RECOMMENDED <br>
Rename your `a.out` file! Despite it being the default name, it's by no means descriptive and will be overwritten if another C file is compiled within the same directory. <br><br>
To do so: <br>
``gcc -o <new_filename> <filename>``<br>
\* `new_filename` does not include an extension.
<br><br>
Example: <br>
If I did: `gcc -o hi hello.c`<br>
I rename the compiled hello.c to `hi`.

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
