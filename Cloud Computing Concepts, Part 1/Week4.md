# Week4

## Introduction

### Week 4 Overview

- This week's first focus : Key-value/NoSQL Storage Systems
- The new generation of cloud storage systems
  - Quickly replacing relational database
- We will see
  - How they are different
  - How they are designed(insides!)
  - Apache Cassandra and HBase in detail
- This is where we put together and combine many of the building blocks you've seen so far
- This week's second focus : ***Time and Ordering***
- Distributed systems in the cloud need to deal with time
  - Or the lack of synchronization among servers and clients
- We will see
  - Time synchronization techniques
  - Logical timestamps
- Both are widely used in cloud systems today
- This is the beginning of our diving into the theoretical concepts underlying cloud/distributed system
  - ***Very important concepts!***

## Lesson 1 : Key-value stores

### 1.1 Why Key-Value/NoSQL

#### The Key-Value Abstraction

What is the key-value store? What kind of an abstraction does it really provide?

-> Key-Value stores map a key to a given value

- (Business) Key->Value
- (twitter.com) tweet id -> information about tweet
- (amazon.com) item number -> information about it
- (kayyak.com) Flight number -> information about flight, e.g., availability
- (yourbank.com) Account number -> information about it

However, you might me saying, that is just a dictionary data structure. So it is a data structure where you can insert, lookup and delete by key.

- It is a dictionary data structure
  - Insert, lookup and delete by key
  - E.g., hash table, binary tree
- But distributed
- Sound familiar? Remember Distributed hash table(DHT) in p2p systems?
- It is not surprising that key-value stores reuse many technique from DHTs
- You cannot maintain all of that on a single server where you have a single process running the strict, dictionary data structure.
- Instead, ***You need to maintain this data on a distributed cluster of servers***
- ***You need to build a distributor hash table***
- It is not a surprise that as a result the new generation of key values towards the no sequence storage systems actually reuse a lot of the design decisions from DHT that we studied in peer to peer system

#### Isn't That Just A Database?

- Yes, sort of
- Relationship Database Management Systems(RDBMSs) have been around for ages
- MySQL is the most popular among them
- Data stored in tables
- ***Schema-based, i.e., structured tables***
- Each row(data item) in a table a primary key that is unique within that table
- Queried using SQL(Structured Query Language)
- Supports joins

#### Mismatch with Today's Workloads

- Data: large and unstructured
- Lots of random reads and writes
- Sometimes write-heavy
- Foreign keys rarely needed
- Joins infrequent

The workload involves a lot of reads and writes coming from multiple clients, millions of clients in some cases and ***this is not necessarily a good match with relation database***

- A lot of these workloads are write-heavy, meaning that there are a lot more writes compared to reads
- ***Relationship databases are typically optimized for read-heavy workloads***

We could potentially come up with a sort of watered down simpler relationship database not even relationship just a database.

- What has happened in this new generation of key value stores and those equal storage systems.

> water down : 물을 타서 희석하다(상황을 부드럽게 하다)

#### Needs of Today's Workloads

So what are the needs of today's workloads?

- Speed
- Avoid Single point of Failure(SPoF)
  - You don't want the failure of one or a few servers in you systems, to lead to loss of data or to unavailability of data for reads and writes
- Low TCO(Total cost of operation - also known as total cost of ownership of infrastructure)
- Fewer system administrations
- Incremental Scalability
- Scale out, not up
  - What?

#### Scale Out, Not Scale Up

- Scale up = grow your cluster capacity by replacing with more powerful machines
  - Traditional approach
  - Not cost-effective, as you are buying above the sweet spot on the price curve
  - And you need to replace machines often
- Scale out = incrementally grow your cluster capacity by adding more COTS machines (Components Off the Shelf)
  - Cheaper
  - Over a long duration, phase in a few newer(faster) machines as you phase out a few older machines
  - Used by most companies who run datacenters and clouds today

#### Key-Value/NoSQL Data Model

- NoSQL = "Not Only SQL"
  - little bit misnomer name
  - Essentially NoSQL and key-Value stores supports are two necessary API operations
- Necessary API operations : ***get(key) and put(key, value)***
  - Get returns value: put updates values if already exists
  - And some extended operations, e.g., "CQL" in Cassandra key-value store
  - ***CQL language supports a small set of extended operations that are more powerful than just get or put*** But are not as powerful as the SQL language

What are the concepts in these key-value store and NoSQL, systems? ***First of all, these new systems maintain tables just like relation database tables, but the tables have different names.***

