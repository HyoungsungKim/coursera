# Week 2

## Lesson 1: Gossip

### 1.1 Multicast Problem

#### Multicast

- ***Node with a piece of information to be communicated to everyone***
- Distributed group of Nodes = Processes at Internet-based host
- Problem: you want to get information out to other members in your group
- There might be multiple multicast message going around at the same time, each of them from potentially, a different sender.

#### Fault-Tolerance and Scalability

> Two of the most important requirements as far as cloud computing is concerned are ***fault tolerance and scalability.***

Multicast sender uses multicast protocol

- node may crash - Because processes are failure prone
- Packet may be dropped(or might also be delayed by the underlying network)
- 1000's of nodes

#### Centralized

When UDP/TCP packets are sended

- Simplest implementation - One of the simplest ways of doing multicast is a ***centralized approach***
- Problems?
  - ***Fault tolerance.*** if the sender failed when it is halfway through its "4" loop, essentially, only half the receivers would have received the multicast.
  - Also ***increases the latency***. The average time for a receiver to receive a multicast could be as high as O(N) which is linear in the size of the group itself

#### Tree-Based

> Spanning tree : 한 노드에서 다른 노드에 이르는 경로가 오직 하나 뿐인 토폴로지
>
> - 스위치나 브리지에서 발생하는 루핑을 막아주기 위한 프로토콜
> - 스위치나 브리지 구성에섯 출발지부터 목적지까지의 경로가 두 개 이상 존재할 때 한 개의 경로만 남겨두고 나머지는 모두 끊어 두었다가 사용하던 경로에 문제가 발생하면 그 때 끊어 두었던 경로를 하나씩 살린다.

When UDP/TCP packets are send

> If you build a tree, a balanced tree with N nodes in your group, the height of the tree is O(log(N)) and ***this means that the latency for a message to reach to the nodes in the group is O(log(N))***

- e.g., IP multicast, SRM, RMTP, TRAM, TMTP
- Tree setup and maintenance
  - when load failures happen, nodes in the system might actually not receive multicast for a while
    (트리 중간에 있는 노드에서 실패 발생하면 밑에 있는 노드들은 정보 못받음)
- Problems?

#### Tree-Based Multicast Protocols

- Build a spanning tree among the processes of the multicast group
- Use spanning tree to disseminate multicasts
- Use either acknowledgements(ACKs) or negative acknowledgements(NAKs) to repair multicasts not received
- SRM(Scalable Reliable Multicast)
  - Use NAKs
  - ***But adds random delays,*** and uses exponential backoff to avoid NAK storms
- RMTP (Reliable Multicast Transport Protocol)
  - Uses ACKs
  - But ACKs only sent to designated receivers, which then re-transmit missing multicasts
- These protocols still cause an O(N) ACK/NAK overhead. So these protocols are not as scalable as they were intended.

### 1.2 The Gossip Protocol

#### A Third Approach

Multicast sender

- Periodically transmit to b random targets
- Other nodes do same(periodically transmit) after receiving Multicast

#### "Epidemic" Multicast(Or "Gossip")

- Protocol Rounds(Local clock) b random targets per round
- Infected -> Uninfected

#### Push vs Pull

Gossips don't necessarily contain messages, instead they contain queries for messages might have been received

- So that was "Push" gossip(메시지 전송)
  - Once you have a multicast message, you start gossiping about it
  - Multiple messages? Gossip a random subset of them, or recently-received ones, or higher priority ones
- There's also "Pull" gossip(메시지 요청)
  - Periodically pull a few randomly selected process for new multicast messages that you haven't received
  - get those messages
- Hybrid variant: Push-Pull
  - As the name suggests

### 1.3 Gossip Analysis

#### Properties

Claim that the simple Push protocol

- Is lightweight in large groups
- Spreads a multicast quickly
- Is highly fault-tolerant

#### Analysis

From old mathematical branch of Epidemiology

- Population of (n + 1) individuals mixing homogeneously

- Contact rate between any individual pair is $\beta$

- At any time, each individual is either uninfected(numbering x) or infected(numbering y)

- Then. $x_0 = n$ , $y_0 = 1$ and at all times $x + y = n + 1$

