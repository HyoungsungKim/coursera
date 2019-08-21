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
  - 노드 degree 높은 노드의 개수는 적고, 노드 degree 낮은 노드는 매우 많음

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
- Not all power-law networks are small world
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

## 4.1 Single-processor Scheduling

### Why Scheduling?

- Multiple "tasks" to schedule
  - The processes on a single-core OS
  - The tasks of a Hadoop job
  - The tasks of multiple Hadoop jobs
- Limited resources that these tasks require
  - processor(s)
  - memory
  - (Less contentious) disk, network
- Scheduling goals
  - Good throughput or response time for tasks(or jobs)
  - High utilization of resources

### FIFO(FCFS) performance

- Average ***completion time may be high***
- For example, length 10, 5, 3 tasks come then completion time is 10, 15, 18
  - Average completion time is 14.33

### STF Scheduling (Shortest Task First or SJF - Shortest Job First)

- Average completion of STF is the shortest among all scheduling approaches!
- For example, length 10, 5, 3 tasks come then completion time is 3, 8, 18
  - Average completion time is 9.66
- In general, STF is a special case of priority scheduling
  - Instead of using time as priority, scheduling could use user-provided priority

### Round-Robin Scheduling

- Round-robin gives a fair share of the resource, the processor, to all the tasks that are present in the system.

### Round-Robin vs STF/FIFO

- Round-Robin preferable for
  - Interactive applications
  - User needs quickly response from system
- FIFO/STF preferable for Batch applications
  - User submit jobs, goes away, comes back to get result

### Summary

- Single processor scheduling algorithms
  - FIFO/FCFS
  - Shortest task first(optimal!)
  - Priority
  - Round-robin
  - Many other scheduling algorithms out there!
- What about cloud scheduling?
  - Next!

## 4.2 Hadoop Scheduling

### Hadoop Scheduling

- A Hadoop job consists of Map tasks and Reduce tasks
- Only one job in entire cluster => it occupies cluster
- Multiple customers with multiple jobs
  - User/jobs = "tenants"
  - Multi-tenant system
- Need a way to schedule all these jobs (and their constituent tasks)
- Need to be fair across the different tenants
- Hadoop YARN has two popular schedulers
  - Hadoop Capacity Scheduler
  - Hadoop Fair Scheduler

### Hadoop Capacity Scheduler

- Contains multiple queues
- Each queue contains multiple jobs
- Each queue guaranteed some portion of the cluster capacity
  - e.g
  - Queue 1 is given 80% of cluster
  - Queue 2 is given 20% of cluster
  - Higher-priority jobs go to Queue 1
- For job within same queue, FIFO typically used
- Administrators can configure queues

### Elasticity in HCS

- Administrators can configure each queue with limits
  - Soft limit: how much % of cluster is the queue guaranteed to occupy
  - (optional) hard limit : max % of cluster the queue is guaranteed
- Elasticity
  - A queue allowed to occupy more of cluster if resources free
  - But if other queues below their capacity limit, now get full, need to give these other queues resources
- Pre-emption now allowed!
  - ***Cannot stop a task part-way through***
  - When reducing % cluster to a queue wait until some tasks of that queue have finished

### Other HCS Features

- Queues can be hierarchical
  - may contain sub-queues, which may contain child sub-queues, and so on
  - Child sub-queues can share resources equally
- Scheduling can take memory requirements into account(memory specified by user)

### Hadoop Fair Scheduler

- ***Goal : all jobs get equal share of resources***
- When only one job present, occupies entire cluster
- As other jobs arrive, each job given equal % of cluster
  - e.g., Each job might be given equal number of cluster-wide YARN containers
  - Each container == 1 task of job
- Divides cluster into pools
  - Typically one pool per user
- resources divide equally among pools
  - Gives each user fair share of cluster
- Within each pool, can use either
  - Fair share scheduling, or
  - FIFO/FCFS
  - (Configurable)

### Pre-emption in HFS

- Some pools may have minimum shares
  - Minimum % of cluster that pool is guaranteed
- When minimum share not met in a pool, for a while
  - Take resources away from other pools
  - By pre-empting jobs in those other pools
  - By killing the currently-running tasks of those jobs
    - tasks can be re-started later
    - Ok since tasks are idempotent!
  - To kill, scheduler picks most-recently-started tasks
    - Minimizes wasted work 

### Other HFS Features

- Can also set limits on
  - Number of concurrent jobs per user
  - Number of concurrent jobs per pool
  - Number of concurrent tasks per pool
