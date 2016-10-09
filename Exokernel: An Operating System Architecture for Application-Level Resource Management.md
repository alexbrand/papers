# Exokernel: An Operating System Architecture for Application-Level Resource Management
https://pdos.csail.mit.edu/6.828/2008/readings/engler95exokernel.pdf

Engler, Kaashoek, O'Toole 

## Summary
* Exokernel securely exports all hardware resources through low-level interface to untrusted library operating systems. 
* Library operating systems implement system objects and policies.
* Typical OS abstractions are implemented entirely at application level (i.e. user space)

## Techniques
* Secure bindings: Applications can securely bind to resources and handle events.
* Visible resource revocation: Applications participate in a resource revocation protocol.
* Abort protocol: Exokernel can break secure bindings of uncooperative applications.

## Secure bindings
* Decouple authorization from actual use of hardware resources
* Improve performance in two ways: a) checks are simple operations implemented by kernel or hardware. b) Authorization 
is performed at bind time. 
* Anything that improves the performance is considered a secure binding
* Hardware mechanisms: TLB entry is a hardware secure binding. 
* Software caching: Shadow TLB - Cache the TLB for each library operating system. On context switch, the TLB is dumped/loaded
from the shadow TLB. 
* Download code into kernel: Packet filter.

## Revocation
* Can be visible or invisible to applications.
* Exokernel uses visible revocation for most resources.
* Library operating system reacts to revocation and is responsible for updating internal state affected by revocation.

## Abort protocol
* Exokernel is capable of revoking resources when library OS is unresponsive to revoke request.
* Breaks all existing secure bindings and informs library OS
* Repossession vector includes list of resources being revoked

## Aegis Data Structures
* PE (Processor Environment): Contains entry points into Library OS for handling traps / discontinuities such as exception handlers, interrupt handlers, Protected Entry Context, and Addressing context. 
