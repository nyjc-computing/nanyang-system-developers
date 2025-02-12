
# Chatbots

## Overview

Chatbots (ChatGPT, Claude, DeepSeek, etc) are large language models (LLMs) with a chat interface. They take input from the user and return a response.

### Multimodal bots

Multimodal bots are bots that can take in other types of input besides text, such as documents, images, audio, or even video. Some bots may also be able to generate such types of files.

## Cautionary measures

When using chatbots to understand and generate code, keep in mind that:

- Chatbots are trained on a large corpus of public code, and will generally give code that generically follows standard practices. However, this is not always what is needed. Review and assess the suitability of the code before use.
- Chatbots do not know the required coding style of the project, and might not follow best practices or coding standards required by the project. Refactor the code as necessary to suit your project’s guidelines.
- Chatbots do not have access to environment variables and other configuration data of the project, and will not use them, or assume the existence of such data (following standard practice) which the project may not be following. Always review the correctness and appropriateness of the generated code for use in the project.
- Chatbot-generated code might interact with other components improperly. Validate integrations with other systems or components before deployment.
