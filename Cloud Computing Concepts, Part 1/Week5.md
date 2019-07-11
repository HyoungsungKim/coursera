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