- Infected-uninfected contact turns latter infected, and it stays infected

- Continuous time process

- Then
  $$
  \frac{dx}{dt} = -\beta{xy} 
  $$
  with solution:
  $$
  x = \frac{n(n+1)}{x + e^{\beta (n + 1)t}}, y = \frac{n+1}{1+ne^{-\beta(n+1)t}}
  $$

> x*y is the total number of potential infected/uninfected contacts per time unit.
>
> All possible x * y contacts only a fraction, beta happened because beta is the contact rate, and for each of those contacts, one uninfected turned in to infected

- When t become extremely large, then x become 0, and y become (n + 1)

#### Epidemic Multicast (log is based on 2)

- Protocol rounds(local clock) b random targets per round(전체 노드의 수 중에 연결 된 노드(b)의 비율)
  $$
  \beta = \frac{b}{n}
  $$
  Substituting, at time t = c*log(n), the number of infected is
  $$
  y \approx (n + 1)- \frac{1}{n^{cb-2}}
  $$

- Set c, b to be small numbers independent of n
- Within c\*log(n) rounds, ***[low latency]***
  - All but $\frac{1}{n^{cb-2}}$ number of nodes receive the multicast ***[reliability]***
  - Each node has transmitted no more than c\*blog(n) gossip message ***[lightweight]***

#### Fault-Tolerance

- With failures, is it possible that the epidemic might die out quickly?
- Possible, but improbable:
  - Once a few nodes are infected, with high probability, the epidemic will not die out
  - So the analysis we saw in the previous slides is actually behavior with high probability
- Think : Why do rumors spreads so fast? Why do infectious diseases cascade quickly into epidemics? Why does a virus or worm spread rapidly?

#### Pull gossip protocol

- In all forms of gossip, it takes O(log(N)) rounds before N/2 gets the gossip
  - Why? Because that's the fastest you can spread a message - a spanning tree with fanout(degree) of constant degree has O(long(N)) total nodes
- Thereafter, pull gossip is faster than push gossip
- ***Second half of pull gossip finished in time O(log(log(N)))***

#### Topology-Aware gossip

- Network topology is hierarchical
- Random gossip target selection => core routers face O(N) load
- Fix: In subnet i, which contains $n_i$ nodes, pick gossip target in your subnet with probability (1-1/$n_i$)
- Router load = O(1)
- Dissemination time = O(log(N))

### 1.4 Gossip Implementations

#### So...

- Is this all theory and a bunch of equations?
- Or are there implementations yet?

#### NNTP Inter-server Protocol

1. Each client uploads and downloads news posts from a news server
2. Server retains news posts for a while, transmits them lazily, deletes them after a while.

#### Summary 

- Multicast is an important problem
- Tree-based multicast protocols
- When concerned about scale and fault-tolerance, gossip is an attractive solution
- Also know as epidemics
- Fast, reliable, fault-tolerant, scalable, topology-aware

## Lesson 2: Membership

### 2.1 What is Group Membership List

#### A Challenge

- You've been put in charge of a datacenter, and your manager has told you, "Oh no! We don't have any failure in our datacenter!"
- Do you believe him/her?
- What would be your first responsibility?

If you didn't believe your manager, what would be your first responsibility? well, your first responsibility would be to build a mechanism that detects failures of servers in your datacenter

- Build a failure detector
- What are some things that could go wrong if you didn't do this

#### Failure Are The Norm

- Say, the rate of failure of one machine (OS/disk/motherboard/network, etc) is once every 10 years (120 months) on average.
- When you have 120 servers in the DC, the mean time to failure(MTTF) of the next machine is 1 month
- When you have 12,000 servers in the DC, the MTTF is about once ***every 7.2 hours!***To Build a Failure Detector
- You have a few options:
  1. Hire 1000 people, each to monitor one machine in the datacenter and report to you when it fails.
  2. Write a failure detector program(distributed) that automatically detects failures and reports to your workstation

Which is more preferable and why?

#### Target Settings

- Process 'group'-based systems
  - Clouds/Datacenters
  - Replicated servers
  - Distributed databases
- Crash-stop/Fail-stop process failures

#### Group Membership Service

