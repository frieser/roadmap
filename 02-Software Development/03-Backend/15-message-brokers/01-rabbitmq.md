---
---

# RabbitMQ

## 1. Summary
RabbitMQ is the most widely deployed open-source message broker. It uses the **AMQP (Advanced Message Queuing Protocol)** 0.9.1 by default and is known for its "Smart Broker, Dumb Consumer" philosophy, where the broker handles complex routing and delivery logic.

## 2. Detailed Explanation
RabbitMQ acts as a post office for your applications. It accepts, stores, and forwards binary blobs of data (messages).

### Core Components:
- **Producer**: Sends messages.
- **Exchange**: Receives messages from producers and pushes them to queues based on routing rules.
- **Queue**: Stores messages until they are consumed.
- **Consumer**: Receives and processes messages.
- **Binding**: The link between an exchange and a queue.

### Exchange Types:
- **Direct**: Routing key matches exactly.
- **Topic**: Pattern matching (e.g., `logs.*`).
- **Fanout**: Broadcasts to all bound queues.
- **Headers**: Uses message headers for routing.

## 3. Go-specific Context & Examples
The official client for Go is `github.com/rabbitmq/amqp091-go`.

### Example: Simple Producer
```go
package main

import (
	"context"
	"log"
	"time"

	amqp "github.com/rabbitmq/amqp091-go"
)

func main() {
	// 1. Connect
	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	if err != nil {
		log.Fatal(err)
	}
	defer conn.Close()

	// 2. Open a channel
	ch, err := conn.Channel()
	if err != nil {
		log.Fatal(err)
	}
	defer ch.Close()

	// 3. Declare a queue
	q, err := ch.QueueDeclare("hello", false, false, false, false, nil)

	// 4. Publish
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	err = ch.PublishWithContext(ctx, "", q.Name, false, false, amqp.Publishing{
		ContentType: "text/plain",
		Body:        []byte("Hello Go!"),
	})
}
```

## 4. Interview Questions
1. **Explain the "Smart Broker, Dumb Consumer" model.**
   - In RabbitMQ, the broker is responsible for managing message states (acks), routing, and ensuring delivery. The consumer just waits for messages to be pushed.
2. **What is a "Dead Letter Exchange" (DLX)?**
   - It is an exchange where messages are sent if they cannot be delivered, are rejected, or expire (TTL).
3. **Difference between a "Fanout" and "Topic" exchange?**
   - Fanout broadcasts to everyone; Topic uses wildcard routing keys to filter which queues get which messages.
