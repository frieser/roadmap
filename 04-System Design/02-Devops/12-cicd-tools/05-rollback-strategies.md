---
---

# Rollback Strategies

Deployment is not just about getting code to production; it's about doing so safely and being able to recover if things go wrong. Rollback strategies define how to revert to a stable state when a deployment fails.

## Summary

The three most common strategies are **Blue/Green**, **Canary**, and **Rolling Update**.
*   **Rolling**: Incremental update. Low resource cost. Slow rollback.
*   **Blue/Green**: Instant switch between two full environments. High resource cost. Instant rollback.
*   **Canary**: Gradual traffic shift to new version. Lowest risk. Complex setup.

## Detailed Explanation

### 1. Rolling Update (Kubernetes Default)
*   **Process**: Replace instances of V1 with V2 one by one (or in batches).
*   **Rollback**: Reverse the process.
*   **Pros**: No extra infrastructure cost.
*   **Cons**: Verification is hard (mixed traffic). Rollback takes time.

### 2. Blue/Green Deployment
*   **Process**: Deploy V2 (Green) alongside V1 (Blue). Once V2 is healthy, switch the Load Balancer to V2.
*   **Rollback**: Switch Load Balancer back to V1 immediately.
*   **Pros**: Instant rollback. No mixed traffic.
*   **Cons**: Requires double the infrastructure capacity (Cost).

### 3. Canary Deployment
*   **Process**: Deploy V2 to a small subset (e.g., 5%). Route 5% of traffic there. Monitor metrics. If healthy, increase traffic gradually (10%, 50%, 100%).
*   **Rollback**: Route 100% traffic back to V1 instantly.
*   **Pros**: Limits blast radius of bugs. Real-user testing.
*   **Cons**: Requires advanced traffic splitting (Service Mesh/ALB).

---

## Go Implementation Example

Simulating a **Blue/Green Switch** using the AWS SDK for Go. This code updates an Application Load Balancer (ALB) Listener to point to a different Target Group.

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/elasticloadbalancingv2"
	"github.com/aws/aws-sdk-go-v2/service/elasticloadbalancingv2/types"
)

func SwitchTraffic(listenerArn, targetGroupArn string) error {
	cfg, err := config.LoadDefaultConfig(context.TODO())
	if err != nil {
		return err
	}

	svc := elasticloadbalancingv2.NewFromConfig(cfg)

	// Modify the listener to forward traffic to the new Target Group
	input := &elasticloadbalancingv2.ModifyListenerInput{
		ListenerArn: &listenerArn,
		DefaultActions: []types.Action{
			{
				Type:           types.ActionTypeEnumForward,
				TargetGroupArn: &targetGroupArn,
			},
		},
	}

	_, err = svc.ModifyListener(context.TODO(), input)
	return err
}

func main() {
	// Example ARNs
	listener := "arn:aws:elasticloadbalancing:us-east-1:123:listener/app/my-lb/..."
	greenTargetGroup := "arn:aws:elasticloadbalancing:us-east-1:123:targetgroup/green-tg/..."

	fmt.Println("Switching traffic to Green environment...")
	if err := SwitchTraffic(listener, greenTargetGroup); err != nil {
		log.Fatal("Rollback failed:", err)
	}
	fmt.Println("Traffic switched successfully!")
}
```

## Interview Questions

**Q: Which strategy minimizes the "Blast Radius"?**
**A:** **Canary Deployment**. By exposing the new version to only a small percentage of users (e.g., 1% or internal users only), any critical bug affects only that small group, leaving the vast majority of users on the stable version.

**Q: Why is Blue/Green considered expensive?**
**A:** It requires you to provision a complete duplicate environment. If you have 100 servers in production (Blue), you must provision another 100 servers for the new version (Green) before switching. This doubles the compute cost during the deployment window.

**Q: How does Kubernetes natively handle rollbacks?**
**A:** Kubernetes Deployments maintain a `ReplicaSet` history. You can run `kubectl rollout undo deployment/my-app` to revert to the previous revision. Kubernetes will perform a rolling update "in reverse," replacing V2 pods with V1 pods.
