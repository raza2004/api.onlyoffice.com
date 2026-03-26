---
sidebar_position: -2
title: User experience considerations
description: UX for AI features.
---

# User experience considerations {#user-experience-considerations}

Well-designed AI tools feel natural and reliable. Poorly designed ones confuse users, produce unexpected changes, and undermine trust. The guidelines below address the most common UX pitfalls when building AI features in ONLYOFFICE plugins.

## Keep tools focused {#keep-tools-focused}

Each registered tool should do exactly one thing. A tool that tries to handle multiple unrelated tasks — for example, both translating and summarizing depending on vague input phrasing — is harder for the AI model to invoke correctly and harder for users to understand.

**Signs a tool is doing too much:**

- The `description` contains the word "or" to describe different behaviors
- The `examples` cover completely different user intents
- The `params` include a mode-switching field like `"action": "translate" | "summarize" | "rewrite"`

**Preferred approach:** Register separate tools for each distinct action. The AI model selects among them automatically based on the user's phrasing.

## Name tools clearly {#name-tools-clearly}

The `func.name` string and the `description` field are both visible signals. `func.name` may appear in logs and developer tooling; `description` is read by the model and may also surface in settings UI.

Guidelines:

- Use verb-noun names for `func.name`: `"translateSelection"`, `"summarizeParagraph"`, `"insertBulletList"`.
- Avoid generic names like `"doAction"` or `"processText"`.
- Make the `description` immediately obvious to a non-technical user reading it for the first time.

## Provide feedback during long operations {#provide-feedback}

Any operation that takes more than approximately one second should indicate progress to the user. Without feedback, users assume the plugin has frozen and may click repeatedly, triggering duplicate requests.

Use `Asc.plugin.showMessage` to display a status message before starting a long-running call:

```ts
async call(params) {
  Asc.plugin.showMessage("Generating summary, please wait...");

  const result = await generateSummary(params);

  Asc.plugin.showMessage(""); // clear the message
  await Asc.Editor.callMethod("InsertText", result);
}
```

For operations with streaming output, the visible text appearing in the document is itself a progress indicator — no additional message is needed once text starts flowing.

## Use `StartAction`/`EndAction` correctly {#start-end-action}

Wrapping all AI-generated document modifications in a `StartAction`/`EndAction` pair lets the user undo the entire change in a single **Ctrl+Z** press.

```ts
await Asc.Editor.callCommand("StartAction", "AI generate");
try {
  await Asc.Editor.callMethod("InsertText", result);
} finally {
  await Asc.Editor.callCommand("EndAction");
}
```

> If `EndAction` is never called — for example, because an exception was thrown — the undo stack becomes inconsistent. Always use a `finally` block.

From the user's perspective:

- A single undo step means the AI's contribution is treated as one atomic action, which matches the mental model of "the AI did this, I want to undo what the AI did."
- Multiple undo steps for a single AI action feel broken and erode trust.

## Use streaming output for text generation {#streaming-output}

For tools that generate or transform text, streaming the output directly into the document as chunks arrive creates a sense of responsiveness and progress. Users see words appearing in real time rather than waiting for a spinner to disappear.

See [Handling async operations](./handling-async-operations.md) for implementation details on streaming with `chatRequest`.

## Do not insert large blocks of text without confirmation {#confirmation-for-large-changes}

Replacing a paragraph, section, or entire document without explicit user consent is disruptive and difficult to recover from. Follow these guidelines:

- **Insert at cursor** rather than replacing existing content unless the user explicitly selected text and asked for it to be replaced.
- For large replacements (more than a few sentences), consider displaying a preview or asking for confirmation before applying the change.
- When replacing selected text, keep the original on the clipboard so the user can paste it back if needed.

## Surface errors in a user-friendly way {#error-messages}

Never let raw error objects or stack traces reach the user. Catch all errors from `chatRequest` and translate them into plain-language messages:

```ts
try {
  await chatRequest(prompt, null, onChunk);
} catch (err) {
  Asc.plugin.showMessage(
    "The AI model did not respond. Please check your connection and try again."
  );
}
```

Common error scenarios to handle:

| Scenario | Suggested message |
|----------|-------------------|
| Network timeout | "The AI model did not respond. Please try again." |
| Invalid or missing API key | "AI is not configured. Open **Settings** to add your API key." |
| Empty selection when selection is required | "Please select some text before running this tool." |
| Provider rate limit exceeded | "Too many requests. Please wait a moment and try again." |

Use `Asc.plugin.showMessage` for transient status messages. For errors that require user action (such as a missing API key), consider directing the user to the relevant settings panel.
