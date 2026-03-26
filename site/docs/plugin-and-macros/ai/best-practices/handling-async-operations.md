---
sidebar_position: -3
title: Handling async operations
description: Managing delays and responses.
---

# Handling async operations {#handling-async-operations}

AI tool functions in ONLYOFFICE plugins are inherently asynchronous. `func.call` is an `async` function, and nearly every operation inside it — reading document state, sending AI requests, and writing results back — must be awaited. Missing an `await` is one of the most common sources of subtle bugs.

## Why async matters {#why-async-matters}

The following operations are all asynchronous and must be awaited:

- `Asc.Editor.callCommand(...)` — executes a command in the editor context
- `Asc.Editor.callMethod(...)` — reads or writes document state
- `chatRequest(...)` — sends a request to the AI provider and waits for the response (or first chunk when streaming)

If any of these calls is not awaited, the plugin continues executing the next line before the previous operation has completed. This leads to:

- Reading stale or empty values from the document
- Inserting text at the wrong cursor position
- A broken undo stack because `StartAction`/`EndAction` brackets are mismatched
- Race conditions that are difficult to reproduce and debug

## Always `await` editor calls {#always-await}

Every call to `Asc.Editor.callCommand` and `Asc.Editor.callMethod` must be prefixed with `await`:

```ts
// Correct
const selectedText = await Asc.Editor.callMethod("GetSelectedText");

// Incorrect — selectedText will be a Promise, not a string
const selectedText = Asc.Editor.callMethod("GetSelectedText");
```

The same rule applies to sequences of commands:

```ts
// Correct — each step completes before the next begins
await Asc.Editor.callCommand("StartAction", "Insert AI result");
await Asc.Editor.callMethod("InsertText", result);
await Asc.Editor.callCommand("EndAction");
```

## The `StartAction`/`EndAction` pattern {#start-end-action}

Wrapping document modifications in `StartAction`/`EndAction` groups all changes into a single undo step. Without this, the user may need to press **Ctrl+Z** many times to undo a single AI-generated insertion.

```ts
async call(params) {
  const result = await generateText(params);

  await Asc.Editor.callCommand("StartAction", "AI insert");
  try {
    await Asc.Editor.callMethod("InsertText", result);
  } finally {
    await Asc.Editor.callCommand("EndAction");
  }
}
```

> Always place `EndAction` inside a `finally` block. If an error occurs between `StartAction` and `EndAction`, omitting `EndAction` leaves the undo stack in a corrupted state.

Why this matters for undo/redo:

- `StartAction` opens a transaction boundary in the editor's history.
- Every modification between `StartAction` and `EndAction` is recorded as a single atomic change.
- `EndAction` closes the boundary and commits the transaction.
- If `EndAction` is never called, the transaction stays open and subsequent user edits may be merged into it unexpectedly.

## Streaming responses {#streaming-responses}

When you pass a callback as the third argument to `chatRequest`, the callback fires multiple times as text chunks arrive from the provider. Each chunk is a partial fragment of the final response — typically a word or a short phrase.

```ts
let buffer = "";

await chatRequest(
  prompt,
  null,
  async (chunk) => {
    buffer += chunk;
    // Insert each chunk as it arrives
    await Asc.Editor.callMethod("InsertText", chunk);
  }
);
```

Guidelines for streaming:

- **Append, do not replace.** Each chunk contains only the new text, not the full response so far.
- **Keep the `StartAction`/`EndAction` bracket outside the callback.** Open `StartAction` before calling `chatRequest` and close `EndAction` after the `await` resolves, so the entire streamed insertion is a single undo step.
- **Avoid expensive operations inside the callback.** The callback fires frequently; any slow synchronous work inside it will delay rendering.

## Handling cancellation {#handling-cancellation}

If the user closes the plugin panel or presses **Esc** while a request is in progress, the plugin context may be torn down before `chatRequest` resolves. To handle this gracefully:

- Check whether the plugin is still active before inserting text from a streaming callback.
- Use a cancellation flag:

```ts
let cancelled = false;

// Set this flag in your plugin's close/unload handler
function onPluginClose() {
  cancelled = true;
}

await chatRequest(prompt, null, async (chunk) => {
  if (cancelled) return;
  await Asc.Editor.callMethod("InsertText", chunk);
});
```

- If `StartAction` was called before the request began, ensure `EndAction` is still called when cancellation is detected — for example, in a `finally` block.

## Timeout patterns {#timeout-patterns}

Cloud AI providers occasionally stall or return no response. To avoid leaving the user waiting indefinitely, wrap the request in a timeout:

```ts
async function chatRequestWithTimeout(prompt, options, onChunk, timeoutMs = 30000) {
  let timedOut = false;

  const timeout = setTimeout(() => {
    timedOut = true;
  }, timeoutMs);

  await chatRequest(prompt, options, async (chunk) => {
    if (timedOut) return;
    if (onChunk) await onChunk(chunk);
  });

  clearTimeout(timeout);

  if (timedOut) {
    throw new Error("The AI model did not respond in time. Please try again.");
  }
}
```

Surface the timeout error to the user with `Asc.plugin.showMessage` rather than silently swallowing it.

## Code example: streaming with full error handling {#streaming-example}

The following example demonstrates a tool that streams a completion into the document and handles partial responses, cancellation, and the undo boundary correctly:

```ts
async call(params) {
  const selectedText = await Asc.Editor.callMethod("GetSelectedText");

  if (!selectedText) {
    Asc.plugin.showMessage("Please select text before running this tool.");
    return;
  }

  const prompt = `Rewrite the following text to be more concise:\n\n${selectedText}`;
  let cancelled = false;

  await Asc.Editor.callCommand("StartAction", "AI rewrite");

  try {
    await chatRequest(prompt, null, async (chunk) => {
      if (cancelled) return;
      await Asc.Editor.callMethod("InsertText", chunk);
    });
  } catch (err) {
    cancelled = true;
    Asc.plugin.showMessage("The AI model did not respond. Please try again.");
  } finally {
    await Asc.Editor.callCommand("EndAction");
  }
}
```
