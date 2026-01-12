# Stability and the Standard Library

## Summary
The road to Zig 1.0 is characterized by a "stabilization race," aiming to finalize the standard library and language features to offer long-term stability. Key milestones include Semantic Versioning compliance, the "freezing" of the standard library API, and the completion of major refactors (like "Writergate").

## Detailed Explanation

### The Stabilization Process
*   **Standard Library Refactor**: The `std` library is undergoing a massive cleanup to remove "hidden" allocations and global state, ensuring that all resource usage is explicit.
*   **Writergate (2025)**: A pivotal moment where the `std.io` API was completely rewritten to support the new `Io` interface model. This broke almost every Zig project but was deemed necessary for the 1.0 vision.
*   **Breaking Change Policy**: Until 1.0, breaking changes are prioritized over stability if they improve the language's "power-to-weight" ratio.

### 1.0 Guarantees
*   **SemVer Compliance**: Post-1.0, Zig will follow strict Semantic Versioning.
*   **Stability of the "Big Three"**: The Build System, the Standard Library, and the Language Syntax will be "frozen" in terms of core architecture.
*   **Transitioning**: Tools like `zig fix` are being developed to help developers migrate through the final pre-1.0 breaking changes.

### Current Status (2026)
The `std` library is approximately 85-90% stabilized. The focus has shifted from *adding* features to *polishing* and *optimizing* existing ones.

## Interview Questions

**Q: What was "Writergate" and why was it significant?**
**A:** It was a major breaking refactor of `std.io` in 2025 to introduce the generic `Io` interface model suitable for colorless async. It signaled the transition to the final async architecture required for 1.0, despite causing significant short-term churn in the ecosystem.

**Q: What can developers expect regarding breaking changes after Zig 1.0?**
**A:** Zig has committed to strict Semantic Versioning (SemVer) after 1.0. This means no breaking changes will occur in minor updates (e.g., 1.1, 1.2), ensuring that code written for 1.0 continues to work for years.

**Q: Why does Zig allow breaking changes right now?**
**A:** Zig is pre-1.0 software. The team prioritizes getting the design *right* over maintaining backward compatibility for a suboptimal design. They believe that fixing fundamental flaws now prevents decades of technical debt later.
