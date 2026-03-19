# changeParagraphStyle

此函数用于修改指定段落的视觉样式。

## 提示词

- 将第 2 段的样式更改为标题 1

## 函数注册 {#function-registration}

```ts
let func = new RegisteredFunction({
    name: "changeParagraphStyle",
    description: "Modifies the visual style of the specified paragraph.",
    parameters: {
        type: "object",
        properties: {
            parNumber: {
                type: "number",
                description: "The paragraph number to apply style changes to.",
            },
            style: {
                type: "string",
                description:
                    "The style name to apply to the paragraph (e.g., 'Heading 1').",
            },
        },
        required: ["parNumber", "style"],
    },
    examples: [
        {
            prompt: "Change the style of paragraph 3 to Heading 1",
            arguments: { parNumber: 3, style: "Heading 1" },
        },
        {
            prompt: "Change the style of paragraph 2 to Heading 1",
            arguments: { parNumber: 2, style: "Heading 1" },
        },
    ],
});
```

### 参数

| 名称      | 类型   | 示例        | 描述                                   |
|-----------|--------|-------------|----------------------------------------|
| parNumber | number | 3           | 要应用样式更改的段落编号。             |
| style     | string | "Heading 1" | 要应用于段落的样式名称。               |

## 函数执行 {#function-execution}

```ts
func.call = async function (params) {
        Asc.scope.parNumber = params.parNumber;
        Asc.scope.styleName = params.style;
        await Asc.Editor.callCommand(function () {
            let doc = Api.GetDocument();
            let par = doc.GetElement(Asc.scope.parNumber - 1);
            if (!par) return;

            let style = doc.GetStyle(Asc.scope.styleName);
            par.SetStyle(style);
    });
};
```

使用的方法：[GetDocument](/docs/office-api/usage-api/text-document-api/Api/Methods/GetDocument.md), [GetElement](/docs/office-api/usage-api/text-document-api/ApiDocument/Methods/GetElement.md), [GetStyle](/docs/office-api/usage-api/text-document-api/ApiDocument/Methods/GetStyle.md), [SetStyle](/docs/office-api/usage-api/text-document-api/ApiParagraph/Methods/SetStyle.md), [Asc.scope object](/docs/plugin-and-macros/interacting-with-editors/overview/how-to-call-commands.md#ascscope-object)

## 结果

<video className="light-video" controls style={{maxWidth: '848px'}}>
  <source src="/assets/images/plugins/functions-video/text-document-editor/changeParagraphStyle.webm" type="video/webm" />
  您的浏览器不支持 HTML5 视频。
</video>
<video className="dark-video" controls style={{maxWidth: '848px'}}>
  <source src="/assets/images/plugins/functions-video/text-document-editor/changeParagraphStyle.dark.webm" type="video/webm" />
  您的浏览器不支持 HTML5 视频。
</video>
