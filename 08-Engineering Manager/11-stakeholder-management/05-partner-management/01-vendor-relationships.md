---
---

## Summary
Vendor Management is a key responsibility for Engineering Managers. It involves selecting third-party tools (SaaS, Cloud, Libraries), negotiating contracts, and ensuring the vendor delivers value (SLA compliance). The core decision framework is often "Build vs. Buy".

## Detailed Explanation

### Build vs Buy
*   **Buy**: Non-core competency (e.g., Email delivery, Payroll, CRM). Speed to market is high. Maintenance cost is externalized (fees).
*   **Build**: Core differentiator (e.g., Google's Search Algo). High customization needed. No vendor fits.

### Evaluation Process
1.  **Requirements**: Must-haves vs Nice-to-haves.
2.  **Security/Compliance**: SOC2, GDPR, Data residency.
3.  **Cost**: TCO (Total Cost of Ownership) = License fee + Integration effort + Training.
4.  **Lock-in**: How hard is it to switch away later?

### Relationship Management
*   **QBR (Quarterly Business Review)**: Meeting with the vendor to review usage and roadmap.
*   **Support**: Escalation paths when things break.

## Go-Specific Context/Examples

Scenario: You need a Feature Flag system.
*   **Buy**: LaunchDarkly.
    *   *Pros*: Dashboard UI, audit logs, ready-to-use Go SDK.
    *   *Cons*: Expensive at scale.
*   **Build**: A simple Go service + Redis.
    *   *Pros*: Cheap, tailored to your exact needs.
    *   *Cons*: You must build the UI, you are on call if it breaks, you must maintain the Go SDK.

## Interview Questions

**Q: When should you fire a vendor?**
**A:** When they consistently miss SLAs, when their price increases outpace the value provided, or when your internal capability has matured enough that "Building" becomes cheaper and more strategic than "Buying".

**Q: How do you mitigate Vendor Lock-in?**
**A:** Use **Abstraction Layers**. Don't import the vendor's library directly in every file. Create an interface (e.g., `EmailProvider`) and wrap the vendor (SendGrid). If you switch to Mailgun, you only change the wrapper implementation.

**Q: What is "Shadow IT"?**
**A:** When teams buy/use software without IT/Security approval. It poses security risks (data leaks) and financial waste (duplicate tools). An EM must balance autonomy with governance.
