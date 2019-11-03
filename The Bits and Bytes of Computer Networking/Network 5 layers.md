# The Bits and Bytes of Computer Networking

## The TCP/IP Five-Layer Network Model

Physical -> Data Link -> Network -> Transport -> Application

### Physical layer

- Protocol : 10 Base T, 802.11
- Protocol Data Unit : Bits
- Addressing : n/a
- Equipment : Hubs
  - A physical device that allows for connection from many computers at once
  - ***Collision domain*** : A network segment where only one device can communicate at a time
    - If multiple systems try sending data at the same time, the electrical pulses sent across the cable can interfere with each other
    - This causes these systems to have to wait for a quiet period before they try sending their data again
  - ***It is really slows down network communications, and is the primary reason hubs are fairly rare.***
  - They're mostly a historical artifact today.

It represents the physical devices that interconnect computers. It is all about cabling, connectors and sending signals.

The physical layer consists of devices and means of transmitting bits across computer networks

- A bit is the smallest representation of data that a computer can understand.
- It is a one or a zero
- Ones and zeros are sent across those network cables through a process called modulation.
- Modulation is a way of varying the voltage of this charge moving across the cable.(Line coding)

> Digital data(0101...1001001) -> Encoder -> Digital signal(Electrical pulse) -> Decoder - > Digital data(0101...1001001)

### Data Link layer

- Protocol : Ethernet, Wi-fi
- Protocol Unit : Frames
- Addressing : MAC Address
- Equipment : Switches
  - A much more common way of connecting many computer
  - Switch can actually inspect the contents of the Ethernet protocol data being sent around the network, determine which system the data is intended for and then only send that data to that one system.
    - A switcher remembers which devices are connected on each interface, while a hub does not.
  - This reduces or even completely eliminates the size of collision domains on a network
  - It will lead to fewer retransmissions and a higher overall throughput

Responsible for defining a common way of interpreting these signals so network devices can communicate. The Ethernet standards also define a protocol responsible for getting data to nodes on the same network or link.

CSMA/CD : Used to determine when the communications channels are clear, and when a device is free to transmit data.

- CSMA/CD : Carrier sense multiple access with collision detection
- Before CSMA, Duplex communication is impossible in hub(Domain collision)

MAC address : A globally unique identifier attached to an individual network interface

- It is a 48-bit number normally represented by six groupings of two hexadecimal numbers.

Ethernet uses MAC addresses to ensure that the data it sends has both an address for the machine that sent the transmission, as well as the one the transmission was intended for.(누가 누구한테 보냈는지 알기 위해 사용)

- In this way, even on a network segment, acting as a single collision domain, each node on that networks knows when traffic is intended for it.

#### Type of Ethernet transmission

First 3 octet is signature of manufactures 

Unicast : A unicast transmission is always meant for just one receiving address

- EX) **** ****0 : **** **** : **** **** : **** **** : **** **** : **** ****
- At the Ethernet level, this is done by looking at a special bit in the destination MAC address
- If the least significant bit in the first octet of a destination address is set to zero, it means that Ethernet frame is intended for only the destination address

Multicast

- If the least significant bit in the first octet of a destination address is set to one, it means you are dealing with a multicast frame.
  - EX) **** ****1 : **** **** : **** **** : **** **** : **** **** : **** ****
  - Multicast frame is similarly set to all devices on the local network signal
  - What's different is that it will be accepted or discarded by each device depending on criteria aside from their own hardware MAC address..
  - Network interface can be configured to accept lists of configured multicast addresses for these sort of communication.

Broadcast : Ethernet broadcast is sent to every single device on a LAN.

- Address : FF:FF:FF:FF:FF:FF
- This is accomplished by using a special destination known as a broadcast address
- Ethernet broadcasts are used so that devices can learn more about each other

#### Internetwork

A collection of networks connected together through routers, the most famous of these being the Internet

#### IP

IP is the heart of the Internet and most smaller networks around the world

#### Ethernet frame

Data packet : An all-encompassing term that represents any single set of binary data being sent across a network link

