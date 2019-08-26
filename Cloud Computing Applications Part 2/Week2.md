## Week 2

## 2.2.1: Eventual Consistency -Part 1

Why is consistency just such a big deal?

### Consistency in a Distributed System

- Why is consistency such a big deal in a distributed system
- Why can reaching consistency be so slow in a distributed system
- What components in a cloud are impacted by consistency issues
- Solution - consistency models
  - ACID, BASE, Paxos
- How are components in a cloud build using these solutions
- Practical issues

### Eric Brewer's CAP theorem

- ***"You can have just two of the Consistency, Availability and Partition Tolerance."***
  - Data centers need to be very responsive, hence availability is vital
  - Responsiveness-even if a transit fault makes it hard to reach some service.
  - Use cached data to respond faster even if the cached entry can't be validated and might be stale!
- Data centers should weakens consistency for faster response

### Fallacies(오류) of Distributed Computing(Peter Deutsch)

- The network is reliable
- latency is zero
- Bandwidth is infinite
- The network is secure
- Topology doesn't change
- There is one administrator
- Transport cost is zero
- The network is homogeneous

-> It is not true(Fallacies)

### Effects of Fallacies

1. Application and transport-layer developers allow unbounded traffic, greatly increasing dropped packets and wasting bandwidth through network latency and packet loss
2. Network security complacency(만족, 안주) allows malicious users and programs that adapt to security measures
3. Multiple administrators, as with subnets for rival companies, may use conflicting policies
4. Building and maintaining a network/subnet incurs large "hidden" costs
5. Bandwidth limits on the part of traffic senders can result in bottlenecks over frequency-multiplexed media

### Brewer's Conjecture(추측)

- ***The CAP theorem***, also known as Brewer's theorem, states that it is impossible for a distributed system to simultaneously provide all three of the following guarantees:
  - Consistency(all nodes see the same data at the same time)
  - Availability (a guarantee that every request receives a response about whether it was successful or failed)
  - Partition tolerance (the system continues to operate despite arbitrary message loss or failure of part of the system)

## 2.2.2 Eventual Consistency - Part 2

### Definition

- Atomic/Linearizable Consistency
  - They look as if they were completed at a single instant
- Availability 
  - Every request received by a non-failing node must result in a response
- Partition-tolerance
  - No set of failures less than total network failure is allowed to cause the system to respond incorrectly

### Result From Asynchronous Systems

- ***It is impossible in the asynchronous network model to implement a read/write data object that guarantees:***
  - Availability
  - Atomic consistency
- In all fair executions (including ones in which message are lost)
- Asynchronous -> There is no clock - decisions made with local computation or messages

### Hint

There is no way to determine whether a message has been lost or just arbitrarily delayed

### Partially Synchronous Model

- ***Allow each node in the network to have a clock, and all clock increase at same rate***
- Can use clocks as timers to know whether received message
- Again, it is impossible in the partially synchronous network model to implement a read/write data object that guarantees the following properties:
  - Availability
  - Atomic consistency
- In all executions (even those in which messages are lost)

### However

- There are partially synchronous algorithms that will return atomic data when all messages in an execution are delivered (no partitions) - and will only return inconsistent data when messages are lost
- Example, use a centralized server to receive all messages and have server respond to messages to all nodes. Use time outs. If time outs fail to predict completion of successful message transfer, then atomic consistency may be violated

### t-Connected Consistency

- If no partitions, then system in consistent
- If partition, then system can return stale data
- Once a partition heals, there is a time limit(clock) on how long the system will take to return to consistency
  - Notion of eventual consistency takes that and runs with that idea.
  - If you are exchanging the messages, if it is asynchronous, you could not do anything about it
  - But it is partially consistent, what you can do is to take account of all these states, and reason about whether you've received all the messages that you needed to

### Is Inconsistency a bad Thing?

- How much consistency is really needed in the first tier of the cloud?
  - Think about YouTube videos. Would consistency be an issue here?
  - What about the Amazon "number of units available" counters. Will people notice if those are a bit off?
- Puzzle : can you come up with a general policy for knowing how much consistency a given thing needs?

### Consistency: Two "Views"

