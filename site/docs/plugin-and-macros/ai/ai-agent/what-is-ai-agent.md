---
sidebar_position: 1
---

# What is the AI agent

The AI agent is an inline contextual assistant integrated into ONLYOFFICE editors. It is available as a beta feature of the AI plugin, starting from version 2.4.2.

## Core concept

The AI agent is invoked with `Ctrl + /` and appears as a floating input panel directly in the editor. Unlike the AI plugin's side panel, the agent works inline — requests are typed in context and results are applied immediately to the document.

![AI agent prompt](/assets/images/plugins/ai-agent-prompt.png#gh-light-mode-only)![AI agent prompt](/assets/images/plugins/ai-agent-prompt.dark.png#gh-dark-mode-only)

The agent:

- Understands natural language requests for common editing tasks.
- Executes those requests by calling registered **tools** — functions that interact with the document via the Office API.
- Maintains conversation history, allowing iterative refinement through follow-up instructions.

## Role in the AI system

The AI agent sits between the AI plugin and custom AI tools:

```
AI plugin (provider connection)
    └── AI agent (natural language interface)
            └── Tools (registered functions that act on the document)
```

The agent does not directly modify the document. It selects the appropriate tool for the user's request, passes arguments to it, and the tool performs the actual document manipulation.

## What the agent can do

- **Text generation and rewriting.** Create new content or enhance existing text — summaries, ideas, rephrasing, tone adjustments.
- **Smart formatting.** Apply formatting changes described in natural language without navigating menus.
- **Data analysis and visualization.** Aggregate, sort, and filter spreadsheet data; generate charts and visual representations from text descriptions.

For a full list of built-in tools, see [Tool calling](tool-calling.md).
