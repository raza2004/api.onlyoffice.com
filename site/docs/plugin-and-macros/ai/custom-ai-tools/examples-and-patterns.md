---
sidebar_position: 3
---

# Examples and patterns

This page covers common custom AI tool designs and recurring patterns you can apply when building your own tools for ONLYOFFICE editors.

## Pattern 1: document content insertion {#pattern-insertion}

A tool that generates content with the AI model and inserts it at the current cursor position. This is the most common pattern.

``` ts
WORD_FUNCTIONS.generateSummary = function() {
    let func = new RegisteredFunction({
        "name": "generateSummary",
        "text": "Generate Summary",
        "description": "Use this function when the user asks to summarize, shorten, or condense the document content.",
        "parameters": {
            "type": "object",
            "properties": {
                "prompt": {
                    "type": "string",
                    "description": "The user's instruction."
                },
                "length": {
                    "type": "string",
                    "enum": ["short", "medium", "long"],
                    "description": "Desired summary length.",
                    "default": "medium"
                }
            },
            "required": ["prompt"]
        },
        "examples": [
            {
                "prompt": "Summarize this document",
                "arguments": { "prompt": "Summarize this document", "length": "medium" }
            },
            {
                "prompt": "Write a short summary",
                "arguments": { "prompt": "Write a short summary", "length": "short" }
            }
        ]
    });

    func.call = async function(params) {
        let length = params.length || "medium";

        // Read the selected text from inside the editor context.
        // Asc.Editor.callCommand runs code where the full document API is available.
        let text = await Asc.Editor.callCommand(function() {
            return Api.GetDocument().GetRangeBySelect()?.GetText() || "";
        });

        // Bail out silently if nothing is selected.
        if (!text)
            return;

        // Initialize the AI request engine for the Chat action type.
        // Returns null if no provider is configured — always guard against this.
        let requestEngine = AI.Request.create(AI.ActionType.Chat);
        if (!requestEngine)
            return;

        // Build the prompt: combine the user's instruction with the selected text.
        let prompt = "Write a " + length + " summary of the following text:\n" + text;

        // checkEndAction ensures EndAction is called exactly once,
        // even though the chatRequest callback fires multiple times during streaming.
        let isSentEndAction = false;
        async function checkEndAction() {
            if (!isSentEndAction) {
                await Asc.Editor.callMethod("EndAction", ["Block", "AI (" + requestEngine.modelUI.name + ")"]);
                isSentEndAction = true;
            }
        }

        // Open a block action — labels the undo entry with the active model name.
        await Asc.Editor.callMethod("StartAction", ["Block", "AI (" + requestEngine.modelUI.name + ")"]);
        // Open a group action — all streamed chunks will form one undoable step.
        await Asc.Editor.callMethod("StartAction", ["GroupActions"]);

        // Send the prompt to the AI. The callback fires once per streamed chunk.
        await requestEngine.chatRequest(prompt, false, async function(data) {
            if (!data)
                return;

            // Close the block action on the first chunk, before writing to the document.
            await checkEndAction();

            // Use Asc.scope to transfer data across the plugin/editor context boundary.
            // Local variables from the outer scope are not accessible inside callCommand.
            Asc.scope.data = data;

            // Insert the chunk at the current cursor position.
            await Asc.Editor.callCommand(function() {
                Asc.Library.PasteText(Asc.scope.data);
            });
        });

        // Guarantee EndAction is called even if the callback never fired (empty response).
        await checkEndAction();
        // Close the group — all chunks now form a single undo entry.
        await Asc.Editor.callMethod("EndAction", ["GroupActions"]);
    };

    return func;
};
```

## Pattern 2: formatting without AI model request {#pattern-formatting}

A tool that applies formatting directly using the Office API, without sending a request to the AI model. Use this pattern for deterministic operations where the AI only needs to identify which formatting to apply.

