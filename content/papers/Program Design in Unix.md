+++
title = "Program Design in the Unix Environment"
description = "This post distills the material presented in the paper titled Program Design in the Unix Environment published by Rob Pike and Brian Kernighan in 1983."
tags = [
    "operating-systems",
    "unix",
    "system-design"
]
date = "2018-01-29"
+++

# Program Design in the Unix Environment: A Summary

![](https://cdn-images-1.medium.com/max/1600/0*O-H_d2lDRBmnCgFG.jpg)
<span class="figcaption_hack">[Credit](http://www.adamalthus.com/blog/2013/04/04/the-composable-enterprise/)</span>

Today, let’s take a look at “[Program Design in the Unix
Environment](http://harmful.cat-v.org/cat-v/unix_prog_design.pdf)” published in
1983 by Pike and Kernighan.

The paper opens by listing why Unix has been successful and is a commentary on
the[ Unix philosophy](https://en.wikipedia.org/wiki/Unix_philosophy) and its
benefits. It does so by taking examples and discussing trade offs where programs
diverged from the Unix philosophy.

The reasons for Unix’s success:

1.  Portability: the kernel & applications are written in C, hence they can be moved
from system to system without being re-written in the assembly language
particular to that system.
1.  The same OS runs on different hardware, so the users are already familiar and
don’t have to relearn when new hardware is released.
1.  The vendors can ship same software with each machine despite changes in
hardware.
1.  The system was not too big and easy to modify since everything was written in C.
1.  It provided a new philosophy based on the use of general purpose tools which did
one thing well and could be combined to do a particular task instead of creating
giant monolithic tools that serve only one purpose.

The paper argues that the use and design of tools is closely related — how they
fit together is the main subject of this essay.

The paper then dives into `cat `— the Unix command line utility for
concatenating and printing files — it copies its input to its output. The input
is usually a sequence of one more of files or the standard input. The output is
a file or the standard output.

The main purpose of `cat` was to act as a utility to concatenate files. It can
be combined with the pipe (`|`) operator to further enhance to extend its
utility through output redirection.

Other systems, on the other hand, try to dump a bunch of related functionality
into a single command which is against the Unix philosophy. It also creates a
lock-in of functionality that might to useful to other programs.

Advantages of the Unix approach:

1.  The shell and the programs it can invoke provide a uniform access to system
facilities. Eg: the filename arguments are expanded by the shell in a similar
fashion for each command. Because of pipes, we don’t need every command to deal
with pre- and post- processing of input.
1.  Growth is easy when functions are well separated.

Example: the ` (backquote) operator was added to convert the output of one
program into input of another without requiring changes in any other program as
it is interpreted by the shell. All programs the shell invokes acquire this
feature automatically. If each program that required this feature interpreted
it, it’d be very hard to enforce uniformity and carry out further
experimentation as each new idea would affects all the programs that would want
to use it.

However, in the future versions of `cat`, many new options were introduced, like
for printing line numbers and non-printable characters.

The authors argue that instead of adding those options to `cat` itself, either
existing programs should’ve been used or new programs should’ve been created.
For example, line number functionality could’ve been provided by using `pr`.
However, there was no program which allowed printing of non-printable characters
hence warranting the creation of a new one.

> Such a modification confuses what `cat`’s job is  concatenating files 
> with<br> what it happens to do in a common special case  showing a file on the
terminal. A UNIX program should do one thing well, and leave unrelated tasks to
other programs. `cat`’s job is to collect the data in files. Programs that
collect data shouldn’t change the data; `cat` therefore shouldn’t transform its
input.

Whenever we split something into multiple programs, we sacrifice some
efficiency. But since `cat` is usually used without any options, it makes sense
to have the most common cases be the most efficient.

> Separate programs are not always better than wider options; which is better
> depends on the problem. Whenever one needs a way to perform a new function, one
faces the choice of whether to add a new option or write a new program (assuming
that none of the programmable tools will do the job conveniently). The<br>
guiding principle for making the choice should be that each program does one
thing. Options are appropriately added to a program that already has the right
functionality. If there is no such program, then a new program is called for. In
that case, the usual criteria for program design should be used: the program
should be as general as possible, its default behavior should match the most
common usage, and it should cooperate with other programs.

*****

Let’s consider another issue: dealing with fast terminal lines. How to deal with
output from `cat` scrolling off the top of the screen?

There are two approaches:

1.  Tell each command about terminal properties so it does the right thing
1.  Write a command that handles only terminals without modifying other programs

Let’s consider examples of both approaches: `lsc` and `ls` which prints out the
list of files in a directory.

`lsc` varies its output depending on the input — it displays the list in a
columnar fashion across the screen so that the o/p fits if it’s outputting to
the terminal whereas `ls` displays everything in a single column.

> By retaining single column output to files or pipes, `lsc `ensures compatibility
> with programs like `grep `or `wc `that expect things to be printed one per line.
This ad-hoc adjustment of the output format depending on the destination is not
only distasteful, it is unique  no standard UNIX command has this property.

The authors argue that the columnation facility is useful in general & shouldn’t
be locked away in just `lsc` and be inaccessible to other programs. They
advocate for a different program whose primary job is columnation.

> Similar reasoning suggests a solution for the general problem of data flowing
> off screens (columnated or not): a separate program to take any input and print
it a screen at a time. Such programs are by now widely available, under names
like pg and more. This solution affects no other programs, but can be used with
all of them. As usual, once the basic feature is right, the program can be
enhanced with options…

*****

Based on the previous example, the authors also talk about different cases where
some functionality is locked away in a specific program, like input history in
the terminal, which would be better off as a central service. All interactive
programs could benefit from it.

They conclude with how augmenting existing commands with features/options is not
desirable in Unix, it goes against its basic philosophy which is to make a
program do one thing well and several such programs can be composed to
accomplish a more complex task.

> The key to problem solving on the UNIX system is to identify the right primitive
> operations and to put them at the right place. UNIX programs tend to solve
general problems rather than special cases. In a very loose sense, the programs
are orthogonal, spanning the space of jobs to be done (although with a fair<br>
amount of overlap for reasons of history, convenience or efficiency). Functions
are placed where they will do the most good: there shouldn’t be a pager in every
program that produces output any more than there should be filename pattern
matching in every program that uses filenames.

> One thing that UNIX does not need is more features. It is successful in part
> because it has a small number of good ideas that work well together. Merely
adding features does not make it easier for users to do things  it just makes
the manual thicker. The right solution in the right place is always more
effective<br> than haphazard hacking.
