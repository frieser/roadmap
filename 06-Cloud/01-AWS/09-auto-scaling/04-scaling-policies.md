#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'auto-scaling']
---

## Summary
AWS Auto Scaling policies allow for automatic adjustment of Auto Scaling Group (ASG) capacity to handle varying loads. **Dynamic Scaling** responds to real-time metrics (like CPU or Request Count) via CloudWatch alarms, while **Predictive Scaling** uses machine learning to forecast future traffic and schedule adjustments in advance. These policies ensure high availability and cost optimization by matching capacity to demand.

## Detailed Explanation

Scaling policies define *how* an Auto Scaling Group should scale in or out. They are typically triggered by CloudWatch alarms or scheduled events.

### 1. Dynamic Scaling Policies
Dynamic scaling policies track a metric and adjust capacity as the metric changes.

#### **Target Tracking Scaling**
*   **Concept**: You select a scaling metric and a target value (e.g., "Maintain average CPU utilization at 50%").
*   **Behavior**: AWS automatically creates and manages the CloudWatch alarms that trigger the scaling policy. It calculates the scaling adjustment based on the metric and the target value.
*   **Use Case**: Most common and recommended for most applications. Works best when the metric increases/decreases proportionally to the load.
*   **Common Metrics**: `ASGAverageCPUUtilization`, `ASGAverageNetworkIn`, `ASGAverageNetworkOut`, `ALBRequestCountPerTarget`.

#### **Step Scaling**
*   **Concept**: You define a set of scaling adjustments, called **steps**, that vary based on the size of the alarm breach.
*   **Behavior**: You specify multiple thresholds. For example:
    *   If CPU is between 50% and 70%: Add 1 instance.
    *   If CPU is between 70% and 85%: Add 2 instances.
    *   If CPU is above 85%: Add 4 instances.
*   **Use Case**: When you need a more aggressive or granular response to large spikes in traffic.

#### **Simple Scaling**
*   **Concept**: A single scaling adjustment is made when an alarm is breached.
*   **Behavior**: "If CPU > 70%, add 1 instance." After a scaling action, it waits for a **cooldown period** before reacting to further alarms.
*   **Use Case**: Legacy scaling. Generally replaced by Step Scaling because Step Scaling can respond to additional alarms while a scaling activity is in progress.

### 2. Predictive Scaling
*   **Concept**: Uses Machine Learning to analyze historical load data (at least 24 hours, ideally 14 days) to forecast future traffic.
*   **Behavior**: It schedules scaling actions *before* the predicted load arrives. It can also "buffer" capacity to handle unexpected spikes.
*   **Use Case**: Highly predictable, cyclical traffic patterns (e.g., daily peaks at 9 AM). Often used in conjunction with Dynamic Scaling.

### 3. Go Application (AWS SDK v2)
Using the AWS SDK for Go to create a Target Tracking Scaling Policy:

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/aws/aws-sdk-go-v2/aws"
	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/autoscaling"
	"github.com/aws/aws-sdk-go-v2/service/autoscaling/types"
)

func main() {
	cfg, err := config.LoadDefaultConfig(context.TODO(), config.WithRegion("us-east-1"))
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	svc := autoscaling.NewFromConfig(cfg)

	// Define Target Tracking Configuration
	targetTrackingCfg := &types.TargetTrackingConfiguration{
		TargetValue: aws.Float64(50.0), // Target 50% CPU
		PredefinedMetricSpecification: &types.PredefinedMetricSpecification{
			PredefinedMetricType: types.MetricTypeASGAverageCPUUtilization,
		},
	}

	_, err = svc.PutScalingPolicy(context.TODO(), &autoscaling.PutScalingPolicyInput{
		AutoScalingGroupName:           aws.String("my-asg-name"),
		PolicyName:                     aws.String("cpu-target-tracking-policy"),
		PolicyType:                     aws.String("TargetTrackingScaling"),
		TargetTrackingConfiguration: targetTrackingCfg,
	})

	if err != nil {
		log.Fatalf("failed to put scaling policy, %v", err)
	}

	fmt.Println("Successfully created Target Tracking Policy")
}
```

## Interview Questions

*   **Q: What is the primary difference between Target Tracking and Step Scaling?**
*   **A:** Target Tracking is easier to configure; you just provide a target value (e.g., 50% CPU) and AWS manages the alarms. Step Scaling gives you more control, allowing you to define specific capacity increments based on different levels of alarm breach (steps).

*   **Q: When would you use Predictive Scaling over Dynamic Scaling?**
*   **A:** Predictive Scaling is ideal for workloads with regular, predictable patterns (e.g., a news site that spikes every morning). It helps avoid the lag time associated with Dynamic Scaling by provisioning capacity *before* the load hits. Usually, they are used together for maximum reliability.

*   **Q: What is the "cooldown period" and why is it important in Simple Scaling?**
*   **A:** The cooldown period is a configurable time during which the ASG will not launch or terminate additional instances, allowing the previous scaling action's effect to be reflected in the metrics. This prevents "flapping" or over-provisioning due to delayed metric updates.

*   **Q: Can a single ASG have multiple scaling policies?**
*   **A:** Yes. An ASG can have multiple policies (e.g., one for CPU and one for Request Count). If multiple policies trigger at once, AWS Auto Scaling chooses the one that provides the largest capacity for scaling out, and the smallest capacity for scaling in, to prioritize availability.
