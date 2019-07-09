# Week 4-1

## 1.3 The Mystery of X-The Cap Theorem

We used this magic value of X, which is specified by the client alongside every read or write query had ***X specifies the number of replicas that the coordinator waits for until it can return either a read value or an acknowledgement for the write for the client.***

### CAP Theorem

- CAP : Consistency, Availability, Partition-tolerance
- In a distributed system you can satisfy at most 2 out of the 3 guarantees:
  - Consistency : all nodes see same data at any time or reads return latest written value by any client
  - Availability : The system allows operations all the time, and operations return quickly
  - Partition-tolerance : the system continues to work in spite of network partitions

### Why is availability important?

- Availability = Reads/Writes complete reliably and quickly
- Measurements have shown that a 500ms increase in latency for operations at Amazon.com or at Google.com can cause a 20% drop in revenue.
- At Amazon, each added millisecond of latency implies a $6M yearly loss.
- They care to make the latencies as small as possible for all reads and writes
- SLAs(Service Level Agreements) written by providers predominantly deal with latencies faced by clients

### Why is Consistency important?

- Consistency = all nodes see same data at any time, or reads return latest written value by any client
- When you access your bank or investment account via multiple clients(laptop, workstation, phone, tablet), you want the updates done from one client to be visible to other clients
- When thousands of customers are looking to book a flight, all updates from any client(e.g., book a flight) should be accessible by other clients.

### Why is partition-tolerance important?

- Partitions can happen across datacenters when the Internet gets disconnected
  - Internet router outages
  - Under-sea cables cut
  - DNS not working
- Partitions can also occur within a datacenter, e.g., a rack switch outage
- Still desire system to continue functioning normally under this scenario

### CAP Theorem Fallout

- Since ***partition-tolerance is essential*** in today's cloud computing systems, ***CAP theorem implies that a system has to choose between consistency and availability***
- Cassandra
  - Eventual(weak) consistency, Availability, Partition-tolerance
  - Always choose availability first and next is partition-tolerance
- Traditional RDBMSs
  - Strong consistency over availability under a partition
  - Prefer strong consistency instead over availability or insert availability whenever you have a partition

### CAP Tradeoff

- Starting point for NoSQL Revolution
- A distributed storage system can achieve at most two of C,A, and P.
- When partition-tolerance is important, you have to choose between consistency and availability

Example

- Consistency & Availability : RDBMSs
- Consistency & Partition-tolerance : HBase, HyperTable, BigTable, Spanner
- Partition-tolerance & Availability : Cassandra, RIAK, Dynamo, Voldemort

### Eventual Consistency

What is the eventual consistency which is provided by Cassandra, what does it really mean?	

- Eventual consistency says the following : given a key, suppose all the writes to that key stop.
- Then all the replicas of that key, all the values that are mating on the server side, ***will eventually converge to the latest write that has been written to that particular key.***
- Typically the latest write, it doesn't need to be the latest write, it is some one value.
- It says that all the values need to converge to being the same value
- If all writes stop (to a key), then all its values (replicas) will converges eventually
- If writes continue, then system always tries to keep converging
  - moving "wave" of updated values lagging behind the latest values sent by clients, but always trying to catch up.
- May still return stale values to clients(e.g., If many back-to-back writes).
- But works well when there a few periods of low writes-system converges quickly