- Tables
  - "Column families"(Columns share something common) in Cassandra, "Table" in HBase, "Collection" in MongoDB
  - Like RDBMS tables, but differences are...
    - Maybe unstructured : may not have schemes
      - Some columns may be missing from some rows
    - Don't always support joins or have foreign keys
    - Can have index tables, just like RDBMSs

What a key-value in NoSQL system would look like for our data model.

- Unstructured
- No schema imposed
- Columns missing from some rows
- No foreign keys, joins may not be supported

#### Column-Oriented Storage

NoSQL systems often use ***column-oriented storage***

- RDBMSs store ***entire row together***(on disk or at a server)
  - This means that the entire key on all the columns associated with that key are stored together.
- NoSQL systems typically ***store a column together***(or a group of columns).
  - Entires within a column are indexed and easy to locate, given a key(and vice-versa)
  - For instance, you can find out which is the URL column that corresponds to the user Charlie
  - Column might be stored in one place.
  - And the user name column might be stored as one in a separate place

Why useful?

- Search that use one or more columns are easier to do an more efficient and they can be done by fetching only a part of the table rather than fetching the entire table.

- Range searches within a column are fast since you don't need to fetch the entire database

- E.g., Get me all the blog_ids from the blog table that were updated within the past month

  - In relationship database
    - You have to refresh the entire table all the rows and then look at the corresponding column in each row
  - In the NoSQL
    - You simply look at the last update column
    - For the entires in the column that match, which are within the past month, you get the corresponding entires in the block ID columns
    - Search in the last_update column, fetch corresponding blog_id column
    - Don't need to fetch the other columns
    - Only columns are sufficient, and that means that you are incurring ***less of an I/O overhead***


### 1.2 Cassandra

#### Cassandra

- A distributed key-value store
- Intended to run in a datacenter(and also across DCs)
- Originally designed at Facebook
- Open-sourced later, today an Apache project
- Some of the companies that use Cassandra in their production clusters
  - IBM, Adobe, HP, eBay, Ericsson, Symantec
  - Twitter, Spotify
  - PBS Kids
  - Netflix: uses Cassandra to keep track of your current position in the video you are watching.

#### Let's go Inside Cassandra: key -> Server mapping

***Cassandra uses a ring-based DHT but without finger tables or routing key*** -> server mapping is the "partitioner"

- How do you decide which server(s) a key-value resides on?
  - It turns out that because key-value stores are similar to distributed hash tables, Cassandra uses the ring that we saw in distributed hash table.
  - Cassandra places servers in virtual ring. The ring here consists of two power endpoints.
  - The server is placed to the point in the ring.
  - The key get stored at servers that are successors to its point on the ring.
  - For example, key is 13 and node is 16, 32, 45, 80, 96, 112, then key is stored at 16, 32, 45(one of theses might be the primary replica and the others might be backup replicas)
  - As Cassandra is concerne, it doesn't necessarily need to have primary backup replicas, or need to know the initial primary or backup replicas, ***it just knows them as replicas***
  - ***The client sends its queries to one of the servers in the system.*** ***the server is called the coordinator.*** and the coordinator need not be on a per data center basis
  - It could be on a per client basis, on a per query basis, it doesn't really matter
  - If you have Cassandra running in two different data centers, ***each of those data centers would be using a separate ring with its own servers mapped to that ring.***
  - Cassandra doesn't have a finger table or routing key. Instead, when the client sends a query to the coordinator, ***the coordinator simply forwards a query to the appropriate replicas or just some of the replicas, and for that particular key.***
  - This means every server which could be the coordinator, ***need to know about where the keys are stored on every other server in the system.***
  - ***The mapping from key to server is called the "Partitioner"*** and that is what is used by the coordinator here to find out which are the replica servers to forward a particular query for key 13 to.

#### Data placement Strategies

- Replication Strategy : two options:
  1. SimpleStrategy
  2. NetworkTopologyStrategy
- SimpleStrategy : uses the Partitioner, of which there are two kinds 
  1. RandomPartitioner : Chord-like hash partitioning
  2. ByteOrderPatitioner : Assigns ranges of keys to servers.
     - You may want to maintain a table in Cassandra that ***preserves the ordering of the keys in there.***
     - For instance, the keys might be timestamps, and you might want to maintain the ordering of the timestamps
     - This does not hash the keys, ***instead it simply maps the key on to a point in the ring based on the value of the key itself***
     - ***So that two keys that are in order in the real space are also in the same order in the ring space itself*** -> This is useful for range queries
     - Easier for range queries(e.g., Get me all twitter users starting with[a-b])
       - ByteOrderedPartitioner by simple searching for the servers that ***will be storing that particular range of keys***
       - If you are using the hash-based partitioning instead, you'd have to ask every single server in that particular ring to see if the server has any key-value, key-value pairs that match this particular range.
