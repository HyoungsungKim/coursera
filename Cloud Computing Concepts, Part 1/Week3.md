# Week3

## Introduction

### Part 1: Week3 Overview

- This week's focus: Peer-to-peer System(P2P)
- Well see both
  - Unstructured p2p system, e.g, Gnutella, napster, etc.
  - Structured p2p system, e.g., Chrod, Pastry, etc.
  - Latter is a pre-cursor to key-value/NoSQL storage system(which comes next week)

## Lesson: P2P System

### Why  Study Peer to Peer System?

- First distributed systems that seriously focused on scalability with respect to number of nodes
- P2P techniques abound in cloud computing systems
  - Key-value storages(e.g., Cassandra, Riak, Voldemort) use Chord p2p hashing

## 2. Napster

### Napster Structure

- Servers run napster.com and store a directory, i.e., filenames with peer pointers
- Peers(Clients) store their own files

### Client

- Connect a napster server
  - Upload list of music files that you want to share
  - Server maintains list of <filename, ip_address, portnum> tuples. ***Server stores no files.***

- Search

  - Send server keywords to search with
  - (Server searches its list with the keywords)
  - Server returns a list of hosts - <ip_address, portnum> tuples - to client
  - Client pings each host in the list to find transfer rates
  - Client fetches file from the best host

  > 클라이언트가 서버에 요청 보내면 서버가 주소 알려줌
  >
  > 클라이언트가 서버에게 받은 주소로 직접 요청(ping)

  

- All communication uses TCP(Transmission Control Protocol)

  - Reliable and ordered networking protocol

None of this communication is the server actually involved in transferring the file. It is used in searching where the file might be, but it is not used in transferring the file.

> 서버는 중계만 하고 직접 파일을 저장하고 있지 않음

### Joining a P2P system

- Can be used for any p2p system
  - send an http request to well-known url for that p2p service
  - Message routed (after lookup in DNS) to introducer, a well known server that keeps track of some recently joined nodes in p2p system
  - Introducer initializes new peers' neighbor table

In any case, DNS is a gate way get introduced into the system, or communicate with an introducer node or introducer peer that ***will tell you some of the recently joined peers or some of the existing peers in the system*** so you can bootstrap your own neighbor list using that

### Problems

- Centralized server a source of congestion
- Centralized server single point of failure
- No security: plaintext messages and passwards
- napster.com declared to be responsible for users' copyright violation
  - "indirect infringement"
  - Next system: Gnutella

## 3. Gnutella

### Gnutella

- Eliminate the servers
- Clients machines search and retrieve amongst themselves
- Client act as servers too, ***called servants***
- Original design underwent several modifications

Gnutella doesn't have any servers. Instead, peers communicate directly with each other. Peers, like in Napster, store their own files; files do not go anywhere by default. But peers also store peer pointers. In other words, every peers has one or more neighbors.

A neighbor means this peer knows about their IP address and port number pairs and can send them messages using TCP(for instance)

Peers connected by overlay graph. Overlay graph is overlaid on top of the Internet.

> overlay : 덮어씌우다.

### How Do I Search for My File?

- Gnutella routes different messages within the overlay graph
- Gnutella protocol has 5 main message types
  - Query(search)
  - QueryHit(Response to query)
  - Ping(to prove network for other peers)
  - Pong(reply to ping, contains address of another peer)
  - Push(used to initiate file transfer) (from requester to receiver)
- We will go into the message structure and protocol now
  - ***All fields excepts IP address are in little-endian format***
  - 0x12345678 stored as 0x78 in lowest address byte, than 0x56 in next higher address, and so on.

#### Gnutella Message Header format

- Descriptor ID(16 bytes) : This is typically generated uniquely for every single search from every peer. In other words, ***Whenever your client generates a search, it has to generate a descriptor ID that is unique throughout the entire system.*** The descriptor ID is useful because the intermediate nodes can then use this to distinguish messages that only belong to this particular search and not confuse it with other searches.
- Type of Payload(2 bytes) : the kind of message that his is one of five types for the five different message(query, queryhit, ping, pong, push)
- Time to live(TTL) (1 byte)  : ***Whenever a message is forwarded by a peer to its neighboring peers, TTL is decremented by one.*** When the TTL hits zero, you do not forward the message anymore.(Usually 7 ~ 10)
  - TTL is set to be a finite number so that essentially a message doesn't keep circulating around the overlay graph forever.
