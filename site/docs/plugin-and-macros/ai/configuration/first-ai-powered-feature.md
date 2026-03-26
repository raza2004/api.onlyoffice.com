---
sidebar_position: -2
title: "First AI-powered feature"
---

# First AI-powered feature {#first-ai-powered-feature}

This page walks you through building a complete, working AI feature end to end: a tool that reads the text the user has selected in the document editor and asks the AI to suggest a highlight color for it.

## Prerequisites {#prerequisites}

Before you start:

- The AI plugin is installed and at least one provider is configured. See [Prerequisites](./prerequisites.md).
- You have a basic understanding of the ONLYOFFICE plugin API. See [AI integration overview](./ai-integration-overview.md).

## Step 1: Set up the plugin structure {#step-1-set-up-the-plugin-structure}

A plugin consists of at minimum a `config.json` manifest and one or more script files.

Create a working directory for your plugin and add the following stub `config.json`:

```json
{
  "name": "Highlight suggester",
  "guid": "YOUR-PLUGIN-GUID-HERE",
  "baseUrl": "",
  "variations": [
    {
      "description": "Suggest a highlight color for selected text",
      "url": "index.html",
      "icons": ["resources/img/icon.png"],
      "isViewer": false,
      "EditorsSupport": ["word"],
      "isVisibleOnlyCallback": false,
      "screens": [],
      "initDataType": "none",
      "initData": "",
      "isUpdateOleOnResize": false,
      "buttons": []
    }
  ]
}
```

Create an `index.html` that loads your script file:

```html
<!DOCTYPE html>
<html>
  <head><meta charset="utf-8" /></head>
  <body>
    <script src="plugin.js"></script>
  </body>
</html>
```

Create an empty `plugin.js` — you will fill it in the steps below.

## Step 2: Register a tool {#step-2-register-a-tool}

Open `plugin.js` and register a tool called `suggestHighlightColor` in the `WORD_FUNCTIONS` map. The tool description is what the AI agent reads when deciding whether to invoke the tool, so make it precise.

```ts
(function () {
  "use strict";

  // Guard: only run inside the ONLYOFFICE plugin runtime.
  if (typeof window.Asc === "undefined") return;

  const tool = {
    name: "suggestHighlightColor",
    description:
      "Reads the currently selected text and asks the AI to suggest " +
      "a suitable highlight color. Returns the color name as a string.",
    parameters: {
      type: "object",
      properties: {},
      required: [],
    },
    func: {
      call: suggestHighlightColor,
    },
  };

  // Register the tool in the word-editor function map.
  if (window.WORD_FUNCTIONS) {
    window.WORD_FUNCTIONS[tool.name] = tool;
  }
})();
```

> `WORD_FUNCTIONS`, `CELL_FUNCTIONS`, and `SLIDE_FUNCTIONS` are global maps maintained by the AI plugin. Your tool is available to the agent as soon as it is added to the appropriate map.

## Step 3: Implement `func.call` {#step-3-implement-func-call}

Add the `suggestHighlightColor` function to `plugin.js`, above the IIFE from Step 2:

```ts
async function suggestHighlightColor() {
  // 1. Read the selected text from the editor context.
  Asc.scope.selectedText = "";

  window.Asc.plugin.callCommand(function () {
    var oDoc = window.Asc.plugin.info.doc;
    Asc.scope.selectedText = oDoc.GetSelectedText() || "";
  }, false);

  var selected = Asc.scope.selectedText;

  if (!selected || selected.trim() === "") {
    window.Asc.plugin.EndAction(
      window.Asc.c_oAscAsyncActionType.Information,
      "No text selected."
    );
    return "No text was selected. Please select some text first.";
  }

  // 2. Ask the AI to suggest a highlight color.
  var prompt =
    "Given the following text, suggest a single highlight color name " +
    "(e.g. yellow, cyan, green, pink) that would help a reader notice it. " +
    "Reply with only the color name, nothing else.\n\nText: " +
    selected;

  var response = await AI.Request.create({
    messages: [{ role: "user", content: prompt }],
  });

  var colorName = response.trim().toLowerCase();

  // 3. Apply the highlight color inside the editor context.
  Asc.scope.colorName = colorName;

  window.Asc.plugin.callCommand(function () {
    var oDoc = window.Asc.plugin.info.doc;
    var oSel = oDoc.GetSelection();
    if (oSel) {
      var oTextPr = oSel.GetTextPr();
      oTextPr.SetHighlight(Asc.scope.colorName);
      oSel.SetTextPr(oTextPr);
    }
  }, false);

  window.Asc.plugin.EndAction(
    window.Asc.c_oAscAsyncActionType.Information,
    "Highlight applied."
  );

  return "Suggested color: " + colorName;
}
```

## Step 4: Test the tool {#step-4-test-the-tool}

