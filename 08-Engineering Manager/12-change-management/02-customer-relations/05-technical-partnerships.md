# Technical Partnerships

## Summary
Technical Partnerships involve collaborating with external companies (Integrations, API Partners, Channel Partners). EMs must manage the technical side of these relationships: API standards, security audits, shared roadmaps, and support channels.

## Detailed Explanation

### 1. Integration Strategy
*   **Native**: We build it inside our app. (High control, High maintenance).
*   **iPaaS**: Use Zapier/MuleSoft. (Low code, lower control).
*   **Partner-Built**: They build to our API. (Zero effort, must support the API).

### 2. Sandbox and Developer Experience (DX)
*   To attract partners, your API must be easy to use.
*   **Artifacts**: Public Docs (Swagger/OpenAPI), SDKs, Sandbox Environment.
*   **Self-Serve**: Partners shouldn't need to email you to get an API key.

### 3. Certification and Security
*   Before listing a partner in your "App Store," you must audit them.
*   **OAuth Scopes**: Ensure they only ask for necessary permissions (Least Privilege).
*   **Data Privacy**: Do they store our customer data securely?

## Go Code Example: Webhook Dispatcher
This example models a system for sending events to partners via Webhooks, including retry logic (essential for partner reliability).

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

type Partner struct {
	Name       string
	WebhookURL string
}

type Event struct {
	ID      string
	Payload string
}

func SendWebhook(p Partner, e Event) bool {
	// Simulate Network Request
	success := rand.Intn(10) > 2 // 70% success rate
	if success {
		fmt.Printf("✅ Sent %s to %s\n", e.ID, p.Name)
		return true
	}
	fmt.Printf("❌ Failed to send to %s\n", p.Name)
	return false
}

func DispatchWithRetry(p Partner, e Event) {
	retries := 3
	for i := 0; i < retries; i++ {
		if SendWebhook(p, e) {
			return
		}
		// Exponential Backoff
		backoff := time.Duration(1<<i) * 100 * time.Millisecond
		fmt.Printf("   Retrying in %s...\n", backoff)
		time.Sleep(backoff)
	}
	fmt.Printf("💀 Gave up on %s after %d attempts\n", p.Name, retries)
}

func main() {
	partner := Partner{"Slack Integration", "https://slack.com/api/webhook"}
	event := Event{"evt_123", "User Signed Up"}

	rand.Seed(time.Now().UnixNano())
	DispatchWithRetry(partner, event)
}
```

## Interview Questions

### Q: "A partner's API is unstable and breaking our app. What do you do?"
**A:**
*   **Circuit Breaker**: Detect the failures and stop calling them immediately to save our UI.
*   **Degrade Gracefully**: Show "Partner Service Unavailable" instead of crashing.
*   **Contact**: Reach out to their technical contact (this is why relationship management matters).
*   **Deprecate**: If it continues, remove the integration.

### Q: "How do you handle API versioning for partners?"
**A:**
*   **Never break changes**: Once an API is public, it is forever.
*   **Versioning**: `/v1/`, `/v2/`.
*   **Sunset Policy**: "We support old versions for 12 months." Communicate aggressively before turning off `/v1/`.

### Q: "What makes a good Developer Portal?"
**A:**
*   **Searchable Docs**: Stripe is the gold standard.
*   **Copy-Paste Code**: "Curl" examples.
*   **Try it now**: Interactive console.
