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