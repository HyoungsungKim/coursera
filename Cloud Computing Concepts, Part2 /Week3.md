# Week3

## Introduction

### Overview of Last 3 Weeks

- you will see topic falling into three categories:
- Hot Topic in cloud computing
  - Our goal : Get your jump-started with this topic, so that you can explore it later yourself
  - Orthogonal but related topics
    - important to know for all cloud/distributed systems developers
    - important Classical Topics
    - These are core concepts

- How Topics in Cloud Computing
  - Stream Processing, e.g., Apache Storm
  - Distributed Graph Processing, e.g., Pregel
  - Out Goal: Get you jump-started with this topic, so that you can it later yourself
- Orthogonal but related topics
  - Structure of Networks
- Important Classical Topics
  - Scheduling

## Lesson 1: Stream Processing in Storm

Stream processing is a very old branch of distributed computing. But in the last few years, it has come into the prominence because of the new workloads that we are seeing.

### What we will cover

- Why Stream Processing
- Storm

### Stream processing Challenge

- large amounts of data => Need for real-time views of data
  - Social network trends, e.g., Twitter real-time search
  - Website statistics, e.g., Google Analytics
  - Intrusion(침범) detection systems, e.g., in most datacenters
- Process large amounts of data
  - Withe latencies of few seconds
  - With high throughput

### MapReduce?

- Batch processing => Need to wait for entire computation on large dataset to complete
  - But essentially MapReduce is a batch processing framework
  - You need to wait for an entire computation on a large dataset to complete before you can get the results.
  - There is no notion of partial results in MapReduce at all
- Even the variants of MapReduce that are tuned to stream-processing or to incremental processing, there are incremental versions of MapReduce as well, still have a fairly high latency
- Essentially MapReduce is not intended for stream-processing applications
- Not intended for long-running stream-processing

### Enter Strom

- Apache Project
- Highly active JVM project
- Multiple language supported via API
  - Pyhthon, Ruby, etc.
- Used by over 30 companies including
  - Twitter : For personalization, search
  - Flipboard : For generating custom feeds
  - Weather Channel, WebMD, etc

### Storm Components

- Tuples
- Streams
- Spouts
- Bolts
- Topologies

### Tuple

- An ordered list of elements
- e.g., <tweeter, tweet>
  - e.g., <"Alice", "hey here is my new song">
  - e,g., <"Bot", "hey here is my new song">
- e.g., <URL, clicker-IP, data, time>
  - e.g., <coursera.org, 101.201.301.401, 4/4/2014, 10:35:40>
  - e.g., <coursera.org, 901.801.701.601, 4/4/2014, 10:35:42>

### Stream

- Sequence of tuples
  - ***Potentially unbounded in number of tuples***

- Social network example:
  - <"Alice", "hey here is my new song">, <"Bot", "hey here is my new song">, ... <"Jones", "hey here is my super hit!">

***But who generate tuples?*** -> Spout

### Spout

- A Storm entity(process) that is a source of streams
- Often reads from a crawler of DB

***How do you process these streams?*** -> Bolt

### Bolt

The entity that processes a stream is known as a bolt in Storm.

- A Storm entity (process) that
  - Processes input streams
  - Outputs more streams for other bolts
- A bolt processes input streams typically just one input stream but it can process multiple input streams as well and in turn it ***generates an output stream which is then fed to other bolts***
  - spout(generate tuples) -> bolt --feed output(another tuple)--> bolt -> --feed output(another tuple)--> bolt --feed output(another tuple)--> ...
- A bolt is essentially a program that is given as input one or more streams and then decides when it gets a tuple from these multiple streams or from just one stream, whichever are its input streams.

### Topology

Bolts along with output bolts is called topology and that is essentially equivalent or corresponds to a Storm application

- A directed graph of spouts and bolts (and output bolts)
- Corresponds to a Storm "Application"
- Can have cycles if the application requires it

### Bolts come in Many Flavors

- Operations that can be performed
  - Filter : forward only tuples which satisfy a condition
  - Joins : When receiving two streams A and B, output all pairs (A, B) which satisfy a condition
  - Apply/transform : Modify each tuple according to a function
  - And many others