- Client sees a snapshot of the database that is internally consistent and "might" be valid
- Internally, database is genuinely consistent, but the states clients saw are not tracked and might sometimes become invalidated by an update
- Inconsistency is tolerated because it yields such big speedups, although some clients see "wrong" results

### Core Issue: How Much Contention?

- Root challenge is to understand
  - How many updates will occur
  - How often those updates conflict with concurrent reads or with concurrent updates
- In most large cloud applications either
  - Contention is rare, in which case transactional database solutions work
  - Use a relaxed consistency
- Is consistency in clouds dead!

### But Inconsistency Brings Risks Too!(Trade-off)

- Inconsistency cause bugs
  - Clients would never be able to trust servers
- Weak or "best effort" consistency?
  - Strong security guarantees demand consistency
  - Would you trust a medical electronic-health records system or a bank that used 'weak consistency" for better scalability?'

### Properties We Might Want

- Consistency : Updates in an agreed order
- Durability : Once accepted, will not be forgotten
- Real-time responsiveness : Replies with bounded delay
- Security : Only permits authorized actions by authenticated parties
- Privacy : will not disclose personal data
- Fault-tolerance : Failures cannot prevent the system from providing desired services
- Coordination : Actions will not interfere with one-another

## 2.2.3 Consistency Trade-Offs

### Trade-Off

- CAP is a warning that strong properties can easily lead to slow services
- ***Weak properties are often a successful strategy that yields a good solution and requires less effort***

## 2.2.4 ACID and trade-Offs

### Consistent Transactions in the Clouds

- What if we do need consistency?
- We will cover consistent transactions today:
  - Easy way : ACID -> BASE
  - Distributed Transactions with Locking
    - Use Locking (Zookeeper, Paxos)
    - Two-phase locking
  - Distributed Transactions with Time Stamps
    - Time
      - Lamport's logical clocks
      - Real GPS-based clocks
  - Distributed Transactions with Snapshots

### The ACID Model

- A model for correct behavior of databases
- Name was coined in '60s
  - Atomicity : even if "transactions" have multiple operations, does them to completion (commit) or rolls back so that they leave no effect(abort)
  - Consistency : A transaction that runs on a correct database leaves it in a correct ("Consistent") state
  - Isolation : It looks as if each transaction ran all by itself. Basically says "We will hide any concurrency"
  - Durability : once a transaction commits, updates cannot be lost or rolled back

### Why is ACID Helpful

- Developer doesn't need to worry about a transaction leaving some sort of partial state
  - For example, showing Tony as retired and yet leaving some customer accounts with him as the account rep
- Similarly, a transaction can't glimpse a partially completed state of some concurrent transaction
  - Eliminates worry about transient database inconsistency that might cause a transaction to crash
  - Analogous situation : thread A is updating a linked list and thread B tries to scan the list while A is running

### Dangers of Replication

(gray, Helland, & Shasha, 1996)

- Investigated the costs of transactional ACID model on replicated data in "typical" settings
  - Found two cases
    - Embarrassingly easy once : transactions that don't conflict at all (like Facebook updates by a single owner to a page that others might read but never change)
    - Conflict-prone ones: transactions that sometimes interfere and in which replicas could be left in conflicting states if care isn't taken to order the updates
  - Scalability for the latter case will be terrible
- Solutions they recommend involve sharding and coding transactions to favor the first case
- Show that even under very optimistic assumptions slowdown will be O(n^2) in size of replica set (shard)
- If approach is naive, O(n^5) slowdown is possible!

### This Motivates BASE

(Pritchett, 2008)

- Proposed by eBay researchers
  - Found that many eBay employees came from transactional database backgrounds and were used to the transactional style of "thinking"
  - But the resulting applications didn't scale well and performed poorly on their cloud infrastructure
- Goal was to guide that kind of programmer to a cloud solution that performs much better
  - Base reflects experience with real cloud applications 
  - "Opposite" of ACID 

### BASE is a methodology

- Base involves step-by-step transformation of a transactional application into one that will be far more concurrent and less rigid
  - But it doesn't guarantee ACID properties
  - Argument parallels (and actually cities) CAP : they believe that ACID is too costly and often, no needed
  - BASE stands for ***"Basically Available Soft-State Services with Eventual Consistency"***
    - Trying to make it eventually consistent
