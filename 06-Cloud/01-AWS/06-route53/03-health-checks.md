#AWS
#Cloud

---
tags: ['aws', 'roadmap', 'route53']
---

## Summary

Amazon Route 53 health checks monitor the health and performance of your application's endpoints, such as web servers and email servers. They are the backbone of **DNS Failover**, allowing Route 53 to automatically route traffic away from unhealthy resources to healthy ones. By combining different types of health checks (Endpoint, Calculated, and CloudWatch-based), you can implement robust **Active-Active** or **Active-Passive** failover strategies to ensure high availability.

## Detailed Explanation

### 1. Health Check Types

Route 53 provides three primary ways to monitor your resources:

#### A. Endpoint Monitoring
Route 53 sends requests to a specific IP address or domain name at regular intervals.
-   **Protocols**: HTTP, HTTPS, and TCP.
-   **Intervals**: Standard (30 seconds) or Fast (10 seconds - higher cost).
-   **Failure Threshold**: Number of consecutive failed checks (1 to 10) before a resource is marked unhealthy.
-   **String Matching**: For HTTP/HTTPS, Route 53 can search for a specific string in the first 5,120 bytes of the response body.
-   **Aggregation Logic**: Route 53 has health checkers worldwide. An endpoint is considered healthy if more than **18%** of the health checkers report it as healthy.

#### B. Calculated Health Checks (Parent-Child)
Used to monitor the status of multiple health checks simultaneously.
-   **Logic**: You can combine up to 255 child health checks.
-   **Use Case**: "Consider the system healthy only if at least 3 out of 5 web servers are up."
-   **Operators**: Supports `AND`, `OR`, and `NOT` logic.

#### C. CloudWatch Alarm Integration
Monitors a CloudWatch alarm instead of a direct endpoint.
-   **Use Case**: Failover based on metrics Route 53 can't see directly, such as DynamoDB throttled events, CPU utilization, or custom application metrics.
-   **Note**: Route 53 monitors the *data stream* of the alarm, not just the state, to react faster.

### 2. Failover Configurations

#### Active-Active Failover
In an Active-Active configuration, all resources are intended to be available at all times.
-   **Routing Policy**: Use Weighted, Latency-based, Geolocation, or Multi-value Answer routing.
-   **Behavior**: Route 53 returns all healthy records. If one fails, it is simply removed from the response pool.
-   **Example**: Two web servers in different regions, both serving traffic.

#### Active-Passive Failover
In an Active-Passive configuration, you have a primary resource (or group) and a secondary/standby resource.
-   **Routing Policy**: Use the **Failover** routing policy.
-   **Behavior**: Route 53 always points to the Primary if it's healthy. It only switches to the Secondary if the Primary is unhealthy.
-   **Example**: A primary site in AWS and a static "Sorry, we're down" page in S3 as the secondary.

### 3. Implementation Example (Go)

Using the AWS SDK for Go v2 to create a basic HTTP health check:

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/route53"
	"github.com/aws/aws-sdk-go-v2/service/route53/types"
	"github.com/aws/aws-sdk-go/aws"
)

func main() {
	cfg, err := config.LoadDefaultConfig(context.TODO(), config.WithRegion("us-east-1"))
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	client := route53.NewFromConfig(cfg)

	input := &route53.CreateHealthCheckInput{
		CallerReference: aws.String("unique-string-12345"), // Idempotency token
		HealthCheckConfig: &types.HealthCheckConfig{
			Type:                     types.HealthCheckTypeHttp,
			FullyQualifiedDomainName: aws.String("myapp.example.com"),
			ResourcePath:             aws.String("/health"),
			Port:                     aws.Int32(80),
			RequestInterval:          aws.Int32(30),
			FailureThreshold:         aws.Int32(3),
		},
	}

	resp, err := client.CreateHealthCheck(context.TODO(), input)
	if err != nil {
		log.Fatalf("failed to create health check, %v", err)
	}

	fmt.Printf("Health Check Created: %s\n", *resp.HealthCheck.Id)
}
```

## Interview Questions

**Q: What is the difference between Active-Active and Active-Passive failover in Route 53?**
**A:** In Active-Active, all healthy resources serve traffic simultaneously using policies like Weighted or Latency. In Active-Passive, a Primary resource serves all traffic, and the Secondary is only used as a standby when the Primary fails its health check.

**Q: Does an HTTPS health check validate the SSL/TLS certificate?**
**A:** No. Route 53 HTTPS health checks only verify that a connection can be established and that it returns a 2xx or 3xx status code. It does not check if the certificate is expired, self-signed, or untrusted.

**Q: How does Route 53 handle a situation where some health checkers see an endpoint as healthy and others as unhealthy?**
**A:** Route 53 uses an aggregation logic where it considers an endpoint healthy if more than 18% of its global health checkers report a "healthy" status. This prevents local network issues between a specific AWS region and your server from causing a false failover.

**Q: What are Calculated Health Checks and when would you use them?**
**A:** Calculated health checks monitor the status of other "child" health checks. You use them when you need complex logic, such as failing over only if a majority of your web servers are down, or when you want to combine multiple independent metrics (e.g., Endpoint health AND CloudWatch alarm state).

**Q: Can Route 53 perform health checks on private IP addresses?**
**A:** No. Route 53 health checkers are located on the public internet. To monitor internal resources, you must either use a public IP/FQDN or create a CloudWatch alarm based on internal metrics (via a VPC endpoint or agent) and have Route 53 monitor that alarm.
