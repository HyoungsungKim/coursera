# Week2

## Lesson 1: Concurrency Control

### 1.1 RPCs

#### Why RPCs

- RPC = Remote Procedure Call
- Proposed by Birrell and Nelson in 1984
- Important abstraction for processes to call functions in other processes
- Allows code reuse
- Implemented and used in most distributed systems, including cloud computing systems
- Counterpart in Object-based setting is called RMI(Remote Method Invocation)

#### Local Procedure Call(LPC)

- Call from one function to another function within the same process
  - Uses stack to pass argument and return values
  - Each process has 1 stack
    - It means local procedure(functions) share same process address space, they can access common objects.
  - Accesses object via pointers(e.g., C) or by reference(e.g., Java)
- LPC has exactly-once semantics
  - If process is alive, called function executed exactly once

#### Remote Procedure Call

- Call from one function to another function, ***where caller and callee function reside in different processes***
  - ***Function call crosses a process boundary***
  - Accesses object via global references
    - ***Can't use pointers*** across processes since a reference address in process P1 may point to a different object in another processes P2
    - e.g., Object address = IP + port + object number
- Similarly, RMI(remote Method Invocation) in Object-based settings
- ***Notice that the remote procedure call here does not cross host boundaries***
  - Same host but different process
- ***If two processes might be on different hosts?***
  - In which case you might need messages that go over the network
  - one message carries the request which is carrying the argument of the call itself(RPC request message)
  - When the RPC is done the RPC reply message is then carried back(RPC reply message)

#### RPC Call Semantics

- Under failures, hard to guarantee exactly-once semantics
- Function may not be executed if
  - Request (call) message is dropped
  - Reply (return) message is dropped
  - Called process fails after executing called function
  - Hard for caller to distinguish these cases
- Function may be executed multiple times if
  - Request(call) message is duplicated

#### Implementing RPC Call Semantics

- Possible semantics
  - At most once semantics (e.g., Java RMI)
  - At least once semantics (e.g., Sun RPC)
  - Maybe, i.e., best-effort (e.g., CORBA)

#### Idempotent Operation

(연산을 여러번 적용해도 달라지지 않음)

- ***Idempotent operations are those that can be repeated multiple times, without any side effects***
- Examples (x is server-side variable)
  - x = 1;
  - x = (argument) y;
- Non-examples
  - x = x + 1;
  - x = x * 2;
- ***Idempotent operations can be used with at-least-once semantics***

#### RPC Components

Client

- Client stub : has same function signature as callee()
  - Allows same caller() code to be used for LPC and RPC
  - It allows the caller to call the callee function just like it was a local function
  - The caller does not need to know that the callee function is not residing in process P1
  - It is calling a function that just looks like a process calling function. Think if a clone that looks like you but is empty inside
  - But when someone tells something to the clone it immediately relays the message to you
- Communication Module : Forwards requests and replies to appropriate hosts

Server

- Dispatcher : selects which server stub to forward request to 
- Server stub : calls callee(), allows it to return a value

#### Generating Code

- Programmer only writes code for caller function and callee function
- Code for remaining components all generated automatically from function signatures (or object interfaces in Object-based languages)
  - e.g., Sun RPC system : Sun XDR interface representation fed into rpcgen compiler
- These components together part of a Middleware system
  - E.g., CORBA (Common Object Request Brokerage Architecture)
  - E.g., SUN RPC
  - E.g., Java RMI

#### Marshalling

> Marshalling이란 한 객체의 메모리에서 표현방식을 저장 또는 전송에 적합한 다른 데이터 형식으로 변환하는 과정이다
>
> 빅엔디안 : 값이 낮은 주소부터 높은 주소 순으로 저장
>
> 리틀엔디안 : 값이 높은 주소부터 낮은 주소 순으로 저장

- Different architecture use different ways of representing data
  - Big endian : Hex 12-AC-33 stored with 12 in lowest address, then AC in next higher address, then 33 in highest address
    - IBM z, System 360
  - Little endian : Hex 12-AC-33 stored with 33 in lowest address, then AC in next higher address, then 12
    - Intel
  - Caller (and callee) process uses its own platform-dependent way of storing data
  - Middleware has a common data representation(CDR)
    - Platform-independent
  - Caller process converts arguments into CDR format
    - Called "Marshalling"
  - Callee process extract arguments from message into its own platform-dependent format
    - Called "Unmarshalling"
  - Return Values are marshalled on callee process and unmarshalled at caller process
    - Caller, callee 둘 다 marshall, unmarshall 함