- It does provide parallelism, it sore of is a parallel to CAP, it doesn't do everything, it doesn't solve CAP problem at all
- What it does instead is to effectively live on this spectrum going from a rigidly acid system to something that is actually serving your purpose, but can sometimes mislead the clients because the values would be out of data

### BASE Terminology

- Basically Available : Fast response even if some replicas are slow or crashed
  - BASE paper : In data centers, partitioning fault are very rare and are mapped to crash failures by forcing the isolated machines to reboot
- Soft-State Service : No durable memory
  - Cannot store any permanent data
  - Restarts in a "clean" state after a crash
  - To remember data, either replicate it in memory in enough copies to never lose all in any crash, or pass it to some other service that keeps "hard state"
- Eventual Consistency :  Ok to send : "optimistic" answers to the external client
  - Could use cached data (without checking for staleness)
  - Could guess at what the outcome of an update will be
  - Might skip locks, hoping that no conflicts will happen
  - Later, if needed, correct any inconsistencies in an off-line cleanup activity
    - If any of these things are not quite right that lead to inconsistencies in the values
    - At some point, you can actually correct the inconsistencies by sort of doing a cleanup of everything that is there

### BASE vs ACID

- ACID Code often much too slow, scale poorly, and end user waits a long time for responses
- With Base
  - Code itself is way more concurrent, hence faster
  - Elimination of locking, early responses, all make end user experience snappy and positive
  - But we do sometimes notice oddities when we look hard
    - Slightly different bidding histories shown to different people shouldn't hurt much if this makes eBay 10x Faster
    - Upload a Youtube video, then search for it -> You may not see it immediately

## 2.2.5 Zookeeper and Paxos: Introduction

### Motivation

- Centralized service : Coordination kernel
- Maintains
  - Configuration information
  - Naming
  - Distributed synchronization
  - Group services
- Avoids synchronization and races
- File-system-based API
  - Manipulates small data nodes: znodes
  - State is a hierarchy of znodes

### Visualizing Paxos

- The ***proposer*** requests that the Paxos system accept some command. Paxos is like a "postal system"
- It thinks about the letter for a while (replicating the data and picking a delivery order)
- Once these are "decided," the learners can execute the command

### Overview of Roles of Processes

- The ***client*** issues a request to the distributed system, and waits for a response (e.g. a write request on a file in a distributed file server).
- The ***Acceptors*** act as the fault-tolerant "memory" of the protocol. Acceptors are collected into groups called Quorums
  - Any message sent to an Acceptor must be sent to a ***Quorum*** of Acceptors
  - Any message sent to an Acceptor is duplicated to that quorum, and any message received from an acceptor is ignored unless you get enough of the acceptors sending the same copy the message
  - Any message received from an Acceptor is ignored unless a copy is received from each Acceptor in a quorum

> 일정 수 이상의 메세지를 acceptors에게 받지 않으면 무시됨

### Paxos Assumptions

Paxos is based on very sort of weak assumption

- Processors operate at arbitrary speed
- Processors may experience failures
- Processors with stable storage may rejoin the protocol after failures
  - Using crash-recovery fault tolerance
- ***Processors do not collude, lie, or otherwise attempt to subvert the protocol***
  - i.e. Byzantine failure don't occur. See Byzantine Paxos for a solution that tolerates failures from arbitrary/malicious behavior of the processes

### Paxos Network

- Processors can send messages to any other processor
- Messages are sent asynchronously and may take arbitrarily long to deliver
- Messages may be lost, reordered, or duplicated
- Messages are delivered without corruption
  - That is, Byzantine network failures don't occur.

### Number of Processors

- In general, a consensus algorithm can make progress using 2F+1 processors despite the simultaneous failure of any F processors
- However, using reconfiguration, a protocol may be employed, which survives any number of total failures as long as no more than F fail simultaneously

## 2.2.6 Paxos

### Overview of Roles of Processes

- A ***proposer*** advocates a client request, attempting to convince the Acceptors to agree on it, and  Learners act as the replication factor for the protocol
- Once a Client request has been agreed on by the Acceptors, the Learner may take action(i.e. execute the request and send a response to the client). To improve availability of processing, additional learners can be added
- Paxos requires a distinguished Proposer (called the ***leader***) to make progress. many processes may believe they are leaders, but the protocol only guarantees progress if one of them is eventually chosen
- If two processes believe they are leaders, they may stall the protocol by continuously proposing conflicting updates. However, the safety properties are still preserved in that case

