---
sidebar_position: -3
title: "AI integration overview"
---

# AI integration overview {#ai-integration-overview}

This page explains the concepts, architecture, and system flow that underpin every AI feature built with the ONLYOFFICE AI plugin.

## High-level architecture {#high-level-architecture}

At the highest level, an AI-powered feature in ONLYOFFICE involves three layers:

1. **ONLYOFFICE editors** — the document editing environment (Writer, Spreadsheet, or Presentation). The editor exposes the plugin API and the `Asc.Editor.callCommand` bridge.
2. **AI plugin** — the intermediary that registers tools, manages the AI agent, constructs requests, and writes results back into the document.
3. **AI provider** — a cloud service (OpenAI, DeepSeek, etc.) or a locally running model server (Ollama) that processes natural-language prompts and returns completions.

The plugin communicates outward to the provider over HTTP/HTTPS and inward to the editor through the plugin API. Neither the editor nor the provider has direct knowledge of the other.

## Key components {#key-components}

### AI plugin {#ai-plugin}

The AI plugin (GUID `{9DC93CDB-B576-4F0C-B55E-FCC9C48DD007}`) is the runtime that hosts all AI features. It ships with ONLYOFFICE Docs and Desktop Editors from v9.0.4 and can also be installed from the App Directory.

### AI agent (beta) {#ai-agent}

Available from plugin **v2.4.2**, the AI agent is an autonomous loop that:

- Receives a free-text prompt from the user (entered via the **Ctrl + /** shortcut).
- Decides which registered tool best matches the prompt.
- Invokes the tool, observes the result, and either returns a final answer or continues reasoning.

The agent is currently in beta. Its behaviour and API surface may change in future releases.

### AI request engine (`AI.Request.create`) {#ai-request-engine}

`AI.Request.create` is the low-level function used to send a prompt to the currently configured provider and receive a completion. Every tool implementation calls this function (directly or indirectly through the agent loop) to interact with the model.

### Custom tools and functions {#custom-tools-and-functions}

Tools are the unit of extension. Each tool is a plain JavaScript object with at minimum:

- A **name** and **description** that the agent uses for tool selection.
- A **`func.call`** implementation that carries out the action when the tool is selected.

Tools are grouped into editor-scoped function maps (see below) so that only the tools relevant to the active editor type are exposed to the agent.

## Editor-scoped function maps {#editor-scoped-function-maps}

The AI plugin maintains three separate maps of registered tools, one per editor type:

| Map name | Editor type |
| --- | --- |
| `WORD_FUNCTIONS` | Document editor |
| `CELL_FUNCTIONS` | Spreadsheet editor |
| `SLIDE_FUNCTIONS` | Presentation editor |

When the agent receives a prompt, it looks up the map that corresponds to the currently open document type, builds a tool list from that map, and passes it to the model as available functions. This ensures that a spreadsheet-only tool is never accidentally invoked inside a presentation.

## How `Asc.Editor.callCommand` bridges contexts {#how-callcommand-bridges-contexts}

The AI plugin runs in an isolated JavaScript context (a sandboxed iframe or a separate plugin thread). The editor document model runs in a different context. `Asc.Editor.callCommand` is the bridge:

- Code inside `callCommand` executes **inside the editor context**, where the full document API is available.
- Code outside `callCommand` executes **inside the plugin context**, where network access, `AI.Request.create`, and the DOM are available.
- Data must be passed between the two contexts explicitly — see the section on `Asc.scope` below.

Forgetting this boundary is the most common source of errors when building AI features.

## The `Asc.scope` data bridge {#asc-scope-data-bridge}

`Asc.scope` is a plain object that is serialised and copied across the plugin/editor context boundary when `callCommand` is called. Use it to pass data in both directions:

- **Into the editor:** set properties on `Asc.scope` before calling `callCommand`, then read them inside the command as `Asc.scope.myProperty`.
- **Out of the editor:** set properties on `Asc.scope` inside the command, then read them in the `callCommand` callback after the command returns.

Any value that cannot be serialised to JSON (functions, DOM nodes, class instances with methods) cannot be transferred via `Asc.scope`.

## System flow {#system-flow}

The following numbered list describes the end-to-end flow for a single AI agent interaction:

1. The user presses **Ctrl + /** in the editor and types a prompt.
2. The AI agent receives the prompt text.
3. The agent selects the correct editor-scoped function map (`WORD_FUNCTIONS`, `CELL_FUNCTIONS`, or `SLIDE_FUNCTIONS`) based on the active editor.
4. The agent calls `AI.Request.create`, passing the prompt and the list of available tools derived from the function map.
5. The AI provider returns a response. If the response contains a tool-call instruction, the agent identifies the matching tool by name.
6. The agent invokes `func.call` on the matched tool, passing any arguments supplied by the model.
7. Inside `func.call`, the implementation typically:
   a. Reads document state by calling `Asc.Editor.callCommand` (with data returned via `Asc.scope`).
   b. Optionally calls `AI.Request.create` again for a follow-up model completion.
   c. Writes results back to the document with a second `Asc.Editor.callCommand`.
8. `func.call` returns its result to the agent loop.
9. The agent determines whether further tool calls are needed. If not, it surfaces the final answer to the user.
10. The plugin calls `Asc.Editor.callMethod("EndAction", [...])` to signal that the operation is complete.