> In wikipedia
>
> **Eventual consistency** is a [consistency model](https://en.m.wikipedia.org/wiki/Consistency_model) used in [distributed computing](https://en.m.wikipedia.org/wiki/Distributed_computing) to achieve [high availability](https://en.m.wikipedia.org/wiki/High_availability)  that informally guarantees that, ***if no new updates are made to a given data item, eventually all accesses to that item will return the last updated value.*** Eventual consistency, also called [optimistic replication](https://en.m.wikipedia.org/wiki/Optimistic_replication), is widely deployed in distributed systems, and has origins in early mobile computing projects. A system that has achieved eventual consistency is often said to have **converged**, or achieved **replica convergence**. Eventual consistency is a weak guarantee – most stronger models, like [linearizability](https://en.m.wikipedia.org/wiki/Linearizability)  are trivially eventually consistent, but a system that is merely eventually consistent does not usually fulfill these stronger constraints. 
>
> Eventually-consistent services are often classified as providing BASE (**B**asically **A**vailable, **S**oft state, **E**ventual consistency) semantics, in contrast to traditional [ACID (**A**tomicity, **C**onsistency, **I**solation, **D**urability)](https://en.m.wikipedia.org/wiki/ACID) guarantees. ***Eventual consistency is sometimes criticized as increasing the complexity of distributed software applications.*** This is partly because eventual consistency is purely a [liveness](https://en.m.wikipedia.org/wiki/Liveness) guarantee (reads eventually return the same value) and does not make [safety](https://en.m.wikipedia.org/wiki/Safety_(distributed_computing)) guarantees: an eventually consistent system can return any value before it converges. 

### RDBMS vs Key-Value stores

- While RDBMS provide ACID
  - Atomicity
  - Consistency
  - Isolation
  - Durability
- Key-Value stores like Cassandra provide BASE
  - Basically Available Soft-state Eventual consistency(BASE)
  - Prefer Availability over Consistency
  - Eventual consistency ensures that when writes stop or become slow, then all the replicas of a given key will converge to the same value
    - All values for a given key will eventually be the same after writes have stopped.

### Back to Cassandra : Mystery of X

How do we specify the value of X( the number of replicas that the coordinator waits for until it can return either a read value or an acknowledgement for the write for the client.) for a given read or write?

- The value of X is what is known as a consistency level in Cassandra

- Cassandra has ***consistency levels***

- Client is allowed to choose a consistency level for each operation(read/write)

  - So for each operation that the clients sends out, it can specify a given consistency level for that particular operation

  - ANY : any server(may not be replica) can store that particular write and then return immediately to clients

    - Fastest : coordinator caches writes and replies quickly to client
    - Even if all the replicas are down, ***the coordinator simply caches the write and returns immediately to the client.***

  - ALL : all replicas

    - Ensures strong consistency, ***but the slowest***. Because some of replicas respond slower that the others and you end up waiting for the slowest replica among the group
    - This says that all the replicas if there are three replicas for a given key, ***then all the replicas need to acknowledge*** to the coordinator before the coordinator can stay that the write has been done and return and acknowledgement to client.
    - This ensures that all the replicas acknowledge and all the replicas have gotten the latest write before an acknowledgement is sent back to the client

  - ONE : at least one replica

    - Faster than ALL, but cannot tolerate a failure
    - Where you say that the coordinator needs to receive back acknowledgement form any ONE of the replicas
    - This is again different from ANY because ANY applies allows even the coordinator to store the written value before returning an ACKs the client, ONE does not allow that

  - QUORUM : quorum across all replicas in all datacenters

    - The coordinator needs to receive a quorum of replicas from replicas ***before it ACKs back***
    - what?

    > quorum : (의사 결정에 필요한)정족수

- All the consistency levels we have discussed in write, can also be applied to read the consistency level specifies. ***how many replicas the coordinator needs to hear from before it returns the latest timestamps value to the client?***

  > read에서는 최신 정보 알아야 하기 때문에 timestamps 필요함

### Quorums?

In a nutshell:

- Quorum = majority

  - \> 50%
  - At least 3 out of 5

- Any two quorums intersect

  - Client 1 does a write in red quorum
  - Then client 2 does read in blue quorum

- At least one server in blue quorum returns latest write

- ***Quorums faster than ALL but still ensure strong consistency***

  - ***Quorums don't require you to wait for all the replicas to return an ACKs***
  - However they are giving you a fairly strong notion of consistency almost as strong as the consistency level in spite of not incurring that much of overhead

- You send out the write to all the replicas for example, all the five replicas and whenever the first three of them acknowledge, you can send back an acknowledgement to the client

  > 과반수 넘으면 coordinator는 client에게 acknowledge 보낼 수 있음

### Quorums in detail

- Several key-value/NoSQL stores(e.g., Riak and Cassandra) use quorums
- Reads
  - Client specifies value of R(<= N = total number of replicas of that key)
  - R = read consistency level
  - Coordinator waits for R replicas to respond before sending result to client
  - In background,coordinator checks for consistency of remaining(N-R) replicas, and initiates read repair if needed
- Writes come in two flavors
  - Clients specifies W (<= N)
  - W = write consistency level
  - Client writes new value to W replicas and returns. Two flavors;
    - Coordinator blocks until quorum is reached(Synchronous 방식 -> write 끝날 때까지 다른 write 못하게 막음(block))
    - Asynchronous: Just write and return
- R = read replica count, W = write replica count
- Two necessary conditions for strong consistency in your system:
  - W+R > N
    - If you have a write quorum and a read quorum they intersect in at least one server among the replicas of that key
    - And this insures that reads will return the latest write.
  - W > N/2
    - If you have two write quorums then they intersect in at least one replica server and this replica server will keep the latest value of the write
    - This is also used for returning conflicts, if you have two conflicting writes, then at least one sever detects that conflict
- Select values based on application
  - (W = 1, R = 1) : very few writes and reads - When you don't expect too many conflicts
  - (W = N, R = 1) : great for read-heavy workloads - Write하는 사람은 많고 Read하는건 적음
  - (W=N/2+1, R=N/2+1) : great for write-heavy workloads
  - (W=1, R=N) : great for write-heavy workloads with mostly one client writing per key

### Cassandra Consistency Levels(Contd)

- clients is allowed to choose a consistency level for each operation(read/write)
  - Quorum : quorum across all replicas in all datacenters
    - Global consistency, but still fast
    - EACH_QUORUM이랑 다르게 datacenter 경유 안하고 바로 replicas로 가나...?
  - LOCAL_QUORUM : quorum in coordinator's datacenter
    - Faster : only waits for quorum in first datacenter client contact
  - EACH_QUORUM : quorum in every datacenter
    - Lets each datacenter do its own quorum : supports hierarchical replies
    - This allows each datacenter do its own quorum and it supports hierarchical replies where the replicas in each datacenter
    - Resend a message back to that datacenters coordinator and then that sends message back to ACKs back to the clients coordinator in the clients contact datacenter and then that coordinator can then send back an ACK to the client itself
    - client -> client coordinator -> each datacenter -> back to client coordinator -> client

## 1.4 The Consistency Spectrum

### Consistency Spectrum

Eventual < - > Strong

- Eventual : Faster reads and writes
- Strong(e.g., Sequential) : More consistency

Cassandra offers Eventual Consistency

- If writes to a key stop, all replicas of key will converge(Typically they will converge the latest written value)
- Originally from Amazon's Dynamo and LinkedIn's Voldmort systems

### Newer Consistency Models

- Striving towards strong consistency
- While still trying to maintain high availability and partition-tolerance

Per-key sequential

- You could ensure this by maintaining a per-key coordinator and ensuring that all the reads and writes are serialized through the coordinator and therefore ***they have a single global order.***
- This may be restrictive because if that coordinator fails and you need free like another coordinator, however it does guarantee some stronger notion of consistency
- Per-key, all operation have a global order

CRDTs(Commutative Replicated Data Types)

commutative : 교환 가능한

- Data structures for which commutated writes give same result
- Commutating writes means that if you reverse the order of two writes, then the end result is the same  
- For instance, if your data structure which is an integer, and the only operation allowed on the data in there is a + 1, essentially this is what is known as a counter. a counter is a commutative data type.
  - If you have two writes coming in from two different clients, each of those writes is essentially a + 1 operation
  - If client one's operation goes first and then client two's operation is applied, you get the same result, which is a + 2, as if you applied client two's operation first and then client one's operation, you will again get a + 2
- e.g., value == int, and only op allowed is + 1
- Effectively, servers don't need to worry about consistency
  - If you have a CRDT the servers really do not need to worry about ordering the writes with respect to each other; all the writes have the same effect
  - Or rather, all the writes can be commutated with  each other and the end result of any permutation of these writes is the same as any other permutation
- CRTDs has been extended to not just counters, which are the simplest notion, but also other complicated data structures such as document which are shared and are being written and read by multiple users at the same time.

Red-Blue consistency

- It is not always possible to write operations or transactions on the client's side which only have commutated the commutative property
- There are some operations that absolutely need and require to be ordered all the data centers in the same order
- Rewrite client transactions to separate operations into red ops vs blue ops
  - Blue ops can be executed(commutated) in any order across Datacenters
  - Red ops need to be executed in the same order at each datacenter
  - By rewriting the original client transactions into read and blue operations, you can support a stringer notion of consistency while still supporting some notions of availability

Causal Consistency : Reads must respect partial order based on information flow

- Causal flows help return same value without direct communicate
- Return the latest value of key
- Causal consistency says that reads obey causality and they return the latest causally written value by client
- However, there might be multiple causally written values as you see elsewhere in this course and the causal consistency allows a read to return one of potentially many different, written values and so it is not as strong as the strongest notion of consistency but it is quite close

### Strong Consistency Model

There are two major strong consistency model. One is linearizability and the other is sequential consistency

- Linerarizability : Each operation by a client is visible(or available) instantaneously to all other clients
  - One of the strongest notions of consistency which says that each operation by client is visible or is available instantaneously and when i say instantaneously i mean in real time to all the other clients.
  - ***This is as if there were only one copy of every key value pair,*** and they were being stored on exactly one server. so this is one of the strongest notions of consistency. ***However it is very difficult to support distributed systems.***
  - Instantaneously in real time
- Sequential Consistency:
  - "... the result of any execution is the same as if the operations of all the processor were executed in some sequential order. and the operations of each individual processor appear in this sequence in the order specified by its program"
  - This is an after the-fact way of reordering the operations that occurred so that there is still sanity in tis new ordering and the sanity also obeys the order at each client
  - This reordering obeys things like causality, it obeys clients being able to read their own writes and it also obeys the ordering-the linear ordering at each client itself
  - After the fact, find a "reasonable" ordering of the operations(can re-order operations) that obeys sanity(consistency) at all clients, and across clients
- Transaction ACID properties, e.g., newer key-value/NoSQL stores(sometimes called"NewSQL")
  - Hyperdex
  - Spanner
  - Transaction chains

## 1.5 HBase

### HBase

- Google's BigTable was first "blob-based" storage system
- yahoo! Open-sourced it -> HBase
- Major Apache project today
- Facebook uses HBase internally
- API functions
  - Get/Put(row)
  - Scan(row range, filter) - range queries
    - For instance, you might fetch all the users whose names start with A and B
  - MultiPut
- Unlike Cassandra, HBase prefers consistency(over availability)

### HBase Architecture

- Zookeeper : Small group of servers running Zab, a consensus protocol(Paxos-like)

### HBase Storage Hierarchy

- HBase Table
  - Split it into multiple ***regions*** : replicated across servers
  - Because you don't want store the entire table, some tables might be large, other tables might be small, you don't want to store large tables as one you want to split them into regions so that they are more manageable
  - These regions are a collection of rows in that HBase Table
    - ColumnFamily = subset of columns with similar query patterns
    - It is within the tables so all the regions for a table contain the same set of columns within their corresponding column families
    - One ***Store*** per combination of ColumnFamily + region
    - Every ColumnFamily within a region, you maintain a store.
      - ***MemStore*** for each Store : in-memory updates to Store: flushed to disk when full
      - This is like the MemTable we discussed in Cassandra
        - ***StoreFiles*** for each store for each region: where the data lives
          - ***HFile***
- HFile
  - SSTable from Google's BigTable

### HFile

- HFile contains a lot of data followed by more data 
- And then there might be some metadata, file information, indices, and a trailer information at the end
  - Data : Data itself contains a magic number to identify that uniquely, followed by a variety of key value pairs
    - Key-value pair : Key-value pair contains information such as the key length first, then the value lengths and the the row length. so you know how many bytes each of these contain.
      - row : particular application which maintains. for instance, census information might be the social security number of that particular individual
      - ColumnFamily length
      - ColumnFamily : demographic information
      - ColumnQualifier
      - TimeStamp
      - keyType
      - value
      - etc...

### Strong Consistency: HBase Write-Ahead Log

HLog is maintained on-a-on a per HRegion server basis

client -> HReigionServer 

- -> HRegion ->  Write in HLog first before actual writing and write in MemStore
- If there is a failure after writing into the log, then you can replay this write by looking into your HLog
- Corresponding operation is sent to the store, which then writes into the MemStore
- When the MemStore is full, it is flushed into a store file in store. and that is stored as an Hfile or HDFS
- Log flush in HReigionServer -> HLog

### Log Replay

- After recovery from failure, or upon bootup(HRegion/Service/HMaster)
  - Replay any table stale logs(use timestamps to find out where the database is w.r.t(write respect to) the logs)
  - Replay : add edits to the MemStore, which eventually will get flushed into HFiles themselves.

### Cross-Datacenter Replication

- Single "Master" cluster
- Other "Slave" cluster replicate the same tables
- Master cluster synchronously sends HLogs over to slave clusters, which then use the logs to simply update the corresponding MemStores, which eventually get flushed to disk
- The Coordination among clusters(master and slave) is done via Zookeeper
- Zookeeper can be used like a file system to store control information
  - Zookeeper not just runs a Paxos-like consensus protocol, it also can be used to store information like a file system in a hierarchical fashion
  - For instance, you can use the address "/hbase/replication/state"to store the current state of the database

### Summary

- Traditional Database(RDBMSs) work with strong consistency, and offer ACID
- ***Modern workloads don't need such strong guarantees, but do need fast response times(availability)***
- Unfortunately, CAP theorem
- Key-value/NoSQL systems offer BASE
  - Eventual consistency, and a variety of other consistency models striving towards strong consistency
  - Because the workloads don't necessarily require them but at the same time they are able to grant very good response times

## 2.1 Introduction and Basics

We will look at time in distributed systems. Time is a very important and challenging concept.

### Why Synchronization?

- You want to catch a but at 6.05 PM, but your watch is off by 15 minutes
  - What if your watch is late by 15 min?
    - You will miss the bus
  - What if your watch is Fast by 15 min?
    - You will end up unfairly waiting for longer time ethan you intended
- Time synchronization is required for both
  - Correctness
  - Fairness

### Synchronization In The Cloud

- Cloud airline reservation system
- Server A receives a client request to purchase last ticket on flight ABC123
- Server A timestamps purchase using local clock 9h:15m:32.45s, and logs it. Replies ok to client
- That was the last seat. Server A sends message to Server B saying "flight full"
- B enters "Flight ABC 123 full" + its own local clock value (which reads 9h:10m10.11s) into its log
- Server C queries A's and B's logs Is confused that a client purchased a ticket A after flight become full at B
- This may lead to further incorrect actions by C  

> A와 B의 기준 시간이 달라서 C가 혼동을 겪고 있음

### Why is it Challenging?

- End hosts in Internet-based systems(like clouds)
  - Each have their own clocks
  - Unlike processors(CPUs) within one server or workstation which share a system clock
- Processes in Internet-based system follow an asynchronous system model
  - No bound on
    - Message delays
    - Processing delays
  - Unlike multi-processor(or parallel) systems which follow a synchronous system model

### Some Definitions

- An Asynchronous Distributed System consists of a number of processes
- Each process has a state (value of variables)
- Each process takes actions to change its state, which may be an instruction or a communication action(send, receive).
- An event is the occurrence of an action
- Each process has a local clock - events within a process can be assigned timestamps, and thus ordered linearly
- But - in a distributed system, we also need to know the time order of event across different processes

### Clock Skew vs Clock Drift

- Each process (running at some end host) has its own clock
- When comparing two clocks at two processes:
  - Clock Skew = Relative Difference in clock values of two processes
    - ***Like distance between two vehicles on a road***
  - Clock Drift = relative Difference in clock frequencies (rates) of two processes
    - ***Like difference in speeds of two vehicles on the road***
- A non-zero clock skew implies clocks are not synchronized
- A non-zero clock drift causes skew to increase (eventually)
  - If faster vehicle is ahead, it will drift away
  - If faster vehicle is behind, it will catch up and then drift away

> **Skew : 거리
>
> Drift : 속도
>
> 이렇게 생각하면 편함

### How Often To Synchronize?

- Maximum Drift Rate(MDR) of a clock
- Absolute MDR is defined relative to Coordinated Universal Time(UTC). UTC is the "correct" time at any point of time.
  - MDR of a process depends on the environment
- Max drift rate between two clocks ***with similar MDR*** is 2 * MDR
  - Why?
  - Because the worst case, one of these clocks may be MDR ahead or faster than UTC. The other clock may be MDR slower than UTC, and so the relative difference between them is (2 * MDR).
  - 기준점(UTC)에서 오른쪽 + 왼쪽 = MDR + MDR = 2 * MDR (Because similar MDR is given)
- ***Given a maximum acceptable skew M between any pair of clocks,*** need to synchronize at least once every: M / (2 * MDR) time units
- Since time = distance/speed

### External vs Internal Synchronization

- Consider a group of processes
- External Synchronization
  - Each process C(i)'s clock is within a bound D of a well-known clock S external to the group
  - |C(i) - S| < D at all times
  - External clock may be connected to UTC(universal Coordinated Time) or an atomic clock
  - e.g.,Christian's algorithm, NTP
- Internal Synchronization
  - every pair of processes in group have clocks within bound D
  - |C(i) - C(j)| < D at all times and for all processes i, j
  - E.g., Berkeley algorithm

- ***External Synchronization with D => Internal Synchronization with 2 \* D***
- Internal Synchronization does not imply External Synchronization
  - ***In fact, the entire system may drift away from the external clock S!***

## 2.2 Cristian's Algorithm

One of the algorithms for external synchronization of clocks

### Basics

- External time synchronization
- All processes P synchronize with a time server S

### What is Wrong

- By the time response message is received at P, time has moved on
  - P의 request에 S가 respone를 보낸뒤에도 계속 시간이 흐르고 있기 때문에 clock이 완전히 일치 할 수가 없음.(latencies)
- P's time set to t is inaccurate
- Inaccuracy a function of message latencies
- Since latencies unbounded in an asynchronous systems, the inaccuracy cannot be bounded

### Christian's Algorithm

- ***P measures the round-trip-time(RTT) of message exchange***
- Suppose we know the minimum P -> S latency min 1
- And the minimum S -> P latency min 2
  - min 1 and min 2 depend on Operating system overhead to buffer messages, TCP time to queue messages, etc
  - If you don't know what min1 and min2 are you can just set them to be zero, that is fine
  - This allows you to incorporate any known minimum overheads on the transmission pads into the error and account for them in the error and retain a better error bound
- The actual time at P when it receives response is between ***[t + min2, t + RTT-min1]***

- P sets its time to halfway through this interval
  - To: t + (RTT + min2 - min1)/2
- Error is at most(RTT-min2-min1)/2
  - Bounded! 

### Gotchas

- Allowed to increase clock value but should never decrease clock value
  - may violate ordering of event within the same process
- Allowed to increase or decrease speed of clock
- If error is too high, take multiple readings and average them

## 2.3 NTP

###  NTP = Network Time Protocol

- NTP Servers organized in a tree
- Each Client = ***a leaf of tree - Primary servers -> Secondary servers -> Tertiary servers***
- Each node synchronizes with its tree parent

### Why O = (tr1 - tr2 + ts2 - ts1) / 2

- Offset o = (tr1 - tr2 + ts2 - ts1)/2
- Let's calculate the error
- Suppose real offset is oreal
  - Child is ahead of parent by oreal
  - Parent is ahead of child by -oreal (opposite direction)
- Suppose one-way latency of Message 1 is L1 (L2 for message 2)
- No one knows L1 or L2!
- Then 
  - tr1 = ts1 + L1 + oreal
  - tr2 = ts2 + L2 - oreal

- Subtracting second equation from the first
  - oreal = (tr1 - tr2 + ts2 - ts1)/2 + (L2 - L1)/2
  - => oreal = o + (L2 - L1)/2
  - => |oreal - o| < |(L2 - L1)/2| < |(L2 + L1)/2|
  - Thus, the error is bounded by the round-trip-time

### And Yet

- we still have a non-zero error!
- We just can't seem to get rid of error
  - Can't , as long as message latencies are non-zeros
- Can we avoid synchronizing clocks altogether, and still be able to order events?

## 2.4 Lamport Timestamps

Lamport Timestamps or logical timestamps, one of the most important building blocks in cloud computing systems and kind of distributed systems.

### Ordering Event in a Distributed Systems

- To order event across process, trying to sync clocks is one approach
- What if we instead assigned timestamps to event that were not absolute time?
- As long as these timestamps obey causality, that would work?
  - If an event A causally happens before another event B, then timestamp(A) < timestamp(B)
  - Human use causality all the time
    - e.g., I enter a house only after I unlock it
    - e.g., You receive a letter only after I send it

### Logical (Or Lamport) Ordering

- Proposed by Leslie Lamport in the 1970s
- Used in almost all distributed systems since then
- Almost all cloud computing systems use some form of logical ordering of events
- -Define a logical relation Happens-Before among pairs of events
- Happens-Before denoted as ->
- Three rules
  1. On the Same process: a -> b, if time(a) < time(b) (using the local clock)
  2. If p1 sends m to p2 : send(m) -> receive(m)
  3. (Transitivity) If a -> b and b -> c then a -> c
- Creates a partial order among events
  - Not all event related to each other via -> 
  - For instance, if processes never exchange messages with each other, then all the events are called as concurrent events, they're not related to each other at all, across different processes
    - Event within a process all still related using the happens-before but across processes, you do not have the happens-before relationship at all.

### In Practice: Lamport Timestamps

- Goal: Assign logical (Lamport) timestamp to each event
- Timestamps obey causality
- Rules
  - Each process uses a local counter(clock) which is an integer
    - initial value of counter is zero
  - A process increments its counter when a send or an instruction happens at it. ***The counter is assigned to the event as its timestamp***
  - A send(message) event carries its timestamp
  - For a receive (message) event the counter is updated by max(local clock, message timestamp)+1

> 각 process는 counter(initial counter = 0) 를 갖고, instruction이 발생 할 때마다 1씩 증가 시킴.
>
> send 할 때 counter(== timestamp)를 같이 보냄
>
> receive는 local-clock과 받은 timestamp(message timestamp) 중 큰 값 + 1로 timestamp 업데이트 함

### Not Always Implying Causality

concurrent event : ***If processes never exchange messages with each other, then all the events are called as concurrent events,*** they're not related to each other at all, across different process. Events within a process all still related using the Happens-before but across processes, you do not have the happens-Before relationship at all.

### Concurrent Events

- A pair of concurrent events doesn't have a causal path from one event to another(either way, in the pair)
- Lamport timestamps not guranteed to be ordered or unequal for concurrent events
- Ok, since concurrent events are not causality relatd!
- Remember
  - E1 -> E2 => timestamp(E1) < timestamp(E2),
  - BUT timestamp(E1) < timestamp(E2) => {E1 -> E2} or {E1 and E2 concurrent}
- ***Lamport timestamps are very causality but they don;t always identify concurrent events***

## 2.5 Vector Clocks

Vector clock : another way of assigning timestamps to events in a distributed system

### Vector Timestamps

- Vector timestamps are used in cloud computing systems such as Riak
- Used in key-value stores like Riak
- Each process uses a vector of integer clocks
- Suppose there are N processes in the group 1..N
- Each vector has N elements
- Process i maintains vector Vi[1,...,N]
- jth element of vector clock at process i, Vi[j], is i's knowledge of latest events at process j

> N개의 프로세스 존재하면 각 프로세스는 N길이의 벡터를 가짐
>
> 다른 프로세스에서 메세지 전달 받으면 자신의 clock 유지하면서 send한 프로세스 index에 있는 벡터값을 업데이트
>
> Ex> p2(0,0,0)이 p1(2,0,0) 메세지 받으면 p2(2,1,0)로 업데이트

### Assigning Vector Timestamps

- Incrementing vector clocks
  1. On an instruction or send event at process i, it increments only its ith element of its vector clock
  2. Each message carries the send-event's vector timestamp V_message[1,..,N]
  3. On receiving a messages at process i:
     1. Vi[i] = Vi[i] + 1
     2. Vi[j] = max(V_message[j],Vi[j]) for j != i

### Causally-Related

- VT1 = VT2
  - iff(if and only if)
    - VT1[i] = VT2[i], for all i = 1, ...,N
- VT1 <= VT2,
  - iff VT1[i] <= VT[2][i], for all i = 1,...,N
- Two events are causally related iff
  - VT1 < VT2, i.e.,
    - iff VT1 <= VT2 &
      - There exists j such that 1 <= j <= N & VT1[j] < VT2[j]

### ... Or Not Causally-Related

- Two events VT1 and VT2 are concurrent
  - iff Not(VT1 <= VT2) AND NOT (VT2 <= VT1) We will denote this as VT2 ||| VT1

>  벡터에서 한쪽의 모든 원소가 다른 원소보다 크거나 같아야 하는데 그렇지 않으면 concurrent임

### Logical Timestamps: Summary

- Lamport timestamps
  - Integer clocks assigned to events
  - Obeys causality
  - Cannot distinguish concurrent events
- Vector timestamps
  - Obey causality
  - By using more space, can also identify concurrent events

### Time And Ordering: Summary

- Clocks are unsynchronized in an asynchronous distributed system
- But need to order events, across processes!
- Time synchronization
  - Cristian's algorithm
  - NTP
  - Berkeley algorithm : using synchronization only within the group, but in all of these cases, when you have a non-zero latencies in the underlying network, you are going to have an error in the clock that you set
  - ***But error a function of round-trip-time***
- Can avoid time sync altogether by instead assigning logical timestamps to events