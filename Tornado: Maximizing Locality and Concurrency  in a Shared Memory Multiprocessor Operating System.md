# Tornado: Maximizing Locality and Concurrency in a Shared Memory Multiprocessor Operating System

http://static.usenix.org/events/osdi99/full_papers/gamsa/gamsa.pdf

Gamsa, Krieger, Appavoo, Stumm

# Summary
* Tornado is described as a brand-new operating system designed specifically for shared memory multiprocessors
* Previous shared memory multiprocessor operating systems do not optimize for locality. They were designed
at a time where sharing costs were low, latency was low, and caches were small. This is no longer true in today's
shared memory multiprocessors
* Tornado has an object-oriented structure. Virtual and physical resources are represented by independent objects
* Tornado achieves locality by employing three techniques: i) Clustered objects, ii) Innovative protected procedure calls, and
iii) New locking strategy
* "Tornado is designed to service all OS requests on the same processor they are issued on"

# Object oriented structure
* Each resource in the operating system is represented by a different object
* Good performance is achieved because each resource requests is fielded by a different object

# Memory management in Tornado
* Page-fault results in exception that is delivered to Process object
* Process object keeps list of memory regions that are in use by process
* Process object uses list to find Region object that should be invoked
* Region translates the fault address into a file offset, and forwards request to File Cache Manager that handles the file 
backing the region of the address space
* The FCM checks if file data is in cache. If it is, the address of the corresponding physical page frame is returned 
to the Region
* The region makes a call to the Hardware Address Translation (HAT) object to map the page, and returns
* If the page frame is not in memory, the FCM requests a page frame from the DRAM object
* The FCM then forwards the request to the Cached Object Representative object to fill the new page frame
* The COR calls the file server to read the file. The thread is restarted when the read is completed

# Clustered Objects
* Clustered object presents the illusion of a single object, but in reallity is composed of multiple underlying components
* These components are called representatives, or reps
* Degree of clustering: One per system, one per CPU, one per cluster of neighbouring CPUs
* Process object is replicated to each processor that has threads running for that process
* Region: Partial replication, one rep per cluster of CPUs
* FCM: Partitioned object
* COR has a single rep throughout the system
* The multiple reps are kept consistent via either shared memory or Tornado's remote PPC (Protected procedure call)

### Clustered object implementation
* Per-processor translation table that translates clustered object reference into the actual location of the local rep
* Each clustered object must define a miss-handler. This is invoked when there is no mapping installed in the translation table
* First invocation of clustered object reference results in invoaction of global miss-handler

# Synchronization
## Locking
* Locks live inside the object, which reduce the scope of locks and contention
* Due to use of clustered objects, the contention for locks is reduced even more. The degree of contention
is defined by the number of processors sharing the same rep for a given clustered object

## Existence guarantees
* Existence guarantees are not implemented with locks. Instead, semi-automatic garbage collection is used
* Tornado uses reference counting for garbage collection

# IPC
* Objects in the OS need to communicate with one another
* Concurrency and locality in these communications is key
* Call from client object to server object is performed via PPC (protected procedure call)
* Reasons: i) Client requests are served on the local processor, ii) Client and server share the processor
* Server threads are created on demand, and they are cached for future calls

