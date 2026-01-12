## Summary
GitHub Classroom is a teacher-facing tool that automates the distribution and collection of programming assignments. It uses Git and GitHub as the foundation for the educational workflow.

## Detailed Explanation

### Workflow
1. **Create an Assignment**: The teacher creates a "template repository" with starter code.
2. **Distribute**: Students receive a unique link. Clicking it creates a private fork of the template for the student.
3. **Work and Submit**: Students push their code to their private repository.
4. **Autograding**: Teachers can set up automatic tests (using GitHub Actions) that run whenever a student pushes code.
5. **Feedback**: Teachers can provide feedback directly via Pull Request comments.

### Use in AI Education
Classroom is ideal for AI courses where students need to implement algorithms (like building a neural network from scratch). The autograding feature can automatically verify if the student's model meets accuracy requirements or passes specific test cases.

## Interview Questions

**Q: What is the benefit of using GitHub Classroom over a traditional LMS (like Canvas or Moodle) for coding assignments?**
**A:** It teaches students industry-standard workflows (Git, PRs, CI/CD) while they learn to code. It also makes it much easier for teachers to clone all student repositories and run local tests if needed.