- NetworkTopologyStrategy : for multi-DC(DataCenter) deployments
  - It supports one configuration ***where you can have two replicas of each key per data center***
  - If you have three data centers, then you have six replicas to each data center for each key.
  - Two replicas per DC
  - Three replicas per DC
  - How do you select replicas within each datacenter?
    - Per DataCenter
      - First replica placed according to partitioner
      - You can use a RandomPartitioner or ByteOrderedpartitioner too.
      - You want to sure that you are storing that second replica on a different rack
      - Then go clockwise around ring until you hit a different rack
        - This ensures that there is rack fault tolerance
        - It also ensures that there are two replicas in this particular case for that key for the data center

#### Snitches

- Maps: IPs to racks and Datacenters. Configured in cassandra.yaml configuration file
- Some options:
  - SimpleSnitch : Unaware of Topology(Rack-unaware)
    - Your application really cannot know much about which IP addresses matter to which racks and which data centers.
  - RackInferring : Assume topology of network by octer of server's IP address
    - Try to guess from the IP address what the rack and the data center might be.
    - 101.102.103.104 = x(first octer is ignored).\<Datacenter octer>.\<rack octer>.\<node octer>
    - PropertiesFileSnitch: uses a config file
    - EC2Snitch : uses EC2
      - EC2 region = Datacenter
      - Availability zone = rack
- Other snitch options available

#### Writes

When a client sends a write to a coordinator, the coordinator needs to forward the write to one or more replicas for that particular key that is present in the query and the write itself

- Need to be lock-free and fast(no reads or disk seeks)
  - Because We are dealing with write heavy workloads here, and they need to be fast
- Client sends write to one coordinator node in Cassandra cluster
  - Coordinator may be per-key, or per-client, or per-query
  - Per-key Coordinator ensures writes for the key are serialized
  - it is not requirement for Cassandra
- Coordinator uses Partitioner to send query to all replica nodes responsible for key
  - It then sends the query to all the replica nodes responsible for that particular key.
- When X replicas respond, coordinator returns an acknowledgement to the client
  - X? We will see later
- Always writable: Hinted Handoff mechanism
  - Hinted Handoff mechanism : ***The coordinator based on the hints that it receives from the failure information.*** it might assume the ownership of that key for just a temporary duration of time
  - If ***any replica is down,*** the coordinator writes to all other replicas, and ***keeps the write locally until down replica comes back up***
    - It is sent a copy of that write
    - This ensure that replicas when they fail and then recover, ***they receive from the coordinator some of the latest writes***
  - ***When all replicas are down,*** the Coordinator(front end)buffers writes(for up to a few hours).
    - When all the replicas might be down, instead of rejecting the write, ***the coordinator stores the write locally at itself. If buffers the writes.***
    - And then, when one or more of the replicas comesback up, it then release the writes to the corresponding replicas
    - The coordinator of course has a time on how long it stores these writes
- One ring per datacenter(datacenter가 노드로 존재하는 ring)
  - If you have multiple data centers running Cassandra, you maintain one ring per datacenter and you might also use a ***per datacenter coordinator.***
    - ***This coordinator is different from the coordinator used by client queries.***
      - The per datacenter coordinator coordinates with the other per datacenter coordinators to make sure that the datacenter to datacenter coordination is done in a correct manner(데이터 센터끼리 coordination 잘 되는지 검사 함)
  - Per-Datacenter coordinator elected to coordinate with other datacenters
  - ***The election of the per  datacenter coordinator is done by a zookeeper,*** which is a system again an open source apache system which runs a Paxos(consensus) variant
    - Paxos : elsewhere in this course

#### Writes at a Replica Node

What does a replica server do when the coordinator forwards it write? ***The first thing is logs information about the write.*** Just a little bit of information in a comment log that is present on disk. ***This is used for failure recovery.***

1. Log it in disk commit log(for failure recovery)
   - If the replica server fails at any point of time here on, it can know by looking at the disk log after it recovers that there were some write that it completed only partially and then it can ask the coordinator for information about this write
