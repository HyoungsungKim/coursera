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