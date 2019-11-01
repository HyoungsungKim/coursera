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

### Transport  layer

- Protocol : TCP/UDP
- Protocol Data Unit : Segment
- Addressing : Port #'s'

### Application layer

- Protocol : HTTP, SMTP, etc...
- Protocol Data Unit : Messages
- Addressing : n/a

