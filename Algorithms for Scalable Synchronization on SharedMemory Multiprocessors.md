# Algorithms for Scalable Synchronization on Shared Memory Multiprocessors

http://www.cs.rice.edu/~johnmc/papers/tocs91.pdf

Mellor-Crummey, Scott

# Summary 
Mellor-Crummey and Scott analyze existing spin-lock and barrier algorithms on shared memory multiprocessors. They
also describe a new spin-lock algorithm and a new barrier algorithm.

# Problem
* Performance of spin-lock and barrier algorithms is of great importance in shared memory multiprocessors. 
* Typical implementions tend to result in high memory contention and interconnection network contention.
* When multiple processors busy-wait on a shared variable, they produce a hot-spot that becomes a serious performance
degradation.

# Main argument
The busy-wait contention can be eliminated by properly designed algorithms that ensure that processors 
are spinning on local variables. The only requirement in terms of hardware is a set of fetch-and-phi operations,
and the ability for a processor to read some portion of the shared memory without using the interconnection network.

# Spin Locks
## Test-and-set Lock
* Polling on a shared variable that indicates whether the lock is held.
* Processor issues test-and-set call when attempting to acquire the lock.
* If the lock is held, the test-and-set call returns true, and the processor keeps trying.
* In the event that the locks is free, one of the test-and-set calls will return false and set lock = true.
* To release the lock, all that is needed is to set the lock state to unlocked.
* Test-and-test-and-set: On cache-coherent hardware, the spinning can be performed on a local variable. Once the lock is
freed, the processor issues the test-and-set operation as an attempt to acquire the lock.
* Test-and-set with exponential backoff: Empirical research shows that adding exponential delays reduces the contention
in the interconnection network.
```
locked = false
func acquire(lock bool) {
  delay = 1
  while test_and_set(lock) == true {
    pause(delay)
    delay = delay * 2
  }
}

func release(lock bool) {
  lock = false
}
```

## Ticket Lock
* In the case of test-and-set locks, all processors are trying to acquire the lock, which results in a high number
of test-and-phi operations.
* The ticket lock reduces this number to one per lock acquisition.
* Also results in FIFO, which means starvation is not possible
* Two counters are employed.
* Next ticket == number of requests to acquire lock
* Now serving == number of times the lock has been released
* Acquire lock by performing fetch_and_increment and then spinning until now_serving == my_ticket
* Release lock by incrementing the now_serving variable
* Delaying also helps in the ticket lock
```
Lock = {now_serving = 0; next_ticket = 0}
def acquire(lock Lock) {
  my_ticket = fetch_and_increment(lock.next_ticket)
  while lock.now_serving != my_ticket {
    pause(my_ticket - lock.now_serving) 
  }
}

def release(lock Lock) {
  lock.now_serving++
}
```

## Array-Based Queueing Lock (Anderson's Lock)
* Use array of lenght P, where P is the number of processors
* The value in the array can have two possible states
* Guarantees FIFO ordering of lock requests
* Downside is space complexity O(P)
```
type Lock {
  has_lock = []bool{true, false, false, ... , false}
  queue_last = 0 // initialize to zero
}
def acquire(lock Lock) {
  my_place = fetch_and_increment(lock.queue_last)
  while (!has_lock[my_place % P]) // spin
  has_lock[my_place % P] = false // reset for next round
}

def release(lock Lock, my_place int) {
  next = (my_place + 1) % P
  lock.has_lock[next] = true
}
```

## MCS Lock
* Guarantees FIFO ordering
* Spins on locally-accessible variables
* O(1) space complexity
* O(1) network transactions per lock acquisition => works well in both cache-coherent and non-cache-coherent systems.
* Downside is that requires more specialized instruction set (`fetch_and_store` and `compare_and_swap`)
* Lock implemented using linked-list structure
* When `me` is acquiring the lock, the lock's `last_requestor` variable is set to `me` using `fetch_and_store` instruction.
* If the `last_requestor` is not null, the `last_requestor`'s `next` pointer is updated to point to `me`
* The processor spins until `me.has_lock` is true
* When releasing the lock, we set `next.has_lock` to true.
* The release case when `next` is null, the `compare_and_swap` operation is required to set the Locks `last_requestor` to null.
* If `compare_and_swap` returns false, this means that there's a new request for the lock, thus we must wait until `me.next` is non-null.
* If `compare_and_swap` returns true, we return, effectively releasing the lock.
```
type QNode {
  next QNode
  has_lock bool
}
type Lock {
  last_requestor QNode
}
func acquire(lock Lock, me QNode) {
  last = fetch_and_store(lock, me)
  last.next = me
  while (!me.has_lock) // spin
}
func release(lock Lock, me QNode) {
  if me.next != null {
    me.next.has_lock = true
    return
  }
  if compare_and_swap(lock, me, null) {
    return // lock.last is me, and swap was succesful
  }
  while me.next == null // spin because another processor is requesting the lock
}  
```

