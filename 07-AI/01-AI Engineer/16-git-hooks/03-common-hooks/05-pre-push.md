# pre-push Hook

## Summary
The `pre-push` hook runs before a `git push` starts, allowing you to validate the state of the local repo against the remote.

## Detailed Explanation
- **When it runs:** After the destination is determined but before any objects are transferred.

### AI Engineering Use Case
- **Large File Block:** A final check to ensure no large binary files (model weights, datasets) are being pushed to the remote repo.
- **Full Test Suite:** Running the complete evaluation suite (which might take several minutes) before allowing a push to the `main` branch.
- **DVC Sync Check:** Ensure that all files tracked by DVC have been pushed to the remote storage (S3/GCS) before allowing the Git push.

## Interview Questions
1.  **Why run tests in `pre-push` instead of `pre-commit`?**
    Tests that take a long time to run are better suited for `pre-push` to avoid slowing down the frequent commit cycle.
