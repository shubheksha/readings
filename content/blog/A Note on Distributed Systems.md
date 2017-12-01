+++
title = "A Note on Distributed Systems"
description = "This post distills the material presented in the paper titled [**“A Note on Distributed Systems”**](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.41.7628) published in 1994 by Jim Waldo et&nbsp;al."
tags = [
    "rpc",
    "distributed-systems",
    "oop",
]
date = "2017-09-18"
+++

### A Note on Distributed Systems: A&nbsp;Summary

![](https://cdn-images-1.medium.com/max/1024/1*tYxWuyksovxA1Thu8PggPQ.jpeg)

This post distills the material presented in the paper titled [**“A Note on Distributed Systems”**](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.41.7628) published in 1994 by Jim Waldo et&nbsp;al.

The paper presents the differences between local & distributed computing in the context of Object Oriented Programming, explaining why treating them to the be same is incorrect and leads to applications that aren’t robust or reliable.

#### Introduction:

The paper kicks off by stating that the current work in distributed systems is modeled around objects, more specifically, a **unified view of objects:** objects are defined by their supported interfaces & the operations they support. Naturally, this can be extended to imply that objects in the same address space, or in a different address space on the same machine, or on a different machine all behave in a similar manner: their location is an implementation detail.

Let’s define the most common terms in this&nbsp;paper:

#### Local Computing:

It deals with programs that are confined to a single address space&nbsp;only.

#### Distributed Computing:

It deals with programs that can make calls to objects in different address spaces either on the same machine or on a different machine.

#### The Vision of Unified&nbsp;Objects:
> Implicit in this vision is that the system will be “objects all the way down”; that is, that all current invocations or calls for system services will be eventually converted into calls that might be to an object residing on some other machine. There is a single paradigm of object use and communication used no matter what the location of the object might&nbsp;be.

This refers to the assumption that all objects are defined only in terms of their interfaces. Their implementation which also includes location of the object (remote/local), is independent of their interfaces and is hidden from the programmer. Hence, as far the programmer is concerned, they write the same type of call for every object, whether local or remote and the system takes care of actually sending the message by figuring out the underlying mechanisms which aren’t visible to the programmer writing the application.

> The hard problems in distributed computing are not the problems of how to get things on and off the&nbsp;wire.

The paper goes on to define what are the toughest challenges of building a distributed systems:

1. Latency
2. Memory Access
3. Partial failure & concurrency

Ensuring reasonable performance while dealing with all of the above doesn’t make the life of the a distributed systems engineer any easier. Moreover, the lack of any central resource or state manager just adds on to the various challenges. Let’s observe each of these one by&nbsp;one.

#### Latency:

This is perhaps the fundamental difference between local & distributed object invocation. The paper claims that a remote call is four to five times slower than a local call. If the design of a system fails to recognize this fundamental difference, it is bound to suffer from serious performance problems especially if it relies heavily on remote communication. A thorough understanding of the application being designed is required to decide which objects should be kept together and which can be placed remotely.

If the goal is to unify the difference in latency, then we’ve two&nbsp;options:

- Rely on the hardware to get faster with time in order to eliminate the difference in effeciency
- Develop tools which allow us to visualize communication patterns between different objects and move them around as required. Since location is an implementation detail, this shouldn’t be too hard to&nbsp;achieve.

#### Memory:

Another difference that’s very relevant to the design of distributed systems is the pattern of memory access between local and remote objects. A pointer in the local address space isn’t valid in a remote address space. We’re left with two&nbsp;choices:

- The developer must be made aware of the difference between the access&nbsp;patterns
- The system handles all memory&nbsp;access

To unify the differences in access between local & remote access, we need to let the system handle all aspects of access to memory. There are several way to do&nbsp;that:

- Distributed shared&nbsp;memory
- Using the OOP paradigm, compose a system entirely of objects, i.e., dealing only with object references. The transfer of data between address spaces can be dealt with by marshalling & unmarshalling the data by the layer underneath. This approach, however, makes the use of address-space-relative pointers obsolete.
> The danger lies in promoting the myth that “remote access and local access are exactly the same” and not enforcing the myth. An underlying mechanism that does not unify all memory accesses while still promoting this myth is both misleading and prone to&nbsp;error.

Hence, it’s important for programmers to be made aware of the various differences between accessing local & remote objects so that they don’t get bitten by not knowing what’s happening under the&nbsp;covers.

#### Partial failure & concurrency
> Partial failure is a central reality of distributed computing.

The paper argues that while both local & distributed systems are subject to failure, it’s a lot harder to discover what went wrong in case of distributed systems. For a local systems, either everything is shut down or there is some central authority which can detect what went wrong (the OS, for example).

However, in the case of a distributed system, determining what has failed is extremely difficult since there is no global state or resource manager available to keep track of everything happening in and across the system, hence there is no way to inform other components which may be functioning correctly which ones have failed. Components in a distributed system fail independently.

> A central problem in distributed computing is insuring that the state of the whole system is consistent after such a failure; this is a problem that simply does not occur in local computing.

In order for a system to withstand partial failure, it’s important that it deals with indeterminacy and the objects react to it in a consistent manner. The interfaces must be able to state the cause of failure if possible and allow the reconstruction of a “reasonable state” in case the cause can’t be determined.

> The question is not “can you make remote method invocation look like local method invocation?” but rather “what is the price of making remote method invocation identical to local method invocation?”

Two approaches come to&nbsp;mind:

1. Treat all interfaces & objects as local. The problem with this approach is that it doesn’t take into account the failure models associated with distributed systems and hence it’s indeterministic by&nbsp;nature.
2. Treat all interfaces & objects as remote. The flaw with this approach is that over-complicates local computing. It adds on a ton of work for objects that are never accessed remotely.
> A better approach is to accept that there are irreconcilable differences between local and distributed computing, and to be conscious of those differences at all stages of the design and implementation of distributed applications.
* * *