# Barriers
### Counting Barrier
* Initialize `count` variable to P, where P is the number of processors
* When arriving at barrier, decrement `count` atomically
* Spin until `count` equals zero
* If `count` equals one, reset `count` to N
* Processors spinning on `count` equals zero, must also spin on `count` equals N to make sure that barrier has been reset
* Requires two spin loops

### Sense-reversing Barrier
* Gets rid of one of the spin loops
* Similar to counting barrier, initialize `count` to N
* There is a shared `sense` variable initialized to `true`
* Each processor also has a local `sense` variable initialized to `true`
* When arriving at barrier, decrement `count` atomically, and reverse local `sense` variable 
* Spin until shared `sense` barrier equals local `sense` variable
* When `count` equals one, reverse the shared `sense` barrier, and reset `count`
```
count = N // number of processors
shared sense = true
processor private local_sense = true
func central_barrier() {
  local_sense = not local_sense
  if fetch_and_decrement(count) == 1 {
    count = N
    sense = local_sense
  } else {
    while sense != local_sense // spin
  }
}
```

### Tree Barrier
* Break up N processors into groups of k
* Results in tree of O(log_k N)
* Last processor in group to arrive goes up the tree
* Keep going up the tree until parent is null
* Benefit is that the spinning var is broken up into multiple vars => reduces contention
* Downside is that spin location is dynamically determined, which is problematic in non-cache-coherent 
systems. That is, the spinning must happen on remote, shared memory.
```
type Node {
  k integer // fan-in of this node (number of children)
  count integer // initialized to k
  locksense bool // initiallized to false
  parent Node
}

shared nodes = []Node{0 ... P-1} // Array of nodes
processor private sense bool = true
processor private mynode Node // my group's leaf in the tree

func combining_barrier() {
  combining_barrier_aux(mynode)
  // the processor that went all the way up to the root issues this call
  // which effectively frees the processors spinning
  sense = not sense 
}

func combining_barrier_aux(node Node) {
  if fetch_and_decrement(node.count) == 1 {
    if node.parent != nil {
      combining_barrier_aux(node.parent) // Go up the tree
    }
    // We have gone all the way up the tree...
    count := node.k // prepare for next barrier
    node.locksense = not node.locksense // reverse my own locksense to avoid spinning
  }
  while (node.locksense == sense) // spin
}
```

### Dissemination barrier
* Number of processors need not be a power of 2
* Multiple rounds are involved = ceil(log N) rounds
* Round is over when processor has sent a message and received a message
* Barrier is complete when all processors have communicated with every other processor
* O(N) number of communication events per round => O(N log N) communication complexity
* No wake up required
* Static spin location for each processor
* Suitable for NCC and clustered machines

### Tournament barrier
* N processors => log N rounds
* Basically a tree where the "winner" is known ahead of time
* The winner spins until loser is done
* Loser spins on global sense reversal variable
* Winner goes up the tree and spins until new loser is done
* Processor 0 sets a global flag when tournament is over, freeing rest of processors
* Use of wake-up tree avoids spinning on global shared variable. The processors are spinning on local variable that is 
changed by winner of each round when traversing the wake-up tree.
* Network communication complexity is O(P) vs O(P log P) in dissemination barrier


### MCS Tree barrier
* Spinning on locally-accessible flag only
* O(P) space complexity
* Performs minimum number of network transactions on non-cache-coherent systems
* O(log P) network transactions
* 4-ary arrival tree
* Binary wake-up tree
* The 4-ary tree is constructed from the N processors in the following manner:
```
N = 16
P0 = {P1, P2, P3, P4}
P1 = {P5, P6, P7, P8}
P2 = {P9, P10, P11, P12}
P3 = {P13, P14, P15, P16}
Rest of processors have no children in tree
```
* Processor does not inspect any other node, except when signaling arrival by setting a flag in the parent's node
* During wake-up, the parent signals each child node
* Upon arrival, the node spins until all children have cleared their child-not-ready flag.
* Once the node's child-not-ready vector is cleared, the node clears its child-not-ready flag in 
the parent's child-not-ready vector.
* All except the root node are spinning on the parentsense flag.
* When the root arrives at barrier, and the child-not-ready vector is clear, the processor at the root toggles the 
parentsense flag on each of its children. This frees spinning nodes, which in turn set their children's parentsense flag.

# Performance considerations
## Locks
* MCS Lock and array-based queueing lock (on cache-coherent machines) are the most scalable algorithms.
* MCS Lock requires `fetch_and_store` and `compare_and_swap`, which might not be available in all systems.
* Test-and-set with exponential backoff is a reasonable alternative when neither `fetch_and_store` nor 
`fetch_and_increment` is available.

## Barriers
* Dissemination barrier is most appropriate for non-cache-coherent systems.
* MCS tree barrier is the most performant on cache-coherent systems. It requires O(P) updates to shared variables
vs O(P log P) in dissemination barrier.
