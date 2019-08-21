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
- Next lecture: Can't we use Paxos?

### 1.3 Election in Chubby and Zookeeper

#### Can use Consensus To Solve Election

- One approach
  - Each process proposes a value
  - Everyone in group reaches consensus on some process Pi's value
  - That lucky Pi is the new leader!

#### Election in Industry

- Several system in industry use paxos-lie approaches for election
  - Paxos is a consensus protocol (safe, but eventually live) : eleswhere in this course
- Google's Chubby system
- Apache Zookeeper

#### Election in Google Chubby

- A system for locking
- essential part of Google's stack
  - Many of Google's internal systems rely on Chubby
  - BigTable, Megastore, etc
- Group of replicas
  - Need to have a master server elected at all times
- Election protocol
  - Potential leader tries to get votes from other servers
  - Each server votes for at most one leader
  - Server with majority of votes becomes new leader, informs everyone

Why safe?

- Essentially, each potential leader tries to reach a quorum(should sound familiar!)
- Since any two quorums intersect, and each server votes at most once, cannot have two leaders elected simultaneously

Why live?

- Only eventually live! Failures may keep happening so that no leader is ever elected
- In practice: elections take a few seconds worst-case noticed by Google : 30s

#### Election in Google Chubby(2)

- After election finishes, other servers promise not to run election again for "a while"
  - "While" = time duration ***called "Master lease"***
  - Set to a few seconds
- Master lease can be renewed by the master as long as it continues to win a majority each time
- Lease technique ensures automatic reelection on master failure

> Master lease 시간 동안 leader election 하지 않음
>
> Master가 다시 master가 되면 master lease 갱신 됨

#### Election in Zookeeper

- Centralized service for maintaining configuration information
- Uses a variant of Paxos called Zab(Zookeeper Atomic Broadcast)
- Needs to keep a leader elected at all times
- Each server creates a new sequence number for itself
  - Let's say the sequence numbers are ids
  - Gets highest id so far(from ZK file system), creates new-higher id, writes it into ZK files system
- Elect the highest-id server as leader

Failure:

- One option: everyone monitors current master(directly or via a failure detector)
  - When there is a failure detected, initiate a new election
  - However, when the master fails, multiple other processes in the group might initiate elections simultaneously and this may lead to a flood of messages. - Leads to a flood of elections
  - Too many messages

- Second option(implemented in Zookeeper)
  - ***Each process monitors its next-higher id process***
  - IF that successor was the leader and it has failed
    - Become the new leader
  - ELSE
    - wait for a timeout, and check your successor again
  - What about id conflicts? What if leader fails during election?
  - To address this, Zookeeper uses a two-phase commit (run after the sequence/id) protocol to commit the leader
    - Leader sends NEW_LEADER message to all
    - Each process responds with ACK to at most one leader, i.e., one with highest process id
    - Leader waits for a majority of ACKs and then sends COMMIT to all
    - On receiving COMMIT, process updates its leader variable
  - No one has a 50% or more of the ACKs in the group. -> Fail
  - And so liveness is not guaranteed here, you may need to read on the lead election protocol again
  - Ensures that safety is still maintained, because of the use of quorums.

### 1.4 Bully Algorithm

#### Bully Algorithm

- All processes know other process' ids When a process finds the coordinator has failed(via the failure detector):
  - If it knows its id is the highest, it elects itself as coordinator, then sends a Coordinator message to all processes with lower identifiers. Election is completed
  - Else if you know that you are not the next highest id process in the system, then you cannot be the bully
    - it initiates an election by sending an election message
    - Whom you send an election message to?
      - You send election messages to only those processes that have a higher id than yourself
    - ***If receives no answer within timeout, calls itself leader and sends Coordinator message to all lower id processes, Election completed***
    - ***If an answer received however, then there is some non-faulty higher process => so, wait for coordinator message, If none received after another timeout, start a new election run***
  - A process that received an Election message replies with OK message within time, and start its own leader election process(unless it has already done so)
- If the bully has crashed, the highest-id process and that you are the next highest id process, then you become the new bully and you tell everyone else with a lower id that in fact you are the new leader and the new bully in the system

#### Failure and Timeouts

- If failures stop, eventually will elect a leader
- How do you set the timeouts
- Based on Worst-case time to complete election
  - 5 message transmission times if there are no failures during the run:
    - Election from lowest id server in group
    - Answer to lowest id server from 2nd highest id process
    - Election from 2nd highest id server to highest id
    - Timeout for answer @ 2nd highest id server
    - Coordinator from 2nd highest id server

#### Analysis