2. Make changes to appropriate memtables
   - ***Memtable = In-memory representation of multiple key-value pairs***
   - It is cached so that can be searched by key fast
   - If it is present on the memtable you update the corresponding value, if it is not present in the memtable then you simply append to the memtable another key value pair
   - Write-back cache as opposed to write-through
     -  Write-back means that you are storing it temporarily and tentatively in memory rather than writing it directly to disk
     - If you are writing it directly to disk then it would be called write-through cache, and write-through caches that are much slower than write-back caches

- ***Later, when memtable is full or old, flush to disk***
  - Data File : An SSTable(Sorted String Table) - lists of key-value pairs, sorted by key
  - Index file : An SSTable of(key, position in data stable) pairs
  - If you want to search for a particular key in the data file itself, in this data file that is the key-value pairs it might take a long time because it consists of both the keys and the values ***so you maintain an index file as well,*** which stores as SSTable of(key, position in data SSTable) pairs so that you can quickly look up the position of a particular key in the data SSTable by looking at the index file SSTable itself
  - However, in the most of the cases you might be checking for existence of a key in SSTable and the key is just not there.
    - In this case doing binary search through the index file to look for that key would just result in a very large overhead
    - If you have a large number of SSTable, each SSTable congaing M entries then you are going to incur an order log M overhead per SSTable(정렬 돼 있는 테이블 이기 때문에 binary search를 이용하여 log(m)) in trying to look for a particular key.
  - And a Bloom filter(for efficient search)
    - Bloom filter is well-known data structure which Cassnadra uses
    - Quick way of looking for whether or not a particular key is present in an SSTable

#### Bloom Filter

A bloom filter is a compact way of representing a set of items so that the most common operation which is checking for existence in that set becomes very cheap and becomes very low overhead

- The Bloom filter has some probability of false positives
- An item not in the set may be returned as true as being in the set and for the Cassandra example, this just results in a little bit of extra overhead when you look through the index file and then subsequently do not find that item
- ***But you never have false negative***

- Compare way of representing a set of items
- Checking for existence in set is cheap
- Some probability of false positives: an item not in set may check true as being in set
- Never false negatives

On insert, set all hashed bits.
***On check-if-present, return true if all hashed bits set.*** (hash set이 다 채워지면 무조건 true 나오기 때문에 안됨)

- False positive

False positive rate low

- k = 4 hash functions
- 100 items inserted
- 3200 bits bloom filter
- False Positive rate = 0.02%

#### Compaction

Over time a particular server might have a lot of SSTables it had written to disk and give key might be present in multiple SSTable.  So whenever in a system is written to disk, you don't check the other SSTable that are present on disk. So given key might be present in multiple SSTables, so you will look out for a particular key, for instance a "Read" comes in for a key you want to look up. You may need to look up multiple SSTables and this is prudentially wasteful. so in order to avoid this waste you do compactions.

***Data updates accumulate over time and SSTable and logs need to be compacted***

- You take the multiple SSTables that are present on disk and you ***merge them.*** you merge the updates for key a given key.
- ***So if a key is present into SSTables, you take the latest updae for that key and you replace the older update with that latest update***
- The process of compaction merges SSTables, i.e., by merging updates for a key
- The compaction run periodically by the server and it is run locally at each server

#### Deletes

Deletes: ***don't delete item right away***

- Instead, you "Write" to the log or th the SSTable what is know as a "Tombstone"
- ***A tombstone is a marker that says, "When you encounter this, please delete his particular key".***
- ***When the compaction algorithm runs, it encounters this tombstone and when it does so,*** it will delete the particular key value pair from the table itself.
- Add a tombstone to the log
- Eventually, when compaction encounters tombstone it will delete item

#### Reads

Read: Similar to writes, excepts. Client sends the "Read" operation to the coordinator, the coordinator can contact set of replicas. ***It doesn't necessary contact all the replicas, it constructs only X replicas***

