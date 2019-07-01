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
> Ping : 현재 제대로 시스템 상에 존재하는지 확인하기 위해 사용 됨
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

- Repeated searchs with same keywords

  - Solution: cache Query, QueryHit m,essages

- Modem-connected hosts do no have enough bandwith for passing Gnutella traffic

  - Solution : use a central server to act as proxy for such peers
  - Another soultion: FastTrack System

- Large number of freeloaders

  - 70% of users in 2000 were freeloadres
  - Only download files, never upload own files

- Flooding cause excessive traffic

  - Is there some way of maintaining meta-information(어떤 목적을 가지고 만들어진 정보) about peers that leads to more intelligent routing? -> Structured Peer-to-peer systems e.g., Chord System.

  