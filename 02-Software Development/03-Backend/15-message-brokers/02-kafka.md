---
---

# Apache Kafka

## 1. Summary
Apache Kafka is a distributed event store and stream-processing platform. Unlike traditional brokers, it is based on a **distributed commit log**. It is designed for high-throughput, fault-tolerance, and low-latency, following a "Dumb Broker, Smart Consumer" philosophy.

## 2. Detailed Explanation
Kafka stores streams of records in categories called **Topics**. Each topic is partitioned across multiple servers (brokers).

### Key Concepts:
- **Partition**: An ordered, immutable sequence of records.
- **Offset**: A unique ID assigned to each record in a partition.
- **Consumer Group**: A group of consumers that work together to consume a topic (each partition is read by exactly one member of the group).
- **Retention**: Kafka keeps all messages for a set period, allowing consumers to "replay" historical data.

### Strengths:
- **Scalability**: Thousands of partitions and millions of messages per second.
- **Durability**: Messages are written to disk and replicated.
- **Replayability**: Consumers can reset their offsets to read old data again.

## 3. Go-specific Context & Examples
Popular libraries include `github.com/segmentio/kafka-go` (pure Go) and `github.com/confluentinc/confluent-kafka-go` (wrapper around `librdkafka`).

### Example using `kafka-go`:
```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/segmentio/kafka-go"
)

func main() {
	// 1. Initialize Reader (Consumer)
	r := kafka.NewReader(kafka.ReaderConfig{
		Brokers:  []string{"localhost:9092"},
		GroupID:  "consumer-group-id",
		Topic:    "my-topic",
		MaxBytes: 10e6, // 10MB
	})
	defer r.Close()

	for {
		// 2. Fetch Message
		m, err := r.ReadMessage(context.Background())
		if err != nil {
			break
		}
		fmt.Printf("message at offset %d: %s = %s\n", m.Offset, string(m.Key), string(m.Value))
	}
}
```

## 4. Interview Questions
1. **What is a "Consumer Group" in Kafka?**
   - A way to parallelize consumption. Each partition of a topic is assigned to one consumer in the group, ensuring load balancing.
2. **How does Kafka achieve such high throughput?**
   - By using sequential disk I/O, zero-copy (sendfile), and batching messages.
3. **What happens if a consumer in a group fails?**
   - Kafka triggers a "Rebalance," and the partitions owned by the failed consumer are reassigned to the remaining active consumers.
