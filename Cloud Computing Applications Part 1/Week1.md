# Week 1 - Module 1

## Lesson 1: Cloud Computing

### Perils of corporate Computing

- Own information systems
- However
  - Capital invesment
  - Heavy fixed costs
  - redundent expenditures
  - High energy cost, low CPU utilization
  - Dealing with unreliable hardware
  - High levels of overcapacity (technology and labor)
- Not Sustainable

### Cloud Computing

What is a cloud?

- Anybody can use it from anywhere
  - Availability is good
  - Accessibility is good
- On-Demand Network Access

### Delivery Models

How the services of the clouds get delivered to the customer

- Software as a Service (SaaS)
  - use provider's applications over a network
  - SalesForce.com
- Platform as a Service (PaaS)
  - Deploy customer-created appications to a cloud
  - AppEng
- Infrastructure as a Service (IaaS) - hardware
  - Rent processing, storage, network
  - Capacity and other fundamental computing resources
  - EC2, S3

#### Big Data Data revolution and Clouds

- Data collection too large to transmit economically over internet == Petabyte data collections
- Computation is Data Intensive
  - Lots of Disks, Networks and CPUs
  - Overhead of maintaining cyber infrastructure is expensive
  - Users buy Big Data services from Clouds to share overhead
  - ***Easy to write programs, fast turn around***
  - MapReduce - Hadoop, PIG, HDFS, HBase

### 1.1.3 Cloudnomics: Part1

#### Cloudnomics: Part1

Economics necessitates Cloud Computing:

- Part 1: utility Pricing
- Part 2: Benefits Common Infrastructure

#### Value of Utility Pricing

- Cloud services don't need to be cheaper to be economic!
- Consider a car
  - Buy or lease for $10 per day
  - Rent a car for $45 a day
  - if you need a car for 2 days in a trip, buying would be much more costly than renting
  - ***It depends on the demand***

#### Utility Pricing in real World

- In Practice demands are often highly spiky
  - News stories, marketing promotions, product launches, Internet flash floods (Slashdot effect), tax season, Christmas shopping, etc.
- Often a hybrid model is the best
  - You own a car for daily commute, and rent a car when traveling or when you need a van to move
  - But we should also consider other costs
    - network cost (both fixed costs and usage costs)
    - Interoperability overhead
    - Consider Reliability, accessibility

### 1.1.5 Big Data

#### Big Data(A singular phrase!)

- A collection of data sets so large and complex, it's impossible to process it on one computer with the usual database and tools
- Because of its its and complexity, Big Data is hard to capture, store, copy, delete(privacy), search, share, analyze and visualize

Big Data represents the information assets characterized by such a High

- Volume,
- Velocity and
- Variety

to require specific Technology and Analytical Methods for its transformation into Value

## Lesson 2: Introduction Part 2

### 1.2.1 Software Defined Architecture

#### Learning Object

- How services are created
- How services can control other services
- The principal architecture components of a cloud and their organization
- How services and orchestration play a role in each layer of a production cloud: IaaS, PaaS, SaaS

#### Software Defined Architecture

- Cloud provides services, service orchestration, and provisioning
- A Cloud may provide IaaS, PaaS, SaaS and have both internal and external Application Programming Interfaces
- ***The mechanisms and concept of providing services, orchestration, and provisioning is called a Software Defined Architecture***
- A Cloud may contain other software defined entities:
  - Software Defined Network
  - Software Defined Storage
  - Software Defined Computer
- Cloud can be used like concatenated service
  - IaaS -> PaaS ->SaaS

#### Orchestration

Cloud service orchestration is the :

- Composing of architecture, tools and processes used by humans to deliver a defined Service
- Stitching of software and hardware components together to deliver a defined Service
- Connection and Automating of work flows when applicable to deliver a defined Service.
- Provides : up and down scaling, assurance, billing, workflows

#### Software Defined Architecture

- services : services are completely abstract. They might be providing application to user. They might be providing the service provider some way to control what his service is.
- Cloud provider : control the allocation of resources and the smooth operations, the maintenance, and so on of that cloud system.
  - All of those come in through APIs, and because of the complexity of all of this, ***because you want customers just to know about their particular piece of the Cloud, and you'd like the Cloud providers to know about all of the Cloud.***(추상화 필요)
- ***What we do is to employ virtualization and access control to actually give them those abstractions so they can manage it.***
- Underneath that layer, There is ***Service Logic,*** to provide the services.
  - ***There is provider logic and there is cloud logic***
  - All trying to provide different aspect of the totality of what the cloud offers
- Cloud Provisioning Middleware : ***A middleware layer offers that type of infrastructure,*** and its design we will touch upon in this course, but not delve too deeply because we are really concerned with applications.
  - The middleware matches all these services with the hardware, software, and data that is on the system.
  - Cloud provider is going to actually offer control, provide resource allocation and access control.
  - That is going to interact with mechanisms to actually allocate the resource
- (High layer) Service -> virtualization -> service logic -> cloud provisioning Middleware -> hardware, software and data (Low layer)

#### Content and Learning Objectives

- ***Virtualization is a key abstraction in building software defined architectures:***
  - Software Defined Networks
  - Software Defined Storage
  - Software Defined Compute
- ***Web Service : A Simple Application build on a Data Center***
- Load Balancing : A simple scheme to distribute the load of multiple servers
- Infrastructure as a Service
- Mirantis and OpenStack
- How systems are structured with orchestration

### 1.2.2 Cloud Services

#### Objective

- Compare IaaS, PaaS, and SaaS
- Look at what services major Cloud companies provide and how they provide them

#### IaaS, PaaS, and SaaS Comparision

- Package Software
  - You manage : Application, Data, Runtime, Middleware, O/s, Virtualization, servers, Storage, Networking
