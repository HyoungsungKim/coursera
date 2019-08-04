# Week2

## Lesson 1: Concurrency Control

### 1.1 RPCs

#### Why RPCs

- RPC = Remote Procedure Call
- Proposed by Birrell and Nelson in 1984
- Important abstraction for processes to call functions in other processes
- Allows code reuse
- Implemented and used in most distributed systems, including cloud computing systems
- Counterpart in Object-based setting is called RMI(Remote Method Invocation)

#### Local Procedure Call(LPC)

- Call from one function to another function within the same process
  - Uses stack to pass argument and return values
  - Each process has 1 stack
    - It means local procedure(functions) share same process address space, they can access common objects.
  - Accesses object via pointers(e.g., C) or by reference(e.g., Java)
- LPC has exactly-once semantics
  - If process is alive, called function executed exactly once

#### Remote Procedure Call

- Call from one function to another function, ***where caller and callee function reside in different processes***
  - ***Function call crosses a process boundary***
  - Accesses object via global references
    - ***Can't use pointers*** across processes since a reference address in process P1 may point to a different object in another processes P2
    - e.g., Object address = IP + port + object number
- Similarly, RMI(remote Method Invocation) in Object-based settings
- ***Notice that the remote procedure call here does not cross host boundaries***
  - Same host but different process
- ***If two processes might be on different hosts?***
  - In which case you might need messages that go over the network
  - one message carries the request which is carrying the argument of the call itself(RPC request message)
  - When the RPC is done the RPC reply message is then carried back(RPC reply message)

#### RPC Call Semantics

- Under failures, hard to guarantee exactly-once semantics
- Function may not be executed if
  - Request (call) message is dropped
  - Reply (return) message is dropped
  - Called process fails after executing called function
  - Hard for caller to distinguish these cases
- Function may be executed multiple times if
  - Request(call) message is duplicated

#### Implementing RPC Call Semantics

- Possible semantics
  - At most once semantics (e.g., Java RMI)
  - At least once semantics (e.g., Sun RPC)
  - Maybe, i.e., best-effort (e.g., CORBA)

#### Idempotent Operation

(연산을 여러번 적용해도 달라지지 않음)

- ***Idempotent operations are those that can be repeated multiple times, without any side effects***
- Examples (x is server-side variable)
  - x = 1;
  - x = (argument) y;
- Non-examples
  - x = x + 1;
  - x = x * 2;
- ***Idempotent operations can be used with at-least-once semantics***

#### RPC Components

Client

- Client stub : has same function signature as callee()
  - Allows same caller() code to be used for LPC and RPC
  - It allows the caller to call the callee function just like it was a local function
  - The caller does not need to know that the callee function is not residing in process P1
  - It is calling a function that just looks like a process calling function. Think if a clone that looks like you but is empty inside
  - But when someone tells something to the clone it immediately relays the message to you
- Communication Module : Forwards requests and replies to appropriate hosts

Server

- Dispatcher : selects which server stub to forward request to 
- Server stub : calls callee(), allows it to return a value

#### Generating Code

- Programmer only writes code for caller function and callee function
- Code for remaining components all generated automatically from function signatures (or object interfaces in Object-based languages)
  - e.g., Sun RPC system : Sun XDR interface representation fed into rpcgen compiler
- These components together part of a Middleware system
  - E.g., CORBA (Common Object Request Brokerage Architecture)
  - E.g., SUN RPC
  - E.g., Java RMI

#### Marshalling

> Marshalling이란 한 객체의 메모리에서 표현방식을 저장 또는 전송에 적합한 다른 데이터 형식으로 변환하는 과정이다
>
> 빅엔디안 : 값이 낮은 주소부터 높은 주소 순으로 저장
>
> 리틀엔디안 : 값이 높은 주소부터 낮은 주소 순으로 저장

- Different architecture use different ways of representing data
  - Big endian : Hex 12-AC-33 stored with 12 in lowest address, then AC in next higher address, then 33 in highest address
    - IBM z, System 360
  - Little endian : Hex 12-AC-33 stored with 33 in lowest address, then AC in next higher address, then 12
    - Intel
  - Caller (and callee) process uses its own platform-dependent way of storing data
  - Middleware has a common data representation(CDR)
    - Platform-independent
  - Caller process converts arguments into CDR format
    - Called "Marshalling"
  - Callee process extract arguments from message into its own platform-dependent format
    - Called "Unmarshalling"
  - Return Values are marshalled on callee process and unmarshalled at caller process
    - Caller, callee 둘 다 marshall, unmarshall 함

