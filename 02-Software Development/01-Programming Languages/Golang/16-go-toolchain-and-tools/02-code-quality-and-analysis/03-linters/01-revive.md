# Revive

## Summary
Revive is a fast, configurable, extensible, and flexible linter for Go. It was created as a modern replacement for the (now deprecated) `golint`. While `golint` was strict and hardcoded to Google's style, Revive allows you to enable/disable rules, configure their severity (warning vs error), and runs up to 6x faster.

## Detailed Explanation

### Key Features
1.  **Configurable**: Uses a `revive.toml` configuration file. You can ignore specific rules for specific files (e.g., ignore comment rules in test files).
2.  **Performance**: Runs faster than `golint` because it runs all rules in a single pass over the AST.
3.  **Extensible**: You can write custom rules easily.

### Comparison to `golint`
`golint` checks for style guide compliance (Effective Go). It complains about missing comments on exported functions, variable names containing underscores (snake_case), etc. Revive implements all these checks but fixes the major complaint about `golint`: the inability to silence false positives or irrelevant rules.

### Configuration Example (`revive.toml`)
```toml
ignoreGeneratedHeader = false
severity = "warning"
confidence = 0.8
errorCode = 0
warningCode = 0

[rule.exported]
  arguments = ["disableStutteringCheck"]

[rule.var-naming]
  severity = "error"
```

## Interview Questions

**Q: Why was `golint` deprecated?**
**A:** The Go team deprecated `golint` because it was often confused with a correctness checker (like `go vet`), but it was purely a style checker enforcing Google's specific preferences. It was frozen in feature set and had no configuration options, leading to frustration. The community moved to tools like Revive and Staticcheck which offer more flexibility and power.

**Q: How does Revive handle "stuttering" in package names?**
**A:** Go style advises against names like `user.UserInfo` (stutter). Revive has a rule to detect this, suggesting you rename the type to `user.Info` so the usage reads cleanly as `user.Info` instead of `user.UserInfo`.

**Q: Can Revive replace `go vet`?**
**A:** No. Revive focuses on **style and coding conventions** (linting). `go vet` focuses on **correctness and bugs** (static analysis). They complement each other.
