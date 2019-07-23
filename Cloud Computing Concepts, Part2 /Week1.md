# Week1

## Week1 Introduction

- This week we complete the remaining theoretical concepts
  - Leader Election
  - Mutual Exclusion
  - Plus, some examples of their use in cloud systems

## Lesson 1: Leader Election

### 1.1 The Election Problem

#### Why Election?

- Example 1 : your bank account details are replicated at a few servers, but one of these servers is responsible for receiving all reads and writes, i.e., it is the leader among the replicas

  - What if there are two leaders per customer?

  - What if servers disagree about who the leader is?

  - What if the leader crashes?

    ***Each of the above scenarios leads to Inconsistency***

#### More Motivating examples

- Example 2: (A few lectures ago) In the sequencer-based algorithm for total ordering of multicasts, the "sequencer" = leader
- Example 3 : Group of NTP servers : who is the root server?
- Other systems that need leader election: Apache Zookeeper, Google's Chubby
- Leader is useful for coordinating among distributed servers

#### Leader Election Problem

- In a group processes, elect a Leader to undertake special tasks
  - And ***let everyone know*** in the group about this Leader
- What happens when a leader fails(crashes)
  - Some process detects this (using a Failure Detector!)
  - Then what?
- Focus of this lecture: ***Election algorithm***. it is goal:
  1. Elect one leader only among the non-faulty processes
  2. All non-faulty processes agree on who is the leader

#### System Model

- N processes
- Each process has a unique id
- Message are eventually delivered
- Failures may occur during the election protocol

#### Calling for an Election

- Any process can call for an election.
- A process can call for at most one election at a time
- Multiple processes are allowed to call an election simultaneously
  - All of them together must yield only a single leader
- The result of an election should not depend on which process calls for it.

#### Election Problem, Formally

- A run of the election algorithm must always guarantee at the end:
  - Safety : For all non-faulty processes p:(p's elected = (q: a particular non-faulty process with the best attribute value) or NULL
  - Liveness: For all election runs: (election run terminates) & for all non-faulty process p:p's elected is not Null
- At the end of the election protocol, the non-faulty process with the best(highest) election attribute value is elected.
  - Common attribute : leader has highest id
  - Other attribute examples : leader has highest IP address, or fastest cpu, or most disk space space, or most number of files, etc.

### 1.2 Ring Leader Election

#### The Ring

- N processes are organized in a logical ring
  - Similar to ring in Chord p2p system
  - i-th processes pi has a communication channel to p_(i+1) mod N
  - All messages are send clockwise around the ring

#### The Ring Election Protocol

- Any process pi that discovers the old coordinator has failed initiates an "Election" message that contains pi's own id:attribute. This is the initiator of the election.
- When a process pi receives an "Election" message, it compares the attr in the message with its own attr
  - If the arrived attr is greater, pi forwards the message.
  - If the arrived attr is smaller and pi has not forwarded an election message earlier, it overwrites the message with its own id:attr, and forwards it.
  - ***If the arrived id:attr matches that of pi then pi's attr must be the greatest(why?),*** and it becomes the new coordinator. This process then sends an "Elected" message to its neighbor with its id, announcing the election result.
    - matched p means that pi's attribute was written into the message the last time it passed pi and ***that the message has made one circle or one circuit around the ring, and has not changed***
    - 원 한바퀴 돌았는데 값 그대로 돌아옴 -> 가지고 있던 값이 가장 큰 값 의미
  - Why do we say that pi needs to check whether it has not forward an election message so far?
    - Because this will come back to the termination criterion later on
    - What this rule is saying over here is that when pi receives an election message, it compares the attribute in the election message with its own.
    - if its own attribute is higher, then it overwrites the message with its own ID and attribute.

#### The Ring Election Protocol(2)

- When a process pi receives an "Elected" message, it
  - sets its variable elected_i <- id of the message
  - forwards the message unless it is the new coordinator

#### Analysis

- Let's assume no failures occur during the election protocol itself, and there are N processes
- How many messages?
- Worst case occurs when the initiator is the ring successors of the would-be leader

#### Worst-Case Analysis

- (N-1) messages for Election message to get from Initiator(N6) to would-be coordinator(N80)
- N messages for Election message to circulate around ring without message being changed
- N messages for Elected message to circulate around the ring
- Message complexity : (3N-1) messages
- Completion time:(3N - 1) message transmission times
  - Election에 2바퀴, Propagation 1바퀴 따라서 3N - 1
  - Election : N6->N80 : N - 1, N80 -> N80 : N
  - Propagation  : N80 -> N80 : N
  - Total : 3N - 1
- Thus, if there are no failures, election terminates(liveness) and everyone knows about highest-attributed process as leader(safety)

#### Best Case?

- Initiator is the would-be leader, i.e., N80 is the initiator
- Message complexity: 2N messages
- Completion time : 2N message transmission

#### Multiple Initiators?

- Each process remember in cache the initiator of each Election/Elected message it receives
- (All the time) Each process suppresses Election/Elected messages of any lower-id initiators
- Updates cache if receives higher-id initiator's Election/Elected message
- Result is that only the highest-id initiator's election run completes
- 가장 높은 attribute가 leader가 되지 않으면 safety 위반
- If there are multiple initiator of each in the group, only the highest id initiator will have its election complete and therefore that will lead to exactly one leader

#### Effect of Failures

- ***N80 fails(the new would be leader fails), when it has already sent an elected message, elected equals 80 message, unfortuately, this is a case where the election protocol may not terminate***

#### Fixing For Failures

- One option : have predecessor (or successor) of would-be leader N80 detect failure and ***start a new election run***
  - May re-initiate election if
    - Receives an election message but times out waiting for an elected message
    - Or after receiving the elected: 80 message
  - But what if predecessor also fails?
  - And its predecessor also fails? (and so on)

#### Fixing For Failures(2)

- Second option : use the failure detector
- Any process, after receiving Election: 80 message, can detect failure of N80 via its own local failure detector
  - If so, start a new run of leader election
- But failure detectors may not be both complete and accurate
  - Incompleteness in FD => N80's failure might be missed => violation of safety
  - Inaccuracy in FD => N 80 mistakenly detected as failed
    - => new election runs initiated forever
    - => Violation of Liveness

#### Why is Election so Hard?

- Because it is related to the consensus problem!
- if we could solve election, then we could solve consensus!
  - Elect a process, use its id's last bit as the consensus decision
- But since consensus is impossible in asynchronous systems, so is election!
- Next lecture: Can;t we use Paxos?