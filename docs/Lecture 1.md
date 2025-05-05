Amdahl's Law: The limit on this class
Speedup due to enhancement `E` is 
Speedup w/E = Exec time w/o E / Exec time w/E

Speedup = 1 / ((1 - F) + F / S)
The non speedup part is the left term, and the speedup part is the right term
![[Screenshot 2025-05-01 at 11.21.18 AM.png]]

The speedup **WITH** the enhancement E is the ratio of the execution time without the enhancement divided by the execution time WITH the enhancement. (Derivation is shown below, the `T` cancels out)
Speedup = 1 / [ (1 - F) + (F / S) ]

For calculating the total execution time with the enhancement in place
what you need to account for is the time that it takes to process the part that is NOT sped up, as well as the time it takes for the part that is sped up to run. 
You add these two terms together

First, you have `T` which is the original time that it takes for the entire program to finish
So for the `slow` part, this means it would take T x (1 - F) time. And for the faster part, this means that the total amount of time that would take would be T x (F/S). The S variable introduced is the *speedup factor.* I think about this as the number of cores being *INCREASED* overall. So we can divide up the work amongst each CPU core. 

So for the execution time with the enhancement, this would be 
exec. time w/ enhancement = exec. time w/o eh (T) x [(1-F) + F/S]
The **speedup** is the (time before) / (time after). 

***Speedup derivation:***
T / T(1 - F) + T(F/S)
- The T's Cancel out, leaving the speedup to be ---> 1 / (1 - F) + (F / S)

![[Screenshot 2025-05-01 at 1.59.24 PM.png]]

The speedup that you get from making the program TWICE as fast is the same as making 25 percent of the program 20 times as fast. BOOOOO

![[Screenshot 2025-05-01 at 2.00.54 PM.png]]

Amdahl's law tells us that in order for us to achieve linear speedup with 100 processors, none of the original computation can be scalar!

To get a speedup of 90 from 100 processors, the percentage of the original program that could be scalar would have to be 0.1% or less.

We are often limited by the sequential stuff

![[Screenshot 2025-05-01 at 2.03.40 PM.png]]

## Strong Scaling vs. Weak Scaling:

**Strong Scaling:** When speedup can be achieved on a parallel processor without increasing the size of the problem.
**Weak Scaling:**  When speedup can be achieved on a parallel processor by increasing the size of the problem proportionally to the increase in the number of processors.

Example: The time it takes to OCR a document
- It make take a minute to OCR a doc
- You can get really good scaling when you want to OCR 1000 docs, just throw a bunch of processors at it (50 to 100 processors)!
	- However, you increased the amount of data as the number of processors increased
	- You still have not solved the issue of processing the 1 OCR doc faster
## Another important term, `Load Balancing`

Another important factor: every processor doing the same amount of work.
- Just one unit with twice the load of others cuts speedup almost in half
Example: we have 50 computers to complete one task, the problem of load balancing can arise if we have 1 computer, doing HALF of the work, and the rest split of the other half. No matter how much you try to make it better, it will never get better cause that ONE computer is doing half the work. 

# Real-world examples 😔

*I don't think this will be super important, but here is the slide*

![[Screenshot 2025-05-01 at 2.37.13 PM.png]]

# Premature Optimization

The runtime of a program is really...
- The runtime of the program on **all** the inputs that you will ever run on it
- The time it takes you to write the program in the first place
So... do **NOT** Prematurely optimize
- If you worry about getting things right first, you may never have to optimize it at all
- Additionally, you should worry about making your code readable and well-documented
	- This is because the runtime of a modified version of the program is the runtime on all inputs plus the time it takes you to relearn what you did in order to modify it

