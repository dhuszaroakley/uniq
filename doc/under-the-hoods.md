# Under the hoods

## Introduction

Imagine a long row of people wanting to cross a bridge or a ferry... 

This ferry only alow one car/person to cross at any time.

The gate is open, the car get in, the crossing happens and after the delivery, the ferry get back to take another.

This operation takes time and has a price.

In computers programming the "transport" problem is about transfering data between different process/actors of the system.

When two processors live in the same memory space, the faster approch is just allow the cores to share the memory instead of transfering the data.

## How to share a memory space between multiple threads? 

The common solution is the use of Semaphores, Mutexes and Critical Sessions to create locks, making the resource unavailable while some operation is running. 

But this pattern **lock/use/release** has a problem: While a operation does not finish, another cannot start. 

# The Queue object 

Here I present the solution to the 3-thread consensus. 

A queue implemented as a lock free circular buffer.

## Methods

## The prototype 
Here is a simple interface and starting point for your Queue object with the minimum properties and methods.

```
class Queue(size) {

  var head = 0, tail = 0
    , buffer = Array(size)

  push(item) // input an item and return an id.

  pop() // remove & return the next item.
}
```

Our queue object, has only 2 methods:

`push()` insert data onto the queue.

`pop()` remove data from the queue.

## Properties
`buffer` A contigous space of memory capable of holding integers or pointers. Allow more than one element to enter the queue at the same time.

`size` must be a power of two. At minimum 1.

`tail` where we register the new elements. Holds a the ID of last inserted element. 

`head` from where elements get out. Registers the next element to be removed.

`tail & head` are simple integers. Topped by the ``size`` of the buffer. At minimum, a single bit.


Now, with the minimum especification defined, its time to put it on fire... 

## Testing - The producer/consumer pattern

Here we create two groups of N threads each, one producing and another consuming data.

The first thread type is called the **producer**, it pushes a bunch of of numbers into the queue `Q`. 

```
producer() 
{
  for(int i=0; i < ITEMS; i++)
    Q.push(1) 
    
  Q.push(-1)

  Total += ITEMS
  log("Produced:', ITEMS)
}
```

To achieve a **mininum working example** with **maximum benchmarks**, we allways pushed the number 1, but a positive `random()` must work equally.

We signalize the termination of the  job with a special constant (-1). Also can be implemented with external flags.

Now, the work of our **consumer** is remove elements out of the queue, until receiving a termination value.  

At minimum:

```
function consumer()
{
  var value, sum = 0

  do { 
    value = Q.pop()
    sum += value
  } while(value > 0) 

  Total -= sum

  log("Consumed:", items)
}
```

Now its a matter of starting the threads and check if what we get out the queue its equal to what we pushed into. Time to fry the processors :D

```
var producers = []
  , consumers = []

for(int i=0; i < 4; i++)
{
  producers[i] = Thread(producer)
  consumers[i] = Thread(consumer)
}

wait(producers, consumers)

log("Sum: %d", Total);

```

Here we create a 8 of threads to flow data (4 produceres & 4 consumers). 

For simplicity, whe create the same number of the producers and consumers, but it works equally in assimetric conditions. 

After creating the Threads, we call the ``wait`` function to wait for the termination of the jobs.

Producers incremented the variable ``Total`` and consumers decrement it. At the end, the ``Total`` value **must** be zero (everytime). This is our proof that there is no leaks in you solution.

The results I get flowing 10M items:

```
Creating 4 producers & 4 consumers
Flowing 10.000.000 items trough the queue.

Produced: 2.500.000
Consumed: 1.349.413
Produced: 2.500.000
Consumed: 1.359.293
Consumed: 3.876.642
Produced: 2.500.000
Consumed: 3.414.652
Produced: 2.500.000

Total: 0

real    0m0,581s
```
Note that all **producers** pushed the same amount of items (2.5M), but the **consumers** consumed different quantities, (ranging from 1.3 to 3.8M). 

The producers known the number of ITEMS produced, but the consumers is open. This is the normal behaviour. The only condition we have, is the sum of items consumed must be equal the produced. This was ensured by the global variable Total.

The last line is the time took by the operation, in my computer, 581ms to flow 10 million itens. A throughput of 17.2 M Items/s, wich is a pretty impressive for an old Dell M6300 Core duo.

## Testing
To get a fully functional C++ implementation just clone this repository:

```
git clone https://github.com/bittnkr/uniq
cd uniq/cpp
sh build.sh
```

## Benchmarks

Here we have the measurements varing the buffer size. 

![Throughput x Buffer Size](https://i.stack.imgur.com/TgkKs.png)

Fixing the buffer size on the default size (64 positions) and varying the number of Threads.
 
![Throughput x Threads](https://i.stack.imgur.com/laMSX.png)

Note how the number of threads does not affect the overall performance of the system. The flow for 2 threads is almost the same for 512. (It's lock-free dude!)

---
This is a work in progress. Comments and benchmarks are wellcome. 

Released under the Creative Commons License (CC BY-SA 3.0)