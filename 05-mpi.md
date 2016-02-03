---
layout: page
title: Parallel Computing
subtitle: Message Passing Interface (MPI)
minutes: 30
---
> ## Learning Objectives {.objectives}
>
> * Learn about basic MPI concepts.
> * Understand what Ranks and Size are used for.
> * Learn what a communicator is.
> * Learn what point-to-point communication is.
> * Learn what message, source, target, and tag are for.
> * Learn the difference beween blocking and non-blocking.
> * Learn about collective operations and when to use them.

This part of the workshop is on the "dry side". To explain the basic principles of MPI programming we have to go through quite a stretch where we can't do much hands-on stuff. Sorry.

As we outlined before, in a serial program, we're starting just one process, and that gets executed on a single processing unit, such as a core. With an MPI program, we're starting several and normally leave it to the Operating System to run each on different CPUs. There are ways to spell it out to the machine what goes where, but we don't bother with that. All we do is specify how many processes we want to run, and the OS takes care of the rest

~~~ {.python}
Picture of parallel and serial code
~~~

Note that the number of cores or CPUs or nodes on a cluster isn't necessarily the same as the number of processes we started. That still works, although it won't give you as much speedup as you may expect. Normally it is best to not start more processes than you have processors available to you. That way you get "dedicated cores" and your program runs as well as it can.

## Ranks and Size

MPI uses the idea of "ranks". These are numbers between 0 and one less than the number of processes, that are used to label each process with a unique number. We can use these to specify where we want to send messages, or from what process we expect messages:

~~~ {.python}
Picture of Rank and Size
~~~

The two functions MPI_COMM_RANK and MPI_COMM_SIZE are used by each process to determine their rank (i.e. their name) and the size. The latter is the total number of processes that are active. This information is essential for all processes, since they need to know how many others there are to properly distribute the workload.


Try running the rootsum program with 12 processes:

~~~ {.python}
$ mpirun -np 12 ./rootsum.x < rootsum.in
 How many terms?
 RANK:           5 MYS=   2409905224976.83       M:  1234567890
 RANK:           6 MYS=   2409905227904.38       M:  1234567890
 RANK:           4 MYS=   2409905222048.78       M:  1234567890
 RANK:           7 MYS=   2409905195696.45       M:  1234567890
 RANK:           2 MYS=   2409905216192.29       M:  1234567890
 RANK:           8 MYS=   2409905198624.17       M:  1234567890
 RANK:           9 MYS=   2409905201551.99       M:  1234567890
 RANK:           3 MYS=   2409905219120.97       M:  1234567890
 RANK:          10 MYS=   2409905204479.74       M:  1234567890
 RANK:           0 MYS=   2409905210335.78       M:  1234567890
 RANK:          11 MYS=   2409905207407.71       M:  1234567890
 RANK:           1 MYS=   2409905213264.25       M:  1234567890
 Total sum:     28918862541603.3
 Total time on rank 0:    0.655900000000000
~~~

As we can see, the rank numbers are all over the place, so obviously the rank doesn't have anything to do with the order in which things happen. They are just labels, so "rank" is a bit of a misnomer. Things are supposed to happen simultaneously, otherwise there wouldn't be much of a point to parallel programming.

## Initialization and Finalization, Error Variable

In order to use the MPI API, one needs to tell "the system" about it. So MPI has two special routines that are used at the beginning and at the end of a program, respectively:

~~~ {.python}
MPI_INIT(IERR)
MPI_FINALIZE(IERR)
~~~

The first one is used to "initialize" MPI, which means setting all sorts of values and declaring all sorts of variables, get the communication system started etc. We don't have to worry about the details, but we need to call that function to make MPI work. This has to be the first MPI call we make. On the other end, when we are done with everything that has to do with MPI, we should call MPI_FINALIZE to let the system know we're done with MPI. That's not as important as MPI_INIT because it happens at the end of the program, so the whole run is over soon anyway, but it should be done to keep things clean.

In case you are wondering what that IERR argument of the function call is about. This is used to check if everything went alright. We can compare this argument after the call to a constant to see if there was a problem. We will ignore this in the following. In fact, we don't have to worry about any of this when we're trying MPI out with Python, since initialization and finalization is automatic there. But if you're going to write heavy-duty MPI program in the future, you need to remember to do this.

## Communicators

One of the ideas introduced in MPI programs is the notion of a "communicator". Communicators are groups of processes and a communication system between them. We need this in an MPI program because we may have several of these groups/systems in a more complicated program. You can think of it like different mailing systems: Canada Post, UPS, Fedex etc… each with a (often overlapping) group of users that you can reach with it. Ranks and size are specific to a communicator. For most MPI function calls we will be asked to specify the communicator. For simple program, we often need only one: the default communicator MPI_COMM_WORLD. As the name indicates, it has all the processes that we started in it.

~~~ {.python}
Picture of communicators
~~~

## Point-2-Point Communication, MPI_Send, MPI_Recv

The most basic communication in MPI is "point-to-point". This means that one process is sending a message, and another process receives it. So point-to-point communication always is a two-part affair:

