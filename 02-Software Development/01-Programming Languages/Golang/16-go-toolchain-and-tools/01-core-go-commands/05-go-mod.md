# Go Mod

## Summary
The `go mod` command manages Go Modules, the dependency management system introduced in Go 1.11. It handles the `go.mod` file (manifest) and `go.sum` file (checksums), allowing for reproducible builds, versioning, and dependency resolution outside of GOPATH.

## Detailed Explanation

### Core Commands

1.  **`go mod init <module-path>`**: Initializes a new module in the current directory, creating `go.mod`.
    ```bash
    go mod init github.com/user/project
    ```

2.  **`go mod tidy`**: The most important command. It scans your source code and:
    *   Adds missing dependencies to `go.mod`.
    *   Removes unused dependencies from `go.mod`.
    *   Updates `go.sum`.

3.  **`go mod vendor`**: Copies all dependencies into a local `vendor/` directory. This allows building without an internet connection.

4.  **`go mod graph`**: Prints the module requirement graph (who depends on what).

5.  **`go mod verify`**: Checks that the dependencies in the local cache match the checksums in `go.sum`, ensuring no one has tampered with the code.

### The `go.sum` file
This file contains the expected cryptographic hashes (checksums) of the content of specific module versions. It ensures that `v1.0.0` of a library downloaded today is bit-for-bit identical to `v1.0.0` downloaded yesterday.

## Interview Questions

**Q: When should you commit `go.sum` to git?**
**A:** Always. `go.sum` is critical for security and reproducibility. It ensures that all developers and CI/CD systems use exactly the same code for dependencies. Without it, a compromised proxy or upstream repo could inject malicious code into a dependency version.

**Q: What does `go mod tidy` do that `go get` doesn't?**
**A:** `go get` adds or updates a specific dependency. `go mod tidy` cleans up the entire manifest. Specifically, if you delete an import from your code, `go get` won't automatically remove it from `go.mod`. `go mod tidy` detects that the package is no longer used and removes the requirement.

**Q: What is the purpose of the `indirect` comment in `go.mod`?**
**A:** It indicates a **transitive dependency**. This means your project doesn't import this module directly, but one of your dependencies does. It can also appear if you import a package but haven't yet updated the `go.mod` file properly (though `tidy` fixes this).
