#### Implementing a `work queue` in C++

```cpp
// Preliminaries
#include <queue>
#include <mutex>

template <class T>
class WorkQueue
{
private:
public:
	WorkQueue() {};
	// It doesn't make sense to hold 0 elements, and 
	// we use 0 capacity to be "unlimited capacity"
	WorkQueue(size_t size)
	{
		capacity = size;
		asser(size > 0);
	}
}
```

Fixed capacity (of at least 1) or unbounded
IF queue if full, pushing will block until it is not full
IF queue is empty, popping will block until it is non-empty
Minimum wake ups necessary
- Only send a wakeup if there may be something waiting
- Only send a single wakeup
- "Unlock than wake" paradigm

```cpp
...
WorkQueue(const WorkQueue &) = delete;
void operator=(const WorkQueue &) = delete;

private:
	std::queue<T> data;
	std::mutex lock;
	std::condition_variable notify_get;
	std::condition_variable notify_put;
	size_t capacity = 0;
```

It makes absolutely no sense to make a work queue copy-able / mov-able.

### Logic on safety regarding this workQueue implementation

We ensure that whenever we are manipulating the queue, that we have a lock
- If the queue is empty there may be something waiting to read
- If the queue is full there may be something waiting to write
- It is safe to send a spurious wakeup if nothing is waiting

What we want to do is avoid the wait and lock motif
Thus, we only signal on the condition variable after we unlock
### Putting elements...

```cpp
void put(T &element)
{
	bool wasempty = false;
	// Paranthesis are important for the 'scope' of what we
	// are doing.
	{
		std::unique_lock l(lock);
		while (capacity != 0 && data.size() >= capacity)
		{
			notify_put.wait(1);
		}
		wasempty = data.empty();
		data.push(element);
	}
	if (wasempty)
		notify_get.notify_one();
}
```

### Getting elements
```cpp
T get()
{
    bool wasfull = false;
    std::unique_lock<std::mutex> l(lock);
	
	// This while-loop right here is critical
    while (data.empty()) { 
        notify_get.wait(l); 
    }
    if (data.size() >= capacity) { 
        wasfull = true; 
    }

    auto ret = data.front();
    data.pop();
    l.unlock(); // Explicit unlock happening
	// Another thread could sneak in here, and it 
	// would not be a problem, this is due to the fact
	// that we have our while 
	// loop above re-checking the condition
    if (wasfull) {
        notify_put.notify_one();
    }

    return ret;
}
```

### What if a writer sneaks in?
Let's suppose that a writer sneaks in between a pop() and the notify?
- That one will sneak in and be able to do a `push()`
- But the other one will just go to sleep and go back to waiting...
- And the next pop() will go "Hey, things are full, I need to notify"
- Have to go through such logic to make sure orderings are good

*The rest of the lecture of Weaver going through coding examples*
- *Setting breakpoints, and the importance of randomization when testing*

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

(Recap from the previous lecture where he left off)

![[Screenshot 2025-05-03 at 10.52.47 AM.png]]

The solution is Coroutines. which exist in Kotlin and Go
Within Go, **every** function is potentially suspending
The user level allocates 1 OS thread for each CPU
Then manually schedules and switches between user threads
### How are go routines (co-routines) so cheap?

Go routines are cheap! They do not use the conventional C Call stack
It uses a set of linked-list connected hunks

Allocates a 4 kilobyte hunk. It does dynamic allocation of growth for the call stack for which is necessary. The call stack for go is NOT contiguous, but is allocated with these disconnected hunks.


![[Screenshot 2025-05-03 at 11.00.16 AM.png]]

### Launching a go routine

```go
go function(args...)
```

This is the syntax to spawn a new go-routine. This is just like a normal function call, but with the word go in front of the function. It evaluates all of the arguments in the current go routine. And then, it launches the function itself in a new go-routine. What it really is, is call this function in a new coroutine, and ignore the return value.  Go does not have a conventional join for threads.

#### Remember...
- In go, every function is potentially suspending
- Which gives far more opportunities for the go scheduler to switch between currently running tasks
- There is s still a timer interrupt
- The scheduler will receive a timer interrupt on a regular basis to execute a mandatory context switch