- Data packets at the Ethernet level are known as Ethernet frames.
-  Ethernet frame is a highly structured collection of information presented in a ***specific order***
  - Preamble(8 bytes)
    - 뜻 : (책의)서문
    - 8 bytes(or 64 bits) long, and can itself be split into two sections
    - ***The first seven bytes**** are a series of alternating ones and zeros.
      - These act partially as a buffer between frames and can also be used by the network interfaces to synchronize internal clocks they use, to regulate the speed at which they send data.
    - This last byte is the preamble is known as the SFD or start frame delimiter(구분문자).
      - This signals to a receiving device that the preamble is over and that the actual frame contents will now follow
  - Destination (MAC) Address(6 bytes)
    - This is the hardware address of the ***intended recipient.***
    - MAC address is (MAC address is 6 byte or 48 bits)
  - Source (MAC) Address(6 bytes)
    - ***Where the frame originated from***
  - VLAN Tag(4 bytes)
    - Indicates that the frame itself is what is called a VLAN frame
    - If a VLAN header is present, the Ether type field follows it
    - ***VLAN stands for virtual LAN, it is a technique that lets you have multiple local LANs operating on the same physical equipment.***
    - Any frame with a VLAN tag will only be delivered out of a switch interface configured to replay that specific tag.
    - ***This way you can have a single physical network that operated like it is multiple LANs.***
    - VLANs are usually used to segregate different forms of traffic. So you might see a company's IP phones operating on one VLAN, while all desktops operate on another.
  - Ether-type(2 bytes)
    - 16 bits long and used to describe the protocol of the contents of the frame
    - After this, you will find a data payload of an Ethernet frame
  - Payload(0 - 1500 bytes)
    - In networking terms, is the ***actual data being transported,*** which is everything that isn't a header
    - 헤더에 없는 모든 것을 전송하는 부분
    - This contains all of the data from higher layers such as the IP, transport and application layers that is actually being transmitted
  - FCS(4 bytes)
    - FCS : Frame check sequence
    - A 4 byte (or 32 bit) number that represents a checksum value for the entire frame
    - This checksum value is calculated by performing what is known as a ***cyclical redundancy check*** against the frame
    - Cyclical Redundancy Check(CRC) : An important concept for data integrity, and is used all over computing, not just network transmissions
    - Anytime you perform a CRC against a set of data, you should end up with the same checksum number
    - sha-256으로 얻는 해시 값 처럼 하나만 바뀌어도 다른 값이 나와서 위변조 여부를 확인 가능
    - If calculation result  is different with a checksum of header, It means frame is corrupted

### Network layer

- Protocol : IP
- Protocol Data Unit : Datagram
- Addressing : IP address
- Equipment : Routers(A device that knows how to forward data between independent network)
  - Router connect IP
  - Router store internal tables containing information about how to route traffic between lots of different networks all over the world
  - Routers share data with each other via Border Gateway Protocol(BGP), which lets them learn about the most optimal paths to forward traffic

Allows different networks to communicate with each other through devices known as routers

#### IP Address

IP addresses belong to networks, not to the devices attached to those networks. For example, laptop always has a same mac address but IP is not.

- IP address will be assigned to it automatically through a technology known as Dynamic Host Configuration Protocol. An Ip address assigned this way is known as a ***dynamic IP address***.
- The opposite of this is known as a ***static IP address,*** which must be a configured on a node, manually.
- In most cases, static IP addresses are reserved for servers and network devices
- Dynamic IP addresses are reserved for clients

IP address can be split into two sections:

- The network ID
- The host ID

#### How MAC address and IP address relate to each other?

- MAC addresses use data link layer
- IP addresses use network layer.

 How these two separate addresses types relate to each other? This is where ***Address Resolution Protocol(ARP)***

Once IP datagram has been fully formed, it needs to be encapsulated inside an Ethernet frame. This means, that the ***transmitting device needs a destination MAC address to complete the Ethernet frame header.*** Almost all network connected devices while retaining local ARP table.

Address class system : A way of defining how the global IP address space is split up

ARP(Address Resolution Protocol) : ***A protocol used to discover the hardware address of a node with a certain IP address***

ARP table : A list of IP addresses and the MAC addresses associated with them

Scenario : Send some data to the IP address 10.20.30.40 and this destination is not in ARP table.

