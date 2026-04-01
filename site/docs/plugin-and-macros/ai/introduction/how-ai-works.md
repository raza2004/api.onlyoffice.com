---
sidebar_position: 2
---

# How AI works

This page explains the execution flow for each layer of the ONLYOFFICE AI system.

## AI plugin example flow

1. The user opens the **AI** tab and clicks **Chatbot** (or right-clicks selected text).
2. The plugin sends the text and user prompt to the configured AI provider via its API.
3. The provider returns a response (streamed or complete).
4. The plugin displays the response and lets the user insert it into the document.

```
User input → AI plugin → Provider API → Response → Document
```
![Activate AI](/assets/images/plugins/activate-ai.png#gh-light-mode-only)![Activate AI](/assets/images/plugins/activate-ai.dark.png#gh-dark-mode-only)

## AI agent example flow

1. The user presses `Ctrl + /` to open the inline agent panel.
2. The user types a natural language request (e.g., "Add a comment explaining this paragraph").
3. The agent examines the request and the list of registered tools, then selects the best matching tool.
4. The selected tool runs: it sends a request to the AI model and applies the result to the document using the Office API.
5. The agent maintains conversation history so the user can refine the result step by step.

```
User request → Agent → Tool selection → Tool execution → Office API → Document
```
![Inline AI Agent](/assets/images/plugins/inline-ai-agent.png#gh-light-mode-only)![Inline AI Agent](/assets/images/plugins/inline-ai-agent.dark.png#gh-dark-mode-only)

## When to use custom AI tools

Use custom AI tools when the built-in agent tools do not cover your use case. Tools are scoped to a specific editor type — register tools in the correct map for the editor your plugin targets.

Common scenarios:

| Scenario | Use custom AI tools |
|----------|-------------------|
| Domain-specific document actions | Yes |
| Workflow automation unique to your team | Yes |
| Extending the agent for a specific editor type | Yes |
| General text generation or summarization | No — use the AI plugin directly |
| Simple formatting changes | No — use the AI agent's built-in tools |

For implementation details, see [Creating a custom AI tool](../custom-ai-tools/creating-a-custom-ai-tool.md).
