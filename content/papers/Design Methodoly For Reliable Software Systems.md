+++
title = "A Design Methodology For Reliable Software Systems"
description = "This post distills the material presented in the paper titled A Design Methodology For Reliable Software Systems published by Barbara Liskov in 1972."
tags = [
    "reliability",
    "software-engineering",
    "system-design"
]
date = "2018-02-03"
+++


Let’s dig into [A Design Methodology For Reliable Software
Systems](https://valbonne-consulting.com/papers/classic/Liskov_72-Design_Methodology_for_Reliable_Software_Systems.pdf)
published by Barbara Liskov in 1972.

![](https://cdn-images-1.medium.com/max/1600/0*YBoSTQ9iJHRhc3jm.jpeg)
<span class="figcaption_hack">[Credit](https://www.defit.org/wp-content/uploads/2012/06/puzzle-modularity-320x259.jpeg)</span>

The focus of this paper is on how to make reliable software systems and what
techniques can help us achieve that. Reliability here implies that a system
works as expected under a given set of conditions.

> The unfortunate fact is that the standard approach to building<br> systems,
> involving extensive debugging, has not proved successful in producing reliable
software, and there is no reason to suppose it ever will. Although improvements
in debugging techniques may lead to the detection of more errors, this does not
imply that all errors will be found. There certainly is no guarantee of this
implicit in debugging: as Dijkstra said, “Program testing can be used to show
the presence of bugs, but never to show their absence.”

To be confident that our system works correctly, we need testing that meets the
following conditions:

1.  We can generate a minimal set of relevant test cases
2.  All test cases in the set can be generated

> The solutions to these problems do not lie in the domain of debugging, which has
> no control over the sources of the problems. Instead, since it is the system
design which determines how many test cases there are and how easily they can be
identified, the problems can be solved most effectively during the design
process: The need for exhaustive testing must influence the design.

The paper further argues that reliability is a major issue with complex systems.
It goes on to define complex systems as follows:

1.  The system has many states and it’s difficult to organize program logic to
handle all of them correctly
2.  It requires several people working together in a coordinated manner

### Criteria for a Good Design:

To tame the design of a complex system, we need to use modularization and divide
the program into several modules (sub programs, later on referred to as
partitions in the paper to avoid overloading the term “modules”) which can be
compiled separately but are connected to other modules.

The connections are defined by Parnas as follows:

> The connections between modules are the assumptions which the modules make about
> each other.

Although the idea of modularity sounds like a great tool for building large
complex software systems, it can introduce additional complexity if not done
right.

> The success of modularity depends directly on how well modules are chosen.

Some common issues are:

1.  A module does too many things
2.  A common function is distributed among many different modules
3.  A module behaves unexpectedly with common data

The next question that arises: **What is good modularity**?

We use two techniques to answer that: **levels of abstraction** to tackle the
inherent complexity of the system and **structured programming** to represent
the design in software.

#### Levels of abstraction:

> Levels of abstraction…provide a conceptual framework for achieving a clear and
> logical design for a system. The entire system is conceived as a hierarchy of
levels, the lowest levels being those closest to the machine.

A group of related functions make up a level of abstraction. Each level can have
the following two types of functions:

1.  External: These functions can be called by functions in other levels
2.  Internal: These functions do a common task within the level and cannot be called
by other functions in a different level

Levels of abstraction are governed by the following two rules:

1.  Each level has exclusive control over some kind of resource
2.  Lower levels aren’t aware of higher levels and can’t reference them in any way.
However, higher levels can ask lower levels to perform an action or for info.

#### Structured Programming:

A structured program defines the way control passes among various partitions in
a system. It is defined by the following two rules:

1.  The program is developed in a top down fashion and divided into levels (the
notion of levels here is different from that of levels of abstraction because
the first rule isn’t satisfied)
2.  Only the following control structures can be used: concatenation, selection<br>
of the next statement based on the testing of a condition,<br> and iteration.
Jumping using `goto` isn’t permitted.

Back to the question that was posed earlier: **how do we define good
modularity**?

In a modular system that is also reliable, the connections between partitions
are limited as follows:

1.  They need to follow the rules imposed by levels of abstraction and the
structured programming
1.  Passing of data between partitions should be done using explicit arguments
passed to external functions of another partition
2.  Partitions should be logically independent — the functions within a partition
should support its own abstraction *only*

The next question that arises after we’ve figured out how to defined good
modularity is — **how do we achieve it in our design**?

> The traditional technique for modularization is to analyze the execution-time
> flow of the system and organize the system structure around each major
sequential task. This technique leads to a structure which has very simple
connections in control, but the connections in data tend to be complex.

Partitions supports abstractions that a system designer finds helpful when
thinking about the system.

> Abstractions are introduced in order to make what the system is doing clearer
> and more understandable; an abstraction is a conceptual simplification because
it expresses what is being done without specifying how it is done.

The paper then presents some guidelines for identifying different types of
abstractions while designing a system:

1. Abstraction of resources: for every hardware resource on the system, we can map
characteristics of the abstract resource to the underlying resource
2. Abstract characteristics of data: how it is stored
3. Simplification via limiting information the partition needs to know or has
access to
4. Simplification via generalization by identifying functions that perform a common
task. Such functions can be grouped together in one partition. “The existence of
such a group simplifies other partitions, which need only appeal to the
functions of the lower partition rather than perform the tasks themselves.”
5. System maintenance and modification: functions performing a task whose
definition is prone to change in the future should be part of independent
partitions. For example, functions which deal with connecting to a particular
kind of storage back end so that if a different back end is used in the future,
only functions in that partition will be affected.

Now that we have some idea about how we can achieve good modularity while
designing our system, **how do we proceed with it**?

The first phase is to identify a set of abstractions that represent the eventual
behavior of the system in a general way. The next phase “establishes the data
connections between the partitions and describes the flow of control among the
partitions”.

> The second phase occurs concurrently with the first; as abstractions<br> are
> proposed, their utility and practicality are immediately investigated.

> A partition has been adequately investigated when<br> its connections with the
> rest of the system are known<br> and when the designers are confident that they
understand<br> exactly what its effect on the system will be.

The next question one would ask is:** how do we identify when the design is
finished**?

1.  All major abstractions have been identified and been linked to a partition. The
system resources have been divided among the various partitions and their
positions in the hierarchy have been defined
2.  The interfaces and flow of control among the partitions is clearly defined. The
test cases for each partition have been identified
3.  A basic user guide for the system can be written
