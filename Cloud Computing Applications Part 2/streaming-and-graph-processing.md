# Streaming and Graph Processing

## 3.1.1 Streaming introduction

### Why Real-time stream processing?

- Real-time data processing at massive scale is becoming a requirement for businesses
  - Real-time search, high frequency trading, social network
  - have a stream of events that flow into the system at a given data rate
- The processing system must keep up with the event rate or degrade gracefully by eliminating events. This is typically called load shedding(나누기)
- MapReduce, hadoop, etc., store and process data at scale, but not for real-time systems
- There is no hack that will turn Hadoop into a real-time streaming system
  - Fundamentally different set of requirements than batch processing
- Lack of a "Hadoop of real-time" has become the biggest hole in the data processing ecosystem

## 4.1.1 Graph Processing

What we mean by graph processing? 

- Lots of different ways to describe it, but essentially you are looking at some storage system and processing system.
- You can have a graph database with all the information in.

### Graph Processing

- A graph database is any storage system that provides index-free adjacency. Has pointers to adjacent elements...
- Nodes represent entities (people, businesses, accounts...)
  - There are all sort of different formulations the difficulty with it that the database that represent graphs tend to be very very large and very sparse and they are kind of difficult to process in straightforward, vectorized way. So new schemes need to be developed for them
- Properties are pertinent(적절한) information that relate to nodes
- Edges interconnect nodes to nodes or nodes to properties and they represent the relationship between the two

### Graph and Relational Databases

***Scaling is one of the big issues***

- Graph Database
  - Associative data sets
  - Structure of object-oriented applications
  - Do not require join operators
- Relational Database
  - Perform same operation on large numbers of data elements
  - Use relational model of data
  - Entity type has own table
    - Rows are instances of entity
    - Columns represent values attributed to that instance
  - Rows in one table can be related to rows in another table via unique key per row

### Graph Computing

- Think like a vertex
- Two basic operations:
  - Fusion : aggregate information from neighbors to a set of entities
  - Diffusion : propagate information from a vertex to neighbors

### Graph Processing

- Graph computations involve local data (small part of graph surrounding a vertex), and the connectivity between vertices is sparse.
- The data may not all fit into one node. This makes it difficult to fit always into the map/reduce model
  - Graph processing deserves it is own sort of way of handling some of this data