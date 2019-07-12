# Week 5

- Continue our deep-dive into the heart of distributed systems
- Snapshots
- Multicast
- Consensus and Paxos
- There are very core and important concept
- These are techniques widely used in today's cloud/distributed systems
- These techniques have been around for decades

## Lesson 1: Snapshots

### 1.1. What is Global Snapshot?

#### Distributed Snapshot

- More often, each country's representative is sitting in their respective capital, and sending messages to each other(say emails).
- How do you calculate a "global snapshot" in that distributed system?
- What does a "global snapshot" even mean?

#### In The Cloud

- In a cloud: ***each application or service is running on multiple servers***
- Servers handling concurrent events and interacting with each other
- The ability to obtain a "global photograph" of the system in important
- Some uses of having a global picture of the system
  - Checkpoint : can restart distributed application on failure
  - Garage collection of objects : objects at servers that don't have any other objects (at any servers) with pointers to them
  - Deadlock detection : Useful in database transaction systems
  - Termination of computation : Useful in batch computing systems like Folding@Home, SETI@Home.

#### What is a Global Snapshot?

- Global Snapshot = Global State = 
  - Individual state of each process in the distributed system
  - and individual state of each communication channel in the distributed system
- Capture the instantaneous state of each process
- And the instantaneous state of each communication channel, i.e., message in transit on the channels

#### Obvious First Solution

- Synchronize clocks of all processes
- Ask all processes to record their states at known time t
- Problems?
  - Time synchronization always has error
    - Your bank might inform you, "We lost the state of our distributed cluster due to a 1 ms clock skew in our snapshot algorithm."
  - Also, does not record the state of messages in the channels
- ***Again: synchronization not required = causality is enough!***

####  Moving From State to State

- Whenever an event happens anywhere in the system, the global state changes
  - Process receives message
  - Process sends message
  - Process takes a step
- State to state movement obeys causality
  - Next : Causal algorithm for Global Snapshot calculation

### 1.2. Global Snapshot Algorithm

#### System Model

- Problem : Record a global snapshot (state for each process, and state for each channel)
- System Model 
  - N processes in the system
  - There are two uni-directional communication channels between each ordered process pair :
    Pj -> Pi and Pi -> Pj
  - Communication channels are FIFO-ordered
    - First in First out
  - No Failure & Not dropped
  - All messages arrive intact(온전한), and are not duplicated
    - Other papers later relaxed some of these assumptions

#### Requirements

- Snapshot should not interfere with normal application actions, and it should not require application to stop sending messages
- Each process is able to record its own state
  - Process state : application-defined state or, in the works case:
  - its heap, registers, program counter, code, etc. (essentially the coredump)
  - Process state really mean either an application-defined state or failing a state that capture the entire state of the process, its heap register, program, code, program counter, code, stack, and anything else that might appear in the coredump(주 기억장치의 모든 내용을 전부 표시) of that process
- Global state is collected in a distributed manner
  - We do not want centralized collection of the state
  - And we want any process to be able to initiate or start the snapshot collection
  - There might be multiple snapshot collection that might occur in the system, perhaps even simultaneously. each of these runs would be distinguished by using a unique ID
- Any process may initiate the snapshot
  - We will assume just one snapshot run for now

#### Chandy-Lamport Global Snapshot Algorithm

- First Initiator Pi recored its own state
- ***Initiator process creates special messages called "Marker" messages***
  - Not an application message, does not interfere with application messages
- For j = 1 to N except i
  - Pi sends out a Marker message on outgoing channel Cij 
  - (N - 1) channels
- Starts recording the incoming messages on each of the incoming channels at Pi : Cij(for j = 1 to N except i)

> Marker를 i와 연결된 모든 채널에 전송하고 받은 곳에서는 그걸 기록함., Pi와 Pj사이에 단방향 채널 생성 됨

#### Chandy-Lamport Global Snapshot Algorithm(2)

Whenever a process Pi receives a marker message on an incoming channel Cki(From Pk to Pi)

- If(this is the first Marker Pi is seeing)
  - Pi records its own state first
  - Marks the state of channel Cki as : "empty"
    - So this is going to be the final state of this channel Cki in the final global snapshot
  - For j = 1 to N except i
    - Pi sends out a marker message on outgoing channel Cij
  - Starts recording the incoming messages on each of the incoming channels at Pi : Cji
    (for j = 1 to N except i and k)
- else // already seen a Marker message
  - Mark the state of channel Cki as all the messages that have arrived on it since recording was turned on for Cki
- All the process will end up sending marker messages to each other, and this will ensure that all the channels have their state snapshotted
- The state of a channel is snapshotted only process that is ***at the receiving end of that channel***

#### Chandy-Lamport Global Snapshot Algorithm(3)

The algorithm terminate when

- All process have received a marker
  - To record their own state
- All processes have received a marker on all the (N - 1) incoming channels at each
  - To record the state of all channels

Then, (if needed), a central server collects all these partial state pieces to obtain the full global snapshot(optional)

> P1이 initiate 되면 C12, C13에게 send하고 C21, C31 채널이 작동 함(turn on)
>
> C12와 C13에 있던 message가 도착하면 C12와 C13의 state는 "empty"가 됨
>
> Recording 중인 process와 연결된 채널에서 들어오면 duplicate -> become empty
>
> Process가 recording 중일때 application message 받으면 duplicate 돼도 사라지지 않고 남아있음 

