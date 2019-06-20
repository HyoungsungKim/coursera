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

## Lesson 2: Clouds are Distributed System

### A Cloud... is a Distributed System

A cloud consist of

- Hundreds to thousands of machines in a datacenter(server side)
- Thousands to millions of machines accessing these services(client side)

Servers communicate amongst one another -> Distributed System

- Essentially cluster(Collection of machine)

Clients communicate with servers

- Also a distributed system

Clients also communicate with each other

- In peer-to-peer systems like BitTorrent
- Also a distributed system

#### Four Features of Clouds = All Distributed System Features

1. Massive Scale: many server
2. On-demand nature
   - Access(multiple) servers anywhere
3. Data-Intensive nature
   - lots of data -> need a cluster(multiple machines) to store
4. New Cloud Programming Paradigms 
   - Hadoop/MapReduce, NoSQL all need clusters

#### Cloud = A Fancy word for a distributed system

A cloud is the latest nickname for a distributed system

Previous nicknames for "Distributed system" have included

- peer-to-peer systems
- Grids -> CPU, RAM, HDD등을 공유하여 컴퓨팅 능력을 향상시키기 위해 사용되는 병렬 분산 시스템
- Clusters
- Timeshared computers(Data Processing Industry)

### What is a distributed system

#### Example of Distributed Systems

- Client communicating with a server
- BitTorrent(peer to peer overlay)
- The internet
- The Web(servers and clients)
- Hadoop
- Datacenters

#### Example of NOT Distributed Systems

- Humans interacting with each other
- A standalone machine not connected to the network, and with only one process running on it

Distributed System:Definition from textbooks

- A distributed system is a collection of independent computers that appear to the users of the systems as a single computer
- A distributed system is several computers doing something together. Thus, a distributed system has three primary characteristics:
  - Multiple computers
  - Interconnections
  - Shared state

***But it is not enough to us.*** Because We are focusing on

- algorithmics
- design and implementation
- maintenance
- study

#### A Working Definition of "Distributed System"

A distributed system is a collection of entities, each of which is autonomous, programmable, asynchronous and failure-prone, and which communicate through an unreliable communication medium.

- Entity = a process on a device(PC, PDA, mote)
- communication Medium = Wired or wireless network
- Asynchronous -> ***Distinguishes distributed systems from parallel system(e.g. multiprocessor systems)***

Parallel systems include multiprocessor systems and super computers. But essentially, very large numbers of processors ***share the same motherboard.***

Distributed system has asynchrony, which means that clocks are unsynchronized.

Distributed system is consisted of many processors

## Lesson 3: MapReduce

### 3.1 MapReduce paradigm

Terms are borrowed from Functional Language(e.g., LISP)

#### Map

- Process individual records to generate intermediate key/value pairs.

#### Reduce

- Reduce processes and merges all intermediate values associated per key  
- Each key assigned to one Reduce
- Parallelly Processes and merges all intermediate values by partitioning keys
- Popular: hash partitioning, i.e., key is assigned to reduce # = hash(key)%number of reduce servers

One way of partitioning is called hash partitioning. You take the key, you hash it by using a consistent hash function, say, simple hash algorithm one one, or message I've used five MD five, or just any other hash function, as long as it's the same hash function applied to all keys.

***Why do we use hashing?*** This leads to fail uniform load balancing of keys across the reduced tasks in the system.

### 3.2 MapReduce Example

#### Some Application of Mapreduce

Distributed Grep:

- Input : large set of files
- Output : lines that match pattern
- Map : Emits a line if it matches the supplied pattern
- Reduce : Copies the intermediate data to output

Reverse Web-Link Graph

- input : Web Graph : tuples(a,b) where(page a -> page b); a and b is URL
- output : For each page, list of pages that link to it
- map : process web log and for each input<source, target>, it outputs <target, source> (reverse inputs)
- reduce : emits <target, list(source)>

Count of URL access frequency

- input : Log of accessed URLs, e.g., from proxy server
- Output : For each URL, % of total accesses for that URL
- Map - Process web log and outputs<URL, 1>
- Multiple Reducers - Emits<URL, URL_count>
- Chain another MapReduce job after above one
- Map - Processes<URL, URL_count> and outputs <1, (<URL, URL_count> )>
- 1 Reducer - Sums up URL_count's to calculate overall_count.
- Emits multiple <URL, URL_count/overall_count>

