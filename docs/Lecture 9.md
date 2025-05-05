## $ and OpenMP
Every processor needs to have the same view of memory. Every thread shares the same view of memory!

If a processor does a write, it has to let **all** of the other processors know. Invalidate the other copies. (Race Conditions only happen on writes). If we are only reading, many processors can have copies of said data. All of the other processors will 'snoop' on the interconnect and see any requests that go on the interconnect (i.e, usually a BUS) and see if it involves them in any way.  We never want all the writes to go back out to main memory, this is incredibly slow. This motivates us to have something called a write-back cache. Yeah, write-back is a jolly good fellow 😎

We also don't want to do a lot of writing in general. Even when using the write-back method. We can communicate by broadcasting requests. Using the bus allows us to communicate with the neighboring caches from the other processor / cores. 

Cache Optimization via these two new states:
3. Exclusive: up-to-date data, no other cache has a copy,  OK to write, memory is up to date
	1. Only the Valid bit is set. 
4. Owner: up to date data, other caches may have a copy, but within the other caches, it must be in the shared state! Main Memory is not up to date. 

### Easy way to think of these states:
For the ***exclusive*** state, I found it the easiest to think of it like this:
“I have the _only_ copy, it matches memory, and I can quietly turn it into a Modified copy whenever I want.”

For the ***owner state***, a mental model:
“Others can read from my data, but memory doesn’t know what’s up. If anyone needs the latest data, they come to me.”

### Idea behind the states:
- M: Modified
	- I have the only copy, and can write, and it's dirty
	- If this gets evicted, I need to flush the entry
- O: Owned
	- I have the official copy, and can write, and it's dirty
	- When writing, I have to tell everyone
	- else I'm writing (And now the state will turn into Modified)
- E: Exclusive
	- I have the only copy
- S: Shared
	- I have a copy and can read away
- I: Invalid
	- Duh, what do you think this means?

Weaver does a walk-through of what it looks like when each of the CPU cores wants to do an access for some data at a designated memory address w/ the cache coherency model.


## State Transition Diagram
![[Screenshot 2025-05-04 at 2.20.07 PM.png]]

High-level understanding:
- All blocks start at the INVALID state, and transition throughout this diagram accordingly
- Probe means that another Processor is asking for thingy
Only one of the caches can have a block set as `Owner`

# This introduces a new type of cache miss, a `coherency` miss:

If two CPU's are reading the same data, there is no problem at all. But, if one thread writes the data it will cause evictions from the other thread's cache. 
It does not even need to occur with the specific same piece of data, it just has to be on the same cache line. If one processor is writing to the beginning of the cache line, and the other processor is reading to the beginning of the cache line, then the entire cache line is trashed. 

When it comes to multithreaded cores, capacity misses increase due to issues like incoherency.
Incoherency occurs when we read or write different data. There is cache pressure that is being increased. This is still an issue!

### Takeaways...
Have each core work on separate pieces of data. Keep the data separate on a reasonable block size (256B will always be safe). Size for a cache about 1/2 of the actual size of the cache (if you face multithreaded CPU's)

You want your working set to be half of what the actual cache size is. You also want `writes` that take place to be on disjoint boundaries. (i.e, What this means is that you do not one thread's writes to be in a place where another thread will be reading from. you want things to be separated effectively). 

What is[[ cache thrashing]]?

If there are multiple threads accessing the same cache, they are essentially putting capacity pressure on each other. This is not good whatsoever.

## Analogy ~ Notebooks!

Imagine that you and your friend (You are both CPUs/Cores), are both using a whiteboard (shared whiteboard) to keep track of each of your to-do lists. To save time, we each keep a little notebook (our respective caches), where we will copy the latest version of the information on the whiteboard. If the friend adds something to the to-do list, then we are gonna have to update our little notebook with what they added. This is b/c we still have the old version of what was on the whiteboard. Our system has to evict (erase) our old notebook page / update it so that we both agree. 
![[Screenshot 2025-05-04 at 4.56.52 PM.png]]
For multithreaded cores, imagine that you and your friend share the same notebook, oof, we won't be able to keep track of all of these changes. There will be a lot of capacity misses, this is because the notebook cannot hold everything that we want it to hold.


Capacity pressure → Capacity miss
If that capacity pressure is *because threads are using different working sets* 
→ Professor calls that an "incoherency miss".
# OpenMP

Language extension used for multi-threaded, shared memory parallelism. 

Key ideas:
- shared vs. private variables

Modeled around the fork-join model.

![[Screenshot 2025-05-04 at 4.59.26 PM.png]]

OpenMP begins as a single process (master thread), and it will execute sequentially until the very first parallel region construct is encountered. The `master` thread waits for all the worker threads to be done, and then, we join all of the threads together when we are done with the said parallel region. 

***FORK***: Master thread creates a team of parallel threads

***JOIN***: When the team threads complete the statements in the parallel region construct, they synchronize and terminate, leaving only the master thread. 

Threads don't ***die***, they go to *sleep*.

**Pragmas** are a preprocessor mechanism C provides for language extensions. 

Good parallel code will still be able to run in a sequential system. 

```cpp
#pragma omp parallel
{
	/* code goes here*/
}
```

