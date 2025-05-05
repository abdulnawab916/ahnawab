### Caches ($)
Computer caches are *critical* for modern performance. This is because accessing main memory can take hundreds of instructions worth of time.  Our computers have relatively small caches, they are just small memories that shadow a larger part of bigger memory. (i.e, RAM)

Caches are usually arranged into blocks. (64B or 128B). Memory transfers usually take place around these block sizes.
Caches are critical for maintaining performance. 
### Caches & Locality (Temporal vs. Spatial)

Temporal Locality: If you use data at location X now, you will likely use it 
Spatial Locality: If you use data at location X now, you will likely use it at X + 1 soon.

The goal is to take advantage of both temporal and spatial locality. If it is able to, it's a hit, if not, it's a miss.

### Cache Miss Types

**Compulsory:** This first type of cache miss must occur because the data has not been accessed ever before yet.
A compulsory miss can be reduced by pre-fetching data.

**Capacity:** The cache miss occurs because the data was evicted because there was too much stuff in the cache

**Conflict:** The cache miss occurred because the data was evicted because there was sharing in the cache: Some other data needed to be put in the same place. 

We cannot avoid compulsory misses completely. We can minimize them though by prefetching data of course, so that we do not have to go to main memory as often. 

Conflict misses are pretty rare.
- This rarely happens because most modern processors have very highly associative caches: 
- 16 way is quite common meaning that you would have 17 different pieces of data mapping the same location
- ![[Screenshot 2025-05-03 at 4.46.47 PM.png]]


Capacity misses are the biggest problem, what we need to do is work on smaller pieces of data. 

A really important fact of optimization is that the sequential part of our program runs really fast / efficiently. Otherwise, there really isn't a point.


## Array Strides

***Key things to know:***
*sizeof(int) = 4 bytes*
*array size = 32 elements*
*Fully assoc. cache*
*line size = 16 bytes*
*number of lines = 16*

```cpp
int sum_array(int *my_array, int size, int stride)
{
	int sum = 0;
	for(int i = 0; i < size; i+= stride)
	{
		sum += my_array[i];
	}
}
```

Let's look at two cases, when the stride is value of 1, and when the stride is  a value of 2.

#### When the stride length is 1
Initially, you have a *compulsory miss*, after the compulsory miss, you then, will bring in (pre-fetch) data from main memory. Filling it up all the way to your line size (16 bytes). After doing so, you will keep getting cache hits until you reach the end of your line, where, you will have to go to main memory, and grab in data again. Visually, here is what that looks like.

We have a good hit / miss ratio because our stride is just `1`. So we are always looking at adjacent elements. 

![[Screenshot 2025-05-03 at 4.56.33 PM.png]]

![[Screenshot 2025-05-03 at 4.56.41 PM.png]]

### When the stride length is 2

Now, here is where issues start to arise

Previously, when the stride length was just 1, we were bringing data into our cache line, and we were using it pretty often, (As noted by the number of hits). However, what if I simply wanted to sum our array up based on every other value? (Every 2nd value)

We introduce a new term called `'brought in, but unused'`, let's take a look

![[Screenshot 2025-05-03 at 5.00.10 PM.png]]
### Stride length of 4, not good!

![[Screenshot 2025-05-03 at 5.00.25 PM.png]]

If our stride is >= block length, we do not have an advantage of bringing the entire line in.


## Matrix Multiply

Everyone times everyone, and boom, you add up to there!
(Noted with the orange colors). Unfortunately, matrix multiply is a very very very common problem in the real-world (i.e, lots of applications)

When you multiply two matrices together, the end result is the product. 
Multiplying the two rows, gets you this one square in the last matrix (i.e, the product)
![[Screenshot 2025-05-03 at 5.21.32 PM.png]]

Note: Arrays are commonly stored in row-major order.

When we do the standard matrix multiply, we make really bad use of our cache.
![[Screenshot 2025-05-03 at 5.23.00 PM.png]]