- IaaS
  - You manage : Application, data, Runtime, Middleware, O/S
  - Managed by vendor : Virtualization, Servers, Storage, Networking
- PaaS 
  - You mange : Applications, data
  - Managed by vendor : Runtime, Middleware, O/S, Virtualization, Servers, Storage, Networking
- SaaS
  - You manage : X
  - Managed by vendor : Application, Data, Runtime, Middleware, O/S, Virtualization, servers, Storage, Networking

#### Cloud Fundamentals

- Infrastructure as a Service (IaaS) : basic compute and storage resources
  - On-demand servers
  - Amazon EC2, VMWare vCloud
- Platform as a Service (PaaS) : cloud application infrastructure
  - On-demand application-hosting environment
  - E.g. Google AppEngine, Salesforce.com, Windows Azure, Amazon
- Software as a Service(SaaS): cloud applications
  - on-demand applications
  - e.g. Gmail, Microsoft Office Web Companions

#### Platform as a Service(PaaS)

- PaaS is a cloud computing service that offers platform for users to run applications on the cloud
- ***It is a level above infrastructure as a service(IaaS)*** because unlike IaaS, PaaS does not require users to develop their own operating system environment

#### Multi-tenancy

- PaaS is better suited for multi-tenancy because the PaaS provider optimizes its infrastructure for use by many providers
- ***Multi-tenancy means that many users may share the same physical computer and database***
- PaaS is better suited for multi-tenancy than an IaaS. because
  - Provide each user with his own virtual machine 
  - Create a clear separation of resources
  - Don't need to switch from on VM to another VM
- ***However, in a PaaS, users may share the same machine, database, etc***
  - As many users are there, It will be slow(Trade off)

> 방 하나 만들어 놓고 거기에 사용자들 다 집어 넣음
>
> - 방을 하나만 관리하면 됨
> - 하지만 사용자가 많아지면 공간이 부족해짐

#### Vendor Lock-in

- PaaS may look in applications by requiring users to develop apps using proprietary interfaces and languages
- This means that it may be difficult of users to go to another vendor to host their app
- ***Businesses may risk their future on the dependability of the PaaS***

> 플랫폼에 종속 될 위험 존재

#### Development Tools

- Often, a PaaS will browser-based development tools
- In this way, developers can create their own applications Online
- Ease of development : the platform takes care of the scaling for you

#### Principles of Software Development

- As a developer, your objective is to create an application in the quickest, most effective way possible
- You should not create applications using convoluted methods that may take a long time to complete
- The user only sees the end product, not the development process

#### PaaS vs IaaS

- When you use the Cloud, remember that your decisions have long-term consequences
- If you choose to use a PaaS and get your application vendor locked in, then your business may fail if the PaaS greatly increase the vendor's prices
- You will not be able to move to another Cloud since your app cannot be easily migrated to somewhere else

> 플랫폼에 족속됐는데 플랫폼 사용 비용 증가하면 사업 망함

### 1.2.3 Infrastructure as a Service

#### Infrastructure as a Service

- Xen
- Xencloud
- OpenStack

#### Xen and the Linux Kernel Distributions

- Xen was initially a university research project
- Invasive changes to the kernel to run Linux as a paravirtualized guest
- Even more change to run Linux as dom0
- Xen support in the Linux kernel; no upstream
- Great maintenance effort on distributions
- Risk of distributions dropping Xen support
- Xen hard to use

#### Marantis Fuel and OpenStack Architecture

- Mirantis provides a complete packaged solution you could download on your laptop if you like, but it will work on bare hardware too
- Image = controller, compute, boot server, network OS, storage OS
- ***Dashboard for controlling for which image you want to actually run.***
  - Dashboard is connected into what they call a controller
- The algorithm, the component that runs all this, is call Fuel and Fuel actually exists for beyond Mirantis in other sort of OpenStack implementations.
  - What fuel is to connect your hardware, to virtual machines.
  - ***So, Fuel runs as a controller in that control space across your system.***
- And through the Dashboard, what it will allow you to do is load and run images that are stored from a disk. ***and they can be computer images.***
  - They can be things like controllers.
  - They can be boot servers.
  - They can be network operating systems to control the network.
  - Thy can be storage operating systems to control the storage.

>Dashboard <-> Fuel controller -> Boot server(<- images) 
>
>-> Open Stack compute, Node, Node...

#### Open stack Architecture

- OpenStack is talking to the OpenStack controller, which is now fully operating as control, as opposed to data for everything. 
- And now the OpenStack Controller allows you to bring up other nodes like Network, Computer, and Storage, all off the same boot server, but now different types of images.
- If you want the Network operating system to control
  - then the switches and routers to give you a network virtualization, that would be one.
- If you want a compute server to provide Ubuntu, then you would bring down the Ubuntu operating system.
  - Put that onto a compute server and create that.
  - if you wanted your storage you would do the same thing,
  - All the different pieces of your virtual data center now on the hardware
- Eventually ***what you're going to do is, to talk to the Network operating system***
  - You're going to publish some IP addresses that the users can actually use to access the compute and storage of your virtual data center. And so the story goes. 
  - So you've taken, if you like, a virtual description of a data center, this OpenStack description, and what you've done is, to map that onto hardware and instantiated it. 
- And all of that mechanism has been allowed because of this infrastructure as a service from Mirantis, which has it's own front-end.
  - It allowed you to allocate machines. Now because you want to automate this.
  - you are allowed to store scripts for all these different operations. So if you want to repeat this, you can do so very readily.
  - If you want to scale it, you will say you want to add processes or storage to it, you can do so very easily. 

#### Mirantis Example

- Mirantis brings up a Fuel controller
- The Fuel controller allows one to bring up OpenStack controller, Network, Compute Servers, and Storage

