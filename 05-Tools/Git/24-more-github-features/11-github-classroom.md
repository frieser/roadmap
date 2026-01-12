# GitHub Classroom

## Summary
A tool for teachers to distribute starter code and collect assignments automatically via GitHub.

## Detailed Explanation

### Workflow
1.  **Assignment**: Teacher creates a "Template Repository" (starter code).
2.  **Link**: Teacher shares an invitation link.
3.  **Accept**: Student clicks link -> GitHub creates a private repo `assignment-studentName`.
4.  **Submit**: Student pushes code.
5.  **Grade**: Teacher can run autograding tests via GitHub Actions.

### Go-specific Context
Teachers often set up a `Makefile` or `go test` workflow in the template. When students push, they get immediate feedback if their solution passes the test cases.

## Interview Questions
**Q: Who owns the student repositories?**
**A:** The Organization (the class), which allows the teacher to see all code, unlike if students created personal repos.
