#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'dynamodb']
---

## Summary
DynamoDB offers two capacity modes for processing reads and writes on your tables: **Provisioned** and **On-Demand**. Provisioned mode allows you to specify the number of reads and writes per second, often used with Auto Scaling to manage costs, while On-Demand mode automatically scales to handle any volume of traffic with a pay-per-request pricing model. Choosing the right mode is critical for balancing performance requirements with cost efficiency.

## Detailed Explanation

### 1. Provisioned Capacity Mode
In this mode, you specify the maximum amount of throughput your application can consume from a table. This is measured in **Read Capacity Units (RCU)** and **Write Capacity Units (WCU)**.

*   **Write Capacity Unit (WCU)**: 1 WCU provides one write per second for an item up to 1 KB in size.
*   **Read Capacity Unit (RCU)**:
    *   1 **Strongly Consistent** read per second for an item up to 4 KB.
    *   2 **Eventually Consistent** reads per second for an item up to 4 KB.
    *   1 **Transactional** read request consumes 2 RCUs (for items up to 4 KB).

**Auto Scaling**: DynamoDB can automatically adjust your table's provisioned capacity based on actual traffic using AWS Application Auto Scaling. You define a target utilization (e.g., 70%), and DynamoDB manages the RCU/WCU within your specified minimum and maximum bounds.

### 2. On-Demand Capacity Mode
On-demand mode is a flexible billing option capable of serving thousands of requests per second without capacity planning.
*   **Scaling**: It instantly accommodates workloads as they ramp up to previously reached peak traffic. If you exceed the peak, it continues to scale, though more gradually.
*   **Pricing**: You pay only for what you use (Read/Write Request Units).
*   **Best For**:
    *   New tables with unknown workloads.
    *   Unpredictable application traffic.
    *   Applications that prefer a simple "pay-as-you-go" model.

### 3. Throughput Calculations
To calculate the units required for an item:
*   **Writes**: `Total WCU = Number of items per second * ceil(Item size in KB / 1 KB)`
*   **Reads**: `Total RCU = Number of items per second * ceil(Item size in KB / 4 KB)`
    *   *Note*: Multiply by 2 for Transactional, or divide by 2 for Eventually Consistent.

### 4. Switching Modes
You can switch between Provisioned and On-Demand modes **once every 24 hours**. When switching from On-Demand to Provisioned, you must specify the initial RCU and WCU values.

### Go Implementation (Capacity Settings)
Using the AWS SDK for Go (v2), here is how you create a table with **On-Demand** (Pay-Per-Request) capacity or update an existing table to **Provisioned** mode.

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/aws/aws-sdk-go-v2/aws"
	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/dynamodb"
	"github.com/aws/aws-sdk-go-v2/service/dynamodb/types"
)

func main() {
	// Load the SDK configuration
	cfg, err := config.LoadDefaultConfig(context.TODO(), config.WithRegion("us-east-1"))
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	client := dynamodb.NewFromConfig(cfg)

	// Example: Updating a table to Provisioned Capacity with Auto Scaling 
	// (Note: Auto Scaling itself is configured via the ApplicationAutoScaling client)
	_, err = client.UpdateTable(context.TODO(), &dynamodb.UpdateTableInput{
		TableName: aws.String("MyTable"),
		ProvisionedThroughput: &types.ProvisionedThroughput{
			ReadCapacityUnits:  aws.Int64(10),
			WriteCapacityUnits: aws.Int64(10),
		},
	})

	if err != nil {
		log.Fatalf("failed to update table capacity, %v", err)
	}

	fmt.Println("Successfully updated table capacity to Provisioned Mode")
}
```

## Interview Questions

**Q: How many RCUs are required to perform 10 strongly consistent reads per second for an item that is 6 KB in size?**
**A:** First, calculate the RCUs per item: `ceil(6 KB / 4 KB) = 2 RCUs`. Then, multiply by the number of reads per second: `2 RCUs * 10 = 20 RCUs`.

**Q: What happens if your application exceeds the provisioned throughput on a table?**
**A:** DynamoDB will return a `ProvisionedThroughputExceededException`. Applications should handle this using exponential backoff (which the AWS SDKs do automatically) or by enabling DynamoDB Auto Scaling.

**Q: When would you choose Provisioned mode over On-Demand?**
**A:** Provisioned mode is more cost-effective for workloads with predictable or steady traffic where you can baseline your capacity. It is also preferred when you want to set a strict "ceiling" on costs to prevent unexpected bills from traffic spikes.

**Q: What is the primary difference between Read Request Units (RRU) and Read Capacity Units (RCU)?**
**A:** RRUs are used in **On-Demand** mode and represent a single request. RCUs are used in **Provisioned** mode and represent a sustained rate of throughput (requests per second). The calculation logic (4 KB chunks) remains the same for both.

**Q: Can you use Auto Scaling with On-Demand capacity mode?**
**A:** No. On-Demand mode handles scaling automatically at the service level. Auto Scaling is a feature specifically designed to manage the RCU/WCU settings for **Provisioned** capacity tables.
