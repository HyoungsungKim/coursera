# Week2

## Lesson 1: Virtualization

### Sharing resource

- Economic of Clouds requires sharing resources
- How do we share a physical computer among multiple users?
- ***Answer : Abstraction***
  - Introduce an abstract model of what a generic computing resource should look like
  - The physical computer resource then provides this abstract model to many users

### Layers of Abstraction

- Introduce an abstract model of what a generic computing resource should look like
- The physical computer resource then provided this abstract model to many users
- ***Virtualization avoids creating dependencies on physical resources***

### Virtualization: Foundation of Cloud Computing

- ***Virtualization allows distributed computing models without creating dependencies on physical resources***
- Clouds are based on Virtualization
  - offer services based mainly on virtual machines, remote procedure calls, and client/servers
  - provide lots of servers to lots of clients(e.g., phones)
- Simplicity of use and ease of programming requires allowing client server paradigms to be used to construct services from lots of resources

### Types of Virtualization

- Native, full
- Hardware assisted
- Para-virtualization
- OS level
  - containers
  - jails
  - chroot
  - zones
  - openVZ -> virtuozzo

### Native and Full Virtualization

- The virtual machine simulates enough hardware to allow an unmodified "guest" OS(one designed for the same CPU) to be run in isolation
- ***Run on hypervisor(VMM)***
- Hardware -> Hypervisor -> Guest OS -> Apps
- Examples:
  - VirtualBox
  - Virtual PC
  - ***Vmware***
  - QEMU

> Hypervisor : 호스트 컴퓨터 1대에서 다수의 운영체제를 동시에 실행 할 수 있도록 해주는 가상 플랫폼 기술

### Hardware enabled virtualization

- ***The virtual machine has its own hardware*** and allows a guest OS to be run in isolation
- Intel VT(IVT)
- AMD virtualization(AMD-V)
- Hardware -> Hardware VM -> Guest OS -> Apps
- Examples:
  - VMware Fusion
  - Parallels Desktop for Mac
  - Parallels Workstation

### Para-virtualization

- The virtual machine does not necessarily simulate hardware, but instead (or in addition) offers ***a special API that can only be used by modifying the "guest" OS***
- Hardware -> Hypervisor -> Stub -> Modified Guest OS -> apps
- we've produced Stubs between the actual Hypervisor and the modified operation system, the guest operation system.
- And those Stubs link all of the systems together with a controlled piece of software called a Hypervisor(VMM) and then all of that virtualization is basically interpreted by the Hypervisor, giving you independence for all the guest operating systems
- Examples:
  - XEN

#### Operating System-level Virtualization

- Virtualizing a physical server at the operating system level, ***enabling multiple isolated and secure virtualized servers to run on a single physical server.***
- Hardware -> OS -> Private Servers
- Examples:
  - Linux-Vserver
  - Solaris Containers
  - FreeBSD Jails
  - Chroot
  - CGroups

### 2.1.2 Virtualizing the Network and Storage

Virtualization is a very powerful technique to organize large data centers.

#### The "Software-defined Network"

1. Open interface to hardware
2. At least one food operating system that is extensible and possibly open source
3. Well-defined open API

- Software-defined network has been one of the steps that is really crucial in developing open stack in building big data centers
  - Because what is allows you to do is to virtualize the network

#### Trend Virtualized OS + Virtualized Network

Simple common stable hardware substrate below + programmability + string isolation model + competition above = faster innovation

- We have virtualized storage as a third leg of this data center, this virtualization going on for data centers.

## Lesson 2: OS Based Virtualization

 What we're going to find is that there's a range of virtualization mechanisms you can use. Some are much more efficient than others, some are much more secure than others. And which one you choose is really going to depend upon your application.

### 2.2.1 OS Based Virtualization

#### Types of Virtualization

- Native, full

- Hardware assisted

- Para-Virtualization

  

