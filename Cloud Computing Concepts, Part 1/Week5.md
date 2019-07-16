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

## Lesson 2: Multicast

### 2.1 Multicast Ordering

#### Multicast Problem

- Node with a piece of information to be communicated to everyone
- Distributed Group of "Nodes" = Processes at Internet-based host

#### Other Communication Forms

- Multicast -> message sent to a group of processes
- Broadcast -> message sent to all processes(anywhere)
- Unicast -> message sent from one sender processes to one receiver process

#### Who Uses Multicast?

- A widely-used abstraction by almost all cloud systems
- Storage systems like Cassandra or a database
  - Replica servers for a key: write/read to the key are multicast within the replicas group
  - All servers: membership information(e.g., heartbeats) is multicast across all servers in cluster
- Online scoreboards(ESPN, French Open, FIFA Cup)
  - Multicast to group of client interested in the scores
- Stock Exchanges
  - Group is the set of broker computers
  - Groups of computers for High frequency Trading
- Air traffic control system
  - All controllers need to receive the same updates in the same order

#### 1. FIFO Ordering

- ***Multicast from each sender are received in the order they are sent, at all receivers***
- Don't worry about multicast from different senders
- More formally
  - If a correct process issues(sends) multicast(g,m) to group g and then multicast(g,m'), then every correct process that deliver m' would already have delivered m.

> 먼저 온게 먼저 나감

#### 2. Causal Ordering

- Multicasts whose send events are causally related, must be received in the same causality-obeying order at all receivers
- Formally
  - If multicast(g,m) -> multicast(g, m') then any correct process that delivers m' would already have delivered m.
  - (-> is Lamport's happens-before)
  - If two message sends, m and m', are not causally related, meaning that their send events do not have a causal path between them.
  - all bets are off and ***it is not necessarily required that their multicasts be delivered in the same order***
- In concurrent, messages can be received in different orders at different receivers

#### Causal VS. FIFO

- Causal Ordering -> FIFO Ordering
- Why?
  - If two multicasts M and M' are sent by the same process P, and M was sent before M', then M->M'
  - Then a multicast protocol that implements causal ordering will obey FIFO ordering since M->M'
- ***Reverse is not true!*** FIFO ordering does not imply causal ordering.

>Causal -> FIFO (O)
>
>FIFO -> Causal (X)

#### Why Causal At All?

- group = set of your friends on a social network
- A friend sees your message m, and she posts a response (comment) m' to it
  - If friends receive m' before m, it wouldn't make sense
  - But if two friends post messages m'' and n'' concurrently, then they can be seen in any order at receivers
- A variety of systems implement causal ordering: Social networks bulletin boards, comments on websites, etc.

#### 3. Total Ordering

- Also known as "Atomic Broadcast"
- Unlike FIFO and causal, ***this does not pay attention to order of multicast sending***
- ***Ensures all receivers receive all multicasts in the same order***
- Formally
  - If a correct process P delivers message m before m' (independent of the senders), then any other correct process P' that delivers m' word already have delivered m.
- Sometimes multicasts wait to ensure same order

#### Hybrid Variants

- ***Since FIFO/Causal are orthogonal to Total, can have hybrid ordering protocols too***
  - FIFO-total hybrid protocol satisfies both FIFO and total orders
  - Casual-total hybrid protocol satisfies both Causal and total orders

### 2.2 Implementing Multicast Ordering 1

#### Multicast Ordering

- How do we implement each of the ordering schemes we've seen
  - FIFO ordering (This lecture)
  - Causal ordering (Next lecture)
  - Total Ordering (This lecture)

#### FIFO Multicast: Data Structures

- Each receiver maintains a per-sender sequence number(integers)
  - Processes P1 through PN
  - Pi maintains a vector of sequence numbers Pi\[1...N] (initially all zeros)
  - Pi[j] is the latest sequence number Pi has received from Pj

#### FIFO Multicast: Updating Rules

- Send multicast at process Pj:
  - Set Pj[j] = Pj[j] + 1
  - Include new Pj[j] in multicast message as its sequence number
- Receive multicast: If Pi receives a multicast from Pj with sequence number S in message
  - If(S == Pj[j] + 1) then
    - deliver message to application
    - Set Pi[j] = Pi[j] + 1
    - ***Else buffer this multicast until above condition is true***
- It may buffer a received multicast message until FIFO ordering is satisfied

#### Total Ordering

- ***Ensure all receivers receive all multicasts in the same order***
- Formally
  - If a correct process P delivers message m before m'(independent of the senders), then any other correct process P' that delivers m' would already m have delivered ,

#### Sequencer-Based Approach

- Special process elected as leader or sequencer

- Send multicast at process Pi:

  - Send multicast message M to group and sequencer

- Sequencer:

  - Maintains a global sequence number S (initially 0)
  - When it receives a multicast message M, it sets S = S + 1, and multicasts<M, S>

- Receive multicast at process Pi:

  - Pi maintains a local received global sequence number Si (initially 0)

  - If Pi receives a multicast M from Pj, it buffers it until it both 

    1. Pi receives<M, S(M)> from sequencer, and
    2. Si + 1 == S(M)

    - Then deliver it message to application and set Si = Si + 1
    - In other words, it has to wait until S(M) value in the message from the sequencer is one more than the global sequence number Si that Pi is waiting for.
    - 로컬 시퀸스가 글로벌 시퀸스보다 1 커질때 까지 기다려야 함.
    - So this ensures that in fact the multicast message that Pi delivers next is in fact the next one that Pi is waiting for.

### 2.3 Implementing Multicast Ordering 2

#### Causal Ordering

- Multicasts whose send events are causally related, must be received in the same causality-obeying order at all receivers
- Formally
  - if multicast(g, m) -> multicast(g, m') then any correct process that delivers m' would already have delivered m.
  - (-> is Lamport's happens before)

#### Causal Multicast: Data structures

- Each receiver maintains a vector of per-sender sequence numbers(integers)
  - Similar to FIFO Multicast, but updating rules are different
  - Process P1 through PN
  - Pi maintains a vector Pi[1...N] (initially all zeros)
  - Pi[j] is the latest sequence number Pi has received from Pj

#### Causal Multicast: updating Rules

- Send Multicast at process Pj:
  - Set Pj[j] = Pj[j] + 1
  - Include new entire vector Pj[1..N] in multicast message as its sequence number
- Receive multicast: If Pi receives a multicast from Pj with vector M\[1...N](=Pj[1...N]) in message, buffer it until both:
  1. This message is the next one Pi is expecting from Pj, i.e.,
     - M[j] = Pi[j] + 1
  2. All multicasts, anywhere in the group, which happened-before M have been received at Pi, i.e.,
     - For all k != j : M[k] <= Pi[k]
     - i.e., Receiver satisfies causality
  3. When above two conditions satisfied, deliver M to application and set Pi[j] = M[j]

#### Summary: Multicast Ordering

- Important because ***Ordering of multicasts affect correctness of distributed systems using multicasts***
- Three popular ways of implementing ordering
  - FIFO, Causal, Total
- And their implementations
- What about reliability of multicasts?
- What about failures?

### 2.4 Reliable Multicast

#### Reliable Multicast

- Reliable multicast loosely says that every process in the group receives all multicasts
  - Reliability is orthogonal to ordering
  - Can implement Reliable-FIFO, or Reliable-Causal, or Reliable-Total, or Reliable-Hybrid protocols
- What about process failures?
  - When you have process failures, you don't necessarily know what the failed or the faulty processes did the definition becomes a little bit vague
- Definition becomes vague

#### Reliable Multicast(Under Failures)

- Need all correct(i.e., non-faulty) processes to receive the same set of multicasts as all other correct processes
  - Faulty processes are unpredictable, so we will not worry about them
  - We were seeing is that is a single run of the system at the end of the run when you have a small set of correct processes, which did not fall.
  - the set of multicasts received at any one of those processes in that small set is the same as the set of multicasts received at any other process in that same set.
  - Reliable multicast sends that all the multicasts that are sent to the group are either received at all the processes that are correct or none of the processes that are correct
    - That is what reliability really means

#### Implementing Reliable Multicast

- Let's assume we have reliable unicast(e.g., TCP) available to us
- First-cut: Sender process (of each multicast M) sequentially sends a reliable unicast message to all group recipients
- First-cut protocol does not satisfy reliability
  - ***If sender fails, some correct processes might receive multicast M, while other correct processes might not receive M*** -> not real implementation

#### Really Implementing Reliable Multicast

- Trick : Have receivers help the sender

1. Sender process (of each multicast M) sequentially sends a reliable unicast message to all group recipients
2. When a receiver receives multicast M, It also sequentially sends M to all the group's processes

#### Analysis

- Not the most efficient multicast protocol, but reliable
- Proof is by contradiction
- Assumption two correct processes Pi and Pj are so that Pi received a multicast M and Pj did not receive that multicast M
  - Then Pi would have sequentially sent the multicast M to all group members, including Pj, and Pj would have received M
  - A contradiction
  - Hence our initial assumption must be false
  - Hence protocol preserves reliability

### 2.5 Virtual Synchrony

#### Virtual Synchrony or View Synchrony

- ***Attempts to preserve multicast ordering and reliability in spite of failures***
- Combines a membership protocol with a multicast protocol
- Systems that implemented in(like Isis) have been used in NYSE, French Air Traffic Control System, Swiss Stock Exchange

#### Views

> Process가 가지고 있는 membership list를 view라고 함.

- ***Each process maintains a membership list***
- The membership list is called ***View***
- An update to the membership list is called a ***View Change***
  - Process join, leave, or failure
- Virtual synchrony guarantee that ***all view changes are delivered in the same order at all correct processes***
  - If a correct P1 process receives views, say {P1}, {P1, P2, P3}, {P1, P2}, {P1, P2, P4} then
  - ***Any other correct process receives the same sequence of view changes(after it joins the group)***
    - P2 receives views{P1, P2, P3}, {P1, P2}, {P1, P2, P4}
- Views may be delivered at different physical times at processes, ***but they are delivered in the same order***

#### VSync Multicasts

- A multicast M is said to be "delivered in a view V at process Pi" if
  - Pi receives view V, and then sometime before Pi receives the next view it delivers multicast M
- Virtual synchrony ensures that
  1. The set of multicasts delivered in a given view is the same set at all correct processes that were in that view
     - What happens in a View, stays in that View
  2. The sender of the multicast message also belongs to that view
  3. If a process Pi does not deliver a multicast M in view V while other processes in the view V delivered M in V, then Pi will be forcibly removed from the next view delivered after V at the other processes
     - So you have to go with what he rest of the group is doing otherwise you might be forcibly removed from that group if you don't satisfy group

> View 전달 받고 받은 view 다른 process에 전달 안하면 탈락 됨

#### What about Multicast Ordering?

- Again, orthogonal to virtual synchrony
- The set of multicasts delivered in a view can be ordered either
  - FIFO
  - Or Causally
  - Or Totally
  - Or using a hybrid scheme

#### About That Name

- Called "virtual synchrony" since in spite of running on an asynchronous network, it gives the appearance of a synchronous network underneath that obeys the same ordering at all processes
- So can this virtually synchronous system be used to implement consensus?
- No! VSync groups susceptible to partitioning
  - E.g., This may happen due to inaccurate failure detections because a process may be mistakenly detected they will be removed from the group

#### Summary

- Multicast an important building block for cloud computing systems
- Depending on application need, can implement
  - Ordering
  - Reliability
  - Virtual synchrony 
- If you need something really strong  then you can implement virtual synchrony or real synchrony

## Lesson 3: Paxos

### 3.1 The Consensus Problem

#### Give it Thought

- Have you ever wondered why distributed server vendors always only offer solutions that promise five-9's reliability, seven-9's reliability, but never 100% reliable?
- The fault does not lie with the companies themselves, or the worthlessness of humanity
- The fault lies in the impossibility of consensus

#### What is common to all of these

- A group of servers attempting:
  - Make sure that all of them receive the same updates in the same order as each other[Reliable Multicast]
  - ***To keep their own local lists*** where they know about each other, and when anyone leaves or fails, ***everyone is updated simultaneously*** [Membership/Failure Detection]
  - Elect a leader among them, and let everyone in the group know about it [Leader Election]
  - To ensure mutually exclusive(one process at a time only) access to a critical resource like a file
    - critical section에 한번에 하나의 프로세스만 접근 가능해야 함[Mutual Exclusion]

#### So What is Common?

- Let's call each server a "process" (think of the daemon at each server)
- All of these were groups of processes attempting to coordinate with each other and reach agreement on the value of something
  - The ordering of message
  - The up/down status of a suspected failed process
  - Who the leader is
  - Who has access to the critical resource
- ***All of these are related to the Consensus problem***

#### What is Consensus?

Formal problem statement

- N process
- Each process p has
  - input variable xp : initially either 0 or 1
  - output variable yp : initially b (can be changed only once)
- Consensus problem : design a protocol so that at the end, either
  1. All processes set their output variable to 0 (all-0's)
  2. Or All processes set their output variables to 1 (all-1's)

#### What is Consensus?(2)

- Every process contributes a value
- ***Goal is to have all processes decide same (some) value***
  - Decision once made can't be changed
- There might be other constraints
  - Validity = If everyone proposes same value, then that's what is decided
  - Integrity = decided value must have been proposed by some process
  - Non-triviality = There is at least one initial system state that leads to each of the all-0's or all-1's outcomes

#### Why is it important?

- Many problems in distributed systems are equivalent to (or harder than) consensus!
  - Perfect Failure Detection
  - Leader election (select exactly one leader, and every alive process know about it)
  - Agreement (harder than consensus)
- So consensus is a very important problem, and solving it would be really useful!
- So, is there a solution to Consensus

#### Two Different Models of Distributed Systems

- Synchronous System Model and Asynchronous System Model

- ***Synchronous Distributed System***

  - Each message is received within bounded time(bound is global bound)
  - Drift of each process' local clock has a known bound
  - Each step in process takes lb(lower bound) < time < ub(upper bound)
    - Each process has a minimum speed and also a maximum, speed at which it executes instructions

  e.g. A collection of processors connected by a communication bus(share a communication bus and are on the same motherboard), e.g., a Cray supercomputer or a multi-core machine

- ***Asynchronous Distributed System***

  - No bounds on process execution
  - The drift rate of a clock is arbitrary
  - No bounds on message transmission delays

  e.g., The Internet is an asynchronous distributed system, so are ad-hoc and sensor networks

- This is a more general(and thus challenging) model than the synchronous system model. A protocol for an asynchronous system will also work for a synchronous system (but not vice-versa)

#### Possible or Not

- In the synchronous system model
  - Consensus is solvable
- In the asynchronous system model
  - ***Consensus is impossible to solve***
  - Whatever protocol/algorithm you suggest, there is always a worst-case possible execution(with failures and message delays) that prevents the system from reaching consensus
  - Powerful result(see the FLP proof in the Optional lecture of this series)
  - Subsequently, safe or probabilistic solutions have become quite popular to consensus or related problems

### 3.2 Consensus in Synchronous Systems

#### Let's Try to Solve Consensus!

- What is the system model?

- Synchronous system : bounds on

  - Message delays

  - Upper bound on clock drift rates

  - Max time for each process step

    e.g., multiprocessor (common clock across processors)

- Processes can fail by stopping(crash-stop or crash failures)

#### Consensus In Synchronous Systems

- For a system with at most f processes crashing(f is the maximum number of processes that crash)
  - All processes are synchronized and operate in "rounds" of time
  - The rounds are essentially demarcated by specific times at which processes finish a round and start the next round.
  - The algorithm proceeds in f + 1 rounds (with timeout), using reliable communication to all members
  - $Value^r_i$ the set of proposed values known to pi at the beginning of round r.

#### Consensus in Synchronous System

- Initially $Values^0_i$ = {}; $Values^1_i = {v_i}$

  ```\
  for round = 1 to f + 1 do
  	multicast($Values^r_i -Values^{r-1}_i$) 
  	$Values^{r+1}_i$ <- $Values^r_i$
  	
  	//Since this is a synchronous system,
  	//if the sending process and the receiving process are both non-faulty,
      //this message will be received before the end of the round.
      //If any one of them is faulty, then the message may not be received.
      
  	for each Vj received
  		$Values^{r+1}_i = Values^{r+1}_i$ ∪ Vj
  	end
  end
  
  di = minimum($Value^{f+1}_i$)
  ```

- This protocol ensures that all the non-faulty processes in the group, the ones that have not crashed, end up with the identical values arrays at the end of the f + 1 rounds.

- so that when they take this minimum operation, they all end up with the same decision value

- At the end, you simply look at your values array, all the values that you have received, and you can look at the minimum value that you have received.

  - This could either be the minimum Id process who has sent you a value, or 
  - it could just be the minimum value that you have received, and you set your decision variable or your output variable to be that minimum value.

#### Why Does The Algorithm Work?

- After f + 1 rounds, all non-faulty processes would have received the same set of Values. Proof by contradiction.
- Assume that two non-faulty processes, say pi and pj, differ in their final set of values(i.e., after f + 1 rounds)
- Assume that pi possesses a value v that pj does not possess.
  - pi must have received v in the very last round
    - Else, pj would have sent v to pj in that last round
  - So in the last roundL a third process, pk must have sent v to pi, but then crashed before sending v to pj
  - Similarly, a fourth process sending v in the last-but-one round must have crashed; otherwise, both pk and pj should have received v.
  - Proceeding in this way, we infer at least one(unique) crash in each of the preceding rounds
  - This means a total of f+1 crashed, while we have assumed at most f crashes can occur => contradiction