sort

Map task's output is sorted(already!)(e.g., quicksort)

Reduce task's input is sorted(e.g, mergesort)

- input : series of (key, value) pairs
- Output: Sorted < value >s
- Map - <key, value> -> <value, _> (identity)
- reducer - <key, value> -> <key, value> (identity)
- identity function does not do anything
- partitioning function - partition keys across reducers based on ranges
  - take data distribution into account to balance reducer tasks

### 3.3 MapReduce Scheduling

#### Programming mapReduce

Externally : For user

1. Write a Map program (short). Write a Reduce program (short)
2. Submit job: wait for result
3. Need to know nothing about parallel/distributed programming!

Internally: For the Paradigm and scheduler

1. Parallelize Map
2. Transfer data from map to Reduce
3. Parallelize Reduce
4. Implement Storage for map input, Map output, Reduce input, and Reduce output

(Ensure that no Reduce starts before all maps are finished. That is, ensure the `barrier` between the map phase and Reduce phase)

For the cloud:

1. parallelize Map:easy! each map task is independent of the other!

   - All Map output records with same key assigned to same Reduce

2. Transfer data from map to reduce:

   - All map output records with same key assigned to same Reduce task
   - use partitioning function, e.g., hash(key)%number of reducers

3. Parallelized Reduce: easy! Each reduce task is independent of the other!

4. Implement Storage for Map input, Map output, Reduce input, and Reduce output

   - Map input : from distributed file system
   - Map output : to local disk(at map node); uses local file system
   - Reduce input : from (multiple) remote disks; uses local file systems
   - Reduce output: to distributed file system

   local file system = Linux FS, etc.

   distributed file system = GFS(Google File System), HDFS(Hadoop Distributed File System)

#### The Yarn Scheduler

- Used in Hadoop 2.x +
- YARN = Yet Another Resource Negotiator
- Treats each server as a collection of containers
  - Container = some CPU + some memory
- Has 3 main components
  - Global Resource Manager(RM)
    - Scheduling
  - Per-server Node manager(NM)
    - Daemon and server-specific functions
  - Per-application(job) Application Master(AM)
    - Container negotiation with RM and NMs
    - Detecting task failures of that job

Node A(node manager + application master) <-> resource manager <-> Node B(node manager + application master)

> Resource manager가 정보 교환 해줌(node B NM -> RM -> node A AM -> node B NM) 교환된 정보 바탕으로 node A의 AM이 node B의 NM에 작업 시작 요청 하는 방식

### 3.4 MapReduce Fault-Tolerance

#### Fault Tolerance

Server Failure

- NM heartbeats to RM
  - If server fails, RM lets all affected AMs know, and AMs take action
- NM keeps track of each task running at its server
  - If task fails while in-progress, mark the task as idle and restart it
- AM heartbeats to RM
  - On failure, RM restarts AM, which then syncs up with it running tasks

RM Failure

- Use old checkpoints and bring up secondary Rm

Heartbeats also used to piggyback container requests(piggyback : 응답이 없을때 요청을 다시 하고 응답 기다림)

- Avoids extra messages

#### Slow servers

Stragglers(slow nodes)

- The slowest machine slows the entire job down(why?)
- Due to bad disks, Network bandwidth, CPU, or Memory
- Keep track of :Progress: of each task(% done)(진행 중인 작업이 아직 종료 안돼서)
- Perform backup(replicated) execution of straggler task: task considered done when first replica complete. called ***Speculative Execution*** (작업이 안끝나면 복사본을 만들어서 먼저 끝나고 나오는 결과를 사용 함)

#### Locality

- since cloud has hierachical topology(e.g., racks)
- GFS/HDFS sotres 3 replicas of each of chunks(e.g., 64MB in size)
  - maybe on different racks, e.g., 2 on a rack, 1 on a different rack
- MapReduce attempts to schedule a map task on
  - A machine that contains a replica of corresponding input data, or failing that,
  - on the same rack as a machine containing the input, or failing that,
  - Anywhere

### MapReduce:Summary

- MapReduce uses parallelization + aggregation to schedule applications across clusters
- Need to deal with failure
- Plenty of ongoing research work in scheduling and fault-tolerance for MapReduce and Hadoop