- Hops(1 byte) : the number of hops that has message has transmitted too
  - Hot is also incremented every time a message is forwarded
  - Why need TTL and Hop both? The reason is the initial TTL may not be the same at all the initial peers.(recently hop is not used)
- Payload : payload differs depending on the kind of message it is, depending on the one of the five kinds of messages.
  - How many bytes are there in the payload is present in the payload length field. so you know how far to look for the end of the message.

#### Query messages(Payload Format in Gnutella Query Message )

The query message contains first of all minimum speed that is request by the querying peers. so if you are connected to a 100 MBps line, you might say  "Well, i need peers that are at least 100 MBps or at least 10 MBps" and then the search criteria, these might be the keywords that you are searching for.

- Query(0x80) : Minimum speed + search criteria(keywords)

### Gnutella Search

How are queries sent out? - Query's flooded out, TTL-restricted, forwarded only once

The query message to all the peers has received except the one from which it just received. other peer doesn't send to peer which already received same query form other peer. the descriptor IDs of these query messages are the same. and they are retained(보유하고 있는) as the messages transit through the overlay graph

- When peer received a query message, then peer decrement TTL and if TTL become 0, then query message is not sent.
- When got a duplicated query, it is not forwarded.

> peer는 얼마나 오래 descriptor id를 보관하고 있는지?

How do search results come back? 

- When a peer finds that it has some matching files which match the incoming query's keywords, ***it creates a QueryHit message***

#### QueryHit(0x81) : successful result to a query

Payload Format in Gnutella QueryHit Message

- Number of hits : the number of files that match.
- The port
- The IP address 
- The speed of the responding peer. This is the peer that is responding to the querying peer 
- Files that match information about that file such as the file-index, file-name  and the file-size
- Servent_id : Unique identifier of responder; a function of its IP address. ***For most part this is not really used in the query and query hit***

How doe the QueryHit make its way back to the querying peer? - QueryHits are reverse routed.

- Remember every peer keeps track of the recent query message it has received.

### Avoiding Excessive Traffic

Each peer maintains a list of recently received messages.

- To avoid duplicate transmissions, each peer maintains a list of recently received messages
- Query forwarded to all neighbors except peer from which received
- Each Query (identified by DescriptorID) forwarded only once
- QueryHit routed back only to peer from which Query received with same DescriptorID
- For flooded messages, duplicates with same DescriptorID and Payload descriptor are dropped
- QueryHit with DescriporID for which Query not seen is dropped - You might receive a Query-hit message for which you don't have any information about a Query message with the same matching descriptor ID(가지고 있는 DescriptorID와 다른 queryhit 받았을때 그냥 drop 함)

The overlay graph may have changed. Because some of the neighbors may have shifted and changed in between the time that the Query went through and until when the queryhit comes back(시간에 따라서 client 바뀔 수 도 있기 때문에 graph도 바뀔 수 있음)

### After Receiving QueryHit Messages

How does the download start?

- Request chooses "best" QueryHit responder

  - Initiates HTTP request directly to responser's ip + port
  - Initially the requester or the querying peer sends an HTTP requests to the responser's IP address and port number. - ***GET http request***

  ```
  (\r :처음으로 커서를 옮김)
  Get /get/<File Index>/<File Name>/HTTP/1.0\r\n
  Connection: Keep-Alive\r\n
  Range: bytes=0-\r\n 
  User-Agent;Gnutella\r\n
  \r\n
  ```

  Range field : To start partial file transfer

For instance, If you started to transfer a 1 MB file and after you'd transferred 512 KB the connection, was killed for some reason then ***you can transfer start another GET transfer by specifying the range field that's 512 KB*** so that the file transfer start from 512 KB on ward rather than transfer only second half of the file rather than transferring the entire file.



When the responder receives this GET message, it responds back with an HTTP OK message with the content length the number of bytes and then this is followed immediately by packets that contain the actual file data itself.

- Responder then replies with file packets after this messages:

```
HTTP 200 OK\r\n
Server:Gnutella\r\n
Content-type:application/binary\r\n
Content-length: 1024\r\n
\r\n
```

- HTTP is the file transfer protocol. WHY?
  -  Because it is standard, well-debugged and widely used
- Why the "range" field in the GET request?
  - To support partial file transfer
  - ***You don't need to restart the entire file transfer whenever a connection gets dropped***
