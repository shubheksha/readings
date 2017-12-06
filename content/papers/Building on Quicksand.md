+++
title = "Building On Quicksand"
description = "This post distills the material presented in the paper titled Building On Quicksand published by Pat Helland and David Campbell in 2009."
tags = [
    "fault-tolerance",
    "distributed-systems",
]
date = "2017-12-04"
+++


Let’s try to break down the paper “[Building On Quicksand](http://arxiv.org/ftp/arxiv/papers/0909/0909.1788.pdf)” published by Pat Helland and David Campbell in 2009. All pull quotes are from the&nbsp;paper.

The paper focuses on the design of large, fault-tolerant, replicated distributed systems and how it’s evolving based on changing requirements over time. It starts of by stating “Reliable systems have always been built out of unreliable components”.

> As the granularity of the unreliable component grows (from a mirrored disk to a system to a data center), the latency to communicate with a backup becomes unpalatable. This leads to a more relaxed model for fault tolerance. The primary system will acknowledge the work request and its actions without waiting to ensure that the backup is notified of the work. This improves the responsiveness of the system because the user is not delayed behind a slow interaction with the&nbsp;backup.

Fault-tolerant systems can be made of many components and their goal is keep functioning correctly when one of those components fail. We don’t consider **_Byzantine_** failures in this discussion, but instead the **_fail fast_** model where either a component works correctly or it&nbsp;fails.

The paper goes on to compare two versions of the Tandem NonStop system — one that used synchronous [checkpointing](https://en.wikipedia.org/wiki/Application_checkpointing) and one that used asynchronous checkpointing. Refer section 3 of the paper for all the details. I’d like to briefly touch upon the difference between the two checkpointing strategies.

- Synchronous checkpointing: in this case, with every write to the primary, state needed to be sent to the backup. Only after the write was acknowledged by the backup did the primary send a response to the client who issued the write request. This ensured that when the primary fails, the backup can take over without losing any&nbsp;work.
- Asynchronous checkpointing: in this strategy, the primary acknowledges & commits the write as soon as it processes it without waiting for a reply from the backup. This technique has improved latency but also poses other challenges addressed later.

#### Log Shipping
> A classic database system has a process that reads the log and ships it to a backup data-center. The normal implementation of this mechanism commits transactions at the primary system (acknowledging the user’s commit request) and asynchronously ships the log. The backup database replays the log, constantly playing catch-up.

The mechanism described above is termed as log shipping. The main problem this poses is that when the primary fails and the back up takes over, some recent transactions might be&nbsp;lost.

> This inherently opens up a window in which the work is acknowledged to the client but it has not yet been shipped to the backup. A failure of the primary during this window will lock the work inside the primary for an unknown period of time. The backup will move ahead without knowledge of the locked up&nbsp;work.

The introduction of asynchrony into the system hasan advantage in terms of latency, response time and performance but it makes the system more prone to the possibility of losing work when the primary fails. There are two ways to deal with&nbsp;this:

1. Discard the work locked in the primary when it fails. Whether a system can do that or not depends on the requirements and business&nbsp;rules.
2. Have a recovery mechanism to sync the primary with backups when it comes back up and retry lost work. This is possible only if the operations can be retried in an idempotent way and the out-of-order retries are possible.

The system loses the notion of what the authors call “an authoritative truth”. Nobody knows the accurate state of the system at any given point of time if the work is allowed to be locked in an unavailable backup or&nbsp;primary.

This leads to the conclusion that business rules in a system with asynchronous checkpointing are probabilistic:

> If a primary uses asynchronous checkpointing and applies a business rule on the incoming work, it is necessarily a probabilistic rule. The primary, despite its best intentions cannot know it will be alive to enforce the business&nbsp;rules.
> When the backup system that participates in the enforcement of these business rules is asynchronously tied to the primary, the enforcement of these rules inevitably becomes probabilistic!

The authors state that commutative operations (operations that can be reordered) can be allowed to execute independently and reordered as long as business rules are preserved. However, this is hard to do with storage systems because the write operation isn’t commutative.

Another consideration is that work of a single operation is idempotent, i.e., executing the operation any number of time should result in the same state of the&nbsp;system.

> To ensure this, applications typically assign a unique number or ID to the work. This is assigned at the ingress to the system (i.e. whichever replica first handles the work). As the work request rattles around the network, it is easy for a replica to detect that it has already seen that operation and, hence, not do the work&nbsp;twice.

The authors suggest that different operations within a system can provide different consistency guarantees depending on the business requirements. Some operations can choose classic consistency over availability and vice&nbsp;versa.

Next, the authors argue that as soon there is no notion of authoritative truth in a system, all of computing boils down to three things: memories, guesses and apologies.

1. Memories: you can only hope that your replica remembers what it has already&nbsp;seen.
2. Guesses: Due to only partial knowledge being available, the replicas take actions based on local state and may be wrong. “In any system which allows a degradation of the absolute truth, any action is, at best, a guess.” Any action in such a system has a high probability of being successful, but it’s still just a&nbsp;guess.
3. Apologies: Mistakes are inevitable, hence every business needs to have an apology mechanism in place either through human intervention or by automating it.

The paper next discusses the topic of eventual consistency by taking the Amazon shopping cart built using Dynamo & a system for clearing checks as examples. The work coming into these systems is uniquely identified and is processed by a single replica. It flows to other replicas as and when connectivity permits. The requests coming into these systems are commutative (reorderable) and can be processed at different replicas in different orders.

> Storage systems alone cannot provide the commutativity we need to create robust systems that function with asynchronous checkpointing. We need the business operations to reorder. Amazon’s Dynamo does not do this by itself. The shopping cart application on top of the Dynamo storage system is responsible for the semantics of eventual consistency and commutativity. The authors think it is time for us to move past the examination of eventual consistency in terms of updates and storage systems. The real action comes when examining application based operation semantics.

Next they discuss two strategies for allocating resources in replicas that might not be able to communicate with each&nbsp;other:

1. Over-provisioning: the resources are partitioned between replicas such that each has a fixed subset of resources they can allocate. No replica can allocate a resource that’s not actually available.
2. Over-booking: the resources are can be individually allocated without ensuring strict partitioning. This may lead to the replicas allocating a resource that’s not truly available, promising something they can’t&nbsp;deliver.

The paper talks also about something termed as the “seat reservation pattern” which is a compromise between over-provisioning and over-booking:

> Anyone who has purchased tickets online will recognize the “Seat Reservation” pattern where you can identify potential seats and then you have a bounded period of time, (typically minutes), to complete the transaction. If the transaction is not successfully concluded within the time period, the seats are once again marked as “available”.

#### ACID 2.0

The classic definition of ACID stands for “Atomic, Consistent, Isolated, and Durable”, it’s goal is to make the application think that there is a single computer which isn’t doing anything else when the transaction is being processed. The authors talk about a new definition for ACID which stands for: Associative, Commutative, Idempotent, and Distributed.

> The goal for ACID2.0 is to succeed if the pieces of the work happen: At least once, anywhere in the system, in any order. This defines a new KIND of consistency. The individual steps happen at one or more system. The application is explicitly tolerant of work happening out of order. It is tolerant of the work happening more than once per machine,&nbsp;too.

Going by the classic definition of ACID, fault tolerance is based on having a linear history. If we want to achieve the same guarantees in a distributed system, it’ll require concurrency control mechanisms which “tend to be fragile”.

> When the application is constrained to the additional requirements of commutativity and associativity, the world gets a LOT easier. No longer must the state be checkpointed across failure units in a synchronous fashion. Instead, it is possible to be very lazy about the sharing of information. This opens up offline, slow links, low quality datacenters, and&nbsp;more.

In conclusion,

> We have attempted to describe the patterns in use by many applications today as they cope with failures in widely distributed systems. It is the reorderability of work and repeatability of work that is essential to allowing successful application execution on top of the chaos of a distributed world in which systems come and go when they feel like&nbsp;it.

P.S. — If you made it this far and would like to receive a mail whenever I publish one of these posts, sign up&nbsp;[here](http://eepurl.com/dcHGFP).