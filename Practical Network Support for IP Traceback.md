# Practical Network Support for IP Traceback

https://cseweb.ucsd.edu/~savage/papers/Sigcomm00.pdf

Stefan Savage, David Wetherall, Anna Karlin and Tom Anderson

2000

## Summary
With the rise of denial-of-service attacks and the difficulty imposed by the
employment of IP address spoofing, new techniques must be devised to
be able to track the source of the attacks. This paper introduces a method
for figuring out the source of a denial-of-service attack after it has occurred.

The ability to identify the machines that generate the attack traffic and the 
network path traversed by these packets is referred to as the "traceback problem".
The solution proposed in this paper is to probabilistically mark packets with
path information as they arrive at routers. Given that an attack involves a large 
number of packets, the path can be reconstructed from the partial markings that are
left on the attack packets.

## Existing solutions
### Ingress Filtering
Ingress filtering involves inspecting every single packet at a router, and 
dropping it if arrives with an illegitimate address. 

Limitations
* Router overhead 
* Inability to deterimine if a packet has a valid source address when multiple ISPs are in play.
* Requires widespread deployment across the network
* Attackers can still fake source IPs by using ones that are valid in the network

### Link Testing
*Input debugging*: The victim reaches out to the network operator with an attack signature.
The operator installs a filter in the router which revelas the origin of packets with the given signature.
This is applied recursively until the network's edge is reached. At this point, the next network operator
must be contacted, and repeat the process. The problem with this approach is the management overhead
required. Furthermore, multiple network providers might have to be involved, which is difficult to coordinate.

*Controlled flooding*: This technique exploits the fact that router buffers are shared across its links.
The victim floods the links with large bursts of traffic and monitors how the incoming traffic from the attacker 
is affected. If the rate of the attack drops, the victim can infer the source link. The problem with this approach is
that this technique is a DOS attack in itself.

### Logging
This traceback technique involves logging all packets that arrive at a router, and then
using data mining to determine the paths the packets traversed. The problem is that this
results in a non-trivial resource overhead.

### ICMP Traceback
Probabilistically copy the contents of incoming packets into special ICMP traceback
messages that can be used by a victim to trace the source and reconstruct the path
to the attacker.

