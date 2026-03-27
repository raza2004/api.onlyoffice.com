---
sidebar_position: -2
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
        "description": "Use this function when the user asks to summarize the document content.",
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

        let text = await Asc.Editor.callCommand(function() {
            return Api.GetDocument().GetRangeBySelect()?.GetText() || "";
        });

        if (!text)
            return;

        let requestEngine = AI.Request.create(AI.ActionType.Chat);
        if (!requestEngine)
            return;

        let prompt = "Write a " + length + " summary of the following text:\n" + text;

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
        let level = params.level || 1;
        if (level < 1 || level > 6)
            return;

        Asc.scope.styleName = "Heading " + level;

        await Asc.Editor.callMethod("StartAction", ["Block", "AI (applyHeading)"]);

        await Asc.Editor.callCommand(function() {
            let doc = Api.GetDocument();
            let range = doc.GetRangeBySelect();
            if (!range)
                return;

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

A tool targeting the spreadsheet editor that reads cell data, processes it with the AI, and writes results back.

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
        let cellValue = await Asc.Editor.callCommand(function() {
            let sheet = Api.GetActiveSheet();
            let range = sheet.GetSelection();
            return range ? range.GetValue() : "";
        });

        if (!cellValue)
            return;

        let requestEngine = AI.Request.create(AI.ActionType.Chat);
        if (!requestEngine)
            return;

        let prompt = params.prompt + ":\n" + cellValue;

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

## Choosing between patterns {#choosing}

| Use case | Pattern to follow |
| -------- | ----------------- |
| Generate or rewrite text and insert it into the document | Pattern 1 (content insertion) |
| Apply a style, format, or structural change determined by the AI | Pattern 2 (formatting without AI request) |
| Read cell or slide data, process it with AI, and write results back | Pattern 3 (spreadsheet/data tool) |
| Combine AI-generated content with programmatic document changes | Combine patterns 1 and 2 |

For more complete tool implementations, see the [custom AI function samples](../../samples/custom-ai-functions-samples/custom-ai-functions-samples.md).
