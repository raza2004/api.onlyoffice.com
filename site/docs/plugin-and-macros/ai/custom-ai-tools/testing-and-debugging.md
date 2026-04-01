---
sidebar_position: 4
---

# Testing and debugging

This page covers how to validate custom AI tools, handle retries, and diagnose incorrect or unexpected outputs.

## Validating tool registration {#validating-registration}

Before testing a tool's logic, confirm it is registered correctly by inspecting the function map in the browser console after the plugin loads:

``` ts
console.log(Object.keys(WORD_FUNCTIONS));
```

If your tool name appears in the output, the factory function ran successfully. If it is missing, check that the factory is called during plugin initialization and that the function returns the `RegisteredFunction` object.

## Testing tool invocation {#testing-invocation}

To test a tool without going through the AI agent, call the factory and invoke `func.call` directly with a mock parameters object:

``` ts
let func = WORD_FUNCTIONS.myTool();
func.call({ prompt: "Test prompt", targetStyle: "Heading 1" });
```

This isolates the tool logic from the AI model and lets you verify document changes independently.

## Debugging in the browser developer tools {#debugging}

Open the browser developer tools (`F12`) while the plugin is active. All `console.log` and `console.error` calls from plugin scripts appear in the **Console** tab. Useful checkpoints to log:

- Parameter values at the start of `func.call`
- The value returned by `callCommand`
- The `data` argument inside the `chatRequest` callback
- Any Office API return values that might be `null`

For instructions on opening developer tools for the web editor plugin environment, refer to [Debugging for web editors](../../tutorials/debugging/for-web-editors.md).

## Handling retries {#retries}

The AI agent does not automatically retry a tool call if it fails silently. If your tool may fail due to transient conditions (for example, a network request inside `chatRequest`), implement retry logic manually:

``` ts
func.call = async function(params) {
    const maxAttempts = 3;
    let attempt = 0;
    let success = false;

    while (attempt < maxAttempts && !success) {
        attempt++;
        try {
            let requestEngine = AI.Request.create(AI.ActionType.Chat);
            if (!requestEngine)
                break;

            await requestEngine.chatRequest(params.prompt, false, async function(data) {
                if (!data)
                    return;

                Asc.scope.data = data;
                await Asc.Editor.callCommand(function() {
                    Asc.Library.PasteText(Asc.scope.data);
                });
                success = true;
            });
        } catch (e) {
            console.warn("Attempt " + attempt + " failed:", e);
        }
    }

    if (!success) {
        console.error("myTool failed after " + maxAttempts + " attempts.");
    }
};
```

## Incorrect or unexpected outputs {#incorrect-outputs}

If the AI agent calls the tool with wrong parameter values, the most common causes are:

- **Ambiguous `params` descriptions.** If the AI cannot determine the correct value from the description, it guesses. Make descriptions explicit and include the expected type and valid values.
- **Too few `examples`.** With only one example, the agent may not generalize correctly. Add examples that cover different parameter combinations, including edge cases.
- **Tool name conflicts.** If two registered tools have similar names or descriptions, the agent may call the wrong one. Make `func.description` distinct and specific.

**Checklist for incorrect outputs:**

1. Log the raw `params` object at the start of `func.call` and verify the values match your expectations.
2. Review the `examples` and `params` fields for ambiguity.
3. Test the tool in isolation (see [Testing tool invocation](#testing-invocation)).
4. If the agent consistently picks the wrong tool, add a more specific `func.description`.

## Undo state issues {#undo-state}

If the editor's undo history becomes corrupted after a tool fails, the most likely cause is an unbalanced `StartAction`/`EndAction` pair. Always use `try/finally` to guarantee `EndAction` is called:

``` ts
func.call = async function(params) {
    await Asc.Editor.callMethod("StartAction", ["Block", "AI (myTool)"]);
    try {
        // Tool logic here.
    } finally {
        await Asc.Editor.callMethod("EndAction", ["Block", "AI (myTool)"]);
    }
};
```

For more details on `StartAction` and `EndAction`, refer to [StartAction](../../interacting-with-editors/api-by-editor-type/text-document-api/Methods/StartAction.md) and [EndAction](../../interacting-with-editors/api-by-editor-type/text-document-api/Methods/EndAction.md).