- What if responder is behind firewall that disallows incoming connections?

### Dealing With Firewalls

Requester sends Push to responder asking for file transfer but file exists behind firewall.

What Gnutella does is something very clever, which is that it uses the overlay links themselves, ***remember that these edges in the overlay are already set up;*** they're already set up TCP connections among the peers or between pairs of peers. so you can route whatever messages you want using these links and so that is what the Gnutella requesting peer does.

1. Gnutella try to set up an HTTP connection to the responding peer
2. When it fails, it guesses that maybe the responding peer is behind a firewall
3. then it routes a Push message via links in the overlay and again
4. the Push message is reverse routed along the QueryHit path
5. so it is reverse routed along the reverse QueryHit path, and so when the peer receives this, Push message it can then generate an outgoing TCP connection.

> 방화벽 있으면 http로 파일 전송 안하고 tcp로 함.

#### Push(0x40)

Same as in received QueryHit

- server_id
- fileindex

Address at which requester can accept incoming connections

- ip_address
- port



- Responder establishes a TCP conenction at ip_address, port specified. Sends

```
GIV<File Index><Servent Identifier>/<File Name>\n\n
```

- Requester then sends GET to responder (as before) and file is transferred as explained earlier
- What if requester is behind firewall too?
  - Gnutella gives up

### Ping-Pong

These are used by peers to update their neighbor lists.

Ping(0x00) : no payload

- When peer receives a Ping message, it responds back in the reverse route to the Ping message using a Pong message

Pong(0x01)

- port
- ip_address
- Number of file shared
- Number of KB shared

> ***These last two fields are used by the pinging peer to prefer neighbors which have more data and more files to share***

- Peers initiate Ping's periodically

- Ping's flooded out like Query's Pong's routed along reverse path like QueryHit's

- Pong replies used to update set of neighboring peers
  -  to keep neighbor lists fresh in spite of peers joining leaving and failing
- Receiver forward ping to neighbors after checking TTL

Why is ping and pong needed? P2P systems have a very high rate of churn(마구 휘돌다)

- Clients are continuously joining and leaving the system but the rate of churn is so high. set of neighbors that you have right now may not be there a few seconds from now because some of them may have left the system
- So it is very important for you to keep your set of neighbors always fresh with the neighbors that are currently there in the system so that you are not disconnected from the system.

> p2p는 매우 혼잡하기 때문에 시스템에 존재하는 neighbors와 계속 연결되어 있는게 중요함.
>
> Query : 파일 있는지 물어볼려구 사용 됨
>
> Ping : 현재 제대로 시스템 상에 peer 존재하는지 확인하기 위해 사용 됨
>
> -> 헷갈리지 말자~

### Gnutella Summary

- No servers
- Peers/servants maintain "neighbors", this forms an overlay graph
- Peers store their own files
- Queries floods out, TTL restricted
- QueryHit(replies) reverse path routed
- Supports file transfer through firewalls
- Periodic Ping-pong to continuously refresh neighbor lists
  - List size specified by user at peer : heterogeneity means some peers may have more neighbors

### Problems

- Ping/Pong constituted(구성된) 50% traffic

  - Solutions : Multiplex, cache and reduce frequency of pings/pongs
  - Multiplex : when receive a multiple ping, then send one pong instead of sending multiple pong messages. similar in ping too

- Repeated search with same keywords

  - Solution: cache Query, QueryHit messages

- Modem-connected hosts do no have enough bandwidth for passing Gnutella traffic

  - Solution : use a central server to act as proxy for such peers
  - Another solution: FastTrack System

- Large number of freeloaders

  - 70% of users in 2000 were freeloaders
  - Only download files, never upload own files

- Flooding cause excessive traffic

  - Is there some way of maintaining meta-information(어떤 목적을 가지고 만들어진 정보) about peers that leads to more intelligent routing? -> Structured Peer-to-peer systems e.g., Chord System.

  

## 4. FastTrack and BitTorrent

### FastTrack

- Hybrid between Gnutella and Napster
- Takes advantages of "healthier" participants in the system
- Underlying technology in Kazaa, KazaaLite, Grokster
- Proprietary protocol, but some details available
- Like Gnutella, but with some peers designated as ***SuperNode***
- SuperNodes have some special duties

### A FastTrack-Like System