- But bolts need to process a lot of data
  - ***Need to make them fast***

### Parallelizing Bolts

- have multiple processes ("takes") constitute a bolt
- Incoming streams split among the tasks
- Typically each incoming tuple goes to one task in the bolt
  - Decided by ***"Grouping strategy"***
- Three types of grouping are popular

### Grouping

- Shuffle Grouping
  - Streams are distributed evenly across the bolt's tasks
  - round-robin fashion
- Fields Grouping
  - Group a stream by a subset of its fields
  - e.g., All tweets where twitter username start with [A-M, a-m, 0-4] goes to task 1, and all tweets starting with [N-Z, n-z, 5-9] go to task 2
- All grouping
  - All tasks of bolt receive all input tuples
  - Useful for joins
  - For instance if you are trying to join the incoming stream with some other stream or with an existing database then this is fairly useful of an all grouping

### Strom Cluster

How is a Storm cluster server run?

- Master node(is elected using leader election)
  - Runs a daemon called Nimbus
  - Responsible for
    - Distributing code around cluster
    - Assigning tasks to machines
    - Monitoring for failures of machines(failure detection)

- Worker node
  - Runs on a machine(server)
  - runs a daemon called Supervisor
  - Listens for work assigned to its machines

- Zookeeper
  - Coordinates Nimbus and Supervisors communication
  - All state of Supervisor and nimbus is kept here

### Failures

- A tuple is considered failed when its topology (graph) of resulting tuples fails to be fully processes within a specified timeout
- Anchoring : Anchor an output to one or more input tuples
  - Failure of one tuple causes one or more tuples to replayed

### API for Fault-Tolerance(outputCollector)

- Emit(tuple, output)
  - Emits an output tuple, perhaps anchored on an input tuple(first argument)
- ACK(tuple)
  - Acknowledge that you finish processing a tuple
- Fail(tuple)
  - Immediately fail the spout tuple at the root of tuple topology if there is an exception from the database, etc.
- Must remember to ack/fail each tuple
  - ***Each tuple consumes memory,*** Failure to do so results in memory leaks

### Summary

- Processing data in real-time a big requirement today
- Storm
  - And other sister system, e.g., Spark Streaming
- Parallelism
- Application topologies
- Fault-tolerance

## Lesson 2: Distributing Graph Processing

### What we will cover

- Distributed Graph Processing
- Google's Pregel system
  - Inspiration for many newer graph processing systems: Piccolo, Giraph, Graphlab, PowerGraph, LFGraph, X-Stream, etc.

### What is Graph?

- A graph is not a plot!
- ***A graph is a "network"***
- A graph has vertices (i.e., nodes)
  - e.g., in the Facebook graph, each user = a vertex(or a node)
- A graph has edges that connect pairs of vertices
  - e.g., in the Facebook graph, a friend relationship = an edge

### Lots of Graphs

- Large graphs are all around us
  - Internet Graph : vertices are routers/switches and edges are links
  - World wide Web : vertices are web-pages, and edges are URL links on a web-page pointing to another web-page
    - Called "Directed" graph as edges are uni-directional
  - Social Graphs : Facebook, Twitter, LinkedIn
  - Biological graphs : DNA interaction graphs, ecosystem graphs, etc.

### Graph Processing Operations

- Need to derive properties from these graphs
- Need to summarize these graphs into statistics
- e.g., find shortest paths between pairs of vertices
  - internet(for routing)
  - LinkedIn(degree of separation)
- e.g., do matching
  - Dating graphs in match.com (for better dates)
- And many (many) other examples!

### Why Hard?

- Because these graphs are large!
  - human social network has 100s Millions of vertices and Billions of edges
  - WWW has Millions of vertices and edges
- ***hard to store the entire graph on one server and process it***
  - Slow on one server(even if beefy!)
- Use distributed cluster/cloud!

### Typical Graph Processing Application

- Works in iterations

- Each vertex assigned a value