The algorithm is terminated when all the processes have calculated their states and all the channel states of all the channels(send(e.g.,C12) <-> send(e.g.,C(21))) in the system have been calculated.

- ***When first marker is initiated, outgoing channel is turned on not recording!***

#### Next

- Global Snapshot calculated by Chandy-lamport algorithm is causally correct
  - What?

### 1.3 Consistent Cuts

#### Cuts

- Cut = time for frontier at each process and at each channel
- Events at the process/channel that happen before the cut are "in the cut"
  - And happening after the cut are "out of the cut"

#### Consistent Cuts

Consistent Cut : a cut that obeys causality

- A cut C is a consistent cut if and only if :
  - for (each pair of events e, f in the system)
    - Such that event e is in the cut C, and if f -> e (f happens-before e)
      - Then : Event f is also in the cut C

> cut 내부에 있는 것끼리 message 주고 받을 때, cut 내부에서 밖으로 메세지 보낼때(왼쪽에서 오른쪽) 
> -> consistent cut
>
> cut 내부에서 외부로 나갈때 (오른쪽에서 왼쪽으로)
> -> Cut but Inconsistent cut

Consistent cut shown by this green dotted line is in face the same as the state captured by the Global Snapshot algorithm

#### In fact...

- Any run of the Chandy-Lamport Global Snapshot algorithm creates a consistent cut

#### Chandy-Lamport Global Snapshot Algorithm Creates A Consistent Cut

Let's quickly look at the proof

- Let ei and ej be events occurring at Pi and Pj, respectively such that
  - ei -> ej (ei happens before ej)
- The snapshot algorithm ensures that
  - If ej is in the cut then ei is also in the cut
- That is : if ej -> \<Pj records its state>, then
  - it must be true that ei -> \<Pi records its state>.
- If ej -> \<Pj records its state>, then it must be true that ei -> \<Pi records its state>. (True)
  - By contradiction, suppose ej -> \<Pj records its state> and \<Pi records its state> -> ei (False)
  - Consider the path of app messages (through other processes) that go from ei -> ej
  - Due to FIFO ordering, markers on each link in above path will precede regular app messages
  - Thus, since \<Pi records its state> -> ei, it must be true that Pj received a marker before ej
  - Thus ej is not in the cut => contradiction

### 1.4 Safety and Liveness

In this lecture, we will see two very important properties that are desired in distributed systems these properties are called ***safety*** and ***liveness***

#### "Correctness" in Distributed Systems

- Can be seen in two ways
- Liveness and Safety
- ***Often confused - it is important to distinguish from each other***

#### Liveness: Exercise

- Liveness = guarantee that something good will happen, eventually
  - Eventually == does not imply a time bound, but if you let the system run long enough, then...
- Example in Real World
  - Guarantee that "at least one of the athletes in the 100 m final will win gold" is liveness
  - A criminal will eventually be jailed
- Examples in Distributed System
  - Distributed computation: Guarantee that it will terminate
  - "Completeness" in failure detectors: every failure is eventually detected by some non-faulty process
  - In Consensus: All Processes eventually decide on a value

#### Safety

- Safety = guarantee that something bad will never happen
- Example in Real World
  - A peace treaty between two nations provides safety
    - War will never happen
  - An innocent person will never be jailed
- Examples in a Distributed System
  - There is no deadlock in a distributed transaction system
  - No object is orphaned in a distributed object system
  - "Accuracy" in failure detectors
  - In Consensus : No two processes decide on different values

#### Can't We Guarantee Both

- ***Can be difficult to satisfy both liveness and safety in an asynchronous distributed system!***
  - Failure Detector: Completeness(Liveness) and Accuracy(Safety) cannot both be guaranteed by a failure detector in an asynchronous distributed system
  - Consensus : Decisions(Liveness) and correct decisions(Safety) cannot both be guaranteed by any consensus protocol in an asynchronous distributed system
  - Very difficult for legal systems(anywhere in the world) to guarante that all criminal are jailed(liveness) and no innocents are jailed(Safety)

#### In The Language Of Global States

- Recall that a distributed systems moves from one global state to another global state, via causal steps
- Liveness w.r.t.(With Respect To) a property Pr in a given state S means
  - S satisfies Pr, or there is some causal path of global states from S to S' where S' satisfies Pr
- Safety w.r.t a property Pr in a given state S means
  - S satisfies Pr, and all global states S' reachable from S also satisfy Pr

#### Using Global Snapshot Algorithm

- Chandy-Lamport algorithm can be used to detect global properties that are stable
  - Stable = once true, stays true forever afterwards
- Stable Liveness examples
  - Computation has terminated
- Stable Non-Safety examples
  - There is a deadlock
  - An object is orphaned (No pointers point to it)
- All stable global properties can be detected using the Chandy-Lamport algorithm 
  - ***Due to its causal correctness***

#### Summary

- The ability to calculate global snapshots in a distributed system is very important
- But don't want to interrupt running distributed application
  - you want to calculate the snapshot concurrently and parallelly while allowing the application to continue proceeding and sending its application messages, and sending its application messages, and the Chandy-Lamport algorithm allows us to do exactly that.
- Chandy-Lamport algorithm calculates global snapshot
  - In other words, it satisfies a consistent cut, it is equivalent to a consistent cut, and it can be used to detect stable global properties which are either liveness properties or non-safety properties
- Obeys causality (creates a consistent cut)
- Can be used to detect stable global properties
- Safety vs Liveness