* One process makes a call to MPI_Send to send the message
* Another process makes a call to MPI_RECV to receive the message

The message is usually some kind of data structure. In low-level programming languages such as Fortran, a lot of things have to be spelled out very carefully, particularly if the message is of a more complicated form. Some of these things are being figured out automatically in higher-level languages such as Python. But some basic stuff will need to be specified:

* The sender needs to specify the recipient, while the recipient needs to specify the sender. This is done by rank numbers, and it needs to match.
* The data are specified through the name of the data structure. The send specifies the name of structure that has data in it,, the recipient specifies the name of an empty data structure that is to be filled (or a non-empty one to be over-written). Sizes, types, etc need to be specified in Fortran or C, but can be automatic in Python.
* An additional "tag" (just an integer) also needs to be specified to distinguish between different messages among the same processes, i.e. to avoid getting "wires crossed".
* The communicator also needs to be specified and needs to match. Often this is just the default one MPI_COMM_WORLD.

~~~ {.python}
Picture of P2P
~~~

If any of the stuff that we need to specify does not match, the communication will not work. MPI is notoriously bad at error reporting, so it is well possible that you'd spend a lot of time staring at a "stuck screen" when that happens. Here are a few scenarios of that kind:

* If process 0 sends a message to process 1, and process one waits for a message from process 2, there's a mismatch, and it's not happening.
* If the tags have different values for some reason, same thing.
* If the two are specifying different communicators, same thing.

It is also possible to create accidental "lockups". If process one waits for a message from process 2, but process two waits for one from process 1 first, then both may be waiting forever. So one has to be very careful with point-to-point communication.

All communication processes in MPI can be reduced to this type, but that often happens "under the hood" and we don't really have to know every detail.

> ## Note {.challenge}
> There is a distinction between "blocking" and "non-blocking" communication. The difference is that in the former, we are waiting around until the 
> communication has finished before the program moves on to do something else. In the latter, we just specify what we want to do (for instance 
> receive a message), and then immediately "move on" to do other things. The blocking one is safe because we're not tempted to do anything with a message
> that we haven't received yet, but the non-blocking one is faster, because we're not spending so much time waiting around for messages, but rather spend
> that time doing something un-related. We're going to stick with blocking in this course.

## Collective Operations: MPI_Bcast, MPI_Reduce, MPI_Barrier

If you have a lot of processes to deal with, point-to-point communication can become very quickly quite tedious, not to mention error-prone. For instance, imaging one of the processes (let's say number 0) has a piece of information that everyone else needs to know about as well. To do this, we have to program a loop. The loop index would be the rank numbers from 1 to N-1. For each of the iterations, the rank 0 process calls MPI_Send, and one of the other processes calls MPI_Recv. 

Since this kind of thing happens often, MPI has "collective operations", i.e function calls where all of this stuff happens "under the hood". That way, only a single function call that has to be done by all processes is required. So here's the definition:

Collective Operations in MPI are function calls that involve all processes. Typically such functions need to be called by all processes. They are usually blocking.


The above situation would require an MPI_BCAST call, a so-called "broadcast". Note that broadcasts involve the recipient (the listener) as much as the sender (broadcaster), so they must be called by all processes:

~~~ {.python}
Picture of Bcast
~~~

We're showing all the required arguments in a FORTRAN call here. In Python things are a bit simpler. But we definitely need to specify what is being broadcast (in this case, array A), which rank has the information at the outset (this is called the "root process", in this and most cases, it's 0), and the communicator (MYCOM, in many cases it's MPI_COMM_WORLD). Never mind the dimension (100) and data type (MPI_REAL). Python will figure that sort of thing out for us once we apply this.

Here's an example for something more complicated in terms of collective operations: an MPI_REDUCE. Remember the sum of the square roots? In that case we computed the partial sum (i.e. only some of the square roots) on each of the processes. At the end, everything got summed up to yield the final result. Exactly that kind of situation is addressed by this operation:

~~~ {.python}
Picture of Reduce
~~~

If each of the processes has "a piece of the puzzle" and we want to combine it on one process, we have to specify:

* The name of pieces (A) in this example
* The name of the total (B)
* Dimensions and types (never mind, not necessary in Python)
* How to combine the pieces (MPI_SUM here, could be something else, like a product or a maximum)
* Which process has the final result (the "root process", 0 here)

And presto, the rest is sorted out by MPI. A single function call does the trick, but we need to make sure everyone calls the function.

Let's look at one more very simple collective operation: a Barrier. Barriers are often used in MPI program to get all the processes back in line. They do nothing but "trapping" a process inside of a function call until all other processes have called the barrier function as well. The only thing one has to specify is the communicator because that determines which processes have to wait for each other:

~~~ {.python}
MPI_BARRIER(COMM,IERR) = After calling wait until all other processes called it as well
~~~

Sorry, I can't really draw a picture of this.

MPI is a pretty big system, so there are many more function call, but the ones we discussed until now should be enough to parallelize our Mandelbrot program. But before that, some information about Python and MPI. 
