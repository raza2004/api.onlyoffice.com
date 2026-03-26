---
sidebar_position: -3
---

# Tool structure

This page describes the complete structure of a custom AI tool, covering parameters, schema conventions, UI naming, translations, and error handling.

## RegisteredFunction fields {#fields}

| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `name` | `string` | Yes | The identifier used by the agent to call the tool. Must be unique across all registered tools. |
| `params` | `string[]` | Yes | A list of parameter descriptions passed to the AI model. Each entry describes one parameter and its expected type. |
| `examples` | `string[]` | Yes | Example invocations that teach the agent when and how to call the tool. |
| `description` | `string` | No | A short sentence describing the tool's purpose. Helps the agent choose the correct tool when multiple tools are registered. |
| `call` | `async function` | Yes | The handler that runs when the agent invokes the tool. Receives a `params` object with the values parsed from the agent's response. |

## Parameter format {#params-format}

Each entry in `func.params` follows this convention:

```
"paramName (type): short description of what the parameter represents"
```

Examples:

``` ts
func.params = [
    "count (number): the number of rows to insert",
    "position (string): where to insert — 'before' or 'after' the selection",
    "content (string): the text to place in the inserted rows (optional)"
];
```

Keep descriptions concise. The AI model uses them to determine what value to pass for each parameter. Mark optional parameters explicitly with `(optional)` at the end of the description.

## Examples format {#examples-format}

Each entry in `func.examples` is a plain string containing a scenario sentence followed by the exact JSON the agent should produce:

``` ts
func.examples = [
    "If you need to insert 3 rows after the selection, respond with:\n" +
    "[functionCalling (insertRows)]: {\"prompt\": \"Add 3 rows\", \"count\": 3, \"position\": \"after\"}",

    "If you need to insert a single row before the selection, respond with:\n" +
    "[functionCalling (insertRows)]: {\"prompt\": \"Insert a row here\", \"count\": 1, \"position\": \"before\"}"
];
```

Provide at least two examples per tool to improve the agent's matching accuracy. Cover edge cases such as optional parameters being omitted.

## UI naming {#ui-naming}

When displaying tool names or results in the editor UI, follow these conventions:

- Use sentence case for labels: **Insert rows**, not **Insert Rows**.
- Refer to the product as **ONLYOFFICE**, not **OnlyOffice** or **Onlyoffice**.
- Use the exact editor names as they appear in the UI: **Text Document Editor**, **Spreadsheet Editor**, **Presentation Editor**.
- When referencing UI elements such as tabs, buttons, or menu items, use **bold**: click **OK**, open the **AI** tab.

## Translations {#translations}

Tool metadata (`params`, `examples`, `description`) is always written in English because it is sent directly to the AI model. Do not localize these fields.

User-visible strings that your tool inserts into the document or displays in a notification should be sourced from the plugin's localization files. For details on plugin localization, refer to the [Localization](../../structure/localization.md) page.

## Error handling {#error-handling}

Handle errors inside `func.call` explicitly. Do not let unhandled rejections propagate — they can leave the editor in an inconsistent undo state if `StartAction` was already called.

A safe pattern:

``` ts
func.call = async function(params) {
    await Asc.Editor.callMethod("StartAction", ["Block", "AI (myTool)"]);

    try {
        Asc.scope.value = params.value;
        await Asc.Editor.callCommand(function() {
            let doc = Api.GetDocument();
            // Tool logic here.
        });
    } catch (e) {
        console.error("myTool failed:", e);
    } finally {
        await Asc.Editor.callMethod("EndAction", ["Block", "AI (myTool)"]);
    }
};
```

Always call `EndAction` in a `finally` block to ensure the undo stack is balanced, even when an error occurs.

### Common error scenarios {#error-scenarios}

| Scenario | Recommended handling |
| -------- | -------------------- |
| Required parameter missing or wrong type | Return early after logging; do not call `StartAction`. |
| `callCommand` returns `null` (no selection) | Display a notification using `Asc.plugin.showMessage` and return. |
| AI model request fails | Check the return value of `chatRequest`; handle `null` or error responses gracefully. |
| Office API method not available in the target editor | Wrap in a `try/catch` and fall back to an alternative implementation. |
