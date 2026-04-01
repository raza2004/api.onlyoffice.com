---
sidebar_position: 5
---

# Creating custom functions (beta)

Custom AI tools are functions that define the functionality of the AI agent. They specify:

- what request to send to the AI model;
- what manipulations to perform on the document.

Adding custom AI tools expands the AI agent's capabilities and allows adapting it to specific use cases. Whether working with documents, spreadsheets, or presentations, custom AI tools let you integrate AI-driven operations directly into your workflow and align the agent's behavior with your requirements.

You can find ready-to-use custom AI tools [here](../../samples/custom-ai-functions-samples/custom-ai-functions-samples.md) or create your own.

:::caution Current limitation
Adding a custom AI tool requires modifying the [AI plugin source code](https://github.com/ONLYOFFICE/onlyoffice.github.io/tree/master/sdkjs-plugins/content/ai) directly — you can then install the modified plugin via a custom [store link](#setup).
:::

## How it works {#how-it-works}

Custom AI tool calling in the ONLYOFFICE AI agent follows a flow similar to [function calling in LLM APIs](https://platform.openai.com/docs/guides/function-calling):

1. **Function registration.** Each function is registered with a name, parameter list, description, and usage examples. This metadata tells the AI model what the function does and when to invoke it.
2. **User prompt.** The user opens the AI agent and types a request.
3. **Function selection.** The AI model examines the prompt and the list of available functions, then decides which function to call and with what arguments.
4. **Execution.** The selected function runs: it sends a request to the AI model and applies the result to the document using the [Office API](../../../office-api/get-started/overview.md).

## Setup {#setup}

To add a custom AI tool and make it available in the AI agent:

1. Clone the [onlyoffice.github.io](https://github.com/ONLYOFFICE/onlyoffice.github.io) repository to your local machine.
2. Write your function in the helpers folder (`sdkjs-plugins/content/ai/.dev/helpers`). Depending on the editor type, place it in the `cell/`, `slide/`, or `word/` subfolder (see [Function registration](#registration) below).
3. Update the current version of the AI plugin in `config.json` to avoid caching issues (for example, `3.0.3` → `3.0.4`).
4. Run the `helpers.py` file.
5. Select all plugin files in the `ai` folder (`sdkjs-plugins/content/ai`), zip them, and rename the archive to `ai.plugin`.
6. Place the file back into `sdkjs-plugins/content/ai/deploy`.
7. Push the changes.
8. Build your GitHub Pages site from this repository (see the [GitHub Pages documentation](https://docs.github.com/en/pages)).
9. Prepare a link to your custom store by appending `/store/index.html` to your GitHub Pages URL: `https://YOUR-USERNAME.github.io/onlyoffice.github.io/store/index.html`.
10. Go to **Plugins > Plugin Manager**.
11. Click the **Store** icon `(</>)` in the top-right corner of the Plugin Manager and enter your custom store URL.
12. Update the AI plugin.

## Example: the commentText function {#example}

The `commentText` function allows adding AI-generated comments directly to the document. Here's how it works:

1. Make sure the AI plugin is installed and [configured correctly](../getting-started/installing-ai-plugin.md).
2. Select a word or sentence to leave a comment on.
3. Open the AI agent dialog box (**Ctrl + /**).
4. Type an instruction for the AI agent, for example: `Explain this text` or `Add a footnote to this text`.
5. Press **Enter**.

![commentText execution](/assets/images/plugins/comment-text-function.png#gh-light-mode-only)![commentText execution](/assets/images/plugins/comment-text-function.dark.png#gh-dark-mode-only)

The AI agent runs the `commentText` function and inserts a comment into the document.

![commentText result](/assets/images/plugins/comment-text-result.png#gh-light-mode-only)![commentText result](/assets/images/plugins/comment-text-result.dark.png#gh-dark-mode-only)

## How to create custom AI tools {#creating-ai-tools}

Creating a custom AI tool involves two phases:

- [Function registration](#registration) — registers the function and its metadata with the agent.
- [Function execution](#execution) — implements the core logic: sending a request to the AI model and writing the result to the document.

### Function registration {#registration}

Use the `RegisteredFunction` object to register a function. Pass a configuration object with the tool's metadata.

#### Parameters {#parameters}

| Name | Type | Example | Description |
| --- | --- | --- | --- |
| `name` | `string` | `"commentText"` | The function name used by the agent to identify and call the tool. |
| `parameters` | `object` | `{ type: "object", properties: { prompt: { type: "string" } }, required: ["prompt"] }` | A [JSON Schema](https://json-schema.org/) object describing the arguments the agent will pass. |
| `examples` | `object[]` | `[{ prompt: "Explain this text", arguments: { prompt: "Explain this text", type: "comment" } }]` | Example invocations that teach the agent when and how to call the function. |
| `description` | `string` | `"Adds a comment or footnote to explain or annotate the selected text."` | Tells the agent what the function does and when to use it. |

The `RegisteredFunction` object is defined in [helperFuncs.js](https://github.com/ONLYOFFICE/onlyoffice.github.io/blob/master/sdkjs-plugins/content/ai/scripts/helpers/helperFuncs.js).

### Function execution {#execution}

The full `commentText` function with inline comments explaining each step:

```js
(function () {
  // Registers the "commentText" tool with the AI agent.
  // The metadata below (name/description/parameters/examples) is used by the model
  // to decide when and how to call this tool.
  let func = new RegisteredFunction({
    name: "commentText",
    description:
      "Adds a comment or footnote to explain or annotate the selected text. If no text is selected, works with the current paragraph.",
    parameters: {
      type: "object",
      properties: {
        // The instruction for the AI model (e.g., "Explain this text").
        prompt: {
          type: "string",
          description: "The instruction for what to explain or comment about the text.",
        },
        // Determines how the response will be inserted into the document.
        // Default behavior is a comment.
        type: {
          type: "string",
          enum: ["comment", "footnote"],
          description: "Whether to add as a comment or as a footnote.",
          default: "comment",
        },
      },
      required: ["prompt"],
    },
    // Example calls that teach the model the correct JSON arguments to provide.
    examples: [
      {
        prompt: "Explain this text",
        arguments: { prompt: "Explain this text", type: "comment" },
      },
      {
        prompt: "Add a historical context as footnote",
        arguments: { prompt: "Add historical context", type: "footnote" },
      },
      {
        prompt: "Comment on the significance",
        arguments: { prompt: "Explain significance", type: "comment" },
      },
    ],
  });

  // The actual logic that runs when the AI agent calls "commentText".
  func.call = async function (params) {
    // Decide whether to insert as a comment or as a footnote.
    let type = params.type;
    let isFootnote = "footnote" === type;

    // 1) Retrieve the selected text (or fall back to the current word if nothing is selected).
    // Asc.Editor.callCommand() executes inside the editor context, where Office API calls are available.
    let text = await Asc.Editor.callCommand(function () {
      let doc = Api.GetDocument();
      let range = doc.GetRangeBySelect();
      let text = range ? range.GetText() : "";

      if (!text) {
        // If nothing is selected, use the current word and select it.
        text = doc.GetCurrentWord();
        doc.SelectCurrentWord();
      }

      return text;
    });

    // 2) Construct the prompt sent to the AI model by combining the user instruction and the selected text.
    let argPrompt = params.prompt + ":\n" + text;

    // 3) Initialize a request engine for communicating with the AI model.
    let requestEngine = AI.Request.create(AI.ActionType.Chat);
    if (!requestEngine) return;

    // Ensures EndAction is called only once, even when the callback fires multiple times during streaming.
    let isSendedEndLongAction = false;
    async function checkEndAction() {
      if (!isSendedEndLongAction) {
        await Asc.Editor.callMethod("EndAction", [
          "Block",
          "AI (" + requestEngine.modelUI.name + ")",
        ]);
        isSendedEndLongAction = true;
      }
    }

    // Start editor actions so the entire operation can be undone/redone as a single block.
    await Asc.Editor.callMethod("StartAction", [
      "Block",
      "AI (" + requestEngine.modelUI.name + ")",
    ]);
    await Asc.Editor.callMethod("StartAction", ["GroupActions"]);

    // 4) Send the request to the AI model and insert the response as a footnote or comment.
    if (isFootnote) {
      let addFootnote = true;

      await requestEngine.chatRequest(argPrompt, false, async function (data) {
        if (!data) return;

        // End the action block as soon as the first data chunk arrives (streaming-safe).
        await checkEndAction();

        // Pass data and model info into the editor scope for use inside callCommand().
        Asc.scope.data = data;
        Asc.scope.model = requestEngine.modelUI.name;

        // Create the footnote once (on the first chunk), then keep inserting text into it.
        if (addFootnote) {
          await Asc.Editor.callCommand(function () {
            Api.GetDocument().AddFootnote();
          });
          addFootnote = false;
        }

        // Insert AI-generated text at the current cursor position (inside the new footnote).
        await Asc.Library.PasteText(data);
      });
    } else {
      let commentId = null;

      await requestEngine.chatRequest(argPrompt, false, async function (data) {
        if (!data) return;

        // End the action block as soon as the first data chunk arrives (streaming-safe).
        await checkEndAction();

        // Store response data, model name, and comment ID in scope so callCommand() can access them.
        Asc.scope.data = data;
        Asc.scope.model = requestEngine.modelUI.name;
        Asc.scope.commentId = commentId;

        // Create the comment once (on the first chunk), then append additional chunks to it.
        commentId = await Asc.Editor.callCommand(function () {
          let doc = Api.GetDocument();

          let commentId = Asc.scope.commentId;
          if (!commentId) {
            // Add a new comment to the selected range.
            let range = doc.GetRangeBySelect();
            if (!range) return null;

            let comment = range.AddComment(
              Asc.scope.data,
              Asc.scope.model,
              "uid" + Asc.scope.model
            );
            if (!comment) return null;

            // Open the comment in the UI and return its ID for future appends.
            doc.ShowComment([comment.GetId()]);
            return comment.GetId();
          }

          // Append new streamed text chunks to the existing comment.
          let comment = doc.GetCommentById(commentId);
          if (!comment) return commentId;

          comment.SetText(comment.GetText() + Asc.scope.data);
          return commentId;
        });
      });
    }

    // Ensure actions are closed even if the response completed in a single chunk.
    await checkEndAction();
    await Asc.Editor.callMethod("EndAction", ["GroupActions"]);
  };

  return func;
})();
```

> The [StartAction](../../interacting-with-editors/api-by-editor-type/text-document-api/Methods/StartAction.md) and [EndAction](../../interacting-with-editors/api-by-editor-type/text-document-api/Methods/EndAction.md) pair ensures the entire operation — including all streaming chunks — can be rolled back with a single **Ctrl + Z**.

The AI agent functionality continues to evolve. Extend its capabilities by creating your own custom tools tailored to your specific use cases.
