---
layout: page
title: Parallel Computing
subtitle: MPI for Python (MPI4Py)
minutes: 20
---
> ## Learning Objectives {.objectives}
>
> *   Learn about MPI bindings for Python.
> *   Learn abouut the simplified form of basic MPI features in Python.

MPI was originally designed for FORTRAN and C. Later (in version 2) C++ was added. Meanwhile, some implementations of MPI cover Java. It was only a matter of time that someone would supply us with a Python version. The MPI interfaces we will work with are called "MPI4Py" (see http://pythonhosted.org/mpi4py/). This is not the only package with this purpose, but it is simple to use and therefore ideally suited for an introductory course.

One nice thing about MPI4Py is that many simple applications don't require as detailed specification as is needed for the native bindings of MPI. For 
beginners this means that they don't have to worry about a lot of details that are required when dealing with MPI in C or C++. On the other hand, once moving on to bigger projects, some of these details become important and one has to re-visit them.

Here's a list of MPI functions that we will be using in our next few mini-projects. They are the absolute minimum needed to write a working MPI program

~~~ {.python}
Table with MPI features
~~~

Note that for most of these there are several ways of specifying details by adding more arguments. For how to use this, you need to study the package in detail. The most compelling aspect of these interfaces is that Python usually can figure out what is the type and shape of the data that are being transmitted. As long as things match on both sides of the communication, you're usually good.

