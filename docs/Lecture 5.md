


# Channels in Go
- A channel is essentially a work queue with a specified capacity

```go
c := make(chan string)
// A channel for strings, with the default capacity of 0
```

When a channel in go has a buffer capacity of some sort, this means that they work like a FIFO queue.

Q: When do the sender and the reciever block when reading and writing from a channel?
A:

Q: How come channels with no buffer in Go are special?
A:

### Channels alone are not enough
A channel is just a work queue, this is not enough, what MAKES a channel special is how it works with the select statement. This is a way to wait on multiple / consecutive channel operations

```go
// Select statements working with Channels in Go
select {
	clause:
		stmt+
		...
}

```

Q: What does the clause have to be?
A: The clause has to be a channel operation
```Go
c <- "foo"
x:= <- c

// 1. Placing the string "foo" onto the channel (i.e, channel write)
// 2. declaring a variable named x, and reading from the channel `c`
```

The part to the right of the channel operation is evaluated. Then if one channel operation can complete, it is run
- Otherwise, things block until a channel is available
Then, when one channel operation can complete, it is run

### Go, by example
```go
func main() {
	c1 := make(chan string)
	c2 := make(chan string)
	go func() {
		time.Sleep(1 * time.Second)
		c1 <- "first"
	}()
	go func() {
		time.sleep(2 * time.Second)
		c2 <- "second"
	}()
	for i:= 0; i < 2; i++ {
		select {
		case msg1 := <-c1:
			fmt.Println("got message for", msg1)
		case msg2 := <-c2:
			fmt.Println("got message for", msg2)
		}
	}
	
}
```

Q: Which select statement will run first?
A: I believe that c1 goes first because we launched that go function first.





```go


```