- Coordinator can contact X replicas(e.g., in same rack)
  - Coordinator sends read to replicas that have responded quickest in past
  - ***The coordinator might prefer a replicas that have responded the quickest in the past(might be in same or near rack)***
  - When  X replicas respond, ***coordinator returns the latest-timestamped value*** from among those X
  - ***The different replicas for a given key might return different values because we didn't say anything about consistency,*** which means that some of the replicas may have received the latest 'Write" to the key and other replicas may still be working with a stale of the value for that particular key.
  - ***So the coordinator looks at the timestamp of these values*** and the highest timestamp or the latest time stamp or the latest time stamp value is returned the client
  - The coordinator also features values from the other replicas for this particular key(Why?)
    - Some of the values may be staler than others and we want to make sure that these values are updated and repaired to the latest value, ***so if any of these values are older then the coordinator initiates a "Read repair" on these older values***
    
    - ***"Read repair"*** informs these older values or rather the replica servers that "Hey here is the new and the latest value, the latest timestamps that is associated with this particular key. ***please update your corresponding memtable and eventually SSTables"***
    
      > memtable : Im memory
      >
      > SSTable : In disk?
      >
      > -> memtable이 꽉 차면 SSTable에 flush
    
      If this goes on for long enough, this mechanism will eventually bring all the replicas for a given key up to date, meaning that all of them will reflect the latest value that has been written to that particular key
  - X is the number again specified by the client.
  - (X? We will see later)
  
- Coordinator also fetches value from other replicas
  - Checks consistency in the background, ***initiating a read reapir if any two values are different***
  - This mechanism seeks to eventually bring all replicas up to date
  
- A row may be split across multiple SSTables -> reads need to touch multiple SSTables => reads slower than writes(But still fast)

- ***If compaction doesn't run often enough then a row maybe split across multiple SSTables then "Reads" need to touch multiple SSTable***

- This means that all the values or all the columns for a given row are not necessarily written in the same SSTable. Because you may have updated only one of the columns for a given row and for a given key, and only that particular column along with a key would be present in that SSTable

- So even aside apart from compaction, you might need to touch multiple SSTables you are trying to read the entire row because the lowest split across multiple SSTables

- This will leads reads is slower than write

> compaction 제대로 안일어나면 SSTable이 증가해서 read에서 병목 발생함

#### Membership

The coordinator needs to know about all the other servers present in the cluster and this is true of all the other servers as well

- Any server in cluster could be the coordinator
- ***So every server needs to maintain a list of all the other servers that are currently in the server***
- ***List needs to be updated automatically*** as servers join, leave, and fail

#### Cluster Membership - Gossip-Style heartbeating

Cassandra uses gossip-based cluster membership

Protocol(gossip-style and heartbeat):

- Nodes periodically gossip their membership list
- On receipt, the local membership list is updated, as shown
- If any heartbeat older than $T_{fail}$, node is marked as a failed

#### Suspicion Mechanisms in Cassandra

- Gossip style membership, Cassandra uses suspicion mechanisms to make sure that when servers are being directed as "Fail", ***there is a low probability that this is a mistaken detection,*** so it tries to increase the accuracy of detections. ***For this, use suspicion mechanisms*** (아직 노드 살아있는데 동작하지 않는다고 보고하는 경우를 줄이기 위해 - 거짓 부정?긍정? 줄이기 위해 사용)
  - This is different from the previous suspicion mechanisms that we have discussed in the membership lectures
  
  - It uses Accrual detector
  
    > In membership list : 만약 노드가 정지됐다고 응답이 오면, 다른 프로세스에게 다시 한번 확인 함.
- Suspicion mechanisms to adaptively set the timeout based on underlying network and failure behavior
- ***Accrual detector*** : Failure detector outputs a value(PHI) representing suspicion
- Apps set an appropriate threshold
- PHI calculation for a member
  - Inter-arrival times for gossip messages
  - It also looks at their cumulative distribution or the probability between when the last cluster was received and the time now and based on that it outputs a value(마지막으로 응답한 시간부터 현재 시간까지 누적된 시간을 고려)
  - PHI(t) = log(CDG of Probability(t_now - t_last))/log10
  - PHI basically determines the detection timeout, but takes into account historical inter=arrival time variations for gossiped heartbeats
  - This is a failure adaptive way of setting the threshold for detection on a per server basis
  - So some servers that are responding slower than others, will end up being given a slightly more laxity(부정확함) in the sense of other servers waiting for a slightly longer for hearbeats from that particular slow server and other servers that are faster, might have a shorter timeline.
  - The suspicion mechanisms are a way of setting a time was adaptively on a server by server basis in the gossip sytle membership protocol
- In practice, PHI = 5 => 10 - 15 sec detection time

#### Cassandra vs RDBMS

- MySQL is one of the most popular(and has been for a while)
- on > 50 GB data
- MYSQL
  - Wrtie 300ms avg
  - Reads 350ms avg
- Cassandra
  - Writes 0.12ms avg
  - Reads 15ms avg
- Orders of magniture faster
- What is the catch? what did we lose?