- OS level

  - Give you faster execution and they are thinner, lighter, than the para-virtualization and therefore execute faster

  - Containers

    - Actually leading contender nowadays when you are building a system and you want to decide what sort of virtualization to use 
    - They may not give you so much security
    - Containers really stick up their benefit and attract a lot of users

  - Jails

    - Old style unix mechanisms which prevented you

    > 관리자 권한 다른 사용자가 사용하지 못하게 막아 놓음

    

  - Chroot

  - Zones

  - Open-VZ -> Virtuozzo

#### Native and Full Virtualization

- The virtual machine simulate enough hardware to allow an unmodified "guest" OS(one designed for the same CPU) to be run in isolation

> 하드웨어를 자체적으로 가진 가상화
>
> -> vmware에서 guest OS 실행 할 때 하드위에 할당 함

- Examples:
  - VirtualBox
  - Virtual PC
  - Vmware
  - QEMU
  - Win4Lin
  - XEN/Virtual Iron

#### Hardware Enabled Virtualization

- The virtual machine has its own hardware and allows a guest OS to be run in isolation.
- Intel VT(IVT)
- AMD virtualization (AMD-V)

> 하드웨어 공유하는 가상화(가상 하드웨어 머신 사용)

- Examples:
  - VMware Fusion
  - Parallels Desktop for Mac
  - Parallels Workstation

#### Partial Virtualization

- The virtual machine simulate multiple instances of much (but not all) of an underlying hardware environment, particularly address spaces.

#### Paravirtualization

Native and Full virtualizing과 유사 하지만 guest OS 수정을 위한 특수한 API 제공

- The virtual machine does not necessarily simulate hardware, but instead (or in addition) ***offers a special API that can only be used by modifying the "guest" OS***
- Terminologies
  - Hypervisor, hypercall
  - Enomalism
- Examples:
  - XEN, KVM, Win4Lin(windows for linux) 9x

#### Operating System-Level Virtualization

- ***Virtualizing a physical server at the operating system level,*** enabling multiple isolated and secure virtualized servers to ***run on a single physical server***
- Examples:
  - Parallels Workstation
  - Linux-VServer, Virtuozzo
  - OpenVZ, Solaris Containers
  - FreeBSD Jails
  - Chroot?

#### Operating System-Level Virtualization

- Hypervisor(VM)
  - One real HW, many virtual HWs, many OSs
  - High versatility - can run different OSs
  - Lower density, performance, scalability
  - <\<Lowers>> are mitigated by new hardware features (such as VT-D)
- Containers(CT)
  - One real HW(no virtual HW), one kernel, many userspace instances
  - Higher density, natural page sharing
  - Dynamic resource allocation
  - Native performance:[almost]no overhead

#### Thinner Containers, Better Performance

- Containers
  - Share host OS and drivers
  - Have small virtualization layer
  - Naturally share pages
- Hypervisors
  - have separate OS plus virtual hardware
  - hardware emulation requires VMM state
  - Have trouble sharing guest OS pages
- ***Containers are more elastic than hypervisors***
- Container slicing of the OS is ideally suited to cloud slicing
- ***Hypervisors' only advantage in IaaS is support for different OS families on one server***

### 2.2.2 Xen

#### Xen 3.0 Guest VM

What you can see is that most of the application space is pretty much what you would expect from a Unix.

Each of instances, guest VMs, are going to have a root. ***And those roots aren't going to interfere with each other which of course they would've done if you'd put them on a single box with Linux***

> 포트가 독립적으로 사용됨 -> 다른 사용자는 접근 불가

What you see is happening is that the guest OS when it is talking to the drivers

I/O Path

- Process to Guest OS
- Guest OS to IDD

Resource control

- map virtual Devices
- CFQ(Completely Fair Queuing) for disk
- HTB(Hierarchical Token Buckets) for network

Schedules All VMs

- Guest VM & IDD scheduled
- Two levels scheduling in guest

Resource control of Hypervisor

- Allocate resources
- Schedule VMs