1. Load the plugin in ONLYOFFICE Docs or Desktop Editors using the plugin manager or by placing it in the plugins directory.
2. Open a document in the document editor.
3. Select a sentence or a few words.
4. Press **Ctrl + /** to open the AI agent input.
5. Type a prompt such as `suggest a highlight color for the selected text` and press **Enter**.
6. The agent matches the prompt to `suggestHighlightColor`, invokes `func.call`, and the selected text receives a highlight color.

## Full working example {#full-working-example}

The following is the complete `plugin.js` with both the function definition and the registration IIFE together:

```ts
async function suggestHighlightColor() {
  Asc.scope.selectedText = "";

  window.Asc.plugin.callCommand(function () {
    var oDoc = window.Asc.plugin.info.doc;
    Asc.scope.selectedText = oDoc.GetSelectedText() || "";
  }, false);

  var selected = Asc.scope.selectedText;

  if (!selected || selected.trim() === "") {
    window.Asc.plugin.EndAction(
      window.Asc.c_oAscAsyncActionType.Information,
      "No text selected."
    );
    return "No text was selected. Please select some text first.";
  }

  var prompt =
    "Given the following text, suggest a single highlight color name " +
    "(e.g. yellow, cyan, green, pink) that would help a reader notice it. " +
    "Reply with only the color name, nothing else.\n\nText: " +
    selected;

  var response = await AI.Request.create({
    messages: [{ role: "user", content: prompt }],
  });

  var colorName = response.trim().toLowerCase();

  Asc.scope.colorName = colorName;

  window.Asc.plugin.callCommand(function () {
    var oDoc = window.Asc.plugin.info.doc;
    var oSel = oDoc.GetSelection();
    if (oSel) {
      var oTextPr = oSel.GetTextPr();
      oTextPr.SetHighlight(Asc.scope.colorName);
      oSel.SetTextPr(oTextPr);
    }
  }, false);

  window.Asc.plugin.EndAction(
    window.Asc.c_oAscAsyncActionType.Information,
    "Highlight applied."
  );

  return "Suggested color: " + colorName;
}

(function () {
  "use strict";

  if (typeof window.Asc === "undefined") return;

  var tool = {
    name: "suggestHighlightColor",
    description:
      "Reads the currently selected text and asks the AI to suggest " +
      "a suitable highlight color. Returns the color name as a string.",
    parameters: {
      type: "object",
      properties: {},
      required: [],
    },
    func: {
      call: suggestHighlightColor,
    },
  };

  if (window.WORD_FUNCTIONS) {
    window.WORD_FUNCTIONS[tool.name] = tool;
  }
})();
```

## Common first-time mistakes {#common-first-time-mistakes}

**Error name:** Tool not registered — missing function map assignment

:::warning[Wrong]
```ts
(function() {
  const func = {};
  func.call = async function(params) { /* ... */ };
  // func is never added to WORD_FUNCTIONS
})();
```
:::

:::tip[Correct]
```ts
(function() {
  const func = {};
  func.call = async function(params) { /* ... */ };
  if (window.WORD_FUNCTIONS) {
    window.WORD_FUNCTIONS[func.name] = func;
  }
})();
```
:::

Error output: The tool is never registered; the AI agent cannot find or invoke it.

---

**Error name:** Local variable used inside `callCommand` without `Asc.scope`

:::warning[Wrong]
```ts
const text = getSelectedText();
window.Asc.plugin.callCommand(function() {
  processText(text); // ReferenceError — text is not in scope
}, false);
```
:::

:::tip[Correct]
```ts
Asc.scope.text = getSelectedText();
window.Asc.plugin.callCommand(function() {
  processText(Asc.scope.text);
}, false);
```
:::

Error output: `ReferenceError` or `undefined` values inside the command — plugin-context variables are not accessible inside `callCommand` callbacks.

---

**Error name:** `EndAction` not called after async operation

:::warning[Wrong]
```ts
func.call = async function(params) {
  await chatRequest(prompt, null, (chunk) => { /* handle */ });
  // EndAction is never called
};
```
:::

:::tip[Correct]
```ts
func.call = async function(params) {
  try {
    await chatRequest(prompt, null, (chunk) => { /* handle */ });
  } finally {
    window.Asc.plugin.EndAction();
  }
};
```
:::

Error output: The editor spinner runs indefinitely and the UI appears frozen after the tool completes.

---

**Error name:** Awaiting `callCommand` directly

:::warning[Wrong]
```ts
const result = await window.Asc.plugin.callCommand(function() {
  Asc.scope.result = Api.GetDocument().GetAllParagraphs();
}, false);
// result is undefined — callCommand is not a Promise
```
:::

:::tip[Correct]
```ts
window.Asc.plugin.callCommand(function() {
  Asc.scope.result = Api.GetDocument().GetAllParagraphs();
}, false);
const result = Asc.scope.result; // read after synchronous call returns
```
:::

Error output: `await` has no effect on `callCommand` — the return value is always `undefined` and the result is lost.