- Supernodes undertake some of the responsibilities very similar Napster. peers in the sense that they might be ***storing directory information*** that are used by its neighboring peers in the overlay to search for files

### FastTrack(contd)

- A supernode stores a directory listing a subset of nearby (<filename, peer pointer>), similar to napster servers
- Supernode membership changes over time
- Any peer can become (and stay) a supernode, provide it has earned enough ***reputation***
  - Peers are selected to be supernodes based on their contributions the past and this contribution is decided from a reputation level
  - Kazaalite: participation level(=reputation) of a user between 0 and 1000, initially 10, then affected by length of periods of connectivity and total number of uploads
  - More sophisticated Reputation schemes invented, especially based on economics 
- A peer searches by contacting a nearby supernode

#### Why is there an advantage to be a supernode? what is the incentive to be a supernode?

- A lot of your queries and searches now become just local searches in your local data structure or your local directory information
- ***So they do not incur any network traffic***

### BitTorrent

BitTorrent is one of these systems that has been very popular and continues to be popular.

1. Get tracker : Website links to .torrent
2. Get peers from tracker: Tracker, per file.(receives heartbeats, joins and leaves from peers)
3. Get file blocks 
   - Seed : has full file
   - leecher : has some blocks

- Usually peers oftentimes do not contribute anything to the system
- BitTorrent addresses this by having incentives for the peers to participate
- so that the peers can benefit from being unselfish and helping other peers

- File split into blocks(32 KB ~ 256 KB)
- Download Local Rarest First block policy : prefer early download of blocks that are least replicated among neighbors
  - Exception : New node allowed to pick one random neighbor : helps in bootstrapping
- ***Tit for tat*** bandwidth usage : Provide blocks to neighbors that provided it the best download rates
  - Incentive for nodes to provide good download rates
  - Seeds do the same too
- Choking : Limit number of neighbors to which concurrent uploads <= a number(5), i.e., the "best" neighbors
  - Everyone else choked
  - Periodically re-evaluate this set(e.g., every 10s)
  - Optimistic unchoked: periodically(e.g., ~30 s), unchoked a random neighbor - helps keep unchoked set fresh(30초 마다 무작위 neighbor에 unchoke함 - 데이터가 더 골고루 퍼질 수 있음)

## 5. Chord

#### DHT = Distributed Hash Table

A hash table is a data structure typically maintained inside one running process on one machine, which allows you to insert, look up, and delete objects with unique key. So you can perform these operations in essentially order 1 time or constant time.

- Hash table stores these object in bucket based on a hash of the key, and this allows you to look up and perform other operations like insert and delete fairly quickly on each key

> unique key < - > file

A distributed hash table known as a DHT is a similar data structure except that it runs in a distributed system rather than in a single process

- However, instead of storing the objects into bucket, ***you store the object at nodes or hosts or machines in a cluster***
- A hash table allows you to insert, lookup and delete objects with keys
- A distributed hash table allows you to do the same in a distributed setting(object = files)
- Performance Concerns:
  - Load balancing
  - Fault-tolerance
  - Efficiency of lookups and inserts
  - Locality : you want the messages that are transmitted among the cluster to be transmitted preferably among nodes that are close by in the underlying network topology or in terms of the Internet distance underneath

- napster, Gnutella, FastTrack are all DHTs(sort of) (however, not optimize)
- So is Chord, a structured peer to peer system that we study next
- Chord is one of the first peer to peer systems which tries to directly address this problem and tries to bring the insert, delete and lookup time down low enough that we can claim that it is, it fact, a DHT or a distributed hash table.

### Comparative Performance

|          | Memory                | Lookup Latency | # messages for a lookup |
| -------- | --------------------- | -------------- | ----------------------- |
| Napster  | O(1) (O(N) at server) | O(1)           | O(1)                    |
| Gnutella | O(N)                  | O(N)           | O(N)                    |
| Chord    | O(log(N))             | O(log(N))      | O(log(N))               |

### Chord

- Chord uses a technique where each of the nodes in the peer-to-peer overlay selects its neighbors in an intelligent fashion
- There are certain rules that the nodes use to decide who their neighbors are. 
- Intelligent choice of neighbors to reduce latency and message cost of routing(lookups/inserts)
- Uses Consistent hashing on node's (peer's) address
  - SHA-1(ip_address, port) -> 160 bit string
  - Truncated to m bits
  - Called peer id(number between 0 and 2^m - 1)
  - ***Not unique but id conflicts very unlikely***
  - Can then map peers to one of 2^m logical points on a circle

