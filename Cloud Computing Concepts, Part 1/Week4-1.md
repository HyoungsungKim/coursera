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

- Partitions can happen across datacenters when the internet gets disconnected
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