### 1.2 Transactions

#### Transaction

- Series of operations executed by client
- Each operation is an RPC to a server
- Transaction either
  - Completes and commits all its operations at server
    - ***Commit = reflect updates on server-side objects***
  - Or aborts and has no effect on server

#### Atomicity And isolation

- Atomicity : All or nothing principle : a transaction should either 
  - Complete successfully, so its effects are recorded in ther server object or
  - the transaction has no effect at all
- Isolation : Need a transaction to be indivisible (atomic) from the point of view of other transactions
  - No access to intermediate results/states of other transactions
  - Free from interference by operations of other transactions
- But...
- Clients and/or servers might crash
- Transactions could run concurrently, i.e., with multiple clients
- Transactions may be distributed, i.e., across multiple servers
- You want to preserve isolation while still maximizing

#### ACID Properties for Transactions

- Atomicity : All or nothing
- Consistency : if the server starts in a consistent state, the transaction ends the server in a consistent state
- Isolation : Each transaction must be performed without interference from other transactions, i.e., non-final effects of a transaction must not be visible to other transactions
- Durability : After a transaction has completed successfully, all its effects are saved in permanent storage

#### Multiple Clients, One Servers

- What could go wrong?

#### 1. Lost Update problem

Transaction T1

```pseudocode
time 00 : x = getSeats(ABC123);
time 01 : if (x > 1)
time 02 : 	x = x - 1;
time 03 : write(x, ABC123);
time 04 : commit
```

Transaction T2

```pseudocode
time 00 : //Start T1 transaction at time 00
time 01 : x = getSeats(ABC123);	//However T2 starts at time 01
time 02 : if(x > 1)
time 03 : 
time 04 : 
time 05 : 	x= x - 1;
time 06 : wrtie(x,ABC123);
time 07 : 
time 08 : commit
```

-> T1 or T2's update is lost

#### 2. Inconsistent Retrieval Problem

Operation result of T1 and T2 is different

How prevent it?

### 1.3 Serial Equivalence

#### Concurrent Transactions

- To prevent transactions from affecting each other
  - Could execute them one at a time at server
  - But reduces number of concurrent transactions
  - Transactions per second directly related to revenue of companies
    - This metric needs to be maximized
- Goal : increase concurrency while maintaining correctness(ACID)

#### Serial Equivalence

- An interleaving (say O) of transaction operations is serially equivalent iff(if and only if):
  - There is some ordering(O') of those transactions, one at a time, which 
  - Gives the same end-result (for all objects and transactions) as the original interleaving O
  - Where the operations of each transaction occur consecutively(in a batch)
- Says: ***Cannot distinguish end-result of real operation O from (fake) serial transaction order O'***

#### Checking for Serial Equivalence

- An operation has an effect on
  - The server object if it is write
  - The client (returned value) if it is a read
- Two operations are said to be ***conflicting operations***, if their combined effect depends on the order they are executed
  - read(x) and write(x)
  - write(x) and read(x)
  - write(x) and write(x)
  - Not read(x) and read(x) : swapping them doesn't change end-results
  - Not read/write(x) and read/write(y) : swapping them OK

#### Checking for Serial Equivalence

- Two transactions are serially equivalent if and only of ***all pairs of conflicting operations*** (pair containing one operation form each transaction) ***are executed in the same order***(transaction order) for all objects(data) they both access.
  - Take all pairs of conflict operations, one from T1 and one from T2
  - If the T1 operation was reflected first on the server, mark the pair as "(T1, T2)", otherwise mark it as "(T2, T1)"
  - All pairs should be marked as either "(T1, T2)" or all pairs should be marked as "(T2, T1)"

#### What is our Response?

- At commit point of a transaction T, check for serial equivalence with all other translations
  - Can limit to transactions that overlapped with T
- ***If not serially equivalent***
  - Abort T
  - Roll back (undo) any write that T did to server objects

#### Can we do better?

- Aborting -> wasted work
- Can you prevent violations from occurring?