- Worst-case completion time : 5 message transmission times 

  - When the process with the lowest id in the system detects the failure

    - (N-1) process altogether begin elections, each sending messages to processes with higher ids.
    - i-th highest id process sends(i - l) election messages

  - number of Election messages

    = N-1 + N-2 + ...+N - (N-1) = (N-1)*N/2 = O(N^2)

    >모든 process의 응답을 기다려야 함

- Best-case

  - Second-highest id detects leader failure
  - Sends (N-2) Coordinator messages
  - Completion time : 1 message transmission time

  > 1개의 process 응답만 기다리면 됨

#### Impossibility

- Since timeouts built into protocol, in asynchronous system model:
  - protocol may never terminate => Liveness not guaranteed
- But satisfied liveness in synchronous system model where
  - Worst-case one-way latency can be calculated = worst-case process time + worst-case message latency

#### Summary

- Leader election an important component of many cloud computing systems
- Classical leader election protocols
  - Ring-based
  - Bully
- But failure-prone
  - Paxos-like protocols used by Google Chubby, Apache Zookeeper

## Lesson 2: Mutual Exclusion

### 2.1 Introduction and Basics

#### Why Mutual Exclusion?

- Bank's Service in the Cloud : Two of your customers make simultaneous deposit of $10,000 into your bank account, each from a separate ATM
  - Both ATMs read initial amount of $1,000 concurrently from the bank's cloud server
  - Both ATM s add $10,000 to this amount(locally at the ATM)
  - Both write the final amount to the server
  - What is wrong?
    - You lost $10,000!
- The ATMs need mutually exclusive access to your account entry at the server
  - or, ***mutually exclusive*** access to executing the code that modifies the account entry

#### More Uses of Mutual Exclusive

- Distributed File system
  - Locking of files and directories
- Accessing objects in a safe and consistent way
  - Ensure at most one server has access to object at any point of time
- Server coordination
  - Work partitioned across servers
  - Servers coordinate using locks
- In industry
  - Chubby is Google's locking service
  - Many cloud stacks use Apache Zookeeper for coordination among servers

#### Problem Statement for Mutual Exclusion

- Critical Section problem : Piece of code(at all processes) for which we need to ensure there is ***at most one process executing it at any point of time***
- Each process can call three functions
  - `enter()` to enter the critical section(CS)
  - `AccessResource()` to run critical section code
  - `exit()` to exit the critical section

#### Approaches to Solve Mutual Exclusion

- Single OS
  -  If all processes are running in one OS a machine(or VM), then
  - Semaphores, mutex, condition variables, monitors, etc
- Distributed system:
  - Processes communicating by passing messages
- Need to guarantee 3 properties:
  - Safety(essential) - At most one process executes in CS(Critical Section) at any time
  - Liveness(essential)- Every request for a CS is granted eventually
  - Ordering(desirable) - Requests are granted in the order they were made

#### Processes Sharing an OS: Semaphores

- Semaphore == an integer that can only be accessed via two special functions
- Semaphore S = 1; // Max number of allowed accessors

#### Next

- In a distributed system, cannot share variables like semaphores
- So how do we support mutual exclusion in a distributed system?

### 2.2 Distributed Mutual Exclusion

#### System Model

- Before solving any problem, specify its System Model:
  - Each pair of processes is connected by reliable channels(such as TCP)
  - Messages are eventually delivered to recipient, and in FIFO(First In First Out) order.
  - Processes do not fail
    - Fault-tolerant variants exists in literature

#### Central Solution

- Elect a central master(or leader)

  - Use one of our election algorithms!

- Master keeps

  - A queue of waiting requests from processes who wish to access the CS
  - A special token which allows its holder to access CS

- Actions of any process in group:

  - `enter()`
    - Send a request to master
    - Wait for token from master
  - `exit()`
    - Send back token to master

- Master Actions:

  - On receiving a request from process Pi

    ```pseudocode
    if(master has token)
    	Send token to Pi
    else
    	Add Pi to queue
    ```

  - On receiving a token from process Pi

    ```pseudocode
    if(queue is not empty)
    	Dequeue head of queue (say Pj), send that process the token
    else
    	Retain token
    ```

#### Analysis of Central Algorithm

- Safety - at most one process in CS
  - Exactly one token
- Liveness - every request for CS granted eventually
  - With N processes in system, queue has at most N processes
  - If each process exits CS eventually and no failures, liveness guaranteed
- FIFO Ordering is guaranteed, in order of requests received at master

#### Analyzing Performance

There are two requirements from mutual exclusion algorithm

- One is that they involve few messages, which means that they are incurring lower overhead on the network it self.
- They incur low delays at the clients that access the critical section

Efficient mutual exclusion algorithms use fewer messages, and make process wait for shorter durations to access resources. Three metrics:

