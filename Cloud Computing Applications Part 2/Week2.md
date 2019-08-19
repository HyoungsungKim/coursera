## Week 2

## Lesson 2.2.1: Eventual Consistency -Part 1

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