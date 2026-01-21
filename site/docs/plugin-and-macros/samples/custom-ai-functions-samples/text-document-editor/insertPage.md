# insertPage

This function inserts a new page into the document at a specified location.

## Prompts

- Insert a page at the current location
- Add a page at the end of the document
- Add a page at the start of the document

## Function registration {#function-registration}

```ts
(function () {
  let func = new RegisteredFunction({
    name: "insertPage",
    description:
      "Inserts a blank page at the specified location in the document.",
    parameters: {
      type: "object",
      properties: {
        location: {
          type: "string",
          enum: ["current", "start", "end"],
          description:
            "Where to insert the new page ('current', 'start', or 'end').",
          default: "current",
        },
      },
      required: ["location"],
    },
    examples: [
      {
        prompt: "Insert a blank page at current position",
        arguments: { location: "current" },
      },
      {
        prompt: "Add a new page at the end",
        arguments: { location: "end" },
      },
      {
        prompt: "Add a page at the beginning",
        arguments: { location: "start" },
      },
    ],
    returns: {
      type: "object",
      description:
        "An object indicating whether the page was successfully inserted.",
      properties: {
        isApply: {
          type: "boolean",
          description:
            "Indicates whether the blank page was successfully inserted at the specified location.",
        },
      },
      required: ["isApply"],
    },
  });

  return func;
})();
```

### Parameters

| Name     | Type   | Example   | Description                                                          |
| -------- | ------ | --------- | -------------------------------------------------------------------- |
| location | string | "current" | Specifies where to insert a new page ("current", "start", or "end"). |

## Function execution {#function-execution}

```ts
func.call = async function (params) {
  Asc.scope.location = params.location;

  await Asc.Editor.callCommand(function () {
    let doc = Api.GetDocument();
    if ("start" === Asc.scope.location) doc.MoveCursorToStart();
    else if ("end" === Asc.scope.location) doc.MoveCursorToEnd();

    Api.GetDocument().InsertBlankPage();
  });
};
```

Methods used: [GetDocument](/docs/office-api/usage-api/text-document-api/Api/Methods/GetDocument.md), [MoveCursorToStart](/docs/office-api/usage-api/text-document-api/ApiDocument/Methods/MoveCursorToStart.md), [MoveCursorToEnd](/docs/office-api/usage-api/text-document-api/ApiDocument/Methods/MoveCursorToEnd.md), [InsertBlankPage](/docs/office-api/usage-api/text-document-api/ApiDocument/Methods/InsertBlankPage.md), [Asc.scope object](/docs/plugin-and-macros/interacting-with-editors/overview/how-to-call-commands.md#ascscope-object)

## Result

<video className="light-video" controls style={{maxWidth: '848px'}}>

  <source src="/assets/images/plugins/functions-video/text-document-editor/insertPage.webm" type="video/webm" />
  Your browser does not support HTML5 video.
</video>
<video className="dark-video" controls style={{maxWidth: '848px'}}>
  <source src="/assets/images/plugins/functions-video/text-document-editor/insertPage.dark.webm" type="video/webm" />
  Your browser does not support HTML5 video.
</video>
