# Lamport, multicast, paxos from wiki

## Chandy-Lamport Algorithm

Chandy-Lamport algorithm is a snapshot algorithm.

### Snapshot algorithm

A snapshot algorithm is used to create a a consistent snapshot of the global state of a distributed system.

### Definition

The assumptions of the algorithm are so follows:

- There are no failures and all messages arrive intact and only once
- The communication channels are unidirectional and FIFO ordered
- There is a communication path between any two processes in the system
- Any process may initiate the snapshot algorithm
- The snapshot algorithm does not interfere with the normal execution of the processes
- Each process in the system records its local state and the state of its incoming channels

The algorithm works using marker messages. Each process that wants to initiate a snapshot records its local state and sends a marker on each of its outgoing channels.

All the other processes, upon receiving a marker, record their local state, the state of the channel from which the marker just came as empty, and send marker messages on all of their outgoing channels.

If a process receives a marker after having recorded its local state, it records the state of the incoming channel from which the marker came as carrying all the messages received since it first recorded its local state

some of the assumptions of the algorithm can be facilitated using a more reliable communication protocol such as TCP/IP. The algorithm can be adapted so that there could be multiple snapshots occurring simultaneously.

### Algorithm

The Chandy-Lamport algorithm works like this

1. The observer process (the process taking a snapshot)
   1. Saves its own local state
   2. Sends a snapshot request message bearing a snapshot token to all other processes
2. A process receiving the snapshot token for the first time on any message:
   1. Sends the observer process its own saved state
   2. Attaches the snapshot token to all subsequent messages (to help propagate the snapshot toke)
3. When a process that has already received the snapshot token receives a message that does not bear the snapshot token, this process will forward that message to the observer process.
   This message was obviously sent before the snapshot "Cut off" (as it does not bear a snapshot token and thus must have come from before the snapshot token was sent out) and needs to be included in the snapshot

From this, the observer builds up a complete snapshot: a saved state for each process and all messages "in the either" are saved

## Multicast

In computing networking, multicast is group communication where data transmission is addressed to a group of destination computers simultaneously. Multicast can be one-to-many or many-to-many distribution.



## Paxos 

Paxos is a family of protocols for solving consensus in a network of unreliable processors. Consensus is the process of agreeing on one result among a group of participants. This problem becomes difficult when the participants or their communication medium may experience failures.

The Paxos family of protocols includes a spectrum of trade-offs between the number of processors, number of message delays before learning the agreed value, the activity level of individual participants, number of messages sent, and types of failures.

### Assumption

In order to simplify the presentation of Paxos, the following assumptions and definitions are made explicit. Techniques to broaden the applicability are known in the literature, and are not covered in this article.

#### Processor

