#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'dynamodb']
---

## Summary
**DynamoDB Streams** is a Change Data Capture (CDC) feature that captures item-level modifications (Create, Update, Delete) in a DynamoDB table in real-time. It provides a time-ordered sequence of changes, ensuring each record appears exactly once and in the correct order per item. Records are stored in shards and retained for exactly **24 hours**.

## Detailed Explanation

### Core Concepts
- **Shards**: The stream is composed of shards, which are containers for stream records. Shards are automatically managed by AWS and can split to handle increased throughput.
- **Retention**: Data is strictly retained for **24 hours**. If processing fails and isn't resolved within this window, the data is lost.
- **Ordered Delivery**: For any given item, the stream preserves the order of changes. However, across different items, the order is not strictly guaranteed.
- **Exactly-once Processing**: Each modification produces exactly one stream record.

### Stream View Types
When enabling a stream, you must choose what information is captured:
1. `KEYS_ONLY`: Only the primary key attributes of the modified item.
2. `NEW_IMAGE`: The entire item as it appears after modification.
3. `OLD_IMAGE`: The entire item as it appeared before modification.
4. `NEW_AND_OLD_IMAGES`: Both the before and after states of the item.

### Lambda Integration
AWS Lambda is the most common consumer for DynamoDB Streams.
- **Triggers**: Lambda polls the stream and invokes your function synchronously when records are available.
- **Batch Size**: You can configure how many records are sent in a single invocation (1 to 10,000).
- **Error Handling**: By default, if a Lambda fails, it will retry the entire batch until it succeeds or the records expire (24h). This can block the shard (Head-of-Line blocking). Modern Lambda features like `Bisect batch on error` and `Maximum record age` help mitigate this.

### Cross-Region Replication (Global Tables)
- **Global Tables** rely on the underlying CDC mechanism to replicate data across regions.
- In **Version 2017.11.29**, you had to manually enable DynamoDB Streams.
- In **Version 2019.11.21**, replication is managed automatically by DynamoDB, though it conceptually performs the same stream-based replication.
- **Multi-active**: Changes in any region are captured and propagated to all other replica regions.

### DynamoDB Streams vs. Kinesis Data Streams
| Feature | DynamoDB Streams | Kinesis Data Streams (KDS) for DynamoDB |
|---------|------------------|-----------------------------------------|
| **Retention** | 24 Hours (fixed) | Up to 1 Year (configurable) |
| **Consumers** | Max 2 per shard | Max 20+ (using Enhanced Fan-Out) |
| **Pricing** | Free for small usage, then read-unit based | Standard Kinesis pricing |
| **Access** | DynamoDB Streams API | Kinesis Data Streams API |

### Implementation in Go
Using the `aws-lambda-go` and `aws-sdk-go-v2` to process stream events:

```go
package main

import (
	"context"
	"fmt"
	"github.com/aws/aws-lambda-go/events"
	"github.com/aws/aws-lambda-go/lambda"
)

func handler(ctx context.Context, e events.DynamoDBEvent) error {
	for _, record := range e.Records {
		fmt.Printf("Processing event %s for item in table %s\n", record.EventID, record.EventSourceArn)

		switch record.EventName {
		case "INSERT":
			fmt.Println("New item added:", record.Change.NewImage)
		case "MODIFY":
			fmt.Println("Item updated. Old:", record.Change.OldImage, "New:", record.Change.NewImage)
		case "REMOVE":
			fmt.Println("Item deleted:", record.Change.OldImage)
		}
	}
	return nil
}

func main() {
	lambda.Start(handler)
}
```

## Interview Questions

**Q: What is the maximum data retention period for DynamoDB Streams?**
**A:** Exactly 24 hours. This is a hard limit and cannot be increased. If you need longer retention, you should use Kinesis Data Streams integration.

**Q: Does DynamoDB Streams guarantee the order of operations?**
**A:** Yes, but only for a specific item. If item A is updated then deleted, the stream will show the update record before the delete record. There is no global order guarantee across different items.

**Q: What happens if a Lambda function fails while processing a DynamoDB Stream batch?**
**A:** By default, Lambda will retry the same batch indefinitely until it succeeds or the records expire (24 hours). This causes "Head-of-Line blocking" where no new records in that shard can be processed until the failing batch is resolved.

**Q: How many consumers can read from a single DynamoDB Stream shard simultaneously?**
**A:** AWS recommends a maximum of 2 simultaneous consumers per shard to avoid throttling. If more consumers are needed, Kinesis Data Streams integration is the preferred solution.

**Q: What information does the `NEW_AND_OLD_IMAGES` view type provide?**
**A:** It provides both the state of the item before the modification (`OldImage`) and the state of the item after the modification (`NewImage`). This is useful for auditing or calculating the "delta" between states.