- `Bandwidth` : the total number of messages sent in each `enter` and `exit` operation
  - 2 messages for enter
  - 1 message for exit
- `Client delay` : the delay incurred by a process at each enter and exit operation(when no other process is in, or waiting) - (We will prefer mostly the enter operation)
  - 2 message latencies(request + grant)
- `Synchronization delay` : the time interval between one process exiting the critical section and the next process entering it(when there is only one process waiting) 
  - 2 message latencies(release + grant)

#### But...

- The master is the performance bottleneck and SPoF(single point of failure)

#### Ring-Based Mutual exclusion

- N processes organized in a virtual ring
- Each process can send message to its successor in ring
- Exactly 1 token
- enter()
  - Wait until you get toekn
- exit()  //already have token
  - Pass on token to ring successor
- If receive token, and not currently in enter(). just pass on token to ring successor

#### Analysis of Ring-Based Mutual Exclusion

- Safety
  - Exactly one token
- Liveness
  - Token eventually loops around ring and reaches requesting process (no failures)
- bandwidth
  - Per enter(), 1 message by requesting process but N Messages throughout system
  - 1 message sent per exit()

***What about performance?***

- Client delay : 0 to N message transmissions after entering enter()
  - Best case : already have token
  - Worst case : just sent token to neighbor
- Synchronization delay between one process' exit() from the CS and the next process' enter():
  - Between 1 and (N - 1) message transmissions.
  - Best case : process in enter() is successor of process in exit()
  - Worst case : process in enter() is predecessor of process in exit()

#### Next

- Client/Synchronization delay to access CS still O(N) in Ring-Based approach
- Can we lower this?

### 2.3 Ricart-Agrawala's Algorithm

#### System Model

- Before solving any problem, specify its System Model:
  - Each pair of processes is connected by reliable channels(such as TCP)
  - Messages are eventually delivered to recipient, and in FIFO(First In First Out) order.
  - Processes do not fail

#### Ricart-Agawala's Algorithm

- Classical algorithm from 1981
- No token
- Uses the notion of causality and multicast
- Has lower waiting time to enter CS than Ring-Based approach

#### Key Idea

- enter() at process Pi
  - Multicast a request to all processes
    - Request : <T, Pi>, where T - current Lamport timestamp at Pi
  - ***Wait until all other processes have responded positively to request***
- Requests are granted in order of causality
- Pi in request <T, Pi> is used to break ties (since Lamport timestamps are not unique for concurrent events)

#### Messages in RA Algorithm

- enter() at process Pi

  - set state to `Wanted`
  - multicast "`Request`" <Ti, Pi> to all processes, where Ti = current Lamport timestamp at Pi
  - wait until all processes send back "`Reply`"
  - change state to `Held` and enter the CS

- On receipt of a Request <Tj, Pj> at Pi (i != j):

  ```pseudocode
  if(state == "Held") or(state == "Wanted" & (Ti,i) < (Tj,j))
  //lexicographic ordering in (Ti, Pj)
  add request to local queue (of waiting requests)
  else send "Reply" to Pj
  //먼저 대기중이었으면 Reply 안보냄
  //모든 노드가 reply 보내면 CS 접근 가능 의미
  ```

- exit() at process Pi

  - change state to `Released` and "`Reply`" to ***all*** queued request

#### Analysis : Ricart-Agrawala's Algorithm

- Safety
  - Two processes Pi and Pj cannot both have access to CS
    - If they did, then both would have sent Reply to each other
    - Thus, (Ti, i) < (Tj, j) and (Tj, j) < (Ti,i), which are together not possible
    - What if(Ti, i) < Tj, j) and Pi replied to Pj's request before it created its own request?
      - Then it seems like both Pi and Pj would approve each others' requests
      - But then, causality and Lamport timestamps at pi implied that Ti > Tj, which is a contradiction
      - So this situation cannot arise

- Liveness
  - Worst-case: wait for all other(N - 1) processes to send Reply
    (모든 노드가 waiting 또는 held 일때)
- Ordering
  - Requests with lower Lamport timestamps are granted earlier

#### Ok, But...

- Compared to Ring-Based approach, in Ricart-Argrwala approach
  - Client/Synchronization delay has now gone down to O(1)
    - Client delay : CS에서 나가서 실행 되는데 까지 시간(enter, exit operation에 걸리는 시간)
    - Synchronization delay : CS진입에 걸리는 시간
  - ***But bandwidth has gone up to O(N)***
- Can we get both down?

### 2.4 Maekawa's Algorithm and Wrap-Up

#### Key Idea

- Ricart-Agrawala requires replies from all processes in group
- ***Instead, get replies from only some processes in group***
- But ensure that only process one is given access to CS(Critical Section) at a time

