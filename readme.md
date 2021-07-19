# Introduction to OpenMP - loop parallelization

Martin Cuma

Date/version information


## Development Notes

These are notes for yourself while you're working on your tutorial - remove them later.

What's your topic?

* Language or Program: C
* Tutorial Type (Task, Technique, Package Overview, Tool):  Technique
* Specific Package/Task/Technique/Tool: Introduction to parallelization with OpenMP



## Introduction

Basic knowledge of C, especially loops.


### Setup

Basic knowledge of Linux terminal and code compilation (e.g. CHPC's Intro to Linux lectures).

Use of text editor in Linux terminal or desktop (vim, emacs, nano, gedit,...).

Remote terminal access to CHPC machines or linuxclass.chpc.utah.edu (FastX preferred).

Download examples....

```
mkdir OpenMP_tutorial
cd OpenMP_tutorial
cp ~u0101881/talks/OpenMP/exercises/* .
```


## Section 1

### Hardware for OpenMP parallel computing - shared memory

Shared memory or distributed memory parallel computers.
(SM schematic)
* All processors have access to local memory
* Simpler programming
* Concurrent memory access
* Representatives: Linux clusters nodes 12-64, your laptop

### Program execution - processes and threads

* Process (task)
** Entity that executes a program â€“ has its own memory
space, execution sequence, is independent from other
processes
* Thread
** Has own execution sequence but shares memory
space with the original process - a process may have
many threads

Common option for program execution is to map a process to a CPU and threads to CPU cores.


### OpenMP basic concepts

Compiler extension using directives to tell compiler to add OpenMP functionalty.
Directives syntax:
```
# pragma omp directive_name [optional clauses]
```

### EXERCISE 1

Compile and run serial code to calculate the value of Pi.

The bulk of the calculation is spent in the loop:
```
h   = 1.0 / (double) n; 
sum = 0.0;
for (i = 0; i < n; i ++)
{
    x = h * ((double)i - 0.5);
    sum += f(x);
}
pi = h * sum;

```

First we have to compile the code with the C compiler, then run it:
```
gcc pi_serial.c
./a.out
```


## Section 2

### OpenMP parallel loop directive.

Loop is parallelized by adding a line with the parallel loop directive right before the loop:
```
#pragma omp parallel for
for (i = 0; i < n; i ++)
...
```

To run in parallel, most commonly we set the environment variable ```OMP_NUM_THREADS``` before executing the program.
```
export OMP_NUM_THREADS=4
```

### EXERCISE 2

Parallelize the pi_serial.c with OpenMP parallel loop directive.
```
cp pi_serial.c pi_openmp.c
```
open your favorite editor and add the OpenMP loop directive at the right place. Then compile the code with the ```-fopenmp``` compiler option which tells the compiler to use OpenMP, and run.
```
gcc -fopenmp pi_openmp.c
export OMP_NUM_THREADS=1
./a.out
export OMP_NUM_THREADS=2 
./a.out
```
Compare the results. They will be different. Can you think why?

## Section 3

### Race conditions in parallel looops.

Our results run with one and two threads differ because of race condition(s). 

Race conditions are situations when two or more threads access, and worse so, write to, the same data variable at the same time.

There are different types of race conditions, and fixing them depends on the type:
* Private (thread-local) variables ```x = i*h```
* Reduction (sum over threads) ```sum += f(x)```
* Flow dependence - loop rearrangement ```a(i)=a(i-1)+h```

Can you think of what race conditions are present in this example?

### Variable privatization

The variable ```x``` in our loop needs to be private as each loop iteration will have a unique value of ```x``` and the loop iterations are distributed over the parallel threads.
```
    x = h * ((double)i - 0.5);
```

Private variables are specified by the scope clause (option) of the parallel loop directive:
```
#pragma omp parallel for private(x)
for (i = 0; i < n; i ++)
...
```


### EXERCISE 3

Privatize the appropriate variable(s) in the parallel loop
```

```
Is the result correct? It still should not be.

## Section 4

### OpenMP reduction clause.

Reduction accumulates partial results from all the threads. It needs to be a commutative operation. Most common reduction is a summation (+).

OpenMP reduction is specified by the reduction clause, which includes the operator colon separated from the variable to be reduced:
```
#pragma omp parallel for reduction(+:sum)
```

### EXERCISE 4

Add the reduction clause to the OpenMP parallel loop directive.
```

```
Now run the program again on one and two threads and compare the result. It should be the same.

## More on OpenMP

OpenMP is much more than loop parallelization but loops are the easies and most commonly parallelized.
Other parts of OpenMP to consider
- tasks - good for non-loop parallelization
- GPU support - offload computation to GPUs
- CPU affinity and vectorization - to get better performance on modern CPUs
- "low level" features, such as critical sections, functions, scheduling, ...

## Wrap-up

We learned to parallelize a loop with OpenMP using the ```#pragma parallel for``` directive.
We need to pay attention to data races and how to take care of them by privatization (```private()``` clause) or reduction (```reduction()``` clause))

More resources: XSEDE Monthly HPC workshop - OpenMP (URL to be added)



