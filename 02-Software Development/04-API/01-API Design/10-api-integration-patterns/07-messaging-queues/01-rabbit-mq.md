#API
---
---

# RabbitMQ (AMQP) for Go Developers

## Summary
**RabbitMQ** is a robust, open-source message broker that implements the **AMQP 0-9-1** (Advanced Message Queuing Protocol). It acts as a middleman between applications, allowing them to communicate asynchronously through messages. For Go developers, it provides a highly scalable way to decouple microservices, handle background tasks, and implement complex routing patterns. The modern standard library for Go integration is `github.com/rabbitmq/amqp091-go`, which is the officially maintained fork of the original `streadway/amqp`.

---

## Detailed Explanation

### Core Topology Concepts
RabbitMQ's architecture is based on the interaction between three main components: **Exchanges**, **Queues**, and **Bindings**.

```mermaid
graph LR
    P[Publisher] -->|Message + Routing Key| E[Exchange]
    E -->|Binding| Q1[Queue 1]
    E -->|Binding| Q2[Queue 2]
    Q1 --> C1[Consumer 1]
    Q2 --> C2[Consumer 2]
```

#### 1. Exchanges
The publisher never sends a message directly to a queue. Instead, it sends messages to an **Exchange**. The exchange is responsible for routing messages to queues based on specific rules (types).

*   **Direct**: Routes messages to queues based on an exact match of the **Routing Key**.
*   **Fanout**: Broadcasts messages to all queues bound to it, ignoring the routing key.
*   **Topic**: Routes messages based on wildcard matches between the routing key and the binding pattern (e.g., `logs.*` or `orders.#`).
*   **Headers**: Uses message headers instead of routing keys for routing.

#### 2. Queues and Bindings
*   **Queues**: Buffer where messages are stored until they are consumed. They are FIFO (First-In-First-Out) by default but can be configured for priorities.
*   **Bindings**: The "link" or rule that connects an exchange to a queue.

#### 3. Routing Keys
A string used by the publisher when sending a message to an exchange. The exchange uses this key (depending on its type) to decide which queue(s) should receive the message.

### The AMQP 0-9-1 Protocol
AMQP is a binary application-layer protocol. It defines:
*   **Channels**: Virtual connections inside a single TCP connection. Creating channels is cheap; creating TCP connections is expensive.
*   **Acknowledgements (Ack/Nack)**: Mechanism to ensure messages are not lost. The consumer tells the broker when a message has been processed successfully.
*   **Durability**: Whether the exchange/queue/message survives a broker restart.

### Go Integration (`amqp091-go`)

The most common library today is `github.com/rabbitmq/amqp091-go`.

#### 1. Establishing a Connection and Channel
```go
import (
    "log"
    amqp "github.com/rabbitmq/amqp091-go"
)

func connect() (*amqp.Connection, *amqp.Channel) {
    conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
    failOnError(err, "Failed to connect to RabbitMQ")

    ch, err := conn.Channel()
    failOnError(err, "Failed to open a channel")
    
    return conn, ch
}
```

#### 2. Publishing a Message
In modern versions, always use `context` for timeouts and cancellations.

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

err = ch.PublishWithContext(ctx,
  "logs_exchange", // exchange
  "info",          // routing key
  false,           // mandatory
  false,           // immediate
  amqp.Publishing{
    ContentType: "text/plain",
    Body:        []byte("Hello World"),
  })
```

#### 3. Consuming Messages
Consumers usually run in a long-lived goroutine.

```go
msgs, err := ch.Consume(
  "my_queue", // queue
  "",         // consumer
  false,      // auto-ack (set to false for manual ack)
  false,      // exclusive
  false,      // no-local
  false,      // no-wait
  nil,        // args
)

go func() {
  for d := range msgs {
    log.Printf("Received: %s", d.Body)
    // Manual acknowledgement
    d.Ack(false) 
  }
}()
```

### Best Practices for Go Developers
1.  **Reuse Connections**: Keep the connection open and use multiple channels for different tasks.
2.  **QoS (Quality of Service)**: Use `ch.Qos(prefetchCount, ...)` to control how many messages a consumer can handle at once. A prefetch of `1` ensures a "fair dispatch" pattern.
3.  **Error Handling**: Monitor the `NotifyClose` channel on both Connection and Channel to detect network failures and trigger reconnection logic.
4.  **Structured Data**: Use `encoding/json` or `protobuf` for the message `Body`.

---

## Interview Questions

### 1. What is the difference between a Direct and a Topic exchange?
*   **Answer**: A **Direct** exchange routes messages based on an exact match of the routing key. A **Topic** exchange allows for wildcard matching. `*` matches exactly one word, and `#` matches zero or more words (e.g., `user.signup` vs `user.#`).

### 2. Why should you use Channels instead of opening multiple Connections?
*   **Answer**: TCP connections are resource-intensive (handshakes, memory). **Channels** are "lightweight connections" that share a single TCP socket. They allow for multiplexing and are much more efficient to create and destroy in highly concurrent Go applications.

### 3. What happens if a consumer dies before acknowledging a message?
*   **Answer**: If `auto-ack` is false and the consumer's channel closes before an `Ack` is sent, RabbitMQ understands that the message wasn't fully processed and will **re-queue** it. This ensures "at-least-once" delivery.

### 4. How do you implement a Work Queue (Task Queue) in RabbitMQ?
*   **Answer**: Use multiple consumers listening to the same queue. By setting `ch.Qos(1, 0, false)`, you ensure that RabbitMQ only gives one message to a worker at a time, preventing a single worker from being overwhelmed while others are idle.

### 5. What is "Dead Lettering"?
*   **Answer**: If a message is rejected (`Nack`), expires due to TTL, or the queue exceeds its length, it can be moved to a **Dead Letter Exchange (DLX)**. This is useful for debugging or implementing retry logic with delays.
