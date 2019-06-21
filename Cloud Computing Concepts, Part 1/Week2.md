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
> - 스위차나 브리지 구성에섯 출발지부터 목적지까지의 경로가 두 개 이상 존재할 때 한 개의 경로만 남겨두고 나머지는 모두 끊어 두었다가 사용하던 경오레 문제가 발생하면 그 때 끊어 두었던 경로를 하나씩 살린다.

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

