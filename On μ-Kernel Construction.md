# On μ-Kernel Construction
http://www.cs.fsu.edu/~awang/courses/cop5611_s2004/microkernel.pdf

Jochen Liedtke 

## Summary
* It is believed that micro-kernel based systems are inefficient and not sufficiently flexible
* Liedtke shows and supports by documentary evidence that inefficiency and inflexibility is due to improper implementation,
and not due to the basic idea being unsound. 

## Rationale
* Kernel: part of the operating system that is common to all other software.
* Micro-kernel: minimize the kernel (i.e. to implement outside the kernel whatever possible)

## Micro-kernel primitives
### Address spaces
Micro-kernel hides the hardware concept of address spaces. Microkernel provides three operations for
building address spaces on top of foundational address space. 
* Grant: allows the owner of an address space to grant any of its pages to another address space.
* Map: allows an owner of an address space to map any of its pages into another address space. 
* Flush: allows owner of address space to remove pages from other address spaces that received the page. 

### Threads and IPC
* Thread: activity executing inside an address space. 
* IPC (aka. cross-address-space communication) is handled by micro-kernel.
* Interrupts are modeled as IPC messages. The hardware is regarded as a set of threads with special thread ids and empty messages. 

### Unique identifiers
* Micro-kernel must provide unique IDs for something: either threads, tasks or communication channels.

## Performance, facts and rumors
### Kernel-user switch
* Bare machine-instructions required for entering kernel mode add up to 107 cycles.
* Kernels measured require ~ 900 cycles to enter kernel mode => ~800 cycle kernel overhead.
* L3 micro-kernel cost for kernel-user switch is 123 cycles => Very close to the minimum number of cycles required.

### Address space switch
* The main cost of address space switch is cost of flushing TLB
* Not a problem with architectures that have address-space tagged TLBs, because flushing TLB is not necessary.
* Exploit hardware features for avoding TLB flushes, such as segment registers in PowerPC and x86.
* Protection domains can be implemented using segment registers instead of actual address space switches. For this reason, there is no need to flush the TLB.
* Expensive context-switching in some existing micro-kernels is due to bad implementation, and not inherent problems with concept of micro-kernel.

### Thread switches and IPC
* Measured various OSs and showed that micro-kernels are at least 2 times faster.
* Proved by construction that a 10 micro-second RPC call is achievable.

## Memory Effects
* Properly constructed micro-kernel automatically avoid memory system degradation because working set of micro-kernel is small.
