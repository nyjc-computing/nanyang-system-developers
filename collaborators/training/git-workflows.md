# git workflows

## Overview

Git workflows are a set of guidelines that help teams collaborate on projects using Git. Common workflows include feature branching, Gitflow, and forking workflows. These workflows help manage code changes and ensure smooth integration.

## Common branch types

### `main`/`produnction`

This is the `main`/`prod` branch from which the current, stable version of the product/project is deployed from. By the time code is merged into `main`, it has been extensively tested for bugs and errors, and these merges are not expected to cause problems.

### `dev`/`testing`/`next`

This is the branch to which most big merges will take place; it is where the "next" version of the product/project is being tested, and where new features are being integrated into.

### feature branches

This is where new features are being developed, and which are eventually merged into `dev` when accepted. These branches may have sub-branches, if the team has multiple developers or if there are multiple parts to the feature to be worked on separately.

### issue/bugfix branches

These are ad-hoc branches which are created to address specific issues or bugs that have cropped up, usually after changes have been merged into `main`. These changes are usually merged directly to `main`, so as to quickly address vulnerablities or user-reported issues.

## Key concepts

### Upstream branch

The upstream branch, sometimes referred to as the parent branch, is the branch that serves as a target for all other branches. Changes made in different branches are eventually merged into the upstream branch. It is typically a main branch like `main` or `dev`, acting as the integration point for the team's work.

### Downstream branch

The downstream branch is derived from the upstream branch and is used to implement specific features or fixes. It does not serve as the primary integration branch but is often merged back into the upstream branch once work is completed. Downstream branches allow developers to work independently without disrupting the overall progress of the project.

### merge

The `merge` command in Git is used to integrate changes from different branches. When you run `git merge`, it combines the differences between two branches, creating a new commit that incorporates those changes. This is often used to bring feature branches into a main branch like `dev` or `main`, allowing new features or bug fixes to be added to the stable codebase. Properly resolving conflicts during a merge ensures that all changes are harmonized.

### pull request

A pull request (PR) is a GitHub feature that allows changes between two branches to be viewed in a web interface. Reviewers and other authorized members may add comments to the PR, or comment on specific lines of code in the files.

## Workflows

Once a projecct gains members, it is important to establish a common workflow. This is usually written as a document, describing the:

- branch hierarchy (i.e. which branches downstream of which other branches)
- members involved in an upstream merge
- process for merging code from a downstream branch to an upstream branch (usually through a PR)
- post-merge steps, e.g. synchronizing upstream changes to downstream branches

## Further reading

- [Git Branching - Branching Workflows](https://git-scm.com/book/en/v2/Git-Branching-Branching-Workflows)
  - Note that the primary branch is nowadays named `main` instead of `master`
- [Atlassian - Git feature branch workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow)
- [nulab: Branching workflows](https://nulab.com/learn/software-development/git-tutorial/git-collaboration/branching-workflows/)
  - This page provides a brief review of alternative git branching workflows
