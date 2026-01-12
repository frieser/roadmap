#AWS
#Cloud

---
tags: ['aws', 'roadmap', 'cloudwatch']
---

## Summary
Amazon CloudWatch metrics are time-ordered sets of data points that represent variables monitoring your resources and applications. They are the fundamental unit in CloudWatch, uniquely identified by a name, a namespace, and a set of dimensions. Metrics allow you to track performance, troubleshoot issues, and set alarms based on historical data.

## Detailed Explanation

### Namespaces
A **Namespace** is a container for CloudWatch metrics. It provides isolation between different applications or services to prevent accidental data aggregation.
- **Convention**: AWS services use the prefix `AWS/` (e.g., `AWS/EC2`, `AWS/S3`).
- **Custom Namespaces**: You can define your own namespaces for custom metrics (e.g., `MyApp/Frontend`).
- **Default**: There is no default namespace; you must specify one for every data point.

### Metrics and Dimensions
A **Metric** is a time-series of data points. Its unique identity is defined by:
1.  **Namespace**
2.  **Metric Name**
3.  **Dimensions**: A set of up to 30 name/value pairs (e.g., `InstanceId=i-12345`).

**Dimensions** are critical because CloudWatch treats every unique combination of dimensions as a separate metric. For example, `Errors` with `Service=Auth` is distinct from `Errors` with `Service=Billing`.

### Resolution
Metrics can be published with two levels of resolution:
- **Standard Resolution**: Data has a granularity of **1 minute**.
- **High Resolution**: Data has a granularity of **1 second**. 
    - Useful for sub-minute activity monitoring.
    - High-resolution alarms can be set with 10s or 30s periods.
    - More expensive due to higher frequency of `PutMetricData` calls.

### Retention Periods
CloudWatch automatically aggregates data as it ages to balance storage efficiency and historical visibility:
| Period | Retention |
| --- | --- |
| < 60 seconds (High-res) | 3 hours |
| 60 seconds (1 minute) | 15 days |
| 300 seconds (5 minutes) | 63 days |
| 3600 seconds (1 hour) | 455 days (15 months) |

---

### Go Implementation (PutMetricData)

In Go, you use the AWS SDK (v2) to publish custom metrics. The `PutMetricData` API allows sending up to 1,000 data points per request.

```go
package main

import (
	"context"
	"fmt"
	"time"

	"github.com/aws/aws-sdk-go-v2/aws"
	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/cloudwatch"
	"github.com/aws/aws-sdk-go-v2/service/cloudwatch/types"
)

func main() {
	// Load AWS configuration
	cfg, _ := config.LoadDefaultConfig(context.TODO())
	cwClient := cloudwatch.NewFromConfig(cfg)

	// Define a custom metric data point
	metricDatum := types.MetricDatum{
		MetricName: aws.String("RequestLatency"),
		Unit:       types.StandardUnitMilliseconds,
		Value:      aws.Float64(150.5),
		Timestamp:  aws.Time(time.Now()),
		Dimensions: []types.Dimension{
			{
				Name:  aws.String("ServiceName"),
				Value: aws.String("OrderService"),
			},
		},
		// StorageResolution: aws.Int32(1), // Uncomment for High-Resolution (1s)
	}

	// Publish to CloudWatch
	_, err := cwClient.PutMetricData(context.TODO(), &cloudwatch.PutMetricDataInput{
		Namespace:  aws.String("MyApp/Performance"),
		MetricData: []types.MetricDatum{metricDatum},
	})

	if err != nil {
		fmt.Printf("Error publishing metric: %v\n", err)
		return
	}
	fmt.Println("Successfully published metric data.")
}
```

## Interview Questions

**Q: What happens if you publish a metric with a new dimension combination?**
**A:** CloudWatch creates a completely new, unique metric. Dimensions are part of the metric's identity. If you retrieve statistics without specifying the exact dimensions used during publication, CloudWatch will not return the data.

**Q: Can you delete a CloudWatch metric?**
**A:** No, metrics cannot be manually deleted. They automatically expire after 15 months if no new data points are published to them.

**Q: What is the difference between Standard and High Resolution metrics?**
**A:** Standard resolution metrics have a 1-minute granularity, while High Resolution metrics allow for 1-second granularity. High resolution is ideal for monitoring rapid spikes or transient issues but incurs higher costs for data points and alarms.

**Q: How does CloudWatch handle metric retention over time?**
**A:** CloudWatch uses a rolling retention schedule. As data ages, it is aggregated into coarser periods (e.g., 1-minute data is kept for 15 days, then aggregated into 5-minute blocks for 63 days, and finally into 1-hour blocks for 15 months).

**Q: Why might an alarm on a custom metric show "Insufficient Data"?**
**A:** This typically happens if the metric hasn't been published recently, if the time stamp of the data points is too far in the past/future, or if the alarm period is shorter than the frequency at which the metric is being published.
