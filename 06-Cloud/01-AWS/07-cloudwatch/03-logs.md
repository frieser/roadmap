#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'cloudwatch']
---

## Summary
Amazon CloudWatch Logs is a fully managed service that centralizes logs from all your systems, applications, and AWS services. It provides a highly scalable infrastructure to monitor, store, and access log files, enabling you to search for specific error codes or patterns, filter fields, and archive data securely. With integrated tools like Logs Insights and Metric Filters, it transforms raw log data into actionable insights and operational metrics.

## Detailed Explanation

### Log Hierarchy
CloudWatch Logs organizes data into three distinct layers:
1.  **Log Events**: The basic unit, consisting of a timestamp and the raw message data.
2.  **Log Streams**: A chronological sequence of log events from a single source (e.g., a specific EC2 instance or Lambda function container).
3.  **Log Groups**: A collection of log streams that share common settings such as retention policies, tags, and IAM access controls.

### Metric Filters
Metric Filters allow you to turn log data into operational metrics that you can graph or use to trigger CloudWatch Alarms.
-   **Pattern Matching**: You can define patterns to find specific terms (e.g., `"Error"`, `404`) or JSON fields (e.g., `{ $.status = 500 }`).
-   **Extraction**: Beyond simple counting, you can extract numerical values from log entries (like latency or memory usage) to create distribution metrics.
-   **No Code Changes**: This allows you to monitor legacy applications that only output to stdout/stderr without modifying their source code.

### CloudWatch Logs Insights
Logs Insights is an interactive query service that uses a purpose-built query language to search and analyze log data.
-   **Query Syntax**: Uses pipe-delimited commands similar to Unix shells or SQL.
    -   `fields`: Select specific fields.
    -   `filter`: Apply conditions (regex supported).
    -   `stats`: Calculate aggregates (e.g., `count()`, `avg()`).
    -   `sort` and `limit`: Manage output presentation.
-   **Visualization**: Supports generating bar, line, and pie charts directly from query results.

### Exporting and Streaming
Logs can be moved to other AWS services for different use cases:

#### 1. Exporting to Amazon S3
*   **Purpose**: Long-term archival and cost optimization.
*   **Mechanism**: A batch process (Export Task).
*   **Limitation**: Data is not available in real-time; it can take up to 12 hours for data to become available in the S3 bucket after the task starts.

#### 2. Subscription Filters (Real-Time)
*   **Lambda**: Trigger a function immediately for every log event (or a filtered subset). Ideal for custom alerting or real-time transformation.
*   **Kinesis Data Streams**: Stream logs to a central data lake or processing pipeline.
*   **Kinesis Data Firehose**: Send logs directly to S3 (near real-time), OpenSearch, or 3rd party providers like Splunk.

### Go Implementation Example
Sending a custom log event to CloudWatch Logs using the AWS SDK for Go (v2):

```go
package main

import (
	"context"
	"fmt"
	"log"
	"time"

	"github.com/aws/aws-sdk-go-v2/aws"
	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/cloudwatchlogs"
	"github.com/aws/aws-sdk-go-v2/service/cloudwatchlogs/types"
)

func main() {
	ctx := context.TODO()
	// Load AWS configuration
	cfg, err := config.LoadDefaultConfig(ctx)
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	client := cloudwatchlogs.NewFromConfig(cfg)

	logGroupName := "AppLogs"
	logStreamName := "Instance-01"

	// Create a log event
	event := types.InputLogEvent{
		Message:   aws.String("User login successful - userID: 12345"),
		Timestamp: aws.Int64(time.Now().UnixNano() / int64(time.Millisecond)),
	}

	// PutLogEvents sends the log data to the stream
	_, err = client.PutLogEvents(ctx, &cloudwatchlogs.PutLogEventsInput{
		LogEvents:     []types.InputLogEvent{event},
		LogGroupName:  &logGroupName,
		LogStreamName: &logStreamName,
		// SequenceToken is no longer required as of Feb 2023
	})

	if err != nil {
		fmt.Printf("Error sending logs: %v\n", err)
		return
	}

	fmt.Println("Successfully sent log event to CloudWatch")
}
```

## Interview Questions

**Q: What is the primary difference between Log Groups and Log Streams?**
**A:** A Log Stream represents a single source of logs (like one server), while a Log Group is a logical container for multiple streams that share the same configuration, such as retention period (how long logs are kept) and IAM permissions.

**Q: How can you process CloudWatch Logs in real-time?**
**A:** Use **Subscription Filters**. These allow you to pipe log events as they are ingested into other services like AWS Lambda (for custom processing), Kinesis Data Streams (for streaming), or Kinesis Data Firehose (for delivery to S3 or OpenSearch).

**Q: What is the purpose of a Metric Filter?**
**A:** A Metric Filter extracts data from logs (using pattern matching or field extraction) and pushes it as a custom metric to CloudWatch. This allows you to create alarms and dashboards based on log content (e.g., counting "404" errors) without modifying application code.

**Q: Can CloudWatch Logs Export to S3 be used for real-time monitoring?**
**A:** No. Exporting to S3 is a batch operation intended for long-term archival. It can take several hours for the data to be fully exported. For real-time monitoring, Subscription Filters or Kinesis integration should be used.

**Q: Does PutLogEvents still require a SequenceToken?**
**A:** No. As of February 2023, AWS updated the CloudWatch Logs API so that `sequenceToken` is no longer required for `PutLogEvents`. This significantly simplifies log ingestion logic and reduces "InvalidSequenceTokenException" errors.
