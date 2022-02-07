# Introduction

## What's Canary?
Canary is a simple distributed systems communication framework.

It provides a simple interface for you to design and code without having
to worry about the communication backend and more.

Unlike most distributed systems libraries, Canary has object-stream-based communication
instead of rpc-based communication.

This means that Canary is usually more performant than
other similar solutions such as gRPC and is comparable to websockets (even though the WASM target uses websockets)
and native sockets, but offers a much better API.

## Core principles
Canary's core principles are a high-level of abstraction and minimalism,
while keeping configurability and flexibility. Canary also strives
to be a zero-cost abstraction, being built around the principle of atomicity (the concept
of atomicity is extremely important to the philosophy of Canary).

Canary is not opinionated on project structure and only provides tools to aid
in creating distributed systems.

Canary strives to be an abstraction over the underlying communications protocols,
and should provide an experience similar to programming to the interface, and not the
implementation.


Still, this library may be a little too low-level for *some* use cases.

Should your project use Canary?
Canary fits these use cases perfectly.

- You want to build a library for distributed systems
- You need to build a communications library that can run on both the browser and native platforms
- You want to build an RPC system or similar
- You want to build a distributed system and you need extreme amounts of control
- Anything that's got to do with distributed systems or communications

**DISCLAIMER**

Canary is not yet on 1.0, which means that the API is prone to changes. Nevertheless,
the concepts should be stable **BUT** they are still prone to changes.