#### Maekawa's Voting Sets

- Each process Pi is associated with voting set Vi(of processes)
- Each process belong to its own voting set
- The intersection of any two voting sets must be non-empty
  - ***Same concept as Quorums!***
- Each voting set is of size K
- Each process belongs to M other voting sets
- Maekawa showed that K = M = sqrt(N) works best
- One way of doing this is to put N processes in a sqrt(N) by sqrt(N) matrix and for each Pi, its voting set Vi = row containing Pi + column containing Pi. Size of voting set = 2 * sqrt(N) - 1

#### Maekawa: Key Differences From Ricart-Agrawala

- Each process request permission from only its voting set members
  - Not from all
- Each process(in a voting set) gives permission to at most one process at a time
  - Not to all

#### Actions

- state = Released, voted = false

- enter() at process Pi:

  - state = Wanted
  - Multicast Request message to all processes in Vi
  - Wait for Reply (vote) messages from all processes in Vi (including vote from self)
  - state = Held

- exit() at process Pi:

  - state = Released
  - Multicast Release to all processes in Vi

- When Pi received a Request from Pj:

  ```pseudocode
  if(state == Held or voted == true)
  	queue request
  else
  	send reply to Pj and set voted = ture
  ```

- When Pi receives a Release from Pj:

  ```pseudocode
  if(queue empty)
  	voted = false
  else
  	dequeue head of queue, say Pk
  	Send reply only to Pk
  	voted = true
  	//queue 앞에 있던 Pk가 reply 받고 CS 진입 준비
  ```

#### Safety

- When a process Pi receives replies from all its voting set Vi members, no other process pj could have received replies from all its voting set members Vj
  - Vi and Vj intersect in at lest one process say Pk
  - But Pk sends only one Reply(vote) at a time, so it could not have voted for both Pi and Pj

#### Liveness

- A process needs to wait for at most (N-1) other processes to finish CS
- ***But does not guarantee liveness***
- ***Since can have a deadlock***
- Example: all 4 processes need access
  - P1 is waiting for P3
  - P3 is waiting for P4
  - P4 is waiting for P2
  - P2 is waiting for P1
  - No progress in the system!
- There are deadlock-free versions

#### Performance

- bandwidth
  - 2*sqrt(N) messages per enter()
  - sqrt(N) messages per exit()
  - Better than Ricart and Agrawala's (2*(N-1) and N-1 messages)
  - sqrt(N) quite small. N~1 million => sqrt(N) = 1K
- Client delay : One round trip time
- Synchronization delay : 2 message transmission times
  - Synchronization delay means that one process is currently in the critical section, and one of the processes is waiting to get in

#### Why sqrt(N)?

- Each voting set is of size K
- Each process belong to M other voting sets
- total number of voting set members (processes may be repeated) = K * N
- But since each process is in M voting sets
  - K*N/M = N => K = M (1)
- Consider a process Pi
  - Total number of voting sets = members presents in Pi's voting set and all their sets = (M-1)*K + 1
    - 자기 voting set 제외한 모든 voting set의 프로세스와 자기자신 의미
  - This must equal the number of process
    - 자기 voting set은 제외하고 카운트하는건가...?
    - rack끼리 CS 정할때 쓰는건가...?
  - To minimize the overhead at each process (K), need each of the above members to be unique, i.e.,
    - N = (M-1) * K + 1
    - N = (K-1)*K + 1(due to (1))
    - K ~sqrt(N)
- 1개의 프로세스당 전체 프로세스의 수만큼 voting set이 intersect 되야 함
  - ex) 4개의 프로세스 있으면 각각의 프로세스는 4개의 voting set과 intersect 되어있어야 함

#### Failures?

- There are fault-tolerant versions of the algorithms we've discussed
  - e.g., Maekawa
- One other way to handler failures: Use paxos-lie approaches!

#### Chubby

- Google's system for locking 
- used underneath Google's system like BigTable, Megastore, etc.
- Not open-sourced but published
- Chubby provides Advisory locks only 
  - Doesn't guarantee mutual exclusion unless every client checks lock before accessing resource

- Can use not only for locking but also writing small configuration files
- relies on Paxos
- Group of servers with one elceted as Master
  - All servers replicate same information
- Clients send read requests to master, which serves it locally
- Clients send write requests to master, which sends it to all servers, gets majority (quorum) among servers and then responds to client
- On master failure, run election protocol
- On replica failure, just replace it an have it catch up

#### Summary

- Mutual exclusion important problem in cloud commuting systems
- Classical algorithms
  - Central
  - Ring-based
  - Ricart-Agrawala
  - Maekawa
- Industry systems
  - Chubby: a coordination service
  - Similarly, Apache Zookeeper for coordination