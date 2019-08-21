# Orientation Overview

## Introducing to Cloud Computing Concepts

### Goal Of C3 - Part2

- Course about the internals of cloud computing
- Distributed systems and algorithms that underlie today's cloud computing technologies
- We will discuss
  - Concepts
  - Techniques
  - Industry systems, including open-source (from the inside)
- Classical algorithms : Leader Election, Mutual Exclusion, Scheduling
- Scalability : Concurrency control, Replication Control
- Trending Areas : Stream processing, Graph processing, Structure of Network, Sensor Network

##  Some Basic Computer Science

### 2. Process  = A Program in action

When we talk about distributed systems, we are going to talk of multiple processes communicating with each other. And we will mostly focus on a process to process communication with each other. And we will also talk about procedure calls or function calls can cross process boundaries

In addition, the other reason we are discussing a process sis that when you take a snapshot of a process, when i say a process takes its own snapshot, this includes many things

### 3. Computer Architecture (simplified)

faster ---- cpu with register <-> cache <-> memory <-> Disk ---- slower

- A program you write (C++/Java etc.) gets compiled to low-level machine instructions
  - Stored in file system on disk
- CPU loads instructions in batches into memory (and cache, and registers)
- As it executes each instruction, CPU loads data for instruction into memory(and cache, and register)
  - And does any necessary stores into memory
- Memory can also be flushed to disk
- This is a highly simplified picture (but works for now)

### 5. DNS

- DNS = Domain Name System
- Colelction of servers, throughout the world
- Input to DNS :  a domain name, e.g., coursera.org
  - Domain name is a name, a human-readable string that uniquely identifies the object
  - Can be derived from the URL (e.g., http://class.coursera.org/)
- Output from DNS : IP address of a web server(that hosts that content)
  - IP address is an ID, a unique string pointing to the object. May not be human readable
  - IP addresses change but names don't
- IP address may refer to either
  - Web server actually hosting that content, or
  - An indirect server, e.g., a CDN (Content distribution Network) server, e.g., from Akamai