# Active network vision and reality: lessons from a capsule-based system

http://research.microsoft.com/en-us/um/people/padmanab/CSE561/papers/Wet99.pdf (1999)

David Wetherall

## Summary
The Active Networks architecture is explored. Active Networks consist of routers that are capable of executing customized
programs, which results in the ability to create innovative applications that exploit this capability. They generated 
a lot of interest, but they also raised performance and security concerns. 

## Background
Styles by which programability can be embedded into the network:
* Individual devices, such as active bridges and router plugins (e.g. Cisco IOS)
* Across multiple nodes for control tasks, rather than new data transfer services
* Users control handling of their own packets. (e.g. ANTS)

## ANTS
* Capsule approach, which associates an application's payload with code that is run on IP routers that are extensible
* Applications send and receive _capsules_ to programmable routers called _active nodes_
* Active nodes execute forwarding routine
* Conventional nodes perform IP forwarding using IP headers

### Interface
* Capsule structure:

| IP Header | Version | Type | Previous Address | Type-dependent header | Payload |
|-----------|---------|------|------------------|-----------------------|---------|

* Type field indicates type of forwarding. Selected by application when creating capsule.
* Forwarding routines are developed in Java in this implementation of ANTS. They are certified and then installed
on the active nodes.
* API contains methods for a) getting information about the node, b) manipulating local data cache (soft-store), 
and c) performing routing decisions

### Implementation
* Transfer of capsules is separate from transfer of service code
* Code is cached locally
* Code is provided to active routers at the edge of the network. Internal routers use a lightweight code distribution system
to obtain the custom forwarding code
* Code verification is performed via MD5 fingerprint, which is put inside the capsule
* When code is not in the active router's cache, a request is made to the previous router using the previous address field of the capsule.
