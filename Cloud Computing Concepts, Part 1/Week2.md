# Week 2

## Lesson 1: Gossip

### 1.1 Multicast Problem

#### Multicast

- Node with a piece of information to be communicated to everyone
- Distributed group of Nodes = Processes at internet-based host
- Problem: you want to get information out to other members in your group
- There might be multiple multicast message going around at the same time, each of them from potentially, a different sender.

#### Fault-Tolerance and Scalability

> Two of the most important requirements as far as cloud computing is concerned are ***fault tolerance and scalability.***

Muilticast sender uses multicast protocol

- node may crash - Because processes are failure prone
- Packet may be dropped(or might also be delayed by the underlying network)
- 1000's of nodes

#### Centralized

When UDP/TCP packets are sended

- Simplest implementation - One of the simplest ways of doing multicast is a ***centralized approach***
- Problems?
  - ***Fault tolerance.*** if the sender faild when it is halfway through its "4" loop, essentially, only half the receivers would have received the multicast.
  - Also ***increases the latency***. The average time for a receiver to receive a multicast could be as high as O(N) which is linear in the size of the group itself

#### Tree-Based

> Spanning tree : 한 노드에서 다른 노드에 이르는 경로가 오직 하나 뿐인 토폴로지
>
> - 스위치나 브리지에서 발생하는 루핑을 막아주기 위한 프로토콜
> - 스위치나 브리지 구성에섯 출발지부터 목적지까지의 경로가 두 개 이상 존재할 때 한 개의 경로만 남겨두고 나머지는 모두 끊어 두었다가 사용하던 경오레 문제가 발생하면 그 때 끊어 두었던 경로를 하나씩 살린다.

When UDP/TCP packets are send

> If you build a tree, a balanced tree with N nodes in your group, the height of the tree is O(log(N)) and ***this means that the latency for a message to reach ant to the nodes in the group is O(long(N))***

- e.g., IP multicast, SRM, RMTP, TRAM, TMTP
- Tree setup and maintenance
  - when load failures happen, nodes in the system might actually not receive multicast for a while
    (트리 중간에 있는 노드에서 실패 발생하면 밑에 있는 노드들은 정보 못받음)
  - 
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

- Protocol rounds(local clock) b random targets per round
  $$
  \beta = \frac{b}{n}
  $$
  Substituting, at time t = c*lo g(n), the number of infected is
  $$
  y \approx (n + 1)- \frac{1}{n^{cb-2}}
  $$

- Set c, b to be small numbers independent of n
- Within c\*log(n) rounds, ***[low latency]***
  - All but $\frac{1}{n^{cb-2}}$ number of nodes receive the multicast ***[reliability]***
  - Each node has transmitted no more than c\*b\*blog(n) gossip message ***[lightweight]***

#### Fault-Tolerance

- With failures, is it possible that the epidemic might die out quickly?
- Possible, but improbable:
  - Once a few nodes are infected, with high probability, the epidemic will not die out
  - So the analysis we saw in the previous slides is actually behavior with high probability
- Think : Why do rumors spreads so fast? Why do infectious diseases cascade quickly into epidemics? Why does a virus or worm spread rapidly?

#### Pull gossip protocol

- In all forms of gossip, it takes O(log(N)) rounds before N/2 gets the gossip
  - Why? Because that's the fatest you can spread a message - a spanning tree with fanout(degree) of constant degree has O(long(N)) total nodes
- Thereafter, pull gossip is faster than pysh gossip
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

If you didn't believe your manager, what would be your first resplnsibility? well, your first responsibility would be to build a mechanism that detects failures of servers in your datacenter

- Build a failure detector
- What are some things that could go wrong if you didn't do this

#### Failure Are The Norm

- Say, the rate of failure of one machine (OS/disk?motherboard/network, etc) is once every 10 years (120 months) on average.
- When you have 120 servers in the DC, the mean time to failure(MTTF) of the next machine is 1 month
- When you have 12,000 servers in the DC, the MTTF is about once ***every 7.2 hours!***To Build a Dailure Detector
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

Membership List(<->Application Queries - communicate with different layere.g., gossip, overlays, DHT's, etc.) < - > Membership Protocol ( <-> Unreliable Communication - communicate with different layer)

#### Two Sub-Protocols

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
- What it does, it can then use the dissemination component to disseminate this information to ther other processes quickly

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

Essentially, this boild down to the fact that the failure detector problem is equivalent to another well-known problem in distributed computing known as consensus.

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