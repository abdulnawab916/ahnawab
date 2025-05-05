### Threads, Synchronization, and coroutines
Another real-world example of locks
Let's say you are working with a piece of dangerous piece of equipment, something that IF it turns on, you will die horribly!
- Power off the system
- Lock the power off
- Add an ownership tag for who turned this shit off
If you want to unlock it, you do everything to find the owner
This relates directly to critical sections

##  The simplest type of lock: The Lock / Mutex
Effectively every single programming language with concurrency has a lock or a mutex.
Only one thread at a time can `own` the lock
There are two primary operations when it comes to working with locks
- Lock: Acquire the lock. If the lock is not available, block until it is available
- Unlock: Release the lock, unblocking another thread if it is appropriate
Often also a `try-lock`
- Returns a boolean: True if it was unlocked (Grabs the lock), false otherwise. This is usually not advised to do however. 

## Weaver's opinion of what the best type of lock is...

## *C++'s std::unique_lock*

```cpp
std::mutex m;
...
{ std::unique_lock ul(m);
	
}
```
- In reality, we could just manipulate the mutex directly
	- Three operations of mutexes: lock, try_lock, and unlock
- C++ has some nice RAII semantics!
- unique_lock is a wrapper around a mutex, and when the unique_lock acquires the lock, it will inherently lock the mutex
- The cool part about unique_locks are that when it goes out of scope, unique_lock will release it! (Unlocking the mutex automatically!)
- You don't have to worry about unlocking it manually

## RAII
- C++ and Rust memory semantics know when an object is unreachable
	- Goes out of scope in a function
	- Reference counted smart pointer decrements all the way down to `0`
	- Unique pointer is unreachable
	- Explicit delete
- Objects have the destructor called promptly

## Features of C++'s std::unique_lock
- It has move, but NOT copy semantics
	- What this means is that we can return it's value, but we cannot pass around the value
- It acts as an RAII wrapper, so you never need to unlock the lock
- Instead, the lock just goes out of scope, and it goes 'poof, unlock' IF it was locked
- We are still able to pass it as an argument via reference
- We also still have the ability to lock and unlock it manually

### But, what about little old Rust?
- There's no unlock function at all
- The lock can only be unlocked automatically, we don't even have the option of doing it manually
- Rust's locks have the notion of being ***[[poisoned]]***, what does this mean?
- Rust allows individual threads to panic. 
- When a thread panics, we assume that the specific data it was operating on **may** be in a corrupt or invalid state, especially if the panic occurred while modifying shared data.
- We assume that the data is in an inconsistent state

# Other language Locks: 
### OpenMP critical...
*An extension to C++, working with `pragmas`*

```c
#pragma omp critical
{
	...
}
```

This designates the region of code as 'critical'
An implicit single lock is added for this particular region 
Starting the block, will acquire the lock
Ending the block, releases the lock

This gives us the ability to say that only one thread can enter this region at a time

## Java's synchronized

Java is one of the languages that allows locks to be reentrant
What this means is that the same thread can acquire the same lock multiple times without deadlocking itself. Every time the thread locks it, increments an internal counter.  The thread must unlock the lock the same number of times before the lock is released for others.

# Go's RWMutex ('Lock')

```go
var l sync.RWMutex
```

There are two different versions of locking:
`RLock` and `Lock`

As many can call RLock as desired. This is because multiple threads can read the same data. This does not cause any type of issues, we just have multiple readers.

However, when a call to Lock happens, no further RLocks will succeed. Until the Lock call gets the lock and then releases it:
- prevents starvation when there are a lot of readers

# Locks as ordering barriers
Within a single thread, writes happen 'in order'
Our code could have the following:
a = x;
y = a;

The write to a happens before the write to y

But, this same visibility does not apply to other threads
- Another thread may see the write to y (of what a becomes) before the write to a is seen!
Locks or any other type of synchronization primitive also act as ordering barriers
Ensuring that all writes will complete before the other thread can see the data.

# Locks have limitations
It isn't the most flexible of synchronization primitives...
This is because the nature of how they work is that it is only 'one at a time', and everything else is locked out
When a lock is released another thread block on the lock resumes
We want much more than that, we want the ability to signal as well
Two options for higher-level operations:
- Condition Variables
- Work Queues / Channels

# Synchronization Variables
In addition to the lock, we have a synchronization variable, that is associated with it.
Two operations that are supported:
1. wait: in an atomic operation it releases the lock and sets this current thread to be in the waiting state
2. notify: take **one** or **all** of the waiting threads and notify them
### What about when we wake up?
Thread:
    Acquire lock
    Change shared data
    (optional) Notify other threads while holding the lock
    Release lock
    (optional) Notify other threads after releasing the lock

# C++'s std::condition_variable

```cpp
std::condition_variable::wait
- void wait( std::unique_lock<std::mutex>& lock );
- void wait( std::unique_lock<std::mutex>& lock, Predicate pred );
```

Wait has two options...
- The first wants just a unique_lock
- The second wants a predicate function as well
- On waking up, the 2nd version evaluates the predicate, and, if false, calls wait again
- Predicate is usually best done as a lambda function

```cpp
[&]{return statement; }
```

Threads have to communicate with each other. This will be in regards to some resource as well, a DB, a particular part of the physical memory, etc.

We want to have overall program control. We cannot / should never have non-deterministic behavior within our program.  Recall, when one of the threads tries to do a *write*, then issues arise. Just one of the threads has to do a write in order for there to be an issue. 

Thread 1 could be trying to do a read, and then, a write to a variable, however, the little gap of time between the read and the write, another thread is able to come in and read in a stale value of the variable that we were reading. (Not good)![[Screenshot 2025-05-02 at 4.43.33 PM.png]]
![[Screenshot 2025-05-02 at 4.44.26 PM.png]]

The middle part is called the critical section. Coordination between threads, get a consistent behavior of our overall program. Mutual exclusiveness.

When a `.wait()` takes place, you do not need to stick around, you can just go to another thread and do something there. (Flipping your core around) get busy, quit being lazy.

# The Producer and the Consumer problem

This is a very common problem. We have a bunch of **producers** that generate data/requests. (This can be one / or more threads)

We have a bunch of **consumers** that accept data / requests (again, this can be one, or more threads)

So, what we need, and want, is a [[queue]]!
# Queue Properties

**Capacity:**
Queue can hold 0, n, or infinite elements with each option specified.

#### How does blocking work when it comes to the producer and consumer problem?

If there is no data available, a **consumer** will block until data is available in the queue

If there is no space in the queue, a **producer** will block unitl there is space that is available

#### Special case of a size '0' work queue
Blocks **until** both a producer and a consumer are available and then the queue opens when they are. 

(Also referred to as a ***work queue***)

## The Expense of OS Threads

- **Memory**
	- An OS Thread neeeds enough space for a large call stack (about 8 MB per thread or so)
	- Has to be a contiguous allocation of memory
- **Startup**
	- Starting an OS thread requires invoking the OS through the interrupt handler
- **Switching**
	- Switching between threads involves the OS through the interrupt handler. (Context switching, round robin, time-slicing, etc)

Note: What is the[[ interrupt handler]]?

Another problem with OS threads are that they are *expensive*!



