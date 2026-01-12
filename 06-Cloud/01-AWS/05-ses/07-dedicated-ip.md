#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'ses']
---

## Summary
Amazon SES Dedicated IP addresses allow senders to isolate their email reputation from other AWS customers. Unlike shared IPs, dedicated IPs are reserved for your exclusive use, providing complete control over sender reputation. SES offers **Standard** dedicated IPs (manually managed) and **Managed** dedicated IPs (automatically warmed up and scaled by AWS).

## Detailed Explanation

### 1. Standard Dedicated IPs
Standard dedicated IPs are leased for a monthly fee and give you full control over the IP addresses used to send your email.
- **Reputation Management**: You are entirely responsible for the reputation. If you send spam, your IP gets blacklisted, and only your account is affected.
- **Manual Warmup**: You must manually "warm up" these IPs by gradually increasing sending volume over several weeks to build trust with ISPs.
- **Static Nature**: These IPs are static and do not change, which is useful if your recipients require IP allowlisting.
- **Configuration**: You can group these IPs into **Dedicated IP Pools** to further isolate reputation (e.g., separating transactional mail from marketing campaigns).

### 2. Managed Dedicated IPs
Managed dedicated IPs automate much of the heavy lifting associated with dedicated IP management.
- **Automated Warmup**: SES uses an adaptive strategy to warm up IPs for each ISP individually, automatically shifting traffic between the shared pool and your dedicated pool.
- **Auto-Scaling**: SES automatically adds more IPs to your pool if your volume increases and removes them if volume drops, ensuring optimal utilization.
- **Reputation Optimization**: AWS monitors ISP-specific performance and adjusts sending patterns to improve deliverability.
- **Cost Structure**: Typically involves a flat monthly fee for the service plus a small per-message charge, rather than a per-IP fee.

### 3. IP Warmup Process
The goal of warmup is to establish a positive sending history with ISPs.
- **The Problem**: ISPs are suspicious of new IPs sending large volumes. This often results in emails being throttled or sent to the spam folder.
- **Standard Warmup Path**:
    1. Start by sending a very small volume (e.g., 50-100 emails/day).
    2. Double the volume every 2-3 days if bounce and complaint rates remain low.
    3. Continue until you reach your target daily volume.
- **Managed Warmup**: SES handles this automatically by using its shared IP pool to "fill the gaps" while the dedicated IPs are still gaining reputation.

### 4. Implementation in Go (AWS SDK v2)
When using Dedicated IP Pools, you associate them with a **Configuration Set**. In your Go code, you specify this Configuration Set when sending an email.

```go
package main

import (
	"context"
	"log"

	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/sesv2"
	"github.com/aws/aws-sdk-go-v2/service/sesv2/types"
)

func main() {
	cfg, err := config.LoadDefaultConfig(context.TODO())
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	client := sesv2.NewFromConfig(cfg)

	input := &sesv2.SendEmailInput{
		FromEmailAddress: "sender@example.com",
		Destination: &types.Destination{
			ToAddresses: []string{"recipient@example.com"},
		},
		Content: &types.EmailContent{
			Simple: &types.Message{
				Subject: &types.Content{Data: "Dedicated IP Test"},
				Body: &types.Body{
					Text: &types.Content{Data: "This email was sent via a specific Configuration Set."},
				},
			},
		},
		// IMPORTANT: Specify the Configuration Set associated with your Dedicated IP Pool
		ConfigurationSetName: "MarketingPoolConfigSet",
	}

	_, err = client.SendEmail(context.TODO(), input)
	if err != nil {
		log.Fatalf("failed to send email, %v", err)
	}

	log.Println("Email sent successfully!")
}
```

### Comparison Table

| Feature | Shared IPs | Dedicated (Standard) | Dedicated (Managed) |
| :--- | :--- | :--- | :--- |
| **Ease of Use** | High (Default) | Medium (Manual Setup) | High (Automated) |
| **Reputation Control** | Shared with others | Full Control | Full Control |
| **Warmup** | None required | Manual | Automated |
| **Scaling** | Automatic | Manual | Automatic |
| **IP Addresses** | Dynamic/Unknown | Static/Known | Dynamic/Known |
| **Cost** | Included | Per-IP Monthly Fee | Monthly Fee + Usage |

## Interview Questions

**Q: What is the primary benefit of using Dedicated IPs in Amazon SES?**
**A:** The primary benefit is **reputation isolation**. It ensures that your email deliverability is not negatively impacted by the sending habits of other SES customers, giving you full control over your sender reputation.

**Q: Describe the manual warmup process for a Standard Dedicated IP.**
**A:** Warmup involves gradually increasing the volume of email sent from the new IP over time (usually 2-4 weeks). You start with a small amount of "safe" traffic (e.g., transactional mail to engaged users) and slowly increase the daily volume to prove to ISPs that the IP is not a source of spam.

**Q: When would a company prefer Managed Dedicated IPs over Standard ones?**
**A:** When they have high sending volumes but less predictable patterns, or when they want to minimize the operational overhead of manually warming up and scaling IP addresses. It’s ideal for "set-and-forget" reputation management.

**Q: Why might you use Dedicated IP Pools?**
**A:** To isolate reputations within your own account. For example, you can use one pool for high-criticality transactional emails (like password resets) and another for marketing newsletters, ensuring that a spike in marketing complaints doesn't delay your transactional mail.

**Q: Can you use Dedicated IPs if you send very low volumes of email?**
**A:** It is generally not recommended. ISPs require a consistent, significant volume of mail to establish a reputation for an IP. Low volume senders are better off using the SES Shared Pool, which has a high collective volume and reputation.
