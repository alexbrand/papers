# Page fault handling

Page fault handling is a key operation during the execution of a program. Depending on the system architecture where the program
is running, the procedure for handling the page fault differs. This is a summary of my understanding of the
different approaches to page fault handling.

## SPIN operating system
In the SPIN operating system, the core memory management service raises a page fault event. The dispatcher 
will send the event to the handler that has registered the page fault event. The extension that implements
this handler is then responsible for handling the page fault.

## Exokernel operating system
* The exokernel fields the page fault. It knows which library operating system should handle the page fault 
from the CPU time slice vector. 
* The exokernel looks in the Processor Environment for the page fault handler entry point.
* Library OS uses an existing page frame, or asks the Exokernel for a new page frame.
* Library OS calls exokernel through API to install the translation in the TLB. It presents the capability 
to the exokernel. 
* Exokernel validates the capability, and installs the mapping into the TLB.

## Virtualization

### Para-virtualized
* Guest OS has already registered each page table with Xen.
* During page fault, hypervisor catches the event and invokes the corresponding handler.
* Hypervisor detects the address that generated the fault, and based on this knows which guest OS needs to 
field the fault.
* The faulting memory address is copied into hypervisor-guest OS shared data structure.
* Hypervisor performs up-call to guest OS.
* Guest OS page fault handler allocates physical page frame from it's pool of memory.
* If necessary, the contents of the page are paged in from disk, which result in I/O interactions between guest OS and hypervisor.
* Once page fault is serviced, the guest OS will issue hypercall to update the page table.

### Fully-virtualized
* Hypevisor catches the page fault.
* Hypervisor passes the virtual address to the guest operating system.
* Guest performs the mapping between virtual address and physical address.
* Attempt to update the page table results in a trap, which is handled by the hypervisor.
* The hypervisor updates the shadow page table to point VPN -> MPN.
* Hypervisor installs translation into TLB and Hardware page table.

## Shared memory multiprocessor architecture
### Tornado OS
* Page fault is an exception that is delivered to the Process object.
* The process objects find which Region object needs to handle the fault, and delegates to it.
* The Region Object calls the File Cache Manager that is responsible for the file that is backing that region.
* The FCM checks if the page content is cached in memory. If it is, it returns the address corresponding to the physical 
page frame.
* The Region object calls the Hardware Address Translation object to install the mapping, and returns.
* If the FCM finds that the content is not cached in memory, it calls out to the DRAM object to allocate a page frame
in memory.
* Once it has the page frame, the FCM calls out to the Cached Object Representation object to fill the new page with
the contents of the file.
* The COR performs the I/O to copy the file contents into the page frame. Once the copy is done, it calls the region with
the address of the page frame.
* The Region object calls the Hardware Address Translation object to install the mapping, and the thread can resume.
