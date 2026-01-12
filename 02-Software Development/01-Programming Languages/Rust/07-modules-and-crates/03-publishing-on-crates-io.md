#Rust
---
---

## Summary
Publishing code to **crates.io** (the Rust package registry) allows the community to easily use your libraries. It requires a valid `Cargo.toml` with specific metadata, an API token, and a verified email address. Once published, versions are immutable (code cannot be changed), but they can be **yanked** to prevent new projects from depending on them.

## Detailed Explanation

### 1. Prerequisites
- **Account**: Create an account on crates.io via GitHub.
- **Token**: Generate an API token on crates.io and run `cargo login <token>` locally.
- **Verification**: Email must be verified.

### 2. Metadata
Before publishing, `Cargo.toml` needs:
- `description`: Short summary.
- `license`: e.g., "MIT" or "Apache-2.0".
- `documentation` / `homepage`: Links to resources.
- `exclude` / `include`: Control which files are uploaded.

### 3. Publishing Workflow
1.  **Dry Run**: `cargo publish --dry-run` checks for errors and verifies the package size.
2.  **Publish**: `cargo publish` uploads the `.crate` file.
3.  **Documentation**: `docs.rs` automatically builds and hosts documentation for every crate on crates.io.

### 4. Yanking
You cannot delete code. If you publish a critical bug, you **yank** the version.
- `cargo yank --vers 1.0.1`
- Yanked versions can still be used by projects that already locked them in `Cargo.lock`, but new projects cannot select them.

## Rust Application

### Preparing for Publish

```toml
[package]
name = "super_math"
version = "0.1.0"
authors = ["Jane Doe <jane@example.com>"]
edition = "2021"
description = "A library for doing super math calculations."
license = "MIT"
repository = "https://github.com/janedoe/super_math"
readme = "README.md"
keywords = ["math", "calculus"]
categories = ["science"]

[dependencies]
```

### CLI Commands

```bash
# 1. Login (only once)
$ cargo login abcdef123456...

# 2. Check content
$ cargo package --list

# 3. Dry run
$ cargo publish --dry-run

# 4. Publish
$ cargo publish

# 5. Yank a bad version
$ cargo yank --vers 0.1.0
```

## Interview Questions

### Q: Can you delete a crate version from crates.io after publishing?
**A:** No. To prevent breaking builds for users who depend on your crate, code is permanent. However, you can **yank** a version. This prevents *new* dependencies on that version while allowing existing lockfiles to continue working.

### Q: How does documentation get onto docs.rs?
**A:** It is automatic. When you publish a crate to crates.io, the docs.rs build servers detect the new release, build the documentation using `cargo doc`, and host it at `docs.rs/crate-name`.

### Q: What is the `cargo package` command used for?
**A:** It assembles the local package into a distributable `.crate` file (a compressed tarball) exactly as it would be uploaded to crates.io. It's useful for verifying exactly what files are included (via `--list`) before actually publishing.