### Peer Pointer (1)  : Successors (Ring of peers)

If m is 7, make the number of 127 dot on circle and point it a result of hash

- What are the neighbors that the peers maintain? 
- The first type of neighbors that the peers maintain are successors
- For example, 16 - > 32, 32 -> 45, 45 -> 80 ...  116 -> 16 can send message directly to successor
- If need, predecessors can be used.

### Peer Pointer (2) : Finger Tables

The ith finger table and the peer that has peer ID n, is the first peer that has ID immediately at or to the clockwise of n + 2^i but then, taking modular 2^m

ith entry at peer with id n is first peer with id >= n+2^i (mod 2^m)

Example n = 80(n is ID), i  = 0

- The 0 finger table at node N80. We get a 96. how?
- 80 + 2^0(n + 2^i) that is 81 and so that is a point somewhere on the ring. The node immediately to the clockwise of that is 96, N96. That is why we say that N96 is the zero-th finger table entry at N80.
- n + 2^i <= 96 -> finger table[i] is 96
- A와 B 사이에 있으면 앞에 있는 것(여기서는 B) 선택

### What About the Files?

How do we place files, how would we decide where files get placed?

- Unlike Napster, Gnutella where clients told their own files and don't upload them by default, here instead find the store in specific notes based on the same rules if your using for placing the servers on the ring.
- In other words, you take the filing which we assume to be unique across the entire system.

- Filenames also mapped using same consistent hash function
  - SHA-1(filename) -> 160 bit string(key)
  - ***File is stored at first peer with id greater than or equal to its key***(mod 2^m)
  - 예를 들어 36과 45 사이에 있는 값 나오면 45에 저장 됨
- File (cnn.com/index.html) that maps to key K42 is stored at first peer with id greater than 42(last slide, it is stored at 45)
  - Note that we are considering a different file-sharing application here : cooperative web caching
  - The same discussion applies to any other file sharing application, including that of mp3 files.
- Consistent hashing => with K keys and N peers, each peer stores O(K/N) keys(i.e., < c*K/N, for some constant c)
- K : keys, N : peers
- O(K/N). It just means that the number of keys at a peer is less than c*K/H for some constant c with a high probability
- It means you have good load balance across the different tiers in your system or the different nodes in your system.
- If name is unique, peer review systems can be used for storing any kind of object.

### Search

Who has file? - hashes to 42 and In 45, there is the File with key K42 stored

Suppose N80 wants search for file,

- the first thing it does us that hashes it and trunks it to embeds, gets 42
- It knows that it needs to route to the point 42 on the ring or rather to the point that is immediately to the north, that is immediately to that clockwise

How does it do?

Search algorithm is applied recursively

- At node n, you have query and it is destined for key k, it this case, k is 42
- You forward the key to the largest successor or finger table entry, essentially your largest neighbor are when i say largest it actually means it is the most to the right wrapping around the ring
- At node n, send query for key k to ***largest successor/finger*** entry <= k.
- ***If not exist, send query to successor(n)***

- Even if the finger table entries are wrong, then as long as the successors are correct you end up routing the query to the correct server eventually
- In finger table the most far N is N16 so N80 will forward the query to N16

N80 -> N16. But still to the left of 42

- You will notice that 16 will have 32 as a finger table entry because 16 + 2^4 is 32

- But 16 + 2^5 is 48 which means that the fifth, the next finger table entry after 32 at 16 ***does not even know about N45's existence***
- N16 has only two choices, 32 or 80 to forward the 32 and since 32 is to the left or counterclockwise of 42, it forwards it to 32

### Analysis

Search takes O(log(N)) time

Proof

- (intuition) : at each step, distance between query and peer-with-file reduces by a factor of at least 2
- (intuition): after log(N) forwardings, distance to key is at most 2^m/2^(log(N)) = 2^m/N
- Number of node identifies in a range of 2^m/N is O(log(N)) with high probability(why? SHA-1! and "Balls and Bins")  So using successors in that range will be OK, using another O(log(N)) hops
- O(log(N)) search time holds for file insertion too(in general for routing to any key)
  - "Routing" can thus be used as a building block for
    - All operations: insert, lookup, delete
- ***O(log(N)) time true only if finger and successor entries correct***
- When might these entries be wrong?
  - When you have failures