The reason that we get a bunch of cache misses within the B matrix, is b/c of the architecture that we are storing our data in. We store things in row-major order. This means that we are essentially always going to be going to main memory when we try to get values for this matrix. (Depending on the arch. matrices can be stored in column major order as well)
When we get to the next column, what we suffer from is something called a cache eviction, so we can't even reuse the things that were stored previously in the cache. We keep having to go to main memory, which is of course not something that we want to do. What we need to do is transpose our matrix to solve this problem of constant cache evictions.

Thus, what this motivates us to do, is to transpose matrix B, and ensure we are utilizing our cache effectively.
![[Screenshot 2025-05-03 at 5.23.37 PM.png]]

When you transpose matrix B, what this ends up doing is reduce your working-set size. The technique of cache blocking comes in b/c instead of working with this large set of data, let's just work with smaller pieces of data to be more effective with how we are storing things in the cache. 
### Cache Blocking

A technique where data accesses are rearranged to make
better use of the data that is brought into the cache. Helps prevent repeatedly evicting and fetching the same data from the main memory. 

The data access being `rearranged` in this case would be just accessing those little blocks and such instead of the entire row. 

Visual representation:
![[Screenshot 2025-05-04 at 10.34.25 AM.png]]

As a result, what he have is a really low cache miss rate. We are not getting as many cache misses as we would have without using the cache blocking technique. 
Find other articles on the concept of cache blocking

Although this is sequential, this is freaking critical for sequential performance. If your cache is cooked, then you are not going to ever get good parallel performance. You will always just be waiting on main memory.

## Loop Un-rolling

Compiler writers spend a huge amount of time spent optimizing loops. This is due to Amdahl's law. The inner loop of the program is where you spend the most amount of your time in. The caveat is that compilers are often optimized around relatively simple loop shapes. So nothing super complicated. (The bounds should be easy to compute and are small in general) Loop unrolling works really well with good caches. 

```cpp
for(auto i = 0; i < 4; ++i)
{
	...
}
```
A loop with small and fixed bounds. 
You gotta make sure that you have the optimizer on! `(O2, O3, etc)`

```cpp
for(auto j = 0; j < (max & ~3); j = 4){
	for(auto i = 0; i < 4; ++i){
		ar[i + j] += src[i + j] * src2[i + j];
	}

}
```

With the optimization flag, what the compiler does is turn the simple loop above, into this:
```cpp
for(auto j = 0; j < (max & ~3); j += 4)
{
	ar[j] += src[j] * src2[j];
	// this is unrolled 4 more times
	ar[j + 1] ... 
	ar[j + 2] ...
	// Continues, all the way to `j + 3`
}
```

*Gonna have to review concepts regarding loop unrolling*

### (Chip) Multicore Multiprocessor
SMP: (Shared Memory) Symmetric Multiprocessor
- Two or more CPUs/Cores
- Single shared coherent memory
![[Screenshot 2025-05-04 at 11.05.01 AM.png]]

Single physical address space shared by all of the processors / cores
Processors coordinate / communicate through shared variables in memory (via loads and stores) 
- Use of shared data has to be coordinated via synchronization primitives (i.e, locks) that allow access to data only one processor at a time. Basically all multicore computers today are SMP

### Multiprocessor Caches:

- Memory will always be a bottle neck, even within a uni-processor system
- We use caches to reduce the bandwidth demands to main memory
- Each core within a multicore system has their own private cache.
- Only cache misses will have to access to the common shared memory


The problem amongst multiprocessor caches: Coherency issues

Two CPU's on a bus both read the same data; each processor gets a copy of the data and stored it within their respective cache. Then, one CPU performed a write, their cache is up to date, but the other processor has stale data (it is unaware of its own data being stale) 

How can we communicate when one processor changes the state of the shared data? Doers every processor action cause data to change state? Who should be responsible for providing the updated data? What happens to memory while all of this is happening?

### What is the goal of cache coherence?
Architect's job: shared memory
-> keeping cache values coherent

When any processor has cache miss or writes, notify other processors via the interconnection network. 
- If you are only reading, many processors can have copies
- If a processor writes, invalidate any of the other copies

Write transactions from one processor, other caches "snoop" the common interconnect checking for tags that they hold.
- We invalidate any copies of same address modified in the other cache

Keeping state & MSI: Just like we have the `valid` and `dirty` bits for each corresponding cache block, we now introduce a new bit called `shared`.
Remember, state is kept at a block by block basis. All of the data within a block should hypothetically have the same state. 