- Processors operate at arbitrary speed.
- Processors may experience failures.
- Processors with stable storage may re-join the protocol after failures (following a crash-recovery failure model).
- Processors do not collude, lie, or otherwise attempt to subvert the protocol. (That is, [Byzantine failures](https://en.wikipedia.org/wiki/Byzantine_fault_tolerance) don't occur. See [Byzantine Paxos](https://en.wikipedia.org/wiki/Paxos_(computer_science)#Byzantine_Paxos) for a solution that tolerates failures that arise from arbitrary/malicious behavior of the processes.)

#### Network

- Processors can send messages to any other processor.
- Messages are sent asynchronously and may take arbitrarily long to deliver.
- Messages may be lost, reordered, or duplicated.
- Messages are delivered without corruption. (That is, Byzantine failures don't occur. See [Byzantine Paxos](https://en.wikipedia.org/wiki/Paxos_(computer_science)#Byzantine_Paxos) for a solution which tolerates corrupted messages that arise from arbitrary/malicious behavior of the messaging channels.)

#### Number of processors

In general, a consensus algorithm can make progress using n = 2F + 1 processors, despite the simultaneous failure of any F processors : ***in other words, the number of non-faulty processes must be strictly greater than the number of faulty processes.*** However, using reconfiguration, a protocol may be employed which survives any number of total failures as long as no more than F fail simultaneously

### Roles

Paxos describes the actions of the processors by their roles in the  protocol: client, acceptor, proposer, learner, and leader. ***In typical implementations, a single processor may play one or more roles at the same time.*** This does not affect the correctness of the protocol—it is  usual to coalesce roles to improve the latency and/or number of messages in the protocol. 

- Client

  The Client issues a *request* to the distributed system, and waits for a *response*. For instance, a write request on a file in a distributed file server.

-  Acceptor (Voters)

  The Acceptors act as the fault-tolerant "memory" of the protocol. Acceptors are collected into groups called Quorums. Any message sent to an Acceptor must be sent to a Quorum of Acceptors. Any message received from an Acceptor is ignored unless a copy is received from each Acceptor in a Quorum.

-  Proposer

  A Proposer advocates a client request, attempting to convince the Acceptors to agree on it, and acting as a coordinator to move the  protocol forward when conflicts occur.

- Learner

  Learners act as the replication factor for the protocol. Once a Client request has been agreed upon by the Acceptors, the Learner may take action (i.e.: execute the request and send a response to the  client). To improve availability of processing, additional Learners can  be added.

-  Leader

  ***Paxos requires a distinguished Proposer (called the leader) to make progress.*** Many processes may believe they are leaders, but the protocol only guarantees progress if one of them is eventually chosen. If two processes believe they are leaders, they may stall the protocol by continuously proposing conflicting updates. However, the [safety properties](https://en.wikipedia.org/wiki/Paxos_(computer_science)#Safety_and_liveness_properties) are still preserved in that case.

#### Quorums

Quorums express the safety (or consistency) properties of Paxos by ensuring at least some surviving processor retains knowledge of the results. 

Quorums are defined as subsets of the set of Acceptors such that any two subsets (that is, any two Quorums) share at least one member. Typically, a Quorum is any majority of participating Acceptors. For example, given the set of Acceptors {A,B,C,D}, a majority Quorum would be any three Acceptors: {A,B,C}, {A,C,D}, {A,B,D}, {B,C,D}. More  generally, arbitrary positive weights can be assigned to Acceptors; ***in that case, a Quorum can be defined as any subset of Acceptors with the summary weight greater than half of the total weight of all Acceptors.***

### Safety and liveness properties

In order to guarantee *safety* (also called "consistency"), Paxos defines three properties and ensures they are always held, regardless of the pattern of failures: 

- Validity (or *non-triviality*)

  Only proposed values can be chosen and learned.

- Agreement (or *consistency*, or *safety*)

  No two distinct learners can learn different values (or there can't be more than one decided value)

- Termination (or liveness)

  If value C has been proposed, then eventually learner L will learn some value (if sufficient processors remain non-faulty)

### Typical deployment

In most deployments of Paxos, each participating process acts in three roles; Proposer, Acceptor and Learner. This reduces the message complexity significantly, without sacrificing correctness

### Basic Paxos

This protocol is the most basic of the Paxos family. Each "instance" (or "execution") of the basic Paxos protocol decides on a single output value. The protocol proceeds over several rounds. A successful round has 2 phases:

phase 1 (which is divided into parts *a* and *b*) and phase 2 (which is divided into parts *a* and *b*). Remember that we assume an asynchronous model, so e.g. a processor may be in one phase while another processor may be in another. 

#### Phase 1

- Phase 1a : prepare
  A Proposer creates a message, which we call a "Prepare", identified with a number *n*. Note that *n* is not the value to be proposed and maybe agreed on, but just a number which uniquely identifies this initial message by the proposer (to be sent to the acceptors). ***The number n must be greater than any number used in any of the previous Prepare messages by this Proposer.*** Then, it sends the *Prepare* message containing *n* to a Quorum of Acceptors. Note that the *Prepare* message only contains the number *n* (that is, it does not have to contain e.g. the proposed value, often denoted by *v*). The Proposer decides who is in the Quorum. A Proposer should not initiate Paxos if it cannot communicate with at least a Quorum of Acceptors.

- Phase 1b: Promise

  Any of the *Acceptors* waits for a *Prepare* message from any of the *Proposers*. If an Acceptor receives a Prepare message, the Acceptor must look at the identifier number *n* of the just received *Prepare* message. There are two cases.

  - ***If n is higher than every previous proposal number received, from any of the Proposers, by the Acceptor, then the Acceptor must return a message, which we call a "Promise", to the Proposer, to ignore all future proposals having a number less than n.*** If the Acceptor accepted a proposal at some point in the past, it must include the previous proposal number, say m, and the corresponding accepted value, say w, in its response to the Proposer.
  - Otherwise (that is, n is less than or equal to any previous proposal number received from any Proposer by the Acceptor) the Acceptor can ignore the received proposal. It does not have to answer in this case for Paxos to work. ***However, for the sake of optimization, sending a denial (Nack) response would tell the Proposer that it can stop its attempt to create consensus with proposal n.***

#### Phase 2

- Phase 2a: Accept ***"Accept this proposal, please!"***
  If a Proposer receives enough Promises[quantify] from a Quorum of Acceptors, it needs to set a value v to its proposal.

  - If any Acceptors had previously accepted any proposal, then they'll have sent their values to the Proposer, who now must set the value of its proposal, v, to the value associated with the highest proposal number reported by the Acceptors, let's call it z.(z : 가장 높은 제안된 숫자)
  - If none of the Acceptors had accepted a proposal up to this point, then the Proposer may choose the value it originally wanted to propose, say x.(x : 원래 제안 하려 했던 숫자 - 만약에 아무도 동의 안해주면 x 선택 함)
  - The Proposer sends an Accept message, (n, v), to a Quorum of Acceptors with the chosen value for its proposal, v, and the proposal number n (which is the same as the number contained in the Prepare message previously sent to the Acceptors). So, the Accept message is either (n, v=z) or, in case none of the Acceptors previously accepted a value, (n, v=x).

- Phase 2b: Accepted

  If an Acceptor receives an Accept message, *(n, v)*, from a Proposer, it must accept it if and only if it has *not* already promised (in Phase 1b of the Paxos protocol) to only consider proposals having an identifier greater than *n*.

      If the Acceptor has not already promised (in Phase 1b) to only consider proposals having an identifier greater than n, it should register the value v (of the just received Accept message) as the accepted value (of the Protocol), and send an Accepted message to the Proposer and every Learner (which can typically be the Proposers themselves).
          Else, it can ignore the Accept message or request.
  Note that an Acceptor can accept multiple proposals. This can happen when another Proposer, unaware of the new value being decided, starts a new round with a higher identification number *n*. In that case, the Acceptor can promise and later accept the new proposed value even though it has accepted another one earlier. These proposals may even have different values in the presence of certain failures. ***However, the Paxos protocol will guarantee that the Acceptors will ultimately agree on a single value.***

### When Rounds Fail

Rounds fail when multiple Proposers send conflicting *Prepare* messages, or when the Proposer does not receive a Quorum of responses (*Promise* or *Accepted*).  In these cases, another round must be started with a higher proposal number.