Security isolation of Hypervisor

- Access physical level
  - PCI address
  - Virtual Memory

### 2.2.3 VServer 2.0

#### VServer 2.0 Guest

Resource Control

- map Container to
  - HTB for network
  - CFQ for Disk
- Logical Limit
  - Processes
  - Open FD
  - Memory Locks

Security Isolation

- Access to Logical Objects
  - Context ID Filter
  - User IDs
  - SHM & IPC address
  - File system Barriers

I/O Path

- Process to COS
- I/O paths are individualized to each of the different processes and threads in their containers so that you can't actually see from one container to another

Scheduler of container OS

- Single level
- Token Bucket Filter preserves O(1) scheduler
- When it does write, it would write to a different place in the disk preventing interference between the different virtualization

Optimizations

- File-level Copy-on-write

Logical virtualization(VServer) is better than physical virtualization(Xen) because physical virtualization use more resource

### 2.2.4 Solaris Zones

#### Solaris Zones

- Zones provide separate virtualized operating system environments that are derived from a Global Zone
- Multiple zones can share file systems, processors, and network interfaces
- Scaling and sharing can be configured on as as-needed basis
- Individual zones gain files and configurations from the Global Zone

There is some benefits a little bit more control perhaps than the Linux world, and the whole thing taken into a Unix world

#### Types of Zones

- Global Zone
  - All Solaris 10 installations contain a Global Zone
  - Only the Global Zone is bootable from the system hardware.
  - The Global Zone contain the complete installation of Solaris, and can contain additional software not installed via packages
- Local Zones
  - Local Zones contain a subset of the complete operating system, and can contain non-shared packages
  - Local Zones have no awareness of other zones.
  - A Local Zone cannot install, manage, or uninstall itself or any other zone.

#### Zone Daemons

- zoneadmd
  - Manages zone booting and shutting down
  - Allocates zone ID and starts the zsched process
  - Sets zone-wide resource controls
  - Allocates devices, including plumbing the virtual interfaces for the zones
  - Manages file-systems including sharing
- zsched
  - Zsched manages thread management per zone.
  - Kernel threads doing work on behalf of the zone are owned by zsched.

#### Zone File Systems

- Sparse Root Model
  - Minimal number of files from the global zone
  - Shared files mounted via read-only loopback file systems
- Whole Root Model
  - No dependency on share file-systems
  - Allows superior customization
  - Local zones cannot be NDF servers!

#### Zones Networking

- Zones have visibility to each other via network interfaces
- Only the Global Zone Administrator can modify the interface configuration and routes
- IPMP is configurable in the Global Zone, and IPMP can be extended to Local zones, allowing failover in the event of an interface failure

## Lesson 3: Containers

### 2.3.1 Docker

- Containers are very useful in building applications, in building parallelism and getting things to switch from to process very fast.
- They are not so good not security, not as good as the virtual machines, however ***they are actually sort of dominating a lot of the way that clouds get used***

#### Overview

- "Docker containers wrap up a piece of software in a complete file-system that contains everything needed to run: code, runtime, system tools, system libraries-anything you can install on server. ***This guarantees that the software will always run the same, regardless of its environment it is running in.***"
- Docker automates the deployment of applications inside software containers
- ***Additional layer of abstraction and automation of operating system level virtualization*** on Linux

#### Basic of Docker

Source code repository

- You are integrating that into a Docker File that contains the file system, the libraries, the interfaces and so on
  - There is a bunch of source code repositories that are used to help build.
  - They will be on your sort of build system, your development environment

source code repository -> Build and create in Docker engine(Developer Linux Host) -> container contain all this parts -> transfer(push) to Docker container Image Registry -> pull to Destination Linux Host

Docker Container Image Registry

- People can actually look up that particular component and pulled it down to whichever of the cluster nodes that they want it to run on or maybe all of them
  - Destination Linux host --search--> Docker Container Image Registry
  - Docker Container Image Registry --Pull Run--> Destination Linux host
