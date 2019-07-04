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

  