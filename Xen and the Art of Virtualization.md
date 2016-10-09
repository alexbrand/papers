# Xen and the Art of Virtualization
http://pages.cs.wisc.edu/~remzi/Classes/838/Spring2013/Papers/xen.pdf

Barham, Dragovic, Fraser, et al. 

## Summary
Presents Xen, a virtual machine monitor (VMM) that allows multiple OS to share conventional
hardware without sacrificing performance or functionality.

## Challenges
* VMs must be isolated from one another
* Necessary to support a variety of operating systems
* Performance overhead introduced by virtualization should be small

## Approach and overview
### Arguments against full virtualization
* x86 was not designed for virtualization
* Certain instructions must be handled by the VMM, and some of these may fail silently
* ESX Server gets around this by dynamically rewriting portions of the hosted OS, inserting
traps wherever necessary
* ESX also implements shadow versions of OS data structures (e.g. page tables), which results in 
high cost for update-intensive operations

### Para-virtualization
* Xen is based on a virtual machine abstraction that is similar, but not identical to underlying hardware
* Guest OS requires code modifications
* ABI is not modified => Guest applications are unaffected

## Virtual Machine Interface
### Memory management
* Guest OSs are responsible for allocating and managing the hardware page table
* Xen exists in a 64MB section at the top of every address space => avoids TLB flush when entering/leaving hypervisor
* Guest OS allocates and initializes a page when it needs a new page table. The guest OS registers the page with Xen.
* Subsequent updates to the page table are validated by Xen. This ensures OS can only update pages that they own.
* Guest OS can batch requests to amortize overhead of entering hypervisor. 

### CPU 
* Guest OSs must be modified to run at a lower privileged level, given that they are no longer the most privileged
entity in the system
* x86 supports ring 0-4. Any operating system can be ported to Xen by modifying it to run on ring 1
* In other architectures, the guest OS runs in the same privilege level as the applications. They are protected
becuase they run in a separate address space
* Privileged instructions are para-virtualized, meaning that they are validated and executed within Xen
* Exceptions are virtualized by allowing the guest OS to register handlers with Xen 
* System call performance is improved by allowing guest OS to register "fast" exception handler, which is
executed directly by the processor without going to Xen

### Device I/O
* Devices are not emulated. Instead, Xen uses shared-memory, asynchronous ring buffers to transfer data
* Xen uses lightweight event delivery mechanism to inform guest OS of I/O
* Guest OS may choose to defer these events to prevent costs from constant wake-up notifications

## Control Transfer
* Hypercall allows domain to perform privileged opeartions by entering into the hypervisor. An example is the update of 
a page table, in which Xen validates and applies the update, and then returns control to the domain.
* Event communication is performed via asynchronous event notifications. 

## Data transfer
* Ring buffer data structure is used to transfer data between Xen and domain
* I/O ring is allocated by domain, but accessible from Xen
* Ring contains descriptors. The descriptors don't contain I/O data, but contain pointers to the actual data buffers
* Two pairs of pointers are used for accessing the ring: request producer/consumer pointers
and response producer/consumer pointers
* Descriptors have a unique ID, which allows Xen to re-order requests as it sees fit
* Production of requests or responses is decoupled from notification to other party

# Subsystem virtualization
## CPU
* Borrowed-virtual time CPU Scheduling algorithm
* Chosen because it is work-conserving and it has low-latency wake-up 
* Per-domain scheduling parameters are adjusted in Domain0

## Time and timers
* Real time expressed in nanoseconds since machine boot
* Domain's virtual time only advances when domain is running
* Wall-clock time is offset added to real time

## Virtual address translation
* Xen is only involved during page table updates when hypercall is invoked
* Avoid overhead and complexity of using shadow page tables
* Guest OS page tables are directly registered with the MMU 

## Physical Memory
* Initial memory allocation for each domain is specified at creation time
* Ballon driver used to pass pages between guest OS and Xen
* Guest OS is responsible for mapping of physical to machine memory