- You can do it under program controller, but pull this stuff down and run associate it with your Linux and it is going to run on top of your Linux almost like another VM
- It is running at user level, so working you have your threads all operating asynchronously.

#### Changes and Updates

Docker Container Image Registry let container can update and upgrade

- Container = App + (Bins & Libs)
  - 업데이트가 있으면 Bins & Libs 만 교체하여 App 업데이트 가능
- Docker system provides a mechanism to simplify the whole distribution of the docker containers and completely automates the way that operates.

### 2.3.2 Kubernetes

This is a very cool development on top of Docker, on top of containers. How do you build sort of complicated systems? Kubernetes will help you

#### Overview

- Kubernetes provides a "platform for automating deployment, scaling, and operations of application containers across clusters of hosts"
- Let's suppose you are doing stream processing and you have five streams and you want to allocate them through a thousand hosts in some way
  - You want to do load balance and all these other good things
  - That is where Kubernetes comes in and it actually provides you a tremendous way of doing this without working too hard

#### Parts

Kinds of interesting to think about how you develop from docker and containers to these parts

One of the issues when you are building these things is it's built on a distributed system

- Loosely coupled building blocks

  - You push those building blocks to talk to other building blocks in other nodes in a loosely coupled way,
  - effectively ignoring the fact that you are going from one system to another

- Pods

  - ***Pod is multiple of dockers,*** but pod restricts and constrains what you can do with all of those containers as a unit
  - ***Unique IP address*** of a collection of co-located(in the same machine or in the same docker) containers
  - No(prevent) collisions on IP port address
    - Effectively sort of making each of those containers a separate entity that you can address with IP addresses and so on
    - SO you don't have to worry about allocating the same IPs
  - It provides a volume(network disk or local directory) shared by containers in a pod
    - It is not going to interfere with other pods
  - What it is going to go is to provide you a way of thinking about this is just your own environment that can't be interacted with by other people, that you can actually know what is going on inside the system.
  - So you've got unique IP port addresses, but naturally enough you'd like to do is also have some sort of search capability, some sort of find me a pod, find me a particular thread inside the container and all this type of request. -> There is a nice interesting abstraction: ***Labels and Selectors***

- Labels and Selectors

  - Key-value pair (label) that resolves (selector) to a pod or node component or container
  - You can give it as a label and then ***use that label to find that particular element without having to know the IP address or knowing where it is***
    - Label을 DNS 처럼 사용하는 듯...?
    - You can actually use that Labels and Selectors to find it
  - Selector
    - Selector is the sort of like query element
    - It is searching through, looking for that particular entity
    - ***It is sort of really mapping whatever you want,*** whatever space of names you those labels on to the actual implementation

  - Controllers
    - Manages pods :
      - Replication Controller (replacement)
        - For example) I want this pod but i am in danger of that perhaps i might just flood this plant and it may die for some strange memory management reason, so i want to make multiple copies of it
      - Daemon Set Controller (Keeps one pod on every machine in a set running)
        - You can set Daemon set controller how it allocates those pods on machines to avoid problems
      - Job controller(runs "batch" pods to completion)
        - A lot of jobs want to run batch you want settle set up this job start it running, go away, later on perhaps come back to collect the result, you don't want to sort of supervise it - job controller does that instead you
        - What it will do is batch pods and then run them to completion so it effectively gives you that type of environment where you can sort of set it in motion go have coffee when it is finished come back collect the final finished product
        - You can of course batch pods one after the other so that one pod builds things that another pod uses and so on
  - Services
    - Set of pods working together
      - you have services inside the pods that the other pods want to talk to
      - So if you like when you are trying to do inter-pod interaction the services enable this(Label selector)
    - Label selector
      - Select the actual pod that you want of selecting
    - Service discovery and request routing - stable IP address and DNS name
    - Round robin load balancer to network IP address

