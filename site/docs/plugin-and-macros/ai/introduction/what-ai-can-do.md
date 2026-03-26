---
sidebar_position: 1
---

# What AI can do in ONLYOFFICE

The ONLYOFFICE AI plugin brings intelligent capabilities directly into your document, spreadsheet, and presentation editors. It is built around three layers that work together:

## AI plugin

The AI plugin is a background plugin that connects an AI provider (e.g., OpenAI, DeepSeek, Anthropic Claude) to ONLYOFFICE editors.

**Supported editors:** documents, spreadsheets, presentations, PDF.

Out of the box it provides:

- **Chatbot** — ask questions, rewrite text, and brainstorm ideas in a side panel.
- **Summarization** — automatically summarize selected text and insert the result.
- **Translation** — translate selected text using the configured AI service.
- **Text analysis** — analyze selected text via the right-click context menu.

## AI agent

The AI agent is an inline assistant invoked with `Ctrl + /`. It understands natural language requests and executes them by calling registered tools.

The agent can:

- Generate and rewrite text directly in the editor.
- Apply formatting without navigating menus.
- Analyze and visualize data in spreadsheets.
- Maintain conversation history for multi-step requests.

## Custom AI tools

Custom AI tools are developer-defined functions that extend what the AI agent can do. Each tool specifies what request to send to the AI model and what action to perform on the document.

Custom tools allow you to:

- Add domain-specific actions tailored to your workflow.
- Integrate AI-driven operations into any editor type.
- Reuse and share tools across teams and deployments.

Ready-to-use examples are available in the [custom AI tool samples](../../samples/custom-ai-functions-samples/custom-ai-functions-samples.md).