### More enhancements: 
We want to use ***write-back*** caches. A ***write-through*** cache design uses much more memory bandwidth. We want to minimize the number of writes overall. So "write-back" even in the case of shared cache blocks!!!

We can communicate by broadcasting requests (i.e, shout out to all of the other processors). What is absolutely insane is that even getting some piece of information from a neighboring cache is still much faster / quicker than trying to access something from main memory. Look in your friend's cache! Fuck it. 

What is the difference between a **write-back** cache and a **write-through** cache? Write back will just update the cache, and mark the block as dirty. The main memory will later get updated, when that block is replaced or explicitly flushed.
Write-through: Every time that the CPU writes data to the cache,  it will also write the same data to the main memory immediately. 

Now, each cache tracks the **state** of each block in the cache:
1. **Shared**: up-to-date data, other caches may have a copy. (Valid bit is set, and the shared bit is set for this block)
2. **Modified:** up-to-date data, changed (dirty), no other cache has a copy, OK to write, memory out-of-date, (Both the valid, and the dirty bit are set)
3. Exclusive: Up-to-date data, no other cache has a copy of it, OK to write, memory is up to date. 
		- If any other of the cache reads this line, the state will become `shared`
		- If I were to write to this particular line, the state would become modified, but I don't need to tell any of the other caches that!
		- Only valid is set
4. Owner: up-to-date data, other caches may have a copy (they must be in the Shared state), memory is NOT up to date.
	- This cache can supplies data on a read instead of trying to go to the main memory. Saves the need for a write back when someone else reads the line
	- Valid, Dirty, and shared bit is set
	- When you write, you once again, have to have the other caches invalidate

Conceptual Understanding:

What is the name of the private cache that each processor core has?

What is the cache coherence problem?
It is the uniformity of shared resource data that ends up stored in multiple local caches.

The Cache Coherence Problem:

	It is the challenge of keeping multiple local caches *synchronized* when one of the processors updates its local copy of data which is shared among multiple caches.

Terminology that is really important to understand:
![[Screenshot 2025-05-04 at 12.12.13 PM.png]]

The highlighted line right here is something referred to as a cache line, (i.e, cache block). We keep track of whatever the hell goes on with this particular component within our cache. 

Q: What is the difference between a *cache line* and a *cache block*?
A: The difference between a cache line and a cache block is the following:

*Protocols to solve this problem:*
Snoopy protocol, this is where we essentially will look at whatever is going on within the interconnect and make sure that coherency is in check.

Each core has a private cache.

![[Screenshot 2025-05-04 at 12.56.53 PM.png]]
Any time that you want to interact with the main memory, you have gotta go through this bus, the other processors are gonna be listening because they have to ensure that they do not need to do anything regarding that particular request. (i.e, An example of this could be a read request, let's say I want to read a value from main memory) 

What happens when I try to update the value of a variable that is in a cache block that's currently in the `shared` state? What is the process of what happens?
- The processor will send a upgrade request to the bus, the bus let's the other processor caches know that the upgrade is happening, and will make that block as invalid due to the fact that there is an inconsistency of what lies in that block amongst the caches now!

The next lecture goes more in depth on this particular cache coherency protocol

A write can be placed on the bus, and the `sharers` have to invalidate themselves.
***Links to external learning material to understand caching at a deeper level:***
1. University of Washington – Cache Introduction https://courses.cs.washington.edu/courses/cse378/09wi/lectures/lec15.pdf
2. UC San Diego – The Basics of Caches https://cseweb.ucsd.edu/classes/su07/cse141/cache-handout.pdf
3. GeeksforGeeks – Cache Memory in Computer Organization https://www.geeksforgeeks.org/cache-memory-in-computer-organization/
4. Wikipedia – CPU Cache https://en.wikipedia.org/wiki/CPU_cache
5. Swarthmore College – Cache Architecture and Design https://www.cs.swarthmore.edu/~kwebb/cs31/f18/memhierarchy/caching.html
6. Dive Into Systems – Caching https://diveintosystems.org/book/C11-MemHierarchy/caching.html