## Lesson 4: Infrastructure as a Service (IaaS)

### 2.4.1 IaaS: OpenStack

#### OpenStack

- "The OpenStack" project has been created with the audacious(대담한) goal of being the ubiquitous software choice for building cloud infrastructures."
- "OpenStack is a collection of open source software projects that enterprises/service providers can use setup and run their cloud compute and storage infrastructure"

#### OpenStack architecture

- Dashboard : the dashboard talks to all of the various elements
  - Controller on the dashboard, they are going to map to the compute networking storage elements that you require for you OpenStack
  - When application is no longer needed, you can just give it to somebody else
  - If you want to bring it back later you can do the same

#### OpenStack's Core Components

- Compute ("Nova")
  - Orchestrates large network for Virtual machines
  - Responsible for VM instance lifecycle, network management, and user access control
- Object Storage ("Swift")
  - Provide scalable, redundant, long-term
  - Storage for things like VM images, data archives & multimedia
- Image Service("Glance")
  - Manages VM disk images
  - Can be stand-alone service
  - Support private/public permissions, and can handle a variety of disk image formats

### 2.4.2 IaaS Providers: Amazon

### Amazon Web Services

- AWS provides a collection of services for building cloud application
- Services for :
  - Storage : S3, EBS
  - Computation : Elastic Cloud Computing(EC2), scaling/load balancer, Elastic MapReduce, Elastic Beanstalk
  - Database : RDS, DynamoDB, ElastiCache
  - Coordination: Simple Notification Service, Simple Workflow Framework
- All services are paid depending on use

- Amazon EC2 : Resizable compute capacity in the Cloud
- Amazon DynamoDB : Fast and flexible NoSQL database with seamless scalability
- AWS Lambda : Compute service that runs your code in response to events and automatically manages the compute resources
- Amazon S3 : Highly scalable, reliable and low-latency data storage infrastructure.

#### 6 Types of Instances

- Micro Instance(Free Tier)
- General Purpose
- Memory Optimized
- Storage Optimized
- Compute optimized
- GPU Instances

#### Storage

- Transient, instance-specific storage
- Persistent, instance-independent Elastic Block Store(EBS) storage (SSD and encryption options)
- Object-based Simple Storage Service(S3)
- Data restricted to region

#### Networking

- Virtual Private Cloud(=VPN)
- Private routing between VPCs
- VPN tunnels can connect your enterprise to Amazon
- DirectConnect allows customer to use carrier for private WAN services

### 2.4.3 IaaS Provider: Microsoft

#### Microsoft

- Cloud first, mobile first
- Virtualization provided by Hyper-V to rival VMware
- Microsoft Azure is IaaS and PaaS
- Office 365 and Office for iPad
- SharePoint
- Yammer (social and collaboration)
- Exchange (primary competitor Google gmail)
- Dynamics CRM

#### Microsoft Azure

- It was launched by Microsoft in 2010
- Provides both PaaS and IaaS services
- It is like a hybrid cloud provider that tries to do multiple things

#### Uses of Azure

- Can be used for anything since it provides IaaS services that can host virtual machines
- however, its PaaS services have been known to host web sites that may receive a lot of traffic
- Food for .NET developer

#### Azure Cloud

- Microsoft developed their own operating system called Windows Azure that is used for their datacenter cluster
- Uses hyper-V, a windows server Hypervisor that can run virtual machines

#### Windows Server

- Has support for Windows server
- Can provision and manage virtual machines
- Can attach and manage disks

#### Windows Azure

- ***Windows Azure is the OS for the data center***
  - Model : treat the data center as a machine
  - Handles resources management, provisioning, and monitoring
  - Manages application lifecycle
  - Allows developer to concentrate on business logic
- Provides shared pool of compute, disk and network
  - Virtualized storage, compute and network
  - Illusion of boundless resources
- Provides common building blocks for distributed applications
  - Reliable queuing, simple structured storage, SQL storage
  - Application services like access control and connectivity

