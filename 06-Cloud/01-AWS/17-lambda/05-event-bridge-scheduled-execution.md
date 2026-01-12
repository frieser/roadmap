#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'lambda']
---

## Summary
AWS Lambda Scheduled Execution allows you to trigger functions at specific times or intervals without managing a persistent server. This is primarily achieved through **Amazon EventBridge Scheduler** (the modern standard) or **EventBridge Rules** (legacy). It supports both simple **Rate expressions** (e.g., every 5 minutes) and complex **Cron expressions** for precise timing. This pattern is essential for automation tasks like database cleanups, report generation, and periodic health checks.

## Detailed Explanation

### How Scheduled Execution Works
In a scheduled execution pattern, Amazon EventBridge acts as the orchestrator. It maintains the timer and sends a JSON event to the target Lambda function when the schedule matches.

1.  **EventBridge Scheduler**: A serverless scheduler that can trigger more than 270 AWS services. It is more scalable and flexible than legacy rules.
2.  **Target**: The specific Lambda function (or other AWS service) to be invoked.
3.  **Permissions**: EventBridge needs permission to invoke the Lambda function (`lambda:InvokeFunction`).

### Schedule Expressions

#### 1. Rate Expressions
Used for simple periodic tasks.
- **Syntax**: `rate(value unit)`
- **Examples**:
    - `rate(1 minute)`
    - `rate(2 hours)`
    - `rate(5 days)`

#### 2. Cron Expressions
Used for complex timing (e.g., "every Monday at 8 AM").
- **Syntax**: `cron(minutes hours day-of-month month day-of-week year)`
- **Wildcards**:
    - `*` : All values.
    - `?` : No specific value (used for day-of-month or day-of-week).
    - `-` : Range.
    - `/` : Increments.
    - `L` : Last day of month/week.
    - `W` : Weekday.
- **Example**: `cron(0 8 ? * MON *)` runs every Monday at 08:00 UTC.

### Input JSON
You can customize the event payload sent to the Lambda function:
- **Constant JSON**: A static JSON object passed every time.
- **Input Transformer**: Extracts specific parts of the EventBridge event or adds metadata.

### Implementation in Go
When triggered by EventBridge, the Lambda function receives a specific event structure.

```go
package main

import (
	"context"
	"fmt"
	"github.com/aws/aws-lambda-go/lambda"
)

// ScheduledEvent represents the basic structure of an EventBridge scheduled event
type ScheduledEvent struct {
	Version    string   `json:"version"`
	ID         string   `json:"id"`
	DetailType string   `json:"detail-type"`
	Source     string   `json:"source"`
	Account    string   `json:"account"`
	Time       string   `json:"time"`
	Region     string   `json:"region"`
	Resources  []string `json:"resources"`
	Detail     struct{} `json:"detail"` // Empty for basic scheduled rules
}

func handler(ctx context.Context, event ScheduledEvent) error {
	fmt.Printf("Lambda triggered at %s\n", event.Time)
	fmt.Printf("Source: %s, ID: %s\n", event.Source, event.ID)
	
	// Perform scheduled task (e.g., Cleanup, Backup)
	return nil
}

func main() {
	lambda.Start(handler)
}
```

## Interview Questions

**Q: What is the difference between EventBridge Scheduler and EventBridge Rules for Lambda scheduling?**
**A:** EventBridge Scheduler is the newer, more capable service. It supports one-time schedules, higher scalability (millions of schedules), and a wider range of targets. EventBridge Rules (legacy) are limited in throughput and primarily intended for event-driven patterns rather than pure scheduling.

**Q: How do you handle time zones in Lambda schedules?**
**A:** EventBridge Scheduler supports specifying a time zone (e.g., `America/New_York`). However, the legacy EventBridge Rules only support UTC.

**Q: Can a scheduled Lambda function receive custom input parameters?**
**A:** Yes. You can configure a "Constant JSON" payload in the EventBridge target configuration. This JSON will be passed as the event argument to the Lambda handler.

**Q: What happens if a scheduled Lambda execution fails?**
**A:** EventBridge Scheduler supports retry policies and Dead Letter Queues (DLQ). You can configure the maximum number of retries and the age of the event before it is sent to the DLQ.
