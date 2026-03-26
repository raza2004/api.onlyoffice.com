---
sidebar_position: 3
---

# Tool calling

The AI agent executes user requests by calling **tools** — registered functions that send requests to the AI model and apply the result to the document using the Office API.

## How tool calling works

Tool calling in the ONLYOFFICE AI agent follows a flow similar to [function calling in LLM APIs](https://platform.openai.com/docs/guides/function-calling):

1. **Registration.** Each tool is registered with a name, parameter schema, description, and usage examples. This metadata tells the AI model what the tool does and when to invoke it.
2. **User request.** The user opens the agent (`Ctrl + /`) and types a natural language request.
3. **Tool selection.** The AI model examines the request and the list of available tools, then selects the most appropriate tool and determines the arguments to pass.
4. **Execution.** The selected tool runs: it sends a request to the AI model and applies the result to the document via the Office API.

## Built-in tools

The AI agent ships with predefined tools for each editor type:

**Text document editor**
- Comment or annotate selected text
- Rewrite or rephrase text
- Check spelling and grammar
- Change text style and paragraph formatting
- Insert pages

**Spreadsheet editor**
- Add charts from data ranges
- Explain formulas
- Insert pivot tables
- Apply filters and sorting

**Presentation editor**
- Add new slides
- Add shapes, tables, and charts to slides
- Change slide backgrounds
- Duplicate or delete slides

## Custom tools

Developers can extend the agent by registering custom tools using the `RegisteredFunction` object. Custom tools follow the same calling pattern as built-in tools.

For implementation details, see [Creating a custom AI tool](../custom-ai-tools/creating-a-custom-ai-tool.md).