- When this happens, the node that wants to send data sends a broadcast ARP message to the MAC broadcast address which is all Fs(FF:FF:FF:FF:FF:FF)
- These kinds of broadcast ARP messages are delivered to all computers on the local network.
- When the network interface that is been assigned an IP of 10.20.30.40 receives this ARP broadcast, it sends back what is known as an ARP response.
  - This response message will contain the MAC address for the network interface in question.
- Now The transmitting computer knows what MAC address to put in the destination hardware address field and the Ethernet frame is ready for delivery.
- It will also likely store this IP address in its local ARP table so that it won't have to send an ARP broadcast the next time he needs to communicate with this IP.
- ARP table entries generally expire after a short amount of time to ensure changes in the network are accounted for

#### IP datagram

A highly structured series of fields that are strictly defined

Two primary sections of an IP datagram

- Header
  - It takes a lot more than Ethernet header
  - Version(4 bits)
    - It indicates what version of internet protocol is being used.
    - The most common version of IP is version 4, or IPv4
    - Version 6 or IPv6 is rapidly seeing more widespread adoption
  - Header Length(4 bits)
    - It declares how long the entire header is
    - Almost always 20 bytes in length when dealing with IPv4
  - Service Type(8 bits)
    - These 8 bits can be used to specify details about quality of service, or QoS, technologies
    - The important takeaway about QoS is that there are services that allow routers to make adecisions about which IP datagram may be more important than others.
  - Total Length Length(16 bit)
    - Indicates the total length of the IP datagram it is attached to
  - Identification field(16 bits)
    - A 16 bit number that is used to group messages together
    - The maximum size of a single datagram is the largest number you can represent with 16 bits: 65,535
    - If the total amount of data that needs to be sent is larger than what can fir in a single datagram, the IP later needs to split this data up into many individual packets.
    - When this happens, the identification field is used so that the receiving end understands that evbery packet with the same value in that field is part of the same transmission.
  - Flags(3 bits)
    - Used to indicate if a datagram is allowed to be fragmented or to indicate that the datagram has already been fragmented
    - Fragmentation : The process of taking a single IP datagram and splitting it up into several smaller datagrams
    - If a datagram has to cross from a network allowing a larger datagram size to one with a smaller datagram size, the datagram would have to be fragmented into smaller ones.
  - Fragment Offset(12 bits)
    - Contains values used by the receiving end to take all the parts of a fragmented packet and put them back together in the correct order.
  - TTL(8 bits)
    - An 8 bit field that indicates how many router hops a datagram can traverse before it is thrown away
    - Every time a datagram reaches a new router, that router decrements the TTL field by one.
    - Once this value reaches zero, a router knows it doesn't have to forward the datagram any further.
    - The main purpose of this field is to make sure that when there is a misconfiguration in routing that causes an endless loop, datagrams don't spend all eternity tying to reach their destination
    - Endless loop : A -> B and B -> A
  - Protocol field(8 bits)
    - Another 8 bits field that contains data about what transport layer protocol is being used
    - The most common transport layer protocols are TCP and UDP
  - Header Checksum(16 bits)
    - A checksum of the contents of the entire IP datagram header
    - It functions very much lie the Etherent checksum field we discussed in the last module.
    - ***Since the TTL field has to be recomputed at every router that a datagram touches, the checksum field necessarily changes too*** 
  - Source IP Address(32 bits)
  - Destination IP Address(32 bits)
  - IP options field(24 bits)
    - An optional field and is used to set speical characteristics for datagrams primarily used for testing purposes
  - Padding(32 - size of IP options fields bits)
    - A series of zeros used to ensure that header is the correct total size
- Payload

Message (Application layer) -> TCP or UDP header + message (Transport layer) -> IP header + TCP segment or UDP datagram (Network layer) -> Ethernet header + IP datagram + Ethernet footer (Data-link)

#### Subnetting

The process of taking a large network and splitting it up into many individual and smaller subnetworks, or subnets.

#### Subnet ID

- Network IDs : used to identify networks
- Host IDs : used to identify individual hosts.

If we want to split things up even further, and we do, we'll need to introduce a third concept, the subnet ID. You might remember IP address is just a 32-bit number.

