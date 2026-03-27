---
sidebar_position: -4
---

# Creating a custom AI tool

This page explains the core concept and implementation steps for creating a custom AI tool that the ONLYOFFICE AI agent can call.

## Core concept {#concept}

A custom AI tool is built around the `RegisteredFunction` object. You create an instance by passing a configuration object with metadata fields, then assign a `call` handler that runs when the AI invokes the tool. The configured object is returned from a factory function and registered with the AI agent via the appropriate editor function map.

## Step 1. Create a factory function {#step-factory}

Wrap your tool definition in a factory function and add it to the function map that matches your target editor:

``` ts
WORD_FUNCTIONS.myTool = function() {
    let func = new RegisteredFunction({ ... });
    return func;
};
```

Use `WORD_FUNCTIONS` for text documents, `CELL_FUNCTIONS` for spreadsheets, and `SLIDE_FUNCTIONS` for presentations.

## Step 2. Define the configuration object {#step-config}

Pass a configuration object to the `RegisteredFunction` constructor. The AI agent reads this metadata to determine when and how to invoke the tool:

``` ts
WORD_FUNCTIONS.myTool = function() {
    let func = new RegisteredFunction({
        "name": "myTool",
        "text": "Apply Paragraph Style",
        "description": "Use this function when the user asks to change the style of a paragraph.",
        "parameters": {
            "type": "object",
            "properties": {
                "prompt": {
                    "type": "string",
                    "description": "The user's instruction."
                },
                "targetStyle": {
                    "type": "string",
                    "description": "The paragraph style to apply, e.g. 'Heading 1'."
                }
            },
            "required": ["prompt"]
        },
        "examples": [
            {
                "prompt": "Make it a heading",
                "arguments": { "prompt": "Make it a heading", "targetStyle": "Heading 1" }
            },
            {
                "prompt": "Reset paragraph style",
                "arguments": { "prompt": "Reset paragraph style", "targetStyle": "Normal" }
            }
        ]
    });

    return func;
};
```

| Field | Description |
| --- | --- |
| `name` | The identifier the agent uses to call the tool. |
| `text` | A short display name shown in the UI. |
| `description` | Tells the agent what this tool does and when to use it. |
| `parameters` | A JSON Schema object describing the arguments the agent will pass. |
| `examples` | Sample prompts with matching argument objects that teach the agent how to invoke the tool. |

Provide at least two `examples` covering different valid inputs to improve matching accuracy.

## Step 3. Implement the call handler {#step-call}

Assign an async function to `func.call`. This handler receives the parameters object that the AI agent parsed from its response:

``` ts
func.call = async function(params) {
    let style = params.targetStyle || "Normal";

    // Starts a block action so the change can be undone in a single step.
    await Asc.Editor.callMethod("StartAction", ["Block", "AI (myTool)"]);

    Asc.scope.style = style;
    await Asc.Editor.callCommand(function() {
        let doc = Api.GetDocument();
        let paragraph = doc.GetElement(0);
        if (paragraph) {
            paragraph.SetStyle(Api.GetStyle(Asc.scope.style));
        }
    });

    await Asc.Editor.callMethod("EndAction", ["Block", "AI (myTool)"]);
};
```

> The `Asc.scope` object is the correct way to pass data from the outer plugin context into `callCommand` closures. Variables declared outside the closure are not directly accessible inside it.

## Full example {#full-example}

``` ts
WORD_FUNCTIONS.myTool = function() {
    let func = new RegisteredFunction({
        "name": "myTool",
        "text": "Apply Paragraph Style",
        "description": "Use this function when the user asks to change the style of a paragraph.",
        "parameters": {
            "type": "object",
            "properties": {
                "prompt": {
                    "type": "string",
                    "description": "The user's instruction."
                },
                "targetStyle": {
                    "type": "string",
                    "description": "The paragraph style to apply, e.g. 'Heading 1'."
                }
            },
            "required": ["prompt"]
        },
        "examples": [
            {
                "prompt": "Make it a heading",
                "arguments": { "prompt": "Make it a heading", "targetStyle": "Heading 1" }
            },
            {
                "prompt": "Reset paragraph style",
                "arguments": { "prompt": "Reset paragraph style", "targetStyle": "Normal" }
            }
        ]
    });

    func.call = async function(params) {
        let style = params.targetStyle || "Normal";

        await Asc.Editor.callMethod("StartAction", ["Block", "AI (myTool)"]);

        Asc.scope.style = style;
        await Asc.Editor.callCommand(function() {
            let doc = Api.GetDocument();
            let paragraph = doc.GetElement(0);
            if (paragraph) {
                paragraph.SetStyle(Api.GetStyle(Asc.scope.style));
            }
        });

        await Asc.Editor.callMethod("EndAction", ["Block", "AI (myTool)"]);
    };

    return func;
};
```

## Using AI inside a tool {#using-ai}

If your tool needs to send a request to the AI model (for example, to generate text before inserting it), initialize a request engine using `AI.Request.create` and wrap the call with `StartAction`/`EndAction`. Use `requestEngine.modelUI.name` so the action label reflects the active model:

``` ts
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

await requestEngine.chatRequest(params.prompt, false, async function(data) {
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
```

> `checkEndAction` ensures `EndAction` is called only once even when the callback fires multiple times during streaming. Always call it both inside the callback (on first data) and after `chatRequest` completes.

The `AI.Request.create` and `chatRequest` methods are defined in [engine.js](https://github.com/ONLYOFFICE/onlyoffice.github.io/blob/master/sdkjs-plugins/content/ai/scripts/engine/engine.js).

For a deeper look at parameters and schema definitions, see [Tool structure](tool-structure.md).
