# Writing Good Commit Messages

Commit messages are an essential part of software development. They explain *why* a change was made and provide context for anyone looking at the project's history. Writing clear commit messages helps your team understand what has been done, making collaboration easier and the development process smoother.

## Why Are Good Commit Messages Important?

1. **Clarity**: A good commit message explains the purpose of the change. This is especially helpful when revisiting the code weeks or months later.
2. **Collaboration**: When working in a team, clear commit messages help others understand your work without needing to ask for explanations.
3. **Debugging**: If something goes wrong, commit messages make it easier to identify when and why a change was introduced.
4. **Professionalism**: Writing good commit messages is a skill that will serve you well in future projects and jobs.

## How to Write a Good Commit Message

A good commit message typically has two parts:
1. A **short summary** (50 characters or less) that describes the change.
2. An **optional detailed description** that explains why the change was made or provides additional context.

### Examples of Good Commit Messages

Here are some examples in the context of a team of students developing internal services for the college:

- **Fix a bug**:  
  ```
  fix: resolve crash when submitting feedback form
  ```
  This message explains that a bug causing a crash was fixed.

- **Add a new feature**:  
  ```
  feat: add email notifications for assignment deadlines
  ```
  This message shows that a new feature (email notifications) was added.

- **Improve existing code**:  
  ```
  refactor: simplify logic for calculating GPA
  ```
  This message indicates that the code was improved without changing its functionality.

- **Update documentation**:  
  ```
  docs: update README with setup instructions
  ```
  This message explains that the documentation was updated.

### Tips for Writing Commit Messages

- Use the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) format for consistency. This format uses prefixes like `fix:`, `feat:`, `docs:`, and `refactor:` to categorize changes.
- Keep the summary short and to the point.
- Use the imperative mood (e.g., "add", "fix", "update") in the summary.
- If needed, add a blank line after the summary and include more details in the description.

### Bad vs. Good Commit Messages

- **Bad**:  
  ```
  fixed stuff
  ```
  This message is vague and doesn't explain what was fixed.

- **Good**:  
  ```
  fix: resolve crash when submitting feedback form
  ```
  This message is clear and provides context.

By writing good commit messages, you contribute to a clear and professional development history that benefits the entire team. Practice this skill, and it will become second nature!