- In a world without without subnets, a certain number of these bits are used for the network ID, and a certain number of the bits are used for the host ID.
- ***In a world with subnetting, some bits that would normally comprise the host ID are actually used for the subnet ID.*** with all three of these IDs representable by a single IP address, we now have a single 32 bits number that can be accurately delivered across many different networks.

At the internet level, core routers only care about the network ID and use this to send the datagram along to the appropriate gateway router to that network. That gateway router then has some additional information that it can use to send that datagram along to the destination machine or the next router in the path to get there. Finally, the host ID is used by that last router to deliver the datagram to the intended recipient machine. Subnet IDs are calculated via what's known as a subnet mask. 

Just like an IP address, subnet masks are 32-bit numbers that are normally written now as four octets in decimal. The easiest way to understand how subnet masks work is to compare one to an IP address.

Let's work with the IP address 9.100.100.100 again. You might remember that each part of an IP address is an octet, which means that it consists of eight bits. The number 9 in binary is just 1001. But since each octet needs eight bits, we need to pad it with some zeros in front. As far as an IP address is concerned, having a number 9 as the first octet is actually represented as 0000 1001. Similarly, the numeral 100 as an eight-bit number is 0110 0100. So, the entire binary representation of the IP address 9.100.100.100 is a lot of ones and zeros.

***A subnet mask is a binary number that has two sections.***

- The beginning part, which is the mask itself is a string of ones just zeros come after this, the subnet mask, which is the part of the number with all the ones, tells us what we can ignore when computing a host ID.
- ***The part with all the zeros tells us what to keep.***
  - Let's use the common subnet mask of 255.255.255.0. This would translate to 24 ones followed by eight zeros. 
    - The purpose of the mask or the part that's all ones is to tell a router what part of an IP address is the subnet ID
    - The numbers in the remaining octets that have a corresponding one in the subnet mask are the subnet ID.
    - The numbers in the remaining octets that have a corresponding zero are the host ID.
    - The size of a subnet is entirely defined by its subnet mask.
  - So for example, with the subnet mask of 255.255.255.0, ***we know that only the last octet is available for host IDs,*** regardless of what size the network and subnet IDs are. A single eight-bit number can represent 256 different numbers, or more specifically, the numbers 0-255. ***This is a good time to point out that, in general, a subnet can usually only contain two less than the total number of host IDs available.***
  - Again, using a subnet mask of 255.255.255.0, we know that the octet available for host IDs can contain the numbers 0-255, but zero is generally not used and 255 is normally reserved as a broadcast address for the subnet. ***This means that, really, only the numbers 1-254 are available for assignment to a host.*** While this total number less than two approach is almost always true, generally speaking, you'll refer to the number of host available in a subnet as the entire number. So, even if it's understood that two addresses aren't available for assignment, you'd still say that eight bits of host IDs space have 256 addresses available, not 254. This is because those other IPs are still IP addresses, even if they aren't assigned directly to a node on that subnet.
  - IP : 9.100.100.100.100 Subnet : 255.255.255.224 -> Notation : 9.100.100.100.100/27 : CIDR notation
    - 255.255.255.224 : 1111 1111 1111 1111 1111 1111 1110 0000 : The number of ones is 27

#### Router

A network device that forwards traffic depending on the destination address of that traffic.

Router step

1. Receive data packet
2. Examines destination IP
3. Looks up IP destination network in routing table
4. Forward traffic to destination

### Transport  layer

- Protocol : TCP/UDP
- Protocol Data Unit : Segment
- Addressing : Port #'s'
  - Port : A 16-bit number that is used to direct traffic to specific services running on a networked computer

Allows traffic to be directed to specific network applications.

Transport layer has the ability to multiplex and demultiplex

- Multiplexing : Nodes on the network have the ability to direct traffic toward many different receiving services.
- Demultiplexing : Same concept with multiplexing, but just at the receiving end
- Processes -> Multiplexer -> IP -> Demultiplexer -> Processes

#### TCP

IP datagram has a payload section and this is made up of what is known as a TCP segment.

- TCP segment : Made up of a TCP header and a data section

TCP Header

- Source port(16 bits)

  - A high-number port chosen from a special section of ports known as ephemeral ports

- Destination port(16 bits)

  - The port of the service the traffic is intended for

