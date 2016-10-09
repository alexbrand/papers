# Extensibility, Safety and Performance in the SPIN Operating System

http://cseweb.ucsd.edu/~savage/papers/Sosp95.pdf

Bershad, Savage, Pardyak, et al. 

## Summary
SPIN is an extensible operating system that can customized dynamically to meet the performance and functional requirements
of applications. The main goal was to build a general purpose OS that provides safety, extensibility and good performance.

## Techniques
* Co-location: Kernel and extensions are co-located in the kernel's virtual address space. This allows performant 
communication between kernel and extensions.
* Enforced modularity: SPIN and extensions are written in Modula-3, a strongly typed programming language. The compiler enforces 
interface boundaries between the kernel and the extensions.
* Logical protection domains: Extensions live inside logical protection domains. These domains contain the code 
and exported interfaces. The extensions are dynamically linked in the kernel, resulting in performant 
cross-domain communication.
* Dynamic call binding: Extensions can subscribe to events by declaring them in their interfaces. Events are dispatched
with the overhead of a procedure call. 

## System overview
* SPIN consists of extension services and a set of core services.
* Extensions can be loaded dynamically at runtime.

## Motivation
* General purpose OSs runs many programs, but few well.
* Specialized OSs run programs well, but a small number of them.
* Propose new structure for an extensible system that can be modified dynamically to meet the needs
of a specific application. 

## Architecture
* Capabilities: All resources are referenced by capabilites. Some examples include page frame, page allocation module, etc. 
Capabilities are implemented using pointers (which are supported by Modula-3). Pointers cannot be forged due to compile-time
checks. 
* Protection domain: Defines accessible names available to an execution context. Domains are created using the _Create_ operation,
which initializes the domain with safe object file. The _Resolve_ operation dynamically links a source and target domain. 
The _Combine_ operation can be used to create an aggregate logical protection domain, resulting in the union of the interfaces
of each domain.
* Extension model: Extensions are defined in terms of events and handlers. Extensions installs event handlers via central
dispatcher. Extensions may register handlers for events that are defined in other modules. When this happens, the kernel 
checks with the primary module. The primary module may accept or deny the request. When allowing, the module may install 
a guard to be associated with the new handler. 

## Core Services
### Memory management
* SPIN decomposes memory management into physical storage, naming, and translation.
* These correspond to physical addresses, virtual addresses, and translation.

### CPU Scheduling
* Global Scheduler: decides how much time is allocated to each extension
* Strand is the unit of scheduling. It is similar to a thread, except that it has no requisite state other than a name.
* Applications can implement their own scheduler and thread package.
