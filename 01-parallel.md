---
layout: page
title: Parallel Computing
subtitle: Sum of square roots
minutes: 20
---
> ## Learning Objectives {.objectives}
>
> *   Explain why we need parallel computing.
> *   Lear what parallel computing is.
> *   Run a simple parallel (multithreaded) program.
> *   Do some external timing to see scaling.

Sometimes computational task are very demanding. A lot of memory my be required to keep large amounts of data around for easy access. Arithmetic operations may be complex and there may be billions of them. A routine may have to be executed a very large number of times to arrive at a result. An increase in computational workload calls for an increase in computational resources. In the past that usually meant that one needed a better, i.e. a faster computer. Luckily, computers had a way to satisfy these increasing demands by doubling their capacity every couple of years. In fact, one could count on it, and this observation was given a name: Moore's Law. Here's a picture of it (thanks, reddit):

~~~ {.python}
There would be a picture here
~~~

The capacity of a computer's processing unit is roughly proportional to the number of transistors you can cram on it. Note that the y-axis of this graph is logarithmic. We can see that the number of transistors increases by an order of magnitude every 6 or 7 years. In recent times though, the "chips"" we are looking at are of the "multicore" variety, i.e. they have more than one processing unit on it. This is generally the case: In the last two decades improvement of computers has been largely due to "more" rather than "faster". Computing has turned parallel. What exactly does that mean?


As usual, Wikipedia has the answer:


~~~ {.python}
Parallel computing is a type of computation in which many calculations are carried out simultaneously, operating on the principle that large problems can often be divided into smaller ones, which are then solved at the same time.
~~~

The key point is of course the division of larger problems into smaller ones. Each of the smaller problems can be tasked to a separate computing units, i.e . A core, a CPU, a chip, a machine etc. For this to work though, there is one more crucial condition:

~~~ {.output}
The sub-tasks in a parallel computation must be independent from each other.
~~~

If one task is independent from the other we can do them all simultaneously on different components. If one depends on an other, i.e. I can only work on one after I have completed another, I end up having to do them in order, and I gain nothing. So the first step in "parallelizing" a program is to identify how to split the workload into preferably equal sub-tasks that have little or nothing to do with each other. The next step is to send these to different processors. There is many ways to do this, and we will learn about some of them.

For now let's look at how a parallel program runs. In the directory ~/sqroots you find a program "sqroots" which computes the sum of the square roots of consecutive integers from 0 to a maximum one N. If N is chosen large enough (of the order of billions) it is worth while to do this in parallel. Most of the computational work is in the computation of the square roots, while the sum is almost negligible. We can therefore compute the sum of a subset of the roots on one processor and others on another. At the end we sum all the partial sums into a total sum. That should reduce the runtime because the partial sums stand to take proportionally less time than the total.

The program "sqroots" was written in Fortran, but that shouldn’t bother us. It uses a parallel system called "OpenMP" that is used for multicore "shared memory" machines, but that shouldn’t bother us either. To tell the system how many processors we want to use, we need to set a so-called environment variable called OMP_NUM_THREADS, which we can do by preceding the program call with the setting:

~~~ {.python}
OMP_NUM_THREADS=2 ./sqroots.exe
1234567
~~~
~~~ {.output}
  Result =   914494295.63120651
~~~

Here we are running with two processors. Since we have chosen the maximum number N a bit small, it was over too quickly for us to tell the difference between different processor numbers. Also it may turn out to be a bit tedious to re-type the number all the time. So we can write the number first into a file, then re-use the file. Finally it would be best to measure the runtime automatically. The easiest way to do this using the "time" Unix function.

~~~ {.python}
$ echo 1234567890 > sqroots.in
$ OMP_NUM_THREADS=1 time -p ./rootsum.exe <rootsum.in
~~~
~~~ {.output}
  Result =   28918862541603.5
real 6.03
user 6.02
sys 0.00
~~~
~~~ {.python}
$ OMP_NUM_THREADS=2 time -p ./rootsum.exe <rootsum.in
~~~
~~~ {.output}
  Result =   28918862541602.9
real 3.60
user 6.62
sys 0.00
~~~
~~~ {.python}
$ OMP_NUM_THREADS=4 time -p ./rootsum.exe <rootsum.in
~~~
~~~ {.output}
  Result =   28918862541602.8
real 2.00
user 7.98
sys 0.00
~~~
~~~ {.python}
$ OMP_NUM_THREADS=8 time -p ./rootsum.exe <rootsum.in
~~~
~~~ {.output}
  Result =   28918862541603.0
real 0.80
user 6.07
sys 0.00
~~~

Looks like the time is substantially reduced in each step. Ideally, we would like the time to be reduced by half whenever we double the number of processes. To be sure that's not quite the case here. This may have various reasons, such as other people using resources on the system, or parts of the program not running properly in parallel. The ratio between the serial and the parallel runtime is called "speedup". We'd like that number to be equal to the number of processors, i.e. if we are using 8 processors then the serial runtime should be 8 times as large as the parallel one. This is called "ideal" or "linear" scaling. It can be achieved for smaller numbers of processors, but as the number increases it gets harder and harder to do it. For limitations, read about "Amdahl's Law".