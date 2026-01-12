#Golang
---
---

## Summary

Channels are the pipes that connect goroutines. They come in two flavors: **unbuffered** (synchronous) and **buffered** (asynchronous). Unbuffered channels block the sender until the receiver is ready, guaranteeing synchronization. Buffered channels have a capacity queue; sending only blocks when full, and receiving only blocks when empty. Choosing the right type is critical for correctness and preventing deadlocks.

## Detailed Explanation

### Unbuffered Channels

Capacity is 0.
*   **Send**: Blocks until someone receives.
*   **Receive**: Blocks until someone sends.
*   **Behavior**: Handshake/Synchronization. The transfer happens instantly when both parties meet.

```go
ch := make(chan int) // Unbuffered

go func() {
    ch <- 1 // Blocks here until main receives
    fmt.Println("Sent")
}()

fmt.Println("Receiving...")
val := <-ch // Blocks here until goroutine sends
fmt.Println("Received", val)
```

### Buffered Channels

Capacity > 0.
*   **Send**: Blocks only if buffer is **full**.
*   **Receive**: Blocks only if buffer is **empty**.
*   **Behavior**: Queue. Decouples sender and receiver speed.

```go
ch := make(chan int, 2) // Buffered (capacity 2)

ch <- 1 // Doesn't block (buffer: [1])
ch <- 2 // Doesn't block (buffer: [1, 2])
// ch <- 3 // Would block now!

fmt.Println(<-ch) // 1
fmt.Println(<-ch) // 2
```

### When to Use Which?

| Feature | Unbuffered | Buffered |
| :--- | :--- | :--- |
| **Guarantee** | Delivery guaranteed (sync) | No delivery guarantee (async) |
| **Coupling** | Tight (Sender waits for Receiver) | Loose (Sender runs ahead) |
| **Use Case** | Synchronization, Handover | Performance, Bursty traffic |
| **Deadlock Risk** | Higher (must match send/recv) | Lower (until buffer fills) |

### Deadlock Example

```go
func main() {
    ch := make(chan int) // Unbuffered
    ch <- 1 // DEADLOCK! No other goroutine is reading.
    // main is waiting for a read that will never happen.
}
```

Fix: Use buffered channel OR another goroutine.

```go
func main() {
    ch := make(chan int, 1) // Buffered
    ch <- 1 // OK
}
```

### Channel State Table

| Operation | Nil Channel | Closed Channel | Open & Full | Open & Empty |
| :--- | :--- | :--- | :--- | :--- |
| **Send** | Block Forever | Panic | Block | Write Value |
| **Receive** | Block Forever | Zero Value, false | Read Value | Block |
| **Close** | Panic | Panic | Close | Close |

### Iterating Over Channels

`range` iterates until the channel is closed.

```go
close(ch) // Sender must close
for msg := range ch {
    // Receives until buffer empty AND closed
}
```

## Interview Questions

**Q: What is the main difference between buffered and unbuffered channels?**

**A:** Unbuffered channels provide **synchronization**. A send operation blocks until a corresponding receive operation is ready, effectively performing a "handshake" or "baton pass." Buffered channels provide a **queue**. A send operation only blocks if the buffer is full. Unbuffered guarantees that the receiver has received the data when the sender unblocks; buffered does not.

**Q: What happens if you send to a closed channel?**

**A:** It causes a runtime **panic**. This is a strict rule in Go: only the "owner" (sender) should close the channel to signal no more data. Receivers should never close channels.

**Q: What happens if you receive from a closed channel?**

**A:** You immediately receive the **zero value** of the channel's type. The operation does not block. To distinguish between a real zero value sent before closing and the closed signal, use the two-value receive form: `val, ok := <-ch`. If `ok` is `false`, the channel is closed and empty.

**Q: When would you use a buffered channel of size 1?**

**A:** A buffered channel of size 1 is often used as a **mutex** or a **semaphore** to limit concurrency or coordinate access to a shared resource without blocking the sender immediately if the resource is briefly busy. It's also useful for ensuring a goroutine can send a result (like an error or a completion signal) and exit without leaking, even if the receiver isn't ready immediately.