- In each iteration, each vertex :

  1. Gathers values from its immediate neighbors(vertices who join it directly with an edge). e.g., A: B->A, C->A, D->A...
  2. Does some computation using its own value and its neighbors values
  3. Updates its new value and sends it out to its neighboring vertices.
     e.g., A->B,C,D,E

- Graph processing terminates after :

  i) fixed iterations,

  ii) vertices stop changing values

### Hadoop/MapReduce to the Rescue

In order to run this in a distributed system, can we use Hadoop?

- If you load this using Hadoop, we would have a multistage Hadoop. Where you would have one MapReduce phase for each iteration. ***So each MapReduce stage would be one iteration***
- So if you had 100 iterations you would have 100 chained Hadoop or MapReduce jobs after the other.

- Multi-stage Hadoop
- Each stage == 1 graph iteration
- ***Assign vertex ids as keys in the reduce phase***
  - Well-known
  - At the end of every stage, transfer all vertices over network
    - All vertex values written to HDFS(file system)
    - ***Very slow!***

> iteration마다 reduce -> assign id -> store in HDFS : 매우 느림

Hadoop is easy to code up but it is very very slow as far as the graph concern of processing

### Bulk Synchronous Parallel Model

- "Think like a vertex"
- Where you have local competition being done at multiple processors parallelly.
- Each vertex is computed separately by a different processor. But at the end of iteration, ***all the processors meet a barrier***
- But all of them wait for all of the other processors to complete there iteration, and only then do they proceed into the next iteration.

### Basic Distributed Graph Processing

- "Think like a vertex"
- Assign each vertex to one server
- Each server thus gets a subset of vertices
- In each iteration, each server performs Gather-Apply-Scatter for all its assigned vertices
  - Gather: get all neighboring vertices' values
  - Apply: compute own new value from own old value and gathered neighbors' values
  - Scatter: send own new value to neighboring vertices

### Assigning Vertices

- How to decide which server a given vertex is assigned to?
- Different options
  - hash-based : hash(vertex id) modulo number of servers
    - Random but it is consistent
    - Remember consistent hashing from P2P systems?!
  - Locality-based : Assigned vertices with more neighbors to the same server as its neighbors
    - Reduce server to server communication volume after each iteration
    - Do not involve any network overhead

### Pregel System By Google

- Pregel uses the master/worker model
  - Theses would also be the master daemons and the worker daemons running on each of the other servers
  - Master(one server) responsible for
    - Maintain list of worker servers
    - Monitors workers; restarts them on failure
    - Provides Web-UI monitoring tool of job progress
  - Worker (rest of the servers) responsible for
    - Processes its vertices
    - Communicates with the other workers
- Persistent data is stored as files on a distributed storage system(such as GFS or BigTable)
- Temporary data is stored on local disk

### Pregel Execution

1. Many copies of the program begin executing on a cluster
2. The master assigns a partition of input (vertices) to each worker
   - Each worker loads the vertices and marks them as active
3. The master instructs each worker to perform a iteration
   - Each worker loops through its ***active vertices*** & computes for each vertex
   - Messages can be sent whenever, but need to be delivered before the end of the iteration(i.e. the barrier)
   - ***When all workers reach iteration barrier, master starts next iteration***
4. ***Computation halts when in some iteration:*** 
   - No vertices are active 
   - When no messages are in transit
5. Master instructs each worker to save its portion of the graph

What about failures?

### Fault-Tolerance In Pregel

- Checkpointing
  - Periodically, master instructs the workers ***to save state of their partitions to persistent storage***
    - e.g., Vertex values, edge values, incoming messages
- Failure detection
  - Using periodic "Ping" messages from master -> worker
- Recovery
  - The master reassigns graph partitions to the currently available workers
  - The workers all reload their partition state from most recent available checkpoint

### How fast is it?

- Shortest paths from one vertex to all vertices
  - SSSP : "Single Source Shortest path"
- On 1 Billion vertex graph(tree)
  - 50 workers : 180 seconds
  - 800 workers : 20 seconds
- 50B vertices on 800 workers : 700 seconds (~12 minutes)
- Pretty Fast!

