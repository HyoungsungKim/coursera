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