- Sequence number(32 bits)

  - A 32 bits number that is used to keep track of where in a sequence of TCP segments this one is expected to be

- Acknowledgement number(32 bits)

  - The number of the next expected segment

- Data offset field(Header length 4 bits)

  - A 4 bits number that communicates how long the TCP header for this segment is

- Control flags(6 bits)

  - The way TCP establishes a connection, is through the use of different TCP control flags, used in a very specific order.

  - The first flag is known as URG(urgent)
    - A value of one here indicates that the segment is considered urgent and that the urgent pointer field has more data about this
  - The second flag is ACK(acknowledge)
    - A value of one in this field means that the acknowledgement number field should be examined
  - The third flag is PSH(push)
    - The transmitting device wants the receiving device to push currently-buffered data to the application on the receiving end as soon as possible
    - By keeping some amount of data in a buffer, TCP can delivery more meaningful chunks of data to the program waiting for it.
    - But in some case, sending a very small amount of information, that you need the listening program to respond to immediately.
      - ***This is what the push flag does***
  - The forth flag is RST(Reset)
    - One of the sides in a TCP connection hasn't been able to properly recover from a series of missing or malformed segments
  - The fifth flag is SYN(Synchronize)
    - It is used when first establishing a TCP connection and makes sure the receiving end knows to examine the sequence number field
  - The sixth flag is FIN(Finish)
    - When this flag is set to one, it means the transmitting computer doesn't have any more data to send and the connection can be closed

- TCP Window(16 bits)

  - Specifies the range of sequence numbers that might be sent before an acknowledgement is required

- Checksum(16 bits)

  - Operate just like the checksum fields at the IP and Ethernet level
  - Check data is wrong, corrupted or lost

- Urgent pointer field(16 bits)

  - Used in conjunction with one of the TCP control flags to point out particular segments that might be more important than others

- Option(0 or 16 if any)

  - It is sometimes used for more complicated flow control protocols

- Padding

  - Sequence of zeros to ensure that the data payload section begins at the expected location

#### TCP connection establishment

Handshake : A way for two devices to ensure that they are speaking the same protocol and will be able to understand each other.

- Three way handshake(When start a connection)

1. A sends a TCP segment to computer B with this SYN flag set
2. B sends a response to A with a TCP segment where both SYN and ACK flags are set
3. Then computer A responds again with just the ACK flag set

- Four way handshake(when close a connection)

1. B sends FIN flag to A
2. A sends ACK to B
3. A sends FIN to B
4. B sends ACK to A

#### Socket

The instantiation of an end-point in a potential TCP connection

- Instantiation : The actual implementation of something defined elsewhere
- Listen : A TCP socket is ready and listening for incoming connections
- SYN_SENT : A synchronization request has been sent, but the connection hasn't been established yet
- SYN_RECEIVED : A socket previously in a LISTEN state has received a synchronization request and sent a SYN/ACK back
- ESTABLISHED : The TCP connection is in working order and both sides are free to send each other data
- FIN_WAIT : A FIN has been sent, but the corresponding ACK from the other end hasn't been received yet
- CLOSE_WAIT : The connection has been closed at the TCP layer, but that the application that opened the socket hasn't released its hold on the socket yet
- CLOSED : The connection has been fully terminated and that no further communication is possible

#### TCP : Connection-oriented protocol

Connection-oriented protocol : Establishes a connection, and uses this to ensure that all data has been properly transmitted.

- At the IP or Ethernet level, if a checksum doesn't compute, all of that data is just discarded.
- It is up to TCP to determine when to resend this data since TCP expects an ACK for every bit of data it sends, it is in the best position to know what data successfully got delivered and it can make the decision to resend a segment if needed.

#### Firewall

Block specific ports

### Session Layer

- Facilitating the communication between actual applications and the transport layer
- Takes application layer data and hands it off th the presentation layer

### Presentation Layer

- Responsible for making sure that the unencapsulated application layer data is able to be understood by the application in question

### Application layer

- Protocol : HTTP, SMTP, etc...
- Protocol Data Unit : Messages
- Addressing : n/a

Allows these applications to communicate in a way they understand.

This payload section is actually the entire contents of whatever data applications want to send to each other. 