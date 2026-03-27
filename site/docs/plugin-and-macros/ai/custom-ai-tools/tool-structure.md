---
sidebar_position: -3
---

# Tool structure

This page describes the complete structure of a custom AI tool, covering the configuration object, JSON Schema parameters, examples format, UI naming conventions, and error handling.

## RegisteredFunction fields {#fields}

| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `name` | `string` | Yes | The identifier used by the agent to call the tool. Must be unique across all registered tools. |
| `text` | `string` | Yes | A short display name shown in the UI (e.g., `"Add Comment to Text"`). |
| `description` | `string` | Yes | A sentence describing the tool's purpose. Helps the agent choose the correct tool when multiple tools are registered. |
| `parameters` | `object` | Yes | A JSON Schema object describing the arguments the agent will pass to the tool. |
| `examples` | `object[]` | Yes | Example invocations that teach the agent when and how to call the tool. |
| `call` | `async function` | Yes | The handler that runs when the agent invokes the tool. Assigned after construction. Receives a `params` object with the values parsed from the agent's response. |

## Parameters format {#params-format}

The `parameters` field follows the [JSON Schema](https://json-schema.org/) format. Define each argument as a property with a `type` and `description`. Mark required arguments in the `required` array.

``` ts
"parameters": {
    "type": "object",
    "properties": {
        "prompt": {
            "type": "string",
            "description": "The user's instruction."
        },
        "count": {
            "type": "number",
            "description": "The number of rows to insert."
        },
        "position": {
            "type": "string",
            "enum": ["before", "after"],
            "description": "Where to insert — before or after the selection.",
            "default": "after"
        }
    },
    "required": ["prompt"]
}
```

Use `enum` to restrict a parameter to a fixed set of values. Use `default` to document the fallback when the argument is omitted.

## Examples format {#examples-format}

Each entry in `examples` is an object with two fields:

- `prompt` — the natural language phrase a user might type.
- `arguments` — the exact parameter object the agent should produce.

``` ts
"examples": [
    {
        "prompt": "Insert 3 rows after the selection",
        "arguments": { "prompt": "Insert 3 rows after the selection", "count": 3, "position": "after" }
    },
    {
        "prompt": "Add a row before this one",
        "arguments": { "prompt": "Add a row before this one", "count": 1, "position": "before" }
    }
]
```

Provide at least two examples per tool. Cover variations in parameters — including cases where optional arguments are omitted — to improve the agent's matching accuracy.

## UI naming {#ui-naming}

When displaying tool names or results in the editor UI, follow these conventions:

- Use sentence case for the `text` field: `"Insert rows"`, not `"Insert Rows"`.
- Refer to the product as **ONLYOFFICE**, not **OnlyOffice** or **Onlyoffice**.
- Use the exact editor names as they appear in the UI: **Text Document Editor**, **Spreadsheet Editor**, **Presentation Editor**.
- When referencing UI elements such as tabs, buttons, or menu items, use **bold**: click **OK**, open the **AI** tab.

## Translations {#translations}

Tool metadata (`parameters`, `examples`, `description`) is always written in English because it is sent directly to the AI model. Do not localize these fields.

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
