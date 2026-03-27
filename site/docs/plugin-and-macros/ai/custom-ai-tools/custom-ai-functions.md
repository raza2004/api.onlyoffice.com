---
sidebar_position: 1
---

# Creating custom functions (beta)

AI functions define the functionality of an AI agent. They specify:

- what request to send to the AI model;
- what manipulations to perform on the document.

Adding custom AI functions expands the AI agent's capabilities and allows adapting it to specific use cases. Whether working with documents, spreadsheets, or presentations, custom functions let you integrate AI-driven operations directly into your workflow and align the agent's behavior with your requirements.

You can find ready-to-use custom AI functions [here](../samples/custom-ai-functions-samples/custom-ai-functions-samples.md) or create your own ones.

## How to use AI functions {#usage}

To use AI functions, follow the steps below:

1. [Add a model](ai-plugin.md#configuring) to the AI plugin.
2. Open the AI agent dialog box by pressing `CTRL + /`.
3. Enter your prompt and press `Enter`.

## Example: the commentText function {#example}

The `commentText` function allows adding AI generated comments directly to the document. Here's how it works:

1. Select a word to leave a comment on.
2. Open the AI agent dialog box (`CTRL + /`).
3. Type in the instruction for the AI agent. For example: `Explain this text`.
4. Press `Enter`.

![commentText execution](/assets/images/plugins/comment-text-function.png#gh-light-mode-only)![commentText execution](/assets/images/plugins/comment-text-function.dark.png#gh-dark-mode-only)

The AI agent will run the `commentText` function and insert relevant comments into the document.

![commentText result](/assets/images/plugins/comment-text-result.png#gh-light-mode-only)![commentText result](/assets/images/plugins/comment-text-result.dark.png#gh-dark-mode-only)

## How to add custom AI functions {#adding-functions}

The process of adding a custom function involves two main phases:

- [Function registration](#registration): Registers the AI function and its metadata within the agent's environment.
- [Function execution](#execution): Implements the core logic, which includes sending requests to the AI model and manipulating document content using our [Office API](../../office-api/get-started/overview.md).

### Function registration {#registration}

To add a new function, the `RegisteredFunction` object is used. Pass a configuration object to the constructor with the tool's metadata:

#### Parameters {#parameters}

| Name | Type | Example | Description |
|---|---|---|---|
| `name` | `string` | `"commentText"` | The function name. |
| `text` | `string` | `"Add Comment to Text"` | A short display name shown in the UI. |
| `description` | `string` | `"Adds a comment or footnote to explain the selected text."` | Explains to the AI what the function is used for. |
| `parameters` | `object` | JSON Schema object | Describes the arguments the AI agent will pass to the function. |
| `examples` | `object[]` | Array of `{prompt, arguments}` objects | Example invocations that teach the agent when and how to call the function. |

#### Example {#example-registration}

``` ts
let func = new RegisteredFunction({
    "name": "commentText",
    "text": "Add Comment to Text",
    "description": "Adds a comment or footnote to explain or annotate the selected text. If no text is selected, works with the current paragraph.",
    "parameters": {
        "type": "object",
        "properties": {
            "prompt": {
                "type": "string",
                "description": "The instruction for what to explain or comment about the text."
            },
            "type": {
                "type": "string",
                "enum": ["comment", "footnote"],
                "description": "Whether to add as a comment or as a footnote.",
                "default": "comment"
            }
        },
        "required": ["prompt"]
    },
    "examples": [
        {
            "prompt": "Explain this text",
            "arguments": { "prompt": "Explain this text", "type": "comment" }
        },
        {
            "prompt": "Add a historical context as footnote",
            "arguments": { "prompt": "Add historical context", "type": "footnote" }
        },
        {
            "prompt": "Comment on the significance",
            "arguments": { "prompt": "Explain significance", "type": "comment" }
        }
    ]
});
```

The `RegisteredFunction()` object is defined in the [helperFuncs.js](https://github.com/ONLYOFFICE/onlyoffice.github.io/blob/master/sdkjs-plugins/content/ai/scripts/helpers/helperFuncs.js) file.

### Function execution {#execution}

After registering the function, implement the actual logic that gets executed when the AI calls this function:

1. Retrieve the selected text using `Asc.Editor.callCommand()`:

    ``` ts
    func.call = async function(params) {
        let type = params.type;
        let isFootnote = "footnote" === type;

        let text = await Asc.Editor.callCommand(function(){
            let doc = Api.GetDocument();
            let range = doc.GetRangeBySelect();
            let text = range ? range.GetText() : "";
            if (!text)
            {
                text = doc.GetCurrentWord();
                doc.SelectCurrentWord();
            }
            return text;
        });
    };
    ```

2. Construct the prompt for the AI by combining `params.prompt` and the selected text:

    ``` ts
    let argPromt = params.prompt + ":\n" + text;
    ```

3. Initialize a request engine using `AI.Request.create`. The object is defined in the [engine.js](https://github.com/ONLYOFFICE/onlyoffice.github.io/blob/master/sdkjs-plugins/content/ai/scripts/engine/engine.js) file:

    ``` ts
    let requestEngine = AI.Request.create(AI.ActionType.Chat);
    if (!requestEngine)
        return;
    ```

4. Send the request using `chatRequest()` and receive the result in a callback:

    ``` ts
    let result = await requestEngine.chatRequest(argPromt, false, async function(data) {
        if (!data)
            return;
    });
    ```

5. Insert the response as a comment or footnote using [AddFootnote()](../../office-api/usage-api/text-document-api/ApiDocument/Methods/AddFootnote.md) or [AddComment()](../../office-api/usage-api/text-document-api/ApiDocument/Methods/AddComment.md).

The entire implementation of the `commentText` function:

``` ts
(function(){
    let func = new RegisteredFunction({
        "name": "commentText",
        "text": "Add Comment to Text",
        "description": "Adds a comment or footnote to explain or annotate the selected text. If no text is selected, works with the current paragraph.",
        "parameters": {
            "type": "object",
            "properties": {
                "prompt": {
                    "type": "string",
                    "description": "The instruction for what to explain or comment about the text."
                },
                "type": {
                    "type": "string",
                    "enum": ["comment", "footnote"],
                    "description": "Whether to add as a comment or as a footnote.",
                    "default": "comment"
                }
            },
            "required": ["prompt"]
        },
        "examples": [
            {
                "prompt": "Explain this text",
                "arguments": { "prompt": "Explain this text", "type": "comment" }
            },
            {
                "prompt": "Add a historical context as footnote",
                "arguments": { "prompt": "Add historical context", "type": "footnote" }
            },
            {
                "prompt": "Comment on the significance",
                "arguments": { "prompt": "Explain significance", "type": "comment" }
            }
        ]
    });

    func.call = async function(params) {
        let type = params.type;
        let isFootnote = "footnote" === type;

        let text = await Asc.Editor.callCommand(function(){
            let doc = Api.GetDocument();
            let range = doc.GetRangeBySelect();
            let text = range ? range.GetText() : "";
            if (!text)
            {
                text = doc.GetCurrentWord();
                doc.SelectCurrentWord();
            }
            return text;
        });

        let argPromt = params.prompt + ":\n" + text;

        let requestEngine = AI.Request.create(AI.ActionType.Chat);
        if (!requestEngine)
            return;

        let isSendedEndLongAction = false;
        async function checkEndAction() {
            if (!isSendedEndLongAction) {
                await Asc.Editor.callMethod("EndAction", ["Block", "AI (" + requestEngine.modelUI.name + ")"]);
                isSendedEndLongAction = true;
            }
        }

        await Asc.Editor.callMethod("StartAction", ["Block", "AI (" + requestEngine.modelUI.name + ")"]);
        await Asc.Editor.callMethod("StartAction", ["GroupActions"]);

        if (isFootnote)
        {
            let addFootnote = true;
            let result = await requestEngine.chatRequest(argPromt, false, async function(data) {
                if (!data)
                    return;

                await checkEndAction();
                Asc.scope.data = data;
                Asc.scope.model = requestEngine.modelUI.name;

                if (addFootnote)
                {
                    await Asc.Editor.callCommand(function(){
                        Api.GetDocument().AddFootnote();
                    });
                    addFootnote = false;
                }
                await Asc.Library.PasteText(data);
            });
        }
        else
        {
            let commentId = null;
            let result = await requestEngine.chatRequest(argPromt, false, async function(data) {
                if (!data)
                    return;

                await checkEndAction();
                Asc.scope.data = data;
                Asc.scope.model = requestEngine.modelUI.name;
                Asc.scope.commentId = commentId;

                commentId = await Asc.Editor.callCommand(function(){
                    let doc = Api.GetDocument();
                    let commentId = Asc.scope.commentId;

                    if (!commentId)
                    {
                        let range = doc.GetRangeBySelect();
                        if (!range)
                            return null;

                        let comment = range.AddComment(Asc.scope.data, Asc.scope.model, "uid" + Asc.scope.model);
                        if (!comment)
                            return null;
                        doc.ShowComment([comment.GetId()]);
                        return comment.GetId();
                    }

                    let comment = doc.GetCommentById(commentId);
                    if (!comment)
                        return commentId;

                    comment.SetText(comment.GetText() + Asc.scope.data);
                    return commentId;
                });
            });
        }

        await checkEndAction();
        await Asc.Editor.callMethod("EndAction", ["GroupActions"]);
    };

    return func;
})();
```

> To ensure the entire block of changes can be rolled back after the request is executed, we use [StartAction](../interacting-with-editors/api-by-editor-type/text-document-api/Methods/StartAction.md) and [EndAction](../interacting-with-editors/api-by-editor-type/text-document-api/Methods/EndAction.md) methods across the `commentText` function.

The AI agent functionality continues to evolve alongside the needs of today's digital world. Extend its capabilities by creating your own custom functions, tailored to your specific use cases.
