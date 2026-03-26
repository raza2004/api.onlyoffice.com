---
sidebar_position: -4
---

# Creating a custom AI tool

This page explains the core concept and implementation steps for creating a custom AI tool that the ONLYOFFICE AI agent can call.

## Core concept {#concept}

A custom AI tool is built around the `RegisteredFunction` object. You create an instance, configure its metadata fields, and assign a `call` handler that runs when the AI invokes the tool. The configured object is returned from a factory function and registered with the AI agent via the appropriate editor function map.

## Step 1. Create a factory function {#step-factory}

Wrap your tool definition in a named factory function. Add it to the function map that matches your target editor:

``` ts
WORD_FUNCTIONS.myTool = function() {
    let func = new RegisteredFunction();
    func.name = "myTool";
    return func;
};
```

Use `WORD_FUNCTIONS` for text documents, `CELL_FUNCTIONS` for spreadsheets, and `SLIDE_FUNCTIONS` for presentations.

## Step 2. Define metadata {#step-metadata}

Set the `params`, `examples`, and `description` fields. The AI agent reads this metadata to determine when and how to invoke the tool:

``` ts
func.params = [
    "targetStyle (string): the paragraph style to apply, e.g. 'Heading 1'"
];

func.examples = [
    "If you need to change the paragraph style to a heading, respond with:\n" +
    "[functionCalling (myTool)]: {\"prompt\": \"Make it a heading\", \"targetStyle\": \"Heading 1\"}"
];

func.description = "Use this function when the user asks to change the style of a paragraph.";
```

Keep `params` entries short and descriptive — they are passed verbatim to the AI model. Provide at least two `examples` covering different valid inputs to improve matching accuracy.

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

## Step 4. Return the function object {#step-return}

Return the fully configured `RegisteredFunction` object from the factory:

``` ts
WORD_FUNCTIONS.myTool = function() {
    let func = new RegisteredFunction();

    func.name = "myTool";
    func.params = [
        "targetStyle (string): the paragraph style to apply, e.g. 'Heading 1'"
    ];
    func.examples = [
        "If you need to change the paragraph style to a heading, respond with:\n" +
        "[functionCalling (myTool)]: {\"prompt\": \"Make it a heading\", \"targetStyle\": \"Heading 1\"}",
        "If you need to reset the paragraph style, respond with:\n" +
        "[functionCalling (myTool)]: {\"prompt\": \"Reset paragraph style\", \"targetStyle\": \"Normal\"}"
    ];
    func.description = "Use this function when the user asks to change the style of a paragraph.";

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

If your tool needs to send a request to the AI model (for example, to generate text before inserting it), initialize a request engine using `AI.Request.create`:

``` ts
let requestEngine = AI.Request.create(AI.ActionType.Chat);
if (!requestEngine)
    return;

let result = await requestEngine.chatRequest(params.prompt, false, async function(data) {
    if (!data)
        return;

    Asc.scope.data = data;
    await Asc.Editor.callCommand(function() {
        // Insert data into the document.
        Asc.Library.PasteText(Asc.scope.data);
    });
});
```

The `AI.Request.create` and `chatRequest` methods are defined in [engine.js](https://github.com/ONLYOFFICE/onlyoffice.github.io/blob/master/sdkjs-plugins/content/ai/scripts/engine/engine.js).

For a deeper look at parameters and schema definitions, see [Tool structure](tool-structure.md).
