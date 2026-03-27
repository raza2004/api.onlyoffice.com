---
sidebar_position: -2
title: "Your first custom AI tool"
---

# Your first custom AI tool

This page walks you through building a complete, working custom AI tool from scratch: a `summarizeSelection` tool that reads the user's selected text, sends it to the AI model, and inserts the summary back into the document.

## What you will build {#what-you-will-build}

The `summarizeSelection` tool registers with the AI agent and becomes available via the **Ctrl + /** dialog. When the user selects text and types a prompt such as "Summarize this", the agent reads the selection, calls the configured AI provider, and streams the result back into the document at the cursor position.

## Prerequisites {#prerequisites}

- The AI plugin is installed and at least one provider is configured. See [Prerequisites](./prerequisites.md).
- You have a working understanding of the ONLYOFFICE plugin architecture. See [AI integration overview](./ai-integration-overview.md).

## Step 1: Register the tool {#step-1-register}

Add the tool to `WORD_FUNCTIONS` so the AI agent can discover it. The `RegisteredFunction` constructor takes a single configuration object describing the tool's identity, parameters, and example invocations.

``` ts
WORD_FUNCTIONS.summarizeSelection = function() {
    let func = new RegisteredFunction({
        "name": "summarizeSelection",
        "text": "Summarize Selection",
        "description": "Use this function when the user asks to summarize, shorten, or condense the selected text.",
        "parameters": {
            "type": "object",
            "properties": {
                "prompt": {
                    "type": "string",
                    "description": "The user's instruction, e.g. 'Summarize this in two sentences'."
                },
                "length": {
                    "type": "string",
                    "enum": ["one sentence", "two sentences", "a short paragraph"],
                    "description": "How long the summary should be.",
                    "default": "two sentences"
                }
            },
            "required": ["prompt"]
        },
        "examples": [
            {
                "prompt": "Summarize this",
                "arguments": { "prompt": "Summarize this", "length": "two sentences" }
            },
            {
                "prompt": "Condense this to one sentence",
                "arguments": { "prompt": "Condense this to one sentence", "length": "one sentence" }
            },
            {
                "prompt": "Give me a short paragraph summary",
                "arguments": { "prompt": "Give me a short paragraph summary", "length": "a short paragraph" }
            }
        ]
    });

    return func;
};
```

> `WORD_FUNCTIONS` scopes the tool to the text document editor. Use `CELL_FUNCTIONS` for spreadsheets and `SLIDE_FUNCTIONS` for presentations.

## Step 2: Implement the call handler {#step-2-implement}

Assign `func.call` to an async function that reads the selected text, sends it to the AI model, and streams the response back into the document.

``` ts
func.call = async function(params) {
    // 1. Read the selected text from the editor context.
    let selectedText = await Asc.Editor.callCommand(function() {
        let doc = Api.GetDocument();
        let range = doc.GetRangeBySelect();
        return range ? range.GetText() : "";
    });

    if (!selectedText)
        return;

    // 2. Build the prompt combining the user's instruction and the selected text.
    let length = params.length || "two sentences";
    let prompt = params.prompt + " in " + length + ":\n\n" + selectedText;

    // 3. Create the AI request engine.
    let requestEngine = AI.Request.create(AI.ActionType.Chat);
    if (!requestEngine)
        return;

    // 4. Set up StartAction / EndAction so the result is a single undoable step.
    let isSentEndAction = false;
    async function checkEndAction() {
        if (!isSentEndAction) {
            await Asc.Editor.callMethod("EndAction", ["Block", "AI (" + requestEngine.modelUI.name + ")"]);
            isSentEndAction = true;
        }
    }

    await Asc.Editor.callMethod("StartAction", ["Block", "AI (" + requestEngine.modelUI.name + ")"]);
    await Asc.Editor.callMethod("StartAction", ["GroupActions"]);

    // 5. Send the request and stream the result into the document.
    await requestEngine.chatRequest(prompt, false, async function(data) {
        if (!data)
            return;

        await checkEndAction();
        Asc.scope.data = data;
        await Asc.Editor.callCommand(function() {
            Asc.Library.PasteText(Asc.scope.data);
        });
    });

    await checkEndAction();
    await Asc.Editor.callMethod("EndAction", ["GroupActions"]);
};
```

Key points:

- `Asc.Editor.callCommand` runs code inside the editor context where the full document API is available. Use `Asc.scope` to pass data in and out — local variables from the outer function are not accessible inside the closure.
- `AI.Request.create(AI.ActionType.Chat)` initializes a request engine bound to the currently configured provider. Always check for `null` before using it.
- `requestEngine.modelUI.name` is the display name of the active model and is used to label the undo action.
- `checkEndAction` ensures `EndAction` is called exactly once even when the callback fires multiple times during streaming.

## Step 3: Test the tool {#step-3-test}

1. Open a text document in ONLYOFFICE Docs or Desktop Editors.
2. Select a paragraph or a few sentences.
3. Press **Ctrl + /** to open the AI agent dialog.
4. Type a prompt such as `Summarize this` and press **Enter**.
5. The agent matches the prompt to `summarizeSelection`, calls `func.call`, and streams the summary into the document.

To undo the insertion, press **Ctrl + Z** — the entire operation reverts in a single step because of the `StartAction`/`EndAction` pair.

## Complete implementation {#complete-implementation}

The following is the full `summarizeSelection` tool ready to be added to the AI plugin source.

``` ts
WORD_FUNCTIONS.summarizeSelection = function() {
    let func = new RegisteredFunction({
        "name": "summarizeSelection",
        "text": "Summarize Selection",
        "description": "Use this function when the user asks to summarize, shorten, or condense the selected text.",
        "parameters": {
            "type": "object",
            "properties": {
                "prompt": {
                    "type": "string",
                    "description": "The user's instruction, e.g. 'Summarize this in two sentences'."
                },
                "length": {
                    "type": "string",
                    "enum": ["one sentence", "two sentences", "a short paragraph"],
                    "description": "How long the summary should be.",
                    "default": "two sentences"
                }
            },
            "required": ["prompt"]
        },
        "examples": [
            {
                "prompt": "Summarize this",
                "arguments": { "prompt": "Summarize this", "length": "two sentences" }
            },
            {
                "prompt": "Condense this to one sentence",
                "arguments": { "prompt": "Condense this to one sentence", "length": "one sentence" }
            },
            {
                "prompt": "Give me a short paragraph summary",
                "arguments": { "prompt": "Give me a short paragraph summary", "length": "a short paragraph" }
            }
        ]
    });

    func.call = async function(params) {
        let selectedText = await Asc.Editor.callCommand(function() {
            let doc = Api.GetDocument();
            let range = doc.GetRangeBySelect();
            return range ? range.GetText() : "";
        });

        if (!selectedText)
            return;

        let length = params.length || "two sentences";
        let prompt = params.prompt + " in " + length + ":\n\n" + selectedText;

        let requestEngine = AI.Request.create(AI.ActionType.Chat);
        if (!requestEngine)
            return;

        let isSentEndAction = false;
        async function checkEndAction() {
            if (!isSentEndAction) {
                await Asc.Editor.callMethod("EndAction", ["Block", "AI (" + requestEngine.modelUI.name + ")"]);
                isSentEndAction = true;
            }
        }

        await Asc.Editor.callMethod("StartAction", ["Block", "AI (" + requestEngine.modelUI.name + ")"]);
        await Asc.Editor.callMethod("StartAction", ["GroupActions"]);

        await requestEngine.chatRequest(prompt, false, async function(data) {
            if (!data)
                return;

            await checkEndAction();
            Asc.scope.data = data;
            await Asc.Editor.callCommand(function() {
                Asc.Library.PasteText(Asc.scope.data);
            });
        });

        await checkEndAction();
        await Asc.Editor.callMethod("EndAction", ["GroupActions"]);
    };

    return func;
};
```

For more tool examples and patterns, see [Examples and patterns](../custom-ai-tools/examples-and-patterns.md).