Membership List(Application Queries e.g., gossip, overlays, DHT's, etc.) which maintains the list of most or all of the other processes that are currently in your system and that have not yet failed(non-faulty processes)

This membership list is accessed by a variety of applications for instance application might be a gossip-based application.

***Accessing the membership list or even sampling the membership list is an important building block of many distributed algorithms*** -> membership protocols

So our goal is build the membership protocol which keeps this membership list up-to-date

> membership list : 아직 오류로 중단 안 된 것들의 리스트

#### Two Sub-Protocols

One is a mechanism that detects failures, the second is a mechanism that ***disseminates information about these failures after detection to the other processes in the system***

> Failure detection : 실패를 찾음
>
> Disseminate : 실패를 알림

Multiple processes might find out about this failure, ***but we need to ensure that at least one non-faulty process finds out about this failure.*** What it does, it can then use the dissemination. ***component to disseminate this information to the other processes quickly.***

So that they get to know about this failure and update their membership lists

> Failure detection 이후에 적어도 failure 상태가 아닌 하나의 프로세스는 이 실패에 대해서 알아야 함. 알리는 일을 하는게 disseminate임

1. Dissemination - Dissemination component can also be used to disseminate information about process joints as well as process leaves in the system
2. Failure detector

- Complete list all the time(strongly consistent)
  - Virtual synchrony
- Almost-Complete list(weakly consistent)
  - Gossip-style, SWIM, ...
- Or Partial-random list(other systems)
  - Scamp, T-MAN, Cyclon

#### Group Membership Protocol

- Multiple processes might find out about this failure, but we need to ensure that at least one non-faulty process finds out about this failure.
- What it does, it can then use the dissemination component to disseminate this information to the other processes quickly

### 2.2  Failure Detectors

#### 1. pj Crashes

- Nothing we can do about it!
- A frequent occurrence
- Common case rather than exception
- Frequency goes up linear with size of datacenter

#### 2. Distributed Failure Detectors: Desirable properties

- Completeness = each failure is detected
- Accuracy = there is no mistaken detection

Completeness & Accuracy(trade-off)

Impossible together in lossy networks. If possible, then can solve consensus

The bad news is that achieving both of these properties, completeness and accuracy, over a network which loses packets, which can delay packets quite a bit, is impossible.

***You cannot build a failure detector that is 100 percent complete, meaning, it detects all failures.***

Essentially, this build down to the fact that the failure detector problem is equivalent to another well-known problem in distributed computing known as consensus.

In real life, failure detectors always guarantee completeness 100 percent of the time. close to the 100%, but never getting exactly at 100 percent.

- Speed 
  - Time to first detection of a failure

Time until SOME process detects the failure. 

The speed basically says that between the time when a failure actually happens, and the first other non-faulty process detects this failure, ***we want that time to first detection of failure to be as small as possible.***

- Scale
  - Equal Load on each member
  - Network Message Load

No bottlenecks/single failure point

A scale is important because you want your system to scale as a number of processes and your group grows. This means that you want to have a low and equally distributed load in terms of number of message on each process or member in your group

#### Centralized Heartbeating

- Heartbeats sent periodically
- If heartbeat not received from pi(sender process) within timeout, pj(recipient process) marks pi as failed
- ***However, center(pj) fails, then there is no grantee about who detect that failure.(Hot spot problem)***

#### Ring Heartbeating

In each particular picture

- Each process sends heartbeats to it's left neighbor, as well as its right neighbor, or its anticlockwise as well as its clockwise neighbor.
- It's a sequence number that is incremented locally and then sent over and the quality of the detection is again the same.
- Still not good enough because ***when you have multiple failures, you might have some failures that go undetected.***

#### All to All Heartbeating

- Where each process pi, sends out heartbeats not to one or the other process in the system but to all the other processes in the system.
- The processes do likewise they timeout waiting for heartbeats. If there time are then they mark the corresponding process having failed

- ***Pretty good.*** 
  1. Equal load per member
  2. The protocol is complete. If pi fails, then as long as there is at least one other non-faulty process in the group, it will time-out waiting for pi's heartbeating. And it will detect pi as having failed.
- Problem : if you have one process pj that is slow and is receiving packets at longer delay than others, it might end up marking all the other or almost all the other processes as having failed, with higher probability.

### 2.3 Gossip-Style Membership

#### Gossip-Style Heartbeating

***The basic concept is that every process wants to send out its heartbeats to all the other processes in the system.*** The heartbeats are again incremented sequence numbers that are incremented locally. Processes timeout waiting for heartbeats and mark the corresponding sources as having failure.

#### Gossip-Style Failure Detection

Protocol:

- Nodes periodically gossip their membership list
- On receipt, the local membership list is updated

Periodically, each process sends over this entire table to a few of ***its neighbors selected at random.***

- If the heartbeat has not increased fro more than $T_{fail}$ seconds, the member is considered failed

- And after $T_{cleanup}$ seconds, it will delete the member from the list 

- Why two different timeouts?

  - If you delete the member right away where an entry might never go away.

  > 만약 clean up time 따로 지정 안하고 바로 제거 하면 노드에서 프로세스 종료 하더라도 다른 노드에서 다시 전파돼서 작업이 종료가 안됨

#### Analysis/Discussion

- What happens if gossip period $T_{gossip}$ is decreased?

- A single heartbeat takes O(log(N)) time to propagate.

  So: N heartbeats take :

  - O(long(N)) time to propagate, if bandwidth allwoed per node is allowed to be O(N)
  - O(Nlog(N)) time to propagate, if bandwidth allowed per node is only O(1)
  - What about O(k) bandwidth? (N/k * log(N) ?)

- What happens to $P_{mistake}$(false positive rate) as $T_{fail}$, $T_{cleanup}$ is increased?

- Trade-off: False positive rate vs. detection time vs. bandwidth

### 2.4 Which is the best failure detector?

#### Failure Detector Properties...

What does optimal mean?

- Completeness - guarantee always
- Accuracy - Probability PM(T) (PM(T) : basically probability of mistake in time T)
- Speed - T time units
  - Time to first detection of a failure
- Scale
  - Equal Load on each member
  - Network Message Load - N * L compare this across protocols

> N : The number of Nodes
>
> T : Time unit
>
> L(load) : N/T

- Load per process is linear in N
- Every tg units = gossip period, send  O(N) gossip message
- T = log(N) * tg
- L = N/(tg) = Nlog(N)/T
- Essentially, the load on each member per gossip period is O(N) because it sends out N entries from its membership lost, so the load is N/(tg)
- You will notice here automatically that the gossip-based heartbeating protocol here has a higher load that the all-to-all heartbeating we discussed in the previous slide
- That is natural because ***gossip is trying to get a slightly better accuracy by using more message***

#### What is the Best/Optimal we can do?

- Worst case load L*(per member) as a function of T, PM(T), N independent Message Loss probability $P_{ml}$

> $P_{ml}$ : Probability of losing a message

$$
L^* = \frac{log(PM(T))}{log(P_{ml})} \frac{1}{T}
$$

- ***Optimal L is independent of N***
- All-to-all and gossip-based: sub-optimal
  - L = O(N/T)
  - Try to achieve simultaneous detection at all processes
  - Fail to distinguish Failure Detection and Dissemination components

> Key:
>
> - Separate the two components
> - Use a non heartbeat-based failure Detection Component

### 2.5 Another Probability Failure Detector

#### Swim Failure Detector Protocol

SWIM(Scalable Weekly consistent Infection style Membership protocol)

>프로세서 A가 B에게 req 보내지만 응답이 없으면 무작위의 프로세서에게 req보내고 req에 응답한 프로세서가 B에게 req 보낸뒤 응답을 A에게 보냄(Second Chance)

#### SWIM VS. Heartbeating

For heart beating,

- When you have a very low process load, another words, when you have a low bound on the bandwidth that can be used, the detection time can be very high.
- If it is constant a load, that is your constraint, the detection time could be as high as order N, On the other hand, when process load is constant, then heartbeating is O(N)

SWIM

- Get both a constant detection time on expectation as well as a constant process load.

#### SWIM Failure Detector

First Detection Time

- Expected $\frac{e}{e-1}$
- Constant(independent of group size)

Process Load

- Constant per period - Because each process is sending out, one ping maybe another K indirect ping message and our expectation is receiving, direct ping and our expectation a few indirect ping message
- <8L*for 15% loss : less than 8 times the optimal load, when you have 15% packet loss.

False Positive Rate

- Tunable(via increasing K - K : The number of random processes)
- Falls exponentially as load is scaled

Completeness

When a process fails, It will eventually be selected for pinging as long as there are any further processes with this failed process in their membership list, this is the nice property about picking, ping, target at random.

- Deterministic time-bounded
- Within O(log(N)) periods w.h.p(With high probability) - Because you have expected $e/(e-1)$ protocol periods until detection, you can show that within log in protocol periods or order log in protocol periods with high probability at least one process will ping the failed process and will mark it as having failed

#### Detection Time

- Prob of being pinged in $T' = 1 - (1 - \frac{1}{N})^{N-1} = 1 - e^{-1}$ 
- E[t] = $T' \frac{e}{e-1}$
- Completeness : Any alive member detects failure
  - Eventually
  - By using a tricks: within worst case O(N) protocol periods

#### Time-Bounded Completeness

> Whenever you pick a membership element pick the next membership element in your list in the liner fashions so essentially you traverse the membership list that you have on per around

You can also reduce this eventual completeness to a time order completeness which is order N rounds in the worst case by using a simple trick

- Key : Select each membership element once as a ping target in a traversal
  - Round robin pinging
  - Random permutation of list after each traversal
- Each failure is detected in worst case 2N-1(local) protocol periods
- Preserves FD(Failure Detection) properties

### 2.6 Dissemination and suspicion

#### Dissemination

failure 발견하면 다른 노드들에게 알림

#### Dissemination options

- Multicast(hardware/ IP)
  - unreliable
  - multiple simultaneous multicast
- Point-to-point(TCP/UDP)
  - expensive
- Zero extra message(SWIM style failure detection) : ***Piggyback on Failure Detector messages***
  - Infection-style Dissemination

Piggyback: Whenever processes receive these message, they do the same as they did before but in addition they also look at what is being piggybacked and use that to update their membership lists.(ack와 데이터를 같이 송신)

#### Infection-Style Dissemination

- Epidemic style dissemination
- Maintain a buffer of recently joined/evicted processes
  - piggyback from this buffer
  - Prefer recent updates
- Buffer elements are garbage collected after a while

#### Suspicion Mechanism

- False detection, due to:
  - Perturbed(교란된) processes
  - packet losses, e.g., from congestion
- Indirect pinging may not solve the problem
  - e.g., corrected message losses near pinged host
- Key: suspect a process(ask again to other process) before declaring it as failed in the group

In other words, you use a state machine where pi maintains a state machine for a given other process, pj which is present in its membership list. In addition, you start disseminating via the SWIM piggyback message the fact that you are suspecting process pj.

When you are in the suspected state for process pj, you might receive an ack for process pj or maybe even another direct ping from process pj, or even a message from someone else in the group saying "Hey ,process pj has not failed, but in fact it is alive" In that case you move the state back again to alive for process pj

- Distinguish multiple suspicions of a process - Typically pi does this when it receives a suspect pi message, meaning that someone else in the group is suspecting process pi as having failed.
  - Per-process incarnation number
  - Inc # for pi can be incremented only by pi
    - e.g., when it receives a (suspect, pi) message
  - Somewhat similar to DSDV
- Higher inc # notifications over-ride lower inc#'s
- Within an inc#: (Suspect inc #) > (Alive, inc #)
- (Failed, inc #) overrides everything else

#### Wrap up

- Failures the norm, not the exception in datacenters
- Every distributed system uses a failure detector
- many distributed systems use a membership service
- Ring failure detection underlies
  - IBM SP2 and many other similar cluster/machines
- Gossip-style failure detection underlies

## Lesson 3: Grids

### 3.1 Grid Application

#### Example: Rapid Atmospheric Modeling System(RAMS), Colorado State Univ

- Can one run such a program without access to a supercomputer?

### 3.2 Grid Infrastructure

#### Scheduling Problem

How to schedule such an application which consists of essentially a DAG of jobs? - Each job is highly parallelizalbe into tasks over here so each of these nodes can be split up into multiple parallel tasks.

#### 2-Level Scheduling Infrastructure(Inter-site <-> intra-site)

Essentially what happens in that Globus decides which job gets scheduled at which site and the intra-site protocol at that site decides how to schedule the different tasks of that job at the different machines in that particular site

#### Intra-site Protocol

- Internal allocation & Scheduling monitoring distributing and publishing of Files

Intra-site protocol runs inside each site is responsible for internal allocation and scheduling so it if is given job 0 and job 3 to run, it then decides which of the tasks of job 3 run on which machines. again job 0 is same as well.

It is responsible for monitoring. So if any of these machines which are running a task fail or crash the ***HT Condor protocol is then responsible for restarting tasks on another machine.***

#### Condor(Now HTCondor) - Example of Intra-site protocol

Globus : inter-site protocol

- High-throughput computing system from Univ Wisconsin Madison
- Belongs to a class of Cycle-scavenging systems such systems
  - Run on a lot of workstations
  - When workstation is free, ***ask site's central server(or Globus) for tasks***
  - If user hits a keystrokes or mouse click, stop task
    - Either kill tasks or ask server to reschedule task
  - Can also run on dedicated machines

> 놀고 있을때 중앙 site에 작업 요청함. 만약에 유저가 워크스테이션에서 작업 시작하면 하던 일을 중단 하고 작업 rescheduling을 요청 함.

#### Inter-site Protocol

The internal structure of the sites is typically invisible to Globus. So this is known as transparency and transparency essentially means invisibility. So Globus does not necessarily need to know about what Condor does inside the Wisconsin site or what MIT's intra-site protocol does inside the MIT site.

> inter-site protocol은 intra-site protocol에서 하는 걸 알 필요 없음

So ***Globus protocol is responsible for external allocation.*** It is responsible for the scheduling to the extent that it talks with the schedulers at the individual sites, but Globus doesn't necessarily do much of scheduling it self.

#### Globus

- Globus Alliance involves universities, national US research labs, and some companies
- Standardized several things, especially software tools
- Separately, but related: Open Grid Forum
- Globus Alliance has developed the Globus Toolkit which is one of the standard way to run the inter-site protocol

#### Globus toolkit

- Open source
- Consist of several components
  - GridFTP : Wide area transfer of bulk data
  - GRAM5(Grid Resource Allocation Manager): submit, locate, cancel, and manage jobs
    - Not a scheduler***(It is a job allocator and manager but it is not a scheduler)***
    - Globus communicates with the schedulers in intra-site protocol like HTCondor or Portable Batch System(PBS)
  - RLS(Replica Location Service): Naming service that translates from a file/dir name to a target location(or another file/dir name)
  - Libriaries like XIO to provide a standard API for all Grid IO functionalities
  - Grid Security Infrastructure

#### Security Issues

Security is important in grids. because grids are essentially federated. ***There is no central authority that manages and owns the entire grid***

> federate : 연합하다.

- Important in Grids because they are federated, i.e., not single entity controls the entire infrastructure

- Single sign-on: collective job set should require once-only user authentication
- Mapping to local security mechanisms: some sites use Kerberos, other using Unix
- Delegation: credentials to accesses resource inherited by subcomputations, e.g., job 0 to job 1
- Community authorization: e.g., third-party authentication
- These are also important in clouds, but less so because clouds are typically run under a central control
- In clouds the focus is on failures, scale, on-demand nature

#### Summary

There is an open question out there which is whether grids and in general high performance computing are in fact, converging towards clouds computing and toward data intensive computing and this is an open question

There is a lot of commonalities among these two different areas but they tend to be disjoint areas in terms of what the software and the standards that they develop as well as the conferences where research get published

So one of the things that i would encourage you to do is compare the architecture of Globus with the architecture of Openstack. OpenStack is one of these cloud computing open source platforms that has emerged so that you can draw your virtualized cloud computing platform

- Grid computing focuses on computation-intensive computing(HPC)
- Though often federated, architecture and key concept have a lot in common with that of clouds
- Are Grids/HPC converging towards clouds?
  - e.g., Compare OpenStack and Globus

