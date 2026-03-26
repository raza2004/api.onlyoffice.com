---
sidebar_position: -4
title: Managing API costs
description: Cost optimization strategies.
---

# Managing API costs {#managing-api-costs}

Every call to a cloud AI provider consumes tokens that are billed by the provider. Thoughtful plugin design can significantly reduce the number of tokens sent and received, keeping costs predictable and low.

## Why API costs matter {#why-api-costs-matter}

Each `chatRequest` call transmits a prompt to the configured AI provider and receives a completion in return. Both the prompt (input tokens) and the completion (output tokens) contribute to the billed usage. In a document editor context, prompts can grow large quickly if they include document content, so small inefficiencies compound across many users.

Key cost drivers:

- The length of the `prompt` string passed to `chatRequest`
- The length of the model's response
- The number of calls made per user action
- The model tier selected by the user (larger models cost more per token)

## Minimize prompt size {#minimize-prompt-size}

Extract only the content required for the task. Do not send the full document when a single paragraph or cell value is sufficient.

**Avoid:**

```ts
const fullText = await Asc.Editor.callMethod("GetDocumentText");
const prompt = `Summarize this document:\n\n${fullText}`;
```

**Prefer:**

```ts
const selectedText = await Asc.Editor.callMethod("GetSelectedText");
const prompt = `Summarize the following paragraph in one sentence:\n\n${selectedText}`;
```

Additional strategies:

- Trim whitespace and normalize line breaks before embedding text in the prompt.
- Remove boilerplate or repeated headers if the model does not need them.
- If you need the model to act on a table, send only the relevant rows, not the entire sheet.

## Avoid unnecessary requests {#avoid-unnecessary-requests}

Validate all inputs before creating an `AI.Request`. If a required parameter is missing or invalid, bail out early and surface a clear error message instead of making a request that will fail or produce garbage output.

```ts
async call(params) {
  const selectedText = await Asc.Editor.callMethod("GetSelectedText");

  if (!selectedText || selectedText.trim().length === 0) {
    Asc.plugin.showMessage("Please select some text before running this tool.");
    return;
  }

  const request = AI.Request.create(/* ... */);
  // proceed only when input is valid
}
```

Additional checks to add before calling `AI.Request.create`:

- Confirm that required `params` fields are present and within expected ranges.
- Check that the selection is not empty when the tool requires selected text.
- Reject inputs that are obviously too large (for example, more than a configurable character limit).

## Use streaming for perceived performance {#use-streaming}

Passing a callback as the third argument to `chatRequest` enables streaming — the callback fires repeatedly as chunks of the response arrive, rather than once when the full response is ready.

```ts
let result = "";
await chatRequest(
  prompt,
  null,
  async (chunk) => {
    result += chunk;
    await Asc.Editor.callMethod("InsertText", chunk);
  }
);
```

> Streaming does not reduce token usage — the same number of tokens is consumed regardless. However, it eliminates the perceived wait time for the user, which is especially valuable for long completions.

When to use streaming:

- Text generation tools where the output is inserted directly into the document
- Summarization or translation of long passages
- Any operation where the user would otherwise stare at a spinner for more than one second

## Use local providers for development and testing {#local-providers}

Cloud API calls incur costs even during development. Using a local provider such as [Ollama](../../ai/configuring-ollama-with-cors.md) means all requests stay on your machine and cost nothing, regardless of how many test iterations you run.

Recommended workflow:

1. Develop and iterate against a local Ollama instance.
2. Test with a small cloud model (such as a lower-tier GPT or Claude model) before promoting to production.
3. Reserve the most capable — and most expensive — models for final validation and production use.

## Choose the right `AI.ActionType` {#action-type}

The `AI.ActionType` enum describes the kind of operation being performed. Using the correct action type helps the plugin runtime and the AI provider route the request appropriately.

| Value | When to use |
|-------|-------------|
| `AI.ActionType.Chat` | General-purpose conversational requests, summarization, rewriting, Q&A |
| Other action types | Consult the API reference for specialized operations |

Using `AI.ActionType.Chat` for every request works but may not be optimal when more specific action types are available. Choosing the most appropriate type allows the runtime to apply the right context window and system prompt, which can reduce the amount of manual context you need to include in your prompt string — directly lowering token usage.