``` ts
WORD_FUNCTIONS.applyHeading = function() {
    let func = new RegisteredFunction({
        "name": "applyHeading",
        "text": "Apply Heading Style",
        "description": "Use this function when the user asks to apply a heading style to the selected paragraph.",
        "parameters": {
            "type": "object",
            "properties": {
                "prompt": {
                    "type": "string",
                    "description": "The user's instruction."
                },
                "level": {
                    "type": "number",
                    "description": "Heading level from 1 to 6."
                }
            },
            "required": ["prompt"]
        },
        "examples": [
            {
                "prompt": "Make this a heading",
                "arguments": { "prompt": "Make this a heading", "level": 1 }
            },
            {
                "prompt": "Apply heading 2",
                "arguments": { "prompt": "Apply heading 2", "level": 2 }
            }
        ]
    });

    func.call = async function(params) {
        // Default to heading level 1 if the agent omits the argument.
        let level = params.level || 1;

        // Validate the level before touching the document.
        if (level < 1 || level > 6)
            return;

        // Asc.scope carries data into the callCommand closure.
        // Direct variable access across the context boundary is not possible.
        Asc.scope.styleName = "Heading " + level;

        // No AI request here — wrap in StartAction/EndAction for undo support only.
        await Asc.Editor.callMethod("StartAction", ["Block", "AI (applyHeading)"]);

        await Asc.Editor.callCommand(function() {
            let doc = Api.GetDocument();
            // GetRangeBySelect returns null if nothing is selected.
            let range = doc.GetRangeBySelect();
            if (!range)
                return;

            // Apply the heading style to the first paragraph in the selection.
            let paragraph = range.GetElement(0);
            if (paragraph) {
                paragraph.SetStyle(Api.GetStyle(Asc.scope.styleName));
            }
        });

        await Asc.Editor.callMethod("EndAction", ["Block", "AI (applyHeading)"]);
    };

    return func;
};
```

## Pattern 3: spreadsheet data tool {#pattern-spreadsheet}

A tool targeting the spreadsheet editor that reads cell data, processes it with the AI, and writes the result back.

``` ts
CELL_FUNCTIONS.explainCell = function() {
    let func = new RegisteredFunction({
        "name": "explainCell",
        "text": "Explain Cell",
        "description": "Use this function to explain the value or formula of the currently selected cell.",
        "parameters": {
            "type": "object",
            "properties": {
                "prompt": {
                    "type": "string",
                    "description": "The user's instruction."
                }
            },
            "required": ["prompt"]
        },
        "examples": [
            {
                "prompt": "Explain this cell",
                "arguments": { "prompt": "Explain this cell" }
            },
            {
                "prompt": "What does this formula do?",
                "arguments": { "prompt": "What does this formula do?" }
            }
        ]
    });

    func.call = async function(params) {
        // Read the value of the currently selected cell from the spreadsheet editor.
        let cellValue = await Asc.Editor.callCommand(function() {
            let sheet = Api.GetActiveSheet();
            // GetSelection returns the current cell range.
            let range = sheet.GetSelection();
            return range ? range.GetValue() : "";
        });

        // Nothing to explain if the cell is empty.
        if (!cellValue)
            return;

        // Initialize the AI request engine.
        let requestEngine = AI.Request.create(AI.ActionType.Chat);
        if (!requestEngine)
            return;

        // Combine the user's instruction with the cell value.
        let prompt = params.prompt + ":\n" + cellValue;

        // Same checkEndAction guard as Pattern 1 — prevents duplicate EndAction calls.
        let isSentEndAction = false;
        async function checkEndAction() {
            if (!isSentEndAction) {
                await Asc.Editor.callMethod("EndAction", ["Block", "AI (" + requestEngine.modelUI.name + ")"]);
                isSentEndAction = true;
            }
        }

        await Asc.Editor.callMethod("StartAction", ["Block", "AI (" + requestEngine.modelUI.name + ")"]);
        await Asc.Editor.callMethod("StartAction", ["GroupActions"]);

        // Stream the AI response and insert each chunk at the cursor.
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

## Choosing between patterns {#choosing}

| Use case | Pattern to follow |
| -------- | ----------------- |
| Generate or rewrite text and insert it into the document | Pattern 1 (content insertion) |
| Apply a style, format, or structural change determined by the AI | Pattern 2 (formatting without AI request) |
| Read cell or slide data, process it with AI, and write results back | Pattern 3 (spreadsheet/data tool) |
| Combine AI-generated content with programmatic document changes | Combine patterns 1 and 2 |

For more complete tool implementations, see the [custom AI function samples](../../samples/custom-ai-functions-samples/custom-ai-functions-samples.md).