- Prevents cluster from being hogged by one user/job

### Estimating Task Lengths

- HCS/HFS use FIFO
  - May not be optimal(as we know!)
  - Why not use shortest-task-first instead? it is optimal (as we know)
    - FIFO is not a optimal way
- ***Challenge : Hard to know expected running time of task(before it is completed)***
- ***Solution : Estimate length of task***
- Some approaches
  - Within a job : Calculate running time of task as proportional to size of its input
  - Across tasks : Calculate running time of task in a given job as average of other tasks in that given job(weighted by input size)
- Lots of recent research in this area!

### Summary

- Hadoop Scheduling in YARN
  - Hadoop Capacity Scheduler
  - Hadoop Fair Scheduler
- Yet, so far we've talked of only one kind of resource
  - Either processor, or memory
  - How about multi-resource requirements?
  - Next!

## 4.3 Dominant-Resource Fair Scheduling

### Challenge

- What about scheduling VMs in a cloud (cluster)?
- Jobs may have multi-resource requirements
  - Job 1's task : 2 CPUs, 8 GB
  - Job 2's task : 6 CPUs, 2 GB
- How do you scheduling these jobs in a 'fair" manner?
- That is, how many tasks of each job do you allow the system to run concurrently
- What does fairness even mean?

### Dominant Resource Fairness (DRF)

- Propose notion of fairness across jobs with multi-resource requirements
- They showed that DRF is
  - Fair for multi-tenant systems
  - Strategy-proof : tenant can't benefit by lying
  - Envy-free : tenant can't envy another tenant's allocations

### Where is DRF Useful?

- DRF is
  - Usable in scheduling VMs in a cluster
  - Usable in scheduling Hadoop in a cluster
- DRF used in Mesos, an OS intended for cloud environments
- DRF-like strategies also used some cloud computing company's distributed OS's

### How DRF works

- Our example

  - Job 1's task : 2 CPUs, 8 GB
    => Job 1's resource vector = <2 CPUs, 8 GB>

  - Job 2's task : 6 CPUs, 2 GB

    => Job 2's resource vector = <6 CPUs, 2 GB>

- Consider a cloud with <18 CPUs, 36 GB RAM>

- Each Job 1's task consumes % of total CPUs = 2/18 = 1/9

- Each Job 1's task consumes % of total RAM = 8/36 = 2/9

- 1/9 < 2/9

  - => Job 1's dominant resource is RAM, i.e., Job 1 is more memory-intensive than it is CPU-intensive

- Each Job 2's task consumes % of total CPUs = 6/18 = 6/18

- Each Job 1's task consumes % of total RAM = 2/36 = 1/18

- 6/18 > 1/18

  - => Job 2's dominant resource is CPU, i.e., Job 1 is more CPU-intensive than it is memory-intensive

### DRF Fairness

- For a given job, the % of its dominant resource type that it gets cluster-wide, is the same for all jobs
  - Job 1's % of RAM = Job 2's % of CPU
  - Because Job 1 is RAM-intensive, Job 2 is CPU-intensive
- Can be written as linear equations, and solved

### DRF Solution, For our Example

- DRF Ensures

  - Job 1's % of RAM = Job 2's % of CPU

- Solution for our example : 

  - Job 1 gets 3 tasks each with <2 CPUs, 8 GB>
  - Job 2 gets 2 tasks each with <6 CPUs, 2 GB>

- Job 1's % of RAM

  = Number of tasks * RAM per task / Total cluster RAM

  = 3 * 8 / 36 = 2/3

- Job 2's % of CPU

  = Number of tasks * CPU per task / Total cluster CPU is

  = 2 * 6 / 18 = 2/3

***So every job gets the same percentage of the cluster as far as its dominant resource is concerned***

### Other DRF Details

- DRF generalizes to multiple jobs
- DRF also generalizes to more than 2 resource types
  - CPU, RAM, Network, Disk, etc
- DRF ensures that each job gets a fair share of that type of resource which the job desires the most
  - Hence fairness
  - That is why DRF is fair across different jobs with multi-resource requirement

### Summary : Scheduling

- Scheduling very important problem in cloud computing
  - limited resources, lots of jobs requiring access to these jobs
- Single-processor scheduling
  - FIFO/FCFS, STF, Priority Round-Robin
- Hadoop scheduling
  - Capacity scheduler, Fair scheduler
- Dominant-Resources Fairness