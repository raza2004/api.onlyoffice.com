---
sidebar_position: -5
---

# Overview

Custom AI tools let you extend the AI agent's built-in capabilities by defining new actions it can perform inside ONLYOFFICE editors. Each tool wraps a specific operation — such as formatting text, inserting structured content, or querying external data — and exposes it to the AI through a structured interface.

## What custom AI tools are {#what-are-tools}

A custom AI tool is a JavaScript function registered with the AI agent that the agent can invoke in response to a user prompt. The tool handles document interaction through the Office API and can optionally send additional requests to an AI model.

A custom AI tool consists of:

- a **name** that the agent uses to identify the tool;
- a list of **parameters** the agent passes when calling the tool;
- **examples** that teach the agent when and how to invoke the tool;
- a **call** function containing the actual implementation.

The `RegisteredFunction` object is the building block of every custom AI tool. It is defined in the [helperFuncs.js](https://github.com/ONLYOFFICE/onlyoffice.github.io/blob/master/sdkjs-plugins/content/ai/scripts/helpers/helperFuncs.js) file.

## When to use custom AI tools {#when-to-use}

Use custom AI tools when:

- the built-in AI agent actions do not cover a specific document operation you need;
- you want to expose business-specific logic to the AI, such as populating a template, validating form fields, or triggering an external integration;
- you are building a plugin that should respond to natural language instructions from the user.

For ready-to-use examples, refer to the [custom AI function samples](../../samples/custom-ai-functions-samples/custom-ai-functions-samples.md).

## How tools fit into the AI workflow {#workflow}

When a user types a prompt in the AI agent dialog (`CTRL + /`), the agent evaluates all registered tools and determines whether any of them match the request. If a match is found, the agent calls the tool with the appropriate parameters.

Custom tools are registered at plugin initialization and remain available for the lifetime of the plugin session:

1. The plugin registers its tools during startup by adding factory functions to the appropriate map (`WORD_FUNCTIONS`, `CELL_FUNCTIONS`, or `SLIDE_FUNCTIONS`).
2. The user opens the AI agent dialog and enters a prompt.
3. The agent matches the prompt against the registered tools using their `examples` and `description` metadata.
4. The agent invokes `func.call` with a parameters object parsed from its response.
5. The tool runs its document logic and optionally returns a result string to the agent.

## Next steps {#next-steps}

- [Creating a custom AI tool](creating-a-custom-ai-tool.md) — set up your first tool from scratch.
- [Tool structure](tool-structure.md) — learn about parameters, schema, UI naming, and error handling.
- [Examples and patterns](examples-and-patterns.md) — explore common tool designs.
- [Testing and debugging](testing-and-debugging.md) — validate and troubleshoot your tools.