### 1.2 Transactions

#### Transaction

- Series of operations executed by client
- Each operation is an RPC to a server
- Transaction either
  - Completes and commits all its operations at server
    - ***Commit = reflect updates on server-side objects***
  - Or aborts and has no effect on server

#### Atomicity And isolation

- Atomicity : All or nothing principle : a transaction should either 
  - Complete successfully, so its effects are recorded in ther server object or
  - the transaction has no effect at all
- Isolation : Need a transaction to be indivisible (atomic) from the point of view of other transactions
  - No access to intermediate results/states of other transactions
  - Free from interference by operations of other transactions
- But...
- Clients and/or servers might crash
- Transactions could run concurrently, i.e., with multiple clients
- Transactions may be distributed, i.e., across multiple servers
- You want to preserve isolation while still maximizing

#### ACID Properties for Transactions

- Atomicity : All or nothing
- Consistency : if the server starts in a consistent state, the transaction ends the server in a consistent state
- Isolation : Each transaction must be performed without interference from other transactions, i.e., non-final effects of a transaction must not be visible to other transactions
- Durability : After a transaction has completed successfully, all its effects are saved in permanent storage

#### Multiple Clients, One Servers

- What could go wrong?

#### 1. Lost Update problem

Transaction T1

```pseudocode
time 00 : x = getSeats(ABC123);
time 01 : if (x > 1)
time 02 : 	x = x - 1;
time 03 : write(x, ABC123);
time 04 : commit
```

Transaction T2

```pseudocode
time 00 : //Start T1 transaction at time 00
time 01 : x = getSeats(ABC123);	//However T2 starts at time 01
time 02 : if(x > 1)
time 03 : 
time 04 : 
time 05 : 	x= x - 1;
time 06 : wrtie(x,ABC123);
time 07 : 
time 08 : commit
```

-> T1 or T2's update is lost

#### 2. Inconsistent Retrieval Problem

Operation result of T1 and T2 is different

How prevent it?

### 1.3 Serial Equivalence

#### Concurrent Transactions

- To prevent transactions from affecting each other
  - Could execute them one at a time at server
  - But reduces number of concurrent transactions
  - Transactions per second directly related to revenue of companies
    - This metric needs to be maximized
- Goal : increase concurrency while maintaining correctness(ACID)

#### Serial Equivalence

