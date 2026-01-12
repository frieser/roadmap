#AWS #Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'route53']
---

## Summary
Amazon Route 53 routing policies determine how the service responds to DNS queries. By selecting the appropriate policy, you can implement highly available, low-latency, and geographically distributed architectures. These policies range from simple single-resource mapping to complex traffic shifting using weights or geographic proximity.

## Detailed Explanation

### 1. Simple Routing Policy
The most basic policy, used to route traffic to a single resource (e.g., one web server IP). If multiple values are specified in one record, Route 53 returns all of them to the client in a random order.
*   **Use Case**: Single-server setups or basic DNS needs.
*   **Health Checks**: Not supported for record selection (though the resource itself can be monitored).

### 2. Weighted Routing Policy
Allows you to assign weights (0-255) to multiple resources for the same domain name. Route 53 calculates the proportion of traffic to send to each resource based on these weights.
*   **Use Case**: A/B testing, blue/green deployments, and gradual migration between versions.
*   **Calculation**: `(Resource Weight / Total Weight) * 100`.

### 3. Latency Routing Policy
Routes traffic to the AWS Region that provides the lowest network latency for the user. It is based on latency measurements performed by AWS over time.
*   **Use Case**: Global applications where performance is critical.
*   **Note**: Lowest latency does not always mean the geographically closest region.

### 4. Failover Routing Policy
Used for **Active-Passive** configurations. Route 53 monitors the health of the primary resource and automatically fails over to the secondary resource if the primary becomes unhealthy.
*   **Use Case**: Disaster Recovery (DR) and high availability.
*   **Requirement**: Requires Route 53 Health Checks.

### 5. Geolocation Routing Policy
Routes traffic based on the user's physical location (continent, country, or even US state).
*   **Use Case**: Content localization, licensing restrictions, or routing to specific regional endpoints.
*   **Default Record**: Should be created to handle queries from locations not explicitly defined.

### 6. Geoproximity Routing Policy
Routes traffic based on the geographic location of your resources and users. You can optionally use **Bias** to expand or shrink the size of the geographic region from which traffic is routed to a specific resource.
*   **Use Case**: Managing traffic flow based on resource proximity while manually "pushing" traffic from one region to another.
*   **Requirement**: Requires Route 53 Traffic Flow.

### 7. Multivalue Answer Routing Policy
Similar to simple routing but allows you to return up to 8 healthy records in response to a DNS query.
*   **Use Case**: Basic load balancing and improved availability without the complexity of ELB.
*   **Health Checks**: Unlike simple routing, it *does* use health checks and only returns healthy records.

---

### Alias Records vs. CNAME
| Feature | CNAME | Alias |
| :--- | :--- | :--- |
| **Zone Apex** | No (cannot use for `example.com`) | **Yes** (can use for `example.com`) |
| **Cost** | Charged per query | **Free** for AWS resources |
| **Automatic Updates** | No (points to a name) | **Yes** (automatically tracks IP changes) |
| **Native AWS Integration** | No | **Yes** (ELB, S3, CloudFront, etc.) |

---

## Go Application (AWS SDK v2)

Using the AWS SDK for Go to create a weighted routing record:

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/route53"
	"github.com/aws/aws-sdk-go-v2/service/route53/types"
	"github.com/aws/aws-sdk-go-v2/aws"
)

func main() {
	cfg, err := config.LoadDefaultConfig(context.TODO(), config.WithRegion("us-east-1"))
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	client := route53.NewFromConfig(cfg)

	input := &route53.ChangeResourceRecordSetsInput{
		HostedZoneId: aws.String("Z1D633PJN98FT9"), // Replace with your Zone ID
		ChangeBatch: &types.ChangeBatch{
			Changes: []types.Change{
				{
					Action: types.ChangeActionUpsert,
					ResourceRecordSet: &types.ResourceRecordSet{
						Name: aws.String("app.example.com"),
						Type: types.RRTypeA,
						TTL:  aws.Int64(60),
						ResourceRecords: []types.ResourceRecord{
							{Value: aws.String("192.0.2.1")},
						},
						SetIdentifier: aws.String("PrimaryServer"),
						Weight:        aws.Int64(70), // 70% of traffic
					},
				},
			},
		},
	}

	_, err = client.ChangeResourceRecordSets(context.TODO(), input)
	if err != nil {
		fmt.Printf("failed to update record, %v\n", err)
		return
	}

	fmt.Println("Successfully created weighted record!")
}
```

## Interview Questions

**Q: Can you use a CNAME for the root domain (Zone Apex)?**
**A:** No. According to DNS standards (RFC 1034), a CNAME cannot coexist with other records (like SOA or NS) which are required at the apex. Use a Route 53 **Alias record** instead.

**Q: What is the difference between Geolocation and Geoproximity routing?**
**A:** Geolocation routes based on the *user's location* (e.g., "Send all users in France to IP X"). Geoproximity routes based on the *distance* between the user and the resource, and allows using "Bias" to shift traffic boundaries.

**Q: How does Multivalue Answer routing differ from a simple Load Balancer?**
**A:** Multivalue Answer is DNS-based; it returns multiple healthy IPs and the client chooses one. A Load Balancer (ELB) provides a single DNS name/IP and performs sophisticated traffic distribution at the protocol level (Layer 4/7).

**Q: When would you use Latency Routing over Geolocation Routing?**
**A:** Use **Latency** when the primary goal is performance (fastest response time). Use **Geolocation** when legal compliance (data residency), content localization, or licensing requires users to stay within specific borders regardless of speed.