### Proposal Number & Agreed Value

- Each attempt to define an agreed value v is performed with proposals, which may or may not be accepted by Acceptors
- Each proposal is uniquely numbered for a given Proposer

### Basic Paxos

- Each instance of the Basic Paxos protocol decides on  a single output value
- The protocol proceeds over several rounds
- A successful round has two phases:
  1. Prepare - Promise
  2. Accept request - Accepted

### Prepare Promise

Prepare:

1. A proposer (the leader) creates a proposal identified with a number N
2. ***This number must be greater than any previous proposal number used by this Proposer***
3. Then, it sends a prepare message containing this proposal to a Quorum of Acceptors

Promise:

1. ***If the proposal's number N is higher than any previous proposal number received from any Proposer by the Acceptor, then the Acceptor must return a promise to ignore all future proposals having a number less than N.***
   - If the Acceptor accepted a proposal at some point in the past, it must include the previous proposal number and previous value in its response to the Proposer
2. Otherwise, the Acceptor can ignore the received proposal. ***It does not have to answer in this case for paxos to work.***
   - However for the sake of optimization, sending a denial (Nack) response would tell the Proposer that it can stop its attempt to create consensus with proposal N

### Accept Request

1. If a Proposer receives enough promises from a Quorum of Acceptors, it needs to set a value to its proposal
2. If any Acceptors had previously accepted any proposal, then they will have sent their values to the Proposer, who now must set the value of its proposal to the value associated with the highest proposal number reported by the Acceptors
3. if none of the Acceptors had accepted a proposal up to this point, then the Proposer may choose any value for its proposal
4. The Proposer sends an Accept Request message to a Quorum  if Accept with the chosen value for its proposal

### Accepted 

- ***If an Acceptor receives an accept request message for a proposal N, it must accept it*** if and only if it has not already promised to only consider proposals having an identifier greater than N
- In this case, it should register the corresponding value v and send an Accepted message to the Proposer and every Learner. Else, it can ignore the Accept request
- ***Rounds fail when multiple Proposers send conflicting prepare messages, or when the Proposer does not receive a Quorum of responses (Promise or Accepted).***
  - In these cases, another round must be stared with a higher proposal number
- Notice that when Acceptors accept a request, they also acknowledge the leadership of the Proposer. Hence, paxos can be used to select a leader in a cluster of nodes

### A Paxos for Every Occasion

- Multi Paxos : Avoids Prepare and Promise
- Cheap paxos : Tolerates F failures with F + 1 processors and F auxiliary
- Fast paxos : reduces end-to-end messages
- Generalized paxos : Exploits communitivity
- Byzantine Paxos

## 2.2.7 Zookeeper

### What is ZooKeeper

- A highly available, scalable, distributed, configuration, consensus, group membership, leader election, naming and coordination service
- Difficult to implement these kinds of services reliably
  - Brittle in the presence of change
  - Difficult to manage
  - Different implementations lead to management complexity when the applications are deployed

### ZooKeeper Properties

- File API without partial reads/writes
  - Simple, wait-free data objects organized hierarchically as in file systems
- Per-client guarantee of FIFO execution of requests
- Linearizability for all requests that change the ZooKeeper state
- Built using ZAB, a totally ordered broadcast protocol(based on Paxos)
- 2F+1 servers can tolerate f crash failures

### Any Guarantees?

1. Clients will never detect old data
   - So There is no stale data
2. Clients will get notified of a change to data they are watching within a bounded period of time
3. All requests from a client will be processes in order
4. All results received by a client will be consistent with results received by all other clients

### ZooKeeper Servers

Once the majority of service have stored on their file systems the value, then that is the value that zookeeper's going to return as the consistent value in this distributed algorithm

1. All servers store a copy of the data on disk
2. A leader is elected at startup
3. Followers service clients; all updates go through leader
4. Update response are sent when a majority of servers have persisted the change

### ZooKeeper Service

- All servers store a copy of the data, logs, and snapshots on disk and use an in-memory database
- A leader is elected at start-up
- Followers service clients; all updates go through leader
- Update response are sent when a majority of servers recorded the change