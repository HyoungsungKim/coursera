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