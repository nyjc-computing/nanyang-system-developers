# Code Assistants

## Overview

Unlike chatbots, code assistants are AI tools that have access to other files in the user’s project. Through a prompt, they can read the contents of other files, and also make changes to them. These assistants typically have a multi-stage reasoning model that enables them to think through the steps required to achieve the requested change.

Examples are [GitHub Copilot](https://github.com/features/copilot) and [Replit Assistant](https://replit.com/ai).

## Cautionary measures

When using AI assistants to make changes to code, keep in mind that:

- some assistants may use git commits to create checkpoints for rollback. These commits often do not follow project standards. It is highly recommended to use the assistant in a separate (throwaway) branch, then transfer the changes to the working branch.
- assistants will do their best to guess the files that should be included in the contxt window. They may not always guess correctly. Some assistants allow users to select/emphasize files that should be included in the context; learn to use this feature to influence the outcome.
- assistants will attempt to follow common/standard practices, which are not always in line with the project requirements. State clearly in the prompt any limitations or constraints the assistant should follow.
- When using a long series of prompts to refine the generated code, previous prompts and results may exceed the context window, resulting in the assistant "forgetting" what you said before. This can lead to the assistant ignoring constraints that were mentioned earlier.  
  If necessary, ask the assistant to summarize points from earlier, and write them to a file (e.g. `specifications.txt`) for it to refer to later.