#### Modeling Cloud Applications

- A cloud application is typically made up of different components
  - Front end : e.g. load-balanced stateless web servers
  - Middle worker tier : e.g. order processing, encoding
  - Backend storage : e.g. SQL tables or files
  - Multiple instances of each for scalability and availability

#### The Windows Azure Service Model

- A windows Azure application is called a "service"
  - Definition information
  - Configuration information
  - At least one "role"
- Roles are like DLLs in the service "process"
  - Collection of code with an entry point that runs in its own virtual machine
- There are currently three role types:
  - Web Role : IIS7 and ASP.Net in Windows Azure-suppkued OS
  - Worker Role : arbitrary code in Windows Azure-supplied OS
  - VM Role : uploaded VHD with customer-supplied OS

#### Role Contents

- Definition:
  - Role name
  - Role type
  - VM size(e.g. small, medium, etc.)
  - Network endpoints
- Code:
  - Web/Worker Role : hosted DLL and other executables
  - VM Role : VHD
- Configuration:
  - Number of instances
  - Number of update and fault domains

### 2.4.5 Serverless Architecture

What does serverless architecture mean?

- What we really mean by serverless architecture is ***pretty much an extension of the platform as a service*** sort of model that we've talked about in previous videos
- Remember in previous videos we say for platform as a service, the cloud provider would provide you with a certain platform
- It kind of encompasses(둘러싼) infrastructure, not infrastructure, platform as a service

### Introduction to Serverless Architecture

- "Application where some amount of server-side logic is still written by the application developer but unlike traditional architectures is run in stateless compute containers that are event-triggered, ephemeral (may only last for one invocation), and fully managed by a 3rd party"
- Functions as as service / FaaS
  - The idea here is that your application is written in terms of very small chunks, just one function,
  - And then the infrastructure would run your function in response to a certain event
- Let me quickly bring up a container-based ephemeral set of resources so that i can quickly run this FaaS
- ***Containers are the foundational technology behind the serverless architecture***
  - Very quickly bring up
  - Very quickly get rid of
- ***AWS Lambda*** is one of the most popular implementations of FaaS at present, but there are others
  - Lambda is game changer

### AWS Cloud Platform 2016

- With the serverless architecture, you have this layer implemented on top f all the clouds services that are implemented before
- So we have something called ***Lambda***
- Now you don't need to interact with infrastructure as a service, which was what?
  - EC2 and elastic load balancer, and whatnot, which you can still use, but you don't need to anymore
  - Now you don't quite even need to use platform as a service like Elastic Beanstalk and whatnot
- Now you just write a function and you let the cloud platform figure out everything about the function
  - How to scale it, when to scale it, where the input data comes from, how the users' requests get routed to your function, everything is pretty much now handled by the cloud infrastructure
- Lambda is quite a game changer now in 2016 because not only you don't need to have your resources, you go to the cloud. you don't need to even get an EC2 instance and pay per hour.
  - You write your function and pay per instance call of your function, which is a very interesting model

### AWS Elastic BeanStalk

Before jumping into Lambda, talk about Elastic BeanStalk

- Deploy and scale web application easily
- Simply upload your code

### AWS Lambda Event-driven Compute

Lambda is a complete event driven compute

> state : 상태를 저장하고 있음. 따라서 이전에 어떤 일이 일어났는지를 포함하고 있음

- Whole Lambda infrastructure just the function, that five lines of code or something. ***This function should be stateless.***
  - So basically the function should not keep track of what it has done before
- Runs stateless, request-driven code called Lambda functions in Java, NodeJS & Python
- Triggered by events (state transitions) in other AWS services
- Pay only for the request served and the compute time
- ***Focus on business logic, not infrastructure.***
- Just upload your code; AWS Lambda handles

### AWS Lambda Execution Environment

- State-less functions
- You can use multi-threading
- Function should finish in a certain time
  - Default 3 seconds, up to 300 seconds