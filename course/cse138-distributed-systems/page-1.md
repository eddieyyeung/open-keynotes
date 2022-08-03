# Page 1

Distributed System = partitial failure + unbounded lantency

Time and clocks

What are clocks for?

1. mark points in time
2. durations or intervals of time

* time-of-day clocks
  * sync'd with NTP (network time protocol)
  * bad for 2 (the time of day time can jump backward and forward) (CloudFlare leap second bug)
  * ok-ish for 1
* monotonic clocks
  * only go forward
  * bad for 1
  * good for 2

physical clocks

* time-of-day and monotonic clocks

logical clocks

* only measure ordering of events
* which events happended before another

Suppose A happened before B, called $A \to B$

* A could have caused B
* B could not have caused A

things that happened-before reasoning is good for:

* debugging (possible cause of a bug)
* desiging systems

***

#### Lamport diagrams / SPACETIME diagrams

**The definition of $\to$**

Given two events A and B, we say $A \to B$ , means A happened before B

if any of the following is true (smallest relation):

* A and B occur on the same process line with B after A
* A is a message send event and B is the corresponding receive
* if $A \to C$ and $C \to B$, then $A \to B$ (transitive closure)

**network model**

* A synchronous network is one where there exists an n s.t. no message take longer than n units of time to be delivered.
* An aysnchronous network is one where there exists no such n.

"partially synchronous" Lynch Distributed of Algorithms

**Partial orders**

a set S, together with ...

a binary relation, usually, but not always, written $\leq$, that lets you compare things from S, and has these properties:

* Relexivility: $a \in S, a \leq a $ ($\to$ an irreflexive partial orders)
* Antisymmetry: $\forall a, b \in S$, if $a \leq b$ and $b \leq a$, then $a = b$ (for $\to$, vacuously true)
* Transitivity: $\forall a, b, c \in S$, if $a \leq b$ and $b \leq c$, then $a \leq c$

#### Lamport clocks

$LC(A) = 3$

if $A \to B$, then $LC(A) < LC(B)$

Lamport clocks are consistent with causality.

it's not the case that if $LC(A) < LC(B)$ then $A \to B$. LCs do not characterize causality.

if $\lnot(LC(A) < LC(B))$, then $\lnot(A \to B)$

**Assigning LCs to events**

1. Every process has a counter, initially O
2. On every event, a process increments its counter
3. When sending a message, a process includes its current counter along with the message
4. When receiving a message, set your counter to max(local counter, message counter) plus one

causality is graph reach-ability in spacetime

#### Vector clocks

$A \to B \Leftrightarrow VC(A) < VC(B)$

* consistent with causality
* characterize causality

1. Every process keeps a vector (length N for N processes) of natural numbers, initialized to zeros. (if N=3, then \[0, 0, 0])
2. on every event, a process imcrements its own position in its vector clocks. (all events: sends, receives, and internal events)
3. when sending a message, a process includes its current vector clock. (after the increment from step 2, because sends are events)
4. when receiving a message, a process updates its vector clock to max(local, received). (after incrementing its position, because receives are events)

**max of vectors**

max (\[1, 12, 4], \[7, 0, 2]) = \[7, 12, 4]
