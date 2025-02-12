
# Code Reviews

## Overview

Code review is a systematic examination of source code by peers to identify bugs, improve code quality, and ensure consistency with project standards.

At least one ode review is usually required for a pull request by contributors to a codebase. The first reviewewr is usually the feature/project lead, who may then tag one or more stakeholder members to also review the code.

For example, a pull request that affects database access or authentication will usually involve tagging the team member(s) in charge of those features to review the code and assess that it follows the relevant guidelines.

## Purpose

The purpose of a code review is to ensure code quality. Code that is of high quality has a low chance of introducing bugs, is sufficiently documented for other team members to understand, and is easily to modify or extend if necessary.

## Requesting a code review

In a GitHub pull request (PR), reviews are requested from reviewers through the Reviewers sidebar.

## Carrying out a code review

The feature/project lead’s main role is to:

- ensure the submitted code follows the project’s style guidelines and standards for documentation.
- ensure that the requisite tests are passed, or written if necessary.
- briefly review the code changes, checking for any unnecessary edits to files.
- tag other reviewers, typically ICs for specific parts of the project who will be affected by or have a stake in the code change.

Other ICs, if tagged, should:

- ensure that the code correctly uses any packages or code written/owned by them.
- ensure that the code will not negatively affect their part of the project in an unexpected way.

## Best practices

1. Make small, atomic commits
   - Each commit should only add code to address one method, feature, or task.
   - Each commit should ideally only edit one or two files.
   - Each commit should be small enough that its specific purpose can be described in one or two lines.
   - Each commit should have a clear commit message, and follow the commit message guidelines of the project.
   - Each commit should be clearly tagged, if required by the project.
2. Discuss the implementation with your feature/project lead before implementing it
   - a code review is much easier to clear if there are no surprising changes.
   - your reviewers should not be "surprised" by any of the changes you included; ideally they would have been informed beforehand, by you or the feature/project lead.
   - if time permits, discuss the changes you intend to make with the affected ICs/leads. They can likely advise you on the best way to go about it, saving you lots of time later having to address the changes requested.
4. Do not change the interface *and* the implementation in the same pull request
   - interface changes usually require other contributors to also change their code to comply.
   - Such changes should usually preserve existing implementation and functionality, to keep testing easy and keep changes manageable.
   - if you realise that your feature will require an interface change, discuss this with your lead beforehand to see if the feature can be implemented in separate pull requests.

## Further reading

- [GitLab: Code Review Guidelines](https://docs.gitlab.com/ee/development/code_review.html)
- [Google Engineering Practices Documentation: How to do a code review](https://google.github.io/eng-practices/review/reviewer/)
- [StackOverflow Blog: How to Make Good Code Reviews Better](https://stackoverflow.blog/2019/09/30/how-to-make-good-code-reviews-better/)
- [Conventional Commits: A specification for adding human and machine readable meaning to commit messages](https://www.conventionalcommits.org/en/v1.0.0/#summary)
