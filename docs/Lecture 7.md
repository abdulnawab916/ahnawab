## Deadlock

**Definition:** *a system state in which no progress is possible because everything is locked waiting for something else.*
#### Dining Lawyer's Problem:

- Pontificate until the left fork is available, when it is, pick it up
- Pontificate until the right fork is avail, when it is, pick it up
- When ya got both of the forks, then start eating big boy
- But you can only eat for a certain amount of time
- Alright, you are done eating big boy, let's let another big boy eat
- Put right fork down
- Put left fork down
- Repeat from the beginning

![[Screenshot 2025-05-03 at 11.43.29 AM.png]]

### Different classes of deadlocks
- Architectural Maldesign
	- The dining lawyers
- No counterparty
- Recursive locking
- Indirect recursive locking

The issue with the dining lawyers is that just **one** fork grab is atomic, but, **both** fork grabs are not atomic. A lawyer will hold onto a fork and cannot steal them from each other.  
The order they attempt to acquire the forks creates a cycle: 

Everyone can acquire lock A and nobody can get lock B.

Our solution to this is to change the overall architecture of our design, to prevent this.

# Techniques of solving the deadlock problem
### Method #1 of fixing a deadlock: ***global lock***
Instead of having the sequential process of grabbing lock A, and then grabbing lock B. What we do is have there be a global lock, and we grab the global lock. 
This allows to have the atomic operation of grabbing both forks. Each lawyer just grabs the global lock, which is equivalent to grabbing 2 locks sequentially, this gets rid of the deadlock problem. Only atomic operations are occurring.
Creating this global lock thingy is not good, because this fucks our throughput over. Things get slow as shit.

### Method #2 of fixing a deadlock: ***Grab even first***

Number the forks 1-n
- Instead of the lawyer grabbing the 'letf' fork first
- Grab the 'even' fork first
Now, there will always be a spare fork for somebody
- Specifically, starting with the last one
What this does is break the cycle, b/c deadlocks only occur when there is a cycle of locks. What we need to do is ensure that this cycle can never occur.

### Method #3 of fixing deadlock: Just Kill One...
Detect that no forward progress is being made...
- Everyone has a fork but no one is eating
Pick a lawyer, and kill that dude 😬
Now, we should be able to make forward progress!
A variant of this solution is that we just never let it happen. 
(i.e, only allowing 'n-1' lawyers, so an extra fork between one pair of lawyers)

### The common deadlock bug: No counterparty...
In the Go PL, this is more common for channels, we can have coroutine 1, which is wanting to write / read to a channel. However, there is no other co-routine that knows about the channel. This is guaranteed to deadlock. Go will just panic this since the Go runtime will have knowledge that no other coroutine knows about the channel we are reading and writing to.

### The common deadlock bug: Direct Recursive Lock / Channel


### A more subtle deadlock bug: Indirect Recursion...



### Race conditions

Two threads accessing the same data, but, one of them is ***writing***



### DNS Overview

DNS will turn a human abstraction (www.google.com) into an IP Address
Can also contain other data.
Known as a ***performance-critical distributed database.***

DNS security is essential for the web. DNS is based on a notion of hiearchical trust. (You trust . for everything, .com. for everything .com, so on and so forth)

### DNS Lookups via a Resolver

![[Screenshot 2025-05-03 at 3.11.07 PM.png]]

We are able to cache entries for a certain period of time

### DNS Protocol

A lightweight exchange of query and reply messages, both with the same message format. DNS is primarily through UDP, if you use TCP, this just adds a 2 byte message length. All servers are on port 53. Clients usually use port 53, but, they can use any port.


![[Screenshot 2025-05-03 at 3.13.44 PM.png]]

![[Screenshot 2025-05-03 at 3.14.00 PM.png]]

### DNS Records and RRSETs
DNS resource records (RR) can be of many types
- Name TYPE Value
	- Also a TTL field, how long in seconds this entry can be cached for
- Addressing
	- A: IPv4 Addresses
	- AAAA: IPv6 Addresses
	- CNAME: Aliases, 'Name X should be name Z'
	- MX: 'The mail server for this name is B'
- DNS related:
	- NS: 'The authority server you should contact is named Y'
	- SOA: "The operator of this domain is Y"
- Other:
	- Text records, cryptographic info
- Groups of records form an **RRSET**
	- i.e, all the nameservers for a given domain

