# insertPage

此函数在指定位置向文档插入新页面。

## 提示词

- 在当前位置插入一页
- 在文档末尾添加一页
- 在文档开头添加一页

## 函数注册 {#function-registration}

```ts
let func = new RegisteredFunction({
    name: "insertPage",
    description:
        "Inserts a new page into the document at a specified location.",
    parameters: {
        type: "object",
        properties: {
            location: {
                type: "string",
                description:
                    "Where to insert the new page ('current', 'start', or 'end')",
            },
        },
        required: ["location"],
    },
    examples: [
        {
            prompt: "Insert a page at the current location",
            arguments: { location: "current" },
        },
        {
            prompt: "Add a page at the end of the document",
            arguments: { location: "end" },
        },
        {
            prompt: "Add a page at the start of the document",
            arguments: { location: "start" },
        },
    ],
});

```

### 参数

| 名称     | 类型   | 示例      | 描述                                                              |
| -------- | ------ | --------- | ----------------------------------------------------------------- |
| location | string | "current" | 指定新页面的插入位置（"current"、"start" 或 "end"）。            |

## 函数执行 {#function-execution}

```ts
(function () {
    let func = new RegisteredFunction({
        name: "insertPage",
        description:
            "Inserts a new page into the document at a specified location.",
        parameters: {
            type: "object",
            properties: {
                location: {
                    type: "string",
                    description:
                        "Where to insert the new page ('current', 'start', or 'end')",
                },
            },
            required: ["location"],
        },
        examples: [
            {
                prompt: "Insert a page at the current location",
                arguments: { location: "current" },
            },
            {
                prompt: "Add a page at the end of the document",
                arguments: { location: "end" },
            },
            {
                prompt: "Add a page at the start of the document",
                arguments: { location: "start" },
            },
        ],
    });

    func.call = async function (params) {
        Asc.scope.location = params.location;

        await Asc.Editor.callCommand(function () {
            let doc = Api.GetDocument();
            if ("start" === Asc.scope.location) doc.MoveCursorToStart();
            else if ("end" === Asc.scope.location) doc.MoveCursorToEnd();

            Api.GetDocument().InsertBlankPage();
        });
    };
    return func;
})();
```

使用的方法：[GetDocument](/docs/office-api/usage-api/text-document-api/Api/Methods/GetDocument.md), [MoveCursorToStart](/docs/office-api/usage-api/text-document-api/ApiDocument/Methods/MoveCursorToStart.md), [MoveCursorToEnd](/docs/office-api/usage-api/text-document-api/ApiDocument/Methods/MoveCursorToEnd.md), [InsertBlankPage](/docs/office-api/usage-api/text-document-api/ApiDocument/Methods/InsertBlankPage.md), [Asc.scope object](/docs/plugin-and-macros/interacting-with-editors/overview/how-to-call-commands.md#ascscope-object)

## 结果

<video className="light-video" controls style={{maxWidth: '848px'}}>

  <source src="/assets/images/plugins/functions-video/text-document-editor/insertPage.webm" type="video/webm" />
  您的浏览器不支持 HTML5 视频。
</video>
<video className="dark-video" controls style={{maxWidth: '848px'}}>
  <source src="/assets/images/plugins/functions-video/text-document-editor/insertPage.dark.webm" type="video/webm" />
  您的浏览器不支持 HTML5 视频。
</video>