> Blockchain network topology랑 비슷한가...?

### Summary

- Lots of (large) graphs around us
- need to process these
- MapReduce(Hadoop) not a good match
  - MapReduce is an attractive approach but it is not very fast, not efficient
- Distributed Graph Processing systems : Pregel by Google
- Many follow-up systems
  - Piccolo, Giraph : Pregel-like
  - GraphLab, PowerGraph, LFPraph, X-Stream : more advanced

## Lesson 3: Structure of Network

### What is a Network/Graph

- Has vertices (i.e., nodes)
  - e.g., in the facebook graph, each user = a vertex(or a node)
- has edges that connect pairs of vertices
  - e.g., in the Facebook graph, a friend relationship = an edge

### Complexity of Network

- Structural : human population has ~ 7B nodes, there are millions of computers on the Internet...
- Evolution : people make new friends all the time, ISP's change hands all the time...
- Diversity : some people are more popular, some friendships are more important...
- Node Complexity : Endpoints have different CPUs Windows is a complicated OS, Mobile devices...
- Emergent phenomena : simple end behavior -> leads to -> complex system-wide behavior
  - If we understand the basics of climate change, why is the weather so unpredictable

### Two Important Network Properties

1. Clustering Coefficient : CC
   - Pr(A-B edge, given an A-C edge and a C-B edge)
2. Path Length of shortest path
   - Extended Ring graph : high CC, long paths
   - Random graph : low CC, short paths
   - Small World Networks : high CC, short paths
   - “Six degrees of Kevin Bacon” : Real world is high CC and low paths(6 paths are enough)

### Small-World Network All Around

Most "natural evolved" networks are small world

- Network of actors -> six degrees of Kevin Bacon
- Network of humans -> Milgram's experiment
- Co-authorship network -> "Erdos Number"
- World Wide Web, the Internet...

Many of these networks also "grow incrementally" "Preferential" model of growth

- When adding a vertex to graph, ***connect it to existing vertex v with probability proportional*** to num_neighbors(v)

### Degree

- Degree of a vertex : number of its immediate neighbor vertices
- Degree distribution : What is the probability of a given node having k edges (neighbors, friends, ...)

- Regular graph : all nodes same degree
- Gaussian
- Node degree : k
- Random graph : Exponential e^(-k*c), c is constant
- Power law : k^(-a)
  - When k is pretty small like 1,2 or 3, this probability is very high
  - But as k increases, this probability drops off very quickly

### Small-World and Power-Law

- A lot of small world network are power law graphs
  - Internet backbone, telephone call graph, protein networks
  - WWW is a small-world graph and also a power-law graph with a = 2.1-2.4
  - Gnutella p2p system network has heavy-tailed degree distribution
- Power law networks also called scale-free
  - Gnutella has 3.4 edges per vertex, independent of scale (i.e., number of vertices)

### Small-World != Power-Law

- Not all small world networks are power law
  - e.g., co-author networks
- Not all power-alw networks are small world
  - e.g., Disconnected power-law networks

### Resilience of Small-world + Power-Law

Most nodes have small degree, but a few nodes have high degree

Attack on small world networks

- Killing a large number of randomly chosen nodes does not disconnect graph
- ***Killing a few high-degree nodes will disconnect graph***

"The Electric Grid is very vulnerable to attacks"

### Routing in Small-World/Power-Law Networks

- Build shortest-path routes between every pair of vertices
- -> Most of these routes ***will pass via few high-degree vertices in the graphs***
  - High-degree vertices are heavily overloaded
  - High-degree vertices more likely to suffer congestions or crash
- Same phenomenon in Electric power gird
- Solution may be to introduce some randomness in path selection; ***don't always use shortest path*** -> That is why probability connection

### Summary

- Network(graphs) are all ground us
  - Man-made networks like Internet, WWW, p2p
  - Natural networks like protein networks, human social network
- Yet, many of these have common characteristics
  - Small-world
  - Power-law
- useful to know this : when designing distributed systems that run on such networks
  - Can better predict how these networks might behave