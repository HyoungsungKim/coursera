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