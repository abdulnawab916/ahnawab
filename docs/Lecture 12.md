# Latency, Throughput, Dependencies, etc...
The real-world is a **distributed**, **parallel system.**

**We see this often in computing:**
DNS is a distributed, parallel-access, mostly read-only datastore.

The *Web* is a distributed, parallel-access computational process.

***Definitions:***

**Latency:** The time it takes for the first ***thingy.***

- For a data communication network, this would be the time that it takes for the first bit to arrive.
- For communication, this is often the round trip time. The time it takes a 'ping' message to transmit the network, be responded to, and for it to return. 
- For memory: The time for the for the first bit to return from accessing the memory device
- For `stuff`: the time it takes for the first thing to arrive
- Units of latency are ***time***: Either are directly just seconds, or in another unit of time (clock cycles)

# Relating Latency and Cost...
If the capabilities were otherwise the same for two fabrics... you will always and always and should choose the *lower latency option*.

Overall, if you have a choice between low-latency vs. a high latency fabric, you will always choose the low-latency option. Assuming everything else is equal / the same. 
# Relating the concept of latency to memory
L1 Cache is low latency, however this cache has a low capacity
L2/L3 cache are **larger** and have a higher latency than L1 cache.
DRAM is much higher latency

**DRAM**:
From a physics standpoint, DRAM is a lot slower to access the *first* element of memory. But, it is much cheaper (i.e, orders of **magnitude** cheaper) per bit stored 

High performance computing is highly memory aware. Most of the focus is centered around ensuring that our memory accesses are fast. It would be amazing if we were able to store everything in L1 Cache.
# Latency & Decision Making 

Your ***decision cycle*** (the time it takes to react to something and **do** the response) has to be ***longer*** than the latency itself.

If the *things* happen faster than you are able to react, what are you going to be able to do about it? You can't really do anything because you didn't have enough time to react!

*Relating this to the real-world:*
Current shipments arriving left before liberation day
So they could only have been affected by the possibility of tariffs not the reality. 

The whipsawing `yes and no` cycle on a daily basis can not be handled when the shipping of things takes weeks alone!

# Bandwidth / Throughput: Stuff per unit time

Bandwidth/Throughput: The ***steady state*** ability to process at a maximum rate of elements. 
Units are the amount of *thingy* per unit time.
- Parallel computing is about *maximizing* the overall throughput on parallel tasks.

If you have a sequential task, it might take longer in a parallel computation (i.e, a higher latency) but that's fine. 

You could want to ship something that will take longer to get to you, but, you are able to get really high-throughput!

# Improving Memory Bandwidth

The [[GPU]]'s *secret* for performance
1. A lot of memory bandwidth simply by adding more wires
	- 900 GB/s max on a 5080 through a combination of wide (256b wide) and fast (very high clock rate) memory busses
	- The memory ***latency*** is still terrible
2. Doing a lot of work on other things while waiting on memory
	- Compute model is focused on being able to stream a massive amount of memory through the device:
		- Generate request for a large hunk
		- Lots of work on previous hunks
		- Get this hunk, generate a request for the next hunk


# Parallel Throughput is an ***insufficient*** metric alone

Throughput alone is not sufficient enough of a metric on a parallel task. If it's parallel, you can always just throw more resources at it! (i.e, real-world examples. Instead of one plane, send 5. Instead of a 64b memory bus, use a 256 memory bus). Sea shipping is so much more attractive because you are moving the same amount of shipment, for a way cheaper cost. This is the way more desirable option. 


# Why latency is >> than parallel throughput

If the task in question is latency bound, then parallel throughput doesn't help. A lot of `things` are latency bound and not `throughput` bound. 

What does it mean for a task to be latency bound? A piece of code is latency bound **when the GPU cannot keep busy with the available/exposed parallel work**. The general strategy for a latency bound code will be to expose more parallel work

If said task is throughput bound, throwing more of a low-latency tool can work.
Each unit of work finishes faster leads to more units processed per second.

***Example:***
Suppose: You can process **1 file every 100 ms** (latency per file = 100 ms). Your **throughput** = 10 files per second. If you switch to a **low-latency tool** that cuts latency to **50 ms per file**: **Throughput now doubles** → 20 files per second.

# Buffering & Caching

A flow through device. Often times, processes have a buffer. A location where one can delay/store up to X thingies. EG, a work queue would be a buffer. This is very common on networking to handle contention. If there are multiple inputs wanting to share an output, buffer up some of the traffic. This is a really popular technique to smooth out a fairly *bursty* process. 

# Caching

Cache is a *store* device. Related are caches that can store and quickly access up to X *thingies*. A small amount of data for a working set. Both the concepts of a *buffer* and a *cache* have the notion of ***capacity***: the number of thingies that it can support. 

# Buffering in the real-world

A business's inventory is a buffer designed to smooth out the bursty process of purchasing. Sell X thingies per day means buying X thingies per day. But buying X thingies is bursty, We get 30X thingies at once. Our solution would be a 30 day buffer inventory. This is very common in parallel services. In the *wait for parallelism model*, we can often get bursts of requests: a bit of buffering might be useful to smooth out the computation. In **parallel services** (like web servers, cloud apps, etc.): **Requests don’t always arrive evenly.** Sometimes, **bursts** happen → e.g., lots of user requests at once. Because of the **wait-for-parallelism model**: Even if the system _can_ process requests in parallel, It might have to **wait** for the next batch of requests or wait for available workers. Even if a huge number of requests arrive at once, they can **wait in the buffer** while the workers process them as fast as they can. Weaver discusses the pool of workers architecture. Throw a bunch of workers into a pool, when tasks come off of the buffer, then one of the workers will do something. 

# Measuring Buffering: Capacity and Cost

Buffers and caches have some type of maximum capacity. The number of thingies that can be held in it. You are paying for what the maximum capacity will be, even if you're not using the space for all of it.  The cost is not associated with what you're actually using. (i.e, the utilized capacity). 

# Option: Minimizing capacity 

The less capacity, the less cost that you have. Make sure the thingies arrive the day before you need it and not the day after. There are ***implicit buffers*** in transit time. High performance DRAM systems do this!

# Dependencies

A dependency relationship where A depends on B requires that B be completed before A can be initiated. Dependencies will ***always*** inhibit parallelism. Broken dependencies cause major issues.

