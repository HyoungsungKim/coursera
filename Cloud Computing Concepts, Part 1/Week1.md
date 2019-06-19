# Week 1 Overview:Introduction to Clouds, MapReduce

## Lesson 1:Introduction to Clouds

### Two categories of clouds

- Public cloud

- Private clouds

## 1.2 What is Cloud?

There is no single definition for a cloud itself. It is often taken to mean not just compute and storage resources, but also include the network, and so on, and so forth. 

***Working definition : Lots of storage + compute cycles nearby***

A single-site cloud(aka "Datacenter") consists of

- Compute nodes
- Switches, connecting the racks
- A network topology(위상 수학)
- Storage(back-end)nodes connected to the network
- Front-end for submitting jobs and receiving client requests
- Software services

A geographically distributed cloud consists of

> 데이터 센터를 여러 자역에 지음

- Multiple such sites
- Each site perhaps with a different structure and services

core switch <-> top of rack switch <-> servers

## 1.4 What is New in Today's Clouds

There are four major characteristics that distinguish today's clouds and the problems that they raise that are different from previous generations of distributed computing systems and problems

1. Massive scale
2. On-demand access : pay-as-you-go, no upfront commitment
3. Data-intensive Nature: What was MBs has now become TBs, PBs and XBs
4. New Cloud Programming Paradigms: MapReduce/Hadoop, NoSQL/Cassandra/MongoDB and many others(key/value store)

## 1.5 New Aspects of Clouds

### 2. On-Demand Access: *aaS Classification

***On-demand: renting a cab vs (previously)renting a car, or buying one. e.g***

- AWS Elastic computer cloud(EC2): a few cents to a few $ per CPU hour
- AWS Simple Storage Service(S3): a few cents to a few $ per GB-month

HaaS: Hardware as a service

- you get access to barebones hardware machines, do whatever you want with them, e.g: your own cluster 
- Not always a good idea because of security risks

IaaS: Infrastructure as a service

- You get access to flexible computing and storage infrastructure. Visualization is one way of achieving this(what's another way, e.g using Linux). Often said to subsume HaaS.
- e.g : Amazon Web Service(AWS:EC2 and S3), Eucalyptus, Rightscale, Microsoft Azure

PaaS : Platform as a Service(Easier than IaaS but less flexible)

- You get access to flexible computing and storage infrastructure, coupled with a software platform(often tightly)
- e.g. google's AppEngine(python, Java, Go)

SaaS : Software as a Service

- You get access to software services, when you need them. Often said to subsume SOA(Service oriented Architectures).
- e.g. Google docs, MS Office on demand

#### 3. Data-Intensive computing

Computation-Intensive Computing

- Example areas : MPI-based, High-performance computing, Grids
- Typically run on spuercomputers

Data-Intensive

- Typically store data at datacenters
- Use compute nodes nearby
- Compute nodes run computation services

***In data-intensive computing; the focus shifts from computation to the data:***CPU utilization no longer the most important resource metric, instead I/O is (disk and/or network)

> Data-intensive computing에서 cpu활용보다 I/O성능이 더 주된 관심사

#### 4. New Cloud programming paradigms

Easy to write and highly parallel programs in new cloud programming paradigms:

- Google: MapReduce and Sawzall
- Amazon: Elastic mapReduce service
- Google(MapReduce)
  - Indexing: a chain of 24 MapReduce jobs
  - ~200K jobs processing 50 PB/month
- Yahoo!(hadoop + Pig)
  - WebMap:a chain of 100 MapReduce jobs
  - 280 TB of data, 2500 nodes, 73 hours
- Facebook(Hadoop + Hive)
  - ~300 TB total, adding 2 TB/day
  - 3K jobs processing 55 TB/day
- Similar numbers from other companies
- NoSQL(key/value): MySQL is  an industry standard, but Cassandra is 2400 times faster!