- An interleaving (say O) of transaction operations is serially equivalent iff(if and only if):
  - There is some ordering(O') of those transactions, one at a time, which 
  - Gives the same end-result (for all objects and transactions) as the original interleaving O
  - Where the operations of each transaction occur consecutively(in a batch)
- Says: ***Cannot distinguish end-result of real operation O from (fake) serial transaction order O'***

#### Checking for Serial Equivalence

- An operation has an effect on
  - The server object if it is write
  - The client (returned value) if it is a read
- Two operations are said to be ***conflicting operations***, if their combined effect depends on the order they are executed
  - read(x) and write(x)
  - write(x) and read(x)
  - write(x) and write(x)
  - Not read(x) and read(x) : swapping them doesn't change end-results
  - Not read/write(x) and read/write(y) : swapping them OK

#### Checking for Serial Equivalence

- Two transactions are serially equivalent if and only of ***all pairs of conflicting operations*** (pair containing one operation form each transaction) ***are executed in the same order***(transaction order) for all objects(data) they both access.
  - Take all pairs of conflict operations, one from T1 and one from T2
  - If the T1 operation was reflected first on the server, mark the pair as "(T1, T2)", otherwise mark it as "(T2, T1)"
  - All pairs should be marked as either "(T1, T2)" or all pairs should be marked as "(T2, T1)"

#### What is our Response?

- At commit point of a transaction T, check for serial equivalence with all other translations
  - Can limit to transactions that overlapped with T
- ***If not serially equivalent***
  - Abort T
  - Roll back (undo) any write that T did to server objects

#### Can we do better?

- Aborting -> wasted work
- Can you prevent violations from occurring?

### 1.4 Pessimistic Concurrently

#### Two Approaches

- Preventing isolation from being violated can be done in two ways
  1. ***Pessimistic*** concurrency control
  2. ***Optimistic*** concurrency control

#### Pessimistic vs. Optimistic

- Pessimistic : assume the worst, prevent transactions from accessing the same object
  - e.g. Locking
- Optimistic : assume the best, allow transactions to write, but check later
  - e.g. Check at commit time, multi-version approaches

#### Pessimistic : Exclusive Locking

- Each object has a lock
- At most one transaction can be inside lock
- Before reading or writing object O, transaction T must call lock(O)
  - Blocks if another transaction already inside lock
- After entering lock T can read and write O multiple times
- When done (or at commit point), T calls unlock(O)
  - If other transactions waiting at lock(O), allows one of them in
- Sound familiar with ***Mutual Exclusion***

#### Can We Improve Concurrency?

- More concurrency -> more transactions per second -> more revenue
- Real-life workloads have a lot of read-only or read-mostly transactions
  - Exclusive locking reduces concurrency
  - Hint: OK to allow two transactions to concurrently read an object, since read-read is not a conflicting pair

> read-read는 Exclusive locking에서 제외 함

#### Another Approach: Read-Write Locks

- Each object has a lock that can be held in one of two modes
  - ***Read mode*** : multiple transactions allowed in
  - ***Write mode*** : exclusive lock
- Before first reading O, transaction T calls read_lock(O)
  - T allowed in only if all transactions inside lock for O all entered via read mode
  - Not allowed if any transaction inside lock for O entered via write mode

#### Read-Write Locks(2)

- Before first writing O, call write_lock(O)
  - Allowed in only if no other transaction inside lock
- If T already holds read_lock(O), and wants to write, call write_lock(O) to promote lock from read to write mode
  - Succeeds only if no other transactions in write mode or read mode
  - Otherwise, T blocks
- Unlock(O) called by transaction T releases any lock on O by T

#### Guaranteeing Serial Equivalence With Locks

- Two-phase locking

  - A transaction cannot acquire (or promote) any locks after it has started releasing locks

- Transaction has two phases

  - These are phases would occur inside the transaction operations themselves, so ***each transaction has these phases while it is executing its operations***

  1. Growing phase : only acquires or promotes locks
  2. Shrinking phase : only release locks
     - Strict two phase locking : released locks only at commit point

#### Why two-phase locking -> Serial equivalence?

- Proof by contradiction
- Assume two phase locking system where serial equivalence is violated for some two transactions T1, T2
- Two facts must then be true:
  - (A) For some object O1, there were conflicting operations in T1 and T2 such that the time ordering pair is (T1, T2)
  - (B) For some object O2, the conflicting operation pair is (T2, T1)
- (A) => T1 released O1's lock and T2 acquired it after that
  => T1's shrinking phase is before or overlaps with T2's growing phase
- Similarly, (B) -> T2's shrinking phase is before or overlaps with T1's growing phase
- But both these cannot be true

#### Downside of Locking

- Deadlocks!

#### When Do Deadlocks Occur?

- 3 necessary conditions for a deadlock to occur

  1. Some object are accessed in exclusive lock modes

  2. Transactions holding locks cannot be preempted

  3. There is a circular wait (cycle) in the Wait-for graph

- "Necessary" = if there is a deadlock, these conditions are all definitely true

- (Conditions not sufficient : If they're present, it doesn't imply a deadlock is present)

#### Combating Deadlocks

1. Lock ***timeout*** : about transaction if lock cannot be acquired within timeout : ***Expensive, wasted work***

2. Deadlock Detection : 

   - Keep track of Wait for graph(e.g. via Global snapshot algorithm), and

   - find cycles in it(e.g., periodically)

   - If find cycle, there's a deadlock => Abort one or more transactions to break cycle 

     -> Still allows deadlocks to occur(Deadlock can be happened again)

3. Deadlock Prevention

   - Set up the system so one of the necessary conditions is violated

     1. Some objects are accessed in exclusive lock modes

        - Fix : Allow read-only access to objects

     2. Transactions holding locks cannot be preempted

        - Fix: Allow preemption of some transactions(Transaction의 우선권을 높여줌)

     3. There is a circular wait(cycle) in the Wait-for graph

        - Fix: Lock all objects in the beginning; if fail any, abort transaction

          => No cycles in Wait-for graph

#### Next

- Can we allow more concurrency?
- Optimistic Concurrency Control

### 1.5 Optimistic Concurrency Control

#### Optimistic Concurrency Control

- Increase concurrency more than pessimistic concurrency control

  (pessimistic concurrency : prevent transactions from accessing the same object)

- Increases transactions per second

- ***For non-transaction systems, increases operations per second and lowers latency***

- Used in Dropbox, Google apps, Wikipedia, key-value stores like Cassandra, Riak, and Amazon's Dynamo

- Preferable than pessimistic when conflicts are expected to be rare

  - But still need to ensure conflicts are caught

#### First-Cut Approach

- Most basic approach
  - Write and read object at will
  - Check for serial equivalence at commit time
  - If abort, roll back updates made
  - An abort may result in other transactions that read dirty data, also being aborted
    - Any transactions that read from those transactions also now need to be aborted - Cascading aborts

#### Second Approach: Timestamp ordering

- Assign each transaction an ID
- Transaction ID determines its position in ***serialization order***
- Ensure that for a transaction T, both are true:
  1. T's ***write*** to object O allowed only if ***transactions that have read or written O had lower ids than T.***
  2. T's ***read*** to object O is allowed only if ***O was last written by a transaction with a lower id that T.***
- Implemented by maintaining read and write timestamps for the object
- Transaction id가 일종의 우선순위 처럼 사용 됨
- If rule violated, abort!
  - Can we do better?

#### Third Approach: Multi-Version Concurrency Control

- For each object
  - A per-transaction version of the object is maintained 
    - Marked as tentative versions
  - And a committed version
- Each tentative version has a timestamp
  - Some systems maintain both a read timestamp and a write timestamp
- On a read or write, find the "Correct" tentative version to read or write from
  - "Correct": based on transaction id, and tries to make transactions only read from "immediately previous" transactions

#### Eventual Consistency...

- That you've seen in key-value stores
- is a form of optimistic concurrency control
  - In Cassandra key-value store
  - In DynamoDB key-value store
  - In Riak key-value store
- But since non-transaction systems, the optimistic approach looks different

#### Eventual Consistency In Cassandra And DynamoDB

- Only one version of each data item(key-value pair)

- Last-write-wins(LWW)

  - Timestamp, typically based on physical time, used to determine whetere to overwrite

    ```pseudocode
    if(new write's timestamp > current object`s timestamp)
    	overwrite;
    else
    	do nothing;
    ```

- With unsynchronized clocks

  - If two writes are close by in time, older write might have a newer timestamp, and might win

#### Eventual Consistency in Riak Key-Value Store

- Uses vector clocks! (Should sound familiar yo you!)
- Implements causal ordering
- uses vector clocks to detect whether
  1. New write is strictly newer than current value, or
  2. If new write conflicts with existing value
- In case (2), a sibling value is created
  - Resolvable by user, or automatically by application (but not by Riak)
- To prevent vector clocks from getting too many entries
  - Size-based pruning
- To prevent vector clocks from having entries updated a long time ago
  - Time-based pruning

#### Summary

- RPCs and RMIs
- Transactions
- Serial Equivalence
  - Detecting it via conflicting operations
- Pessimistic Concurrency Control: locking
- Optimistic Concurrency Control

## Lesson 2: Replication Control

### 2.1 Replication

#### Server-Side Focus

- Concurrency Control = Hot to coordinate multiple concurrent clients executing operations(or transactions) with a server

Next:

- Replication Control = How to handle operations(or transactions) when there are ***objects are stored at multiple servers, with or without replication***

#### Replication: What and Why

- Replication = An object has identical copies, each maintained by a separate server
  - Copies are called "replicas"
- Why replication?
  - ***Fault-tolerance*** : With k replicas of each object, can tolerate failure of any(k - 1) servers in the system
  - ***Load balancing***: Spread read/write operations out over the k replicas => load lowered by a factor of k compared to a single replica
  - Replication => Higher Availability

#### Availability

- If each server is down a fraction f of the time

  - Server's failure probability

- With no replication, availability of object 

  ​	= Probability that single copy is up

  ​	= (1 - f)

  - With k replicas, availability of object

    = Probability that at least one replica is up

    = 1 - Probability that ***all replicas are down***

    = (1 - f^k) 

#### What is the Catch?

- Challenge is to maintain two properties
  1. Replication ***transparency***
     - A client ought not to be aware of multiple copies of object existing on the server side
  2. Replication ***Consistency***
     - All clients see single consistent copy of data, in spite of replication
     - For transactions, guarantee ACID

#### Replication Consistency

- Two ways to forward updates from front-ends(FEs) to replica group
  - Passive replication : uses a primary replica(master)
  - Active Replication : treats all replicas identically
- Both approaches use the concept of ***"Replicated State Machines"***
  - Each replica's code runs the same state machine
  - Multiple copies of the same State machine begun in the Start state, and receiving the same Inputs in the same order will arrive at the same State having generated the same Outputs

#### Passive Replication

- Request
  - Master => total ordering of all updates
  - On master failure, run election

#### Active Replication

- Request
  - Front ends provide replication transparency
  - Front ends multicast inside replica

> Passive Replication : 저장소에서 election 다시 진행
>
> Active Replication : Front ends에서 저장소에 보낼때 multicast로 보냄

#### Active Replication Using Concept You've Learnt Earlier

- Can use any flavor of multicast ordering, depending on application

  - FIFO ordering
  - Causal ordering
  - Total ordering
  - Hybrid ordering

- Total or Hybrid(*-Total) ordering + replicated State machine approach

  => All replicas reflect the same sequence of updates to the object

- What about failures?

  - use virtual synchrony (i.e., view synchrony)

- Virtual synchrony with total ordering for multicasts =>

  - ***Virtual synchrony guarantee that all view changes are delivered in the same order at all correct processes***
  - All replicas see all failures/joins/leaves and all multicasts in the same order
  - Could also use causal (or even FIFO) ordering if application can tolerate it

#### Transactions And Replication

- ***One-copy serialization***
  - A concurrent execution of transactions in a replicated database is one-copy-serializable if it is equivalent to a serial execution of these transactions over a single logical copy of the database
  - (Or) The effect of transactions performed by clients on replicated objects ***should be the same*** as if they had been performed one at a time on a single set of objects(i.e., 1 replica per object)
- In a non-replicated system, transactions appear to be performed one at a time in some order.
  - Correctness means serial equivalence of transactions
- When objects are replicated transaction systems for correctness need
  - Serial equivalence + One-copy serializability

#### 2.2 Two-Phase Commit

#### Transaction With Distributed Servers

- Transaction T may touch object that reside on different servers
- When T tries to commit
  - Need to ensure all these servers commit their updates from T => T will commit
  - Or none of these servers commit => T will abort
- What problem is this?
  - Consensus!
  - It is also called the "Atomic Commit Problem"

#### One-Phase Commit

- Special server called "Coordinator" which is elected by leader election initiates atomic commit
  - Tells other servers to either commit or abort

#### One-Phase Commit: Issues

- Server with object has no say in whether transaction commits or aborts
  - If object corrupted, it just cannot commit (while other servers have committed)
- Server may crash before receiving commit message, with some updates still in memory

#### Two-phase Commit

One of the most widely used protocols for committing transactions.

- When server receives a message from coordinator server, 
  - Save updates to disk
  - Respond with "Yes" or "No"
  - If the server fails, it can then recover starting from this log on disk
- When Coordinator server receive a message and any message is "No" vote or timeout before all votes - ***Abort***
- All "Yes" votes received within timeout - ***Commit***
- Servers(not coordinator server) cannot commit or abort before receiving next message!
  - After receiving, Commit updates from disk to store
- When Coordinator server receive "Ack" from all server, then two-phase commit is done

#### Failures in Two-Phase Commit

- If server voted Yes, ***it cannot commit unilaterally before receiving Commit message***
  - Have to wait commit message from coordinator server
- ***If server voted No, can abort right away(why?)***
- To deal with server crashes 
  - Each server saves tentative updates into permanent storage, right before replying Yes/No in first phase. Retrievable after crash recovery
- To deal with coordinator crashed
  - Coordinator logs all decisions and received/sent messages on disk
  - After recovery or new election => new coordinator takes over
- To deal with Prepare message loss
  - The server may decide to abort unilaterally(일방적으로) after timeout for first phase(server will vote No, and so coordinator will also eventually abort) 
- To deal with Yes/No message loss, coordinator aborts the transaction after a timeout(pessimistic!). It must announce Abort message to all.
- To deal with Commit or abort message loss
  - Server can poll coordinator(repeatedly)

#### Using Paxos In Distributed Servers

Atomic Commit

- Can instead use paxos to decide whether to commit a transaction or not
- But need to ensure that if any server votes No, everyone aborts

Ordering updates

- paxos can also be used by replica group (for an object) to order all updates - iteratively do:
  - Server proposes message for next sequence number
  - Group reaches consensus (or not)

#### Summary

- Multiple servers in cloud
  - Replication for Fault-tolerance
  - Load balancing across objects
- Replication Flavors using concepts we learnt earlier
  - Active replication
  - Passive replication
- transactions and distributed servers
  - Two phase commit