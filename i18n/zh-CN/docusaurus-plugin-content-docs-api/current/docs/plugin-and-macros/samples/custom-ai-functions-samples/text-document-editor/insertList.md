# insertList

该函数在当前光标位置或文档开头/结尾创建简单的编号或项目符号列表。

## 提示词

- 创建一个包含以下项目的编号列表："第一项"、"第二项"、"第三项"
- 在文档开头插入一个项目符号列表，包含以下项目："任务 1"、"任务 2"、"任务 3"

## 函数注册 {#function-registration}

```ts
let func = new RegisteredFunction({
    name: "insertList",
    description: "Use this function to create simple numbered or bulleted lists at the current cursor position or at the start/end of the document.",
    parameters: {
        type: "object",
        properties: {
            items: { type: "array", description: "array of strings representing list items", items: { type: "string" } },
            listType: { type: "string", description: "'numbered' for numbered list, 'bulleted' for bulleted list (default is 'bulleted')" },
            position: { type: "string", description: "where to insert the list - 'current', 'start', or 'end' (default is 'current')" }
        },
        required: ["items"]
    }
});
```

### 参数

| 名称     | 类型   | 示例                                        | 描述                                                                          |
|----------|--------|---------------------------------------------|-------------------------------------------------------------------------------|
| items    | array  | ["First item", "Second item", "Third item"] | 表示列表项的字符串数组。                                                      |
| listType | string | "numbered"                                  | 'numbered' 表示编号列表，'bulleted' 表示项目符号列表（默认为 'bulleted'）。   |
| position | string | "current"                                   | 列表插入位置——'current'（当前）、'start'（开头）或 'end'（结尾）（默认为 'current'）。 |

## 函数执行 {#function-execution}

```ts
func.call = async function(params) {
    Asc.scope.items = params.items || ["Item 1", "Item 2", "Item 3"];
    Asc.scope.listType = params.listType || "bulleted";
    Asc.scope.position = params.position || "current";

    await Asc.Editor.callCommand(function() {
        let doc = Api.GetDocument();

        if (Asc.scope.position === "start") {
            doc.MoveCursorToStart();
        } else if (Asc.scope.position === "end") {
            doc.MoveCursorToEnd();
            let newParagraph = Api.CreateParagraph();
            doc.InsertContent([newParagraph]);
        } else if (Asc.scope.position === "current") {
            let newParagraph = Api.CreateParagraph();
            doc.InsertContent([newParagraph]);
        }

        let paragraphs = [];
        let numbering;

        if (Asc.scope.listType === "numbered") {
            numbering = doc.CreateNumbering("numbered");
        } else {
            numbering = doc.CreateNumbering("bullet");
        }

        let numLvl = numbering.GetLevel(0);

        for (let i = 0; i < Asc.scope.items.length; i++) {
            let item = Asc.scope.items[i];
            let paragraph = Api.CreateParagraph();
            paragraph.AddText(item);
            paragraph.SetNumbering(numLvl);
            paragraph.SetContextualSpacing(true);
            paragraphs.push(paragraph);
        }

        doc.InsertContent(paragraphs);
    });
};

return func;
```

使用的方法：[GetDocument](/docs/office-api/usage-api/text-document-api/Api/Methods/GetDocument.md), [MoveCursorToStart](/docs/office-api/usage-api/text-document-api/ApiDocument/Methods/MoveCursorToStart.md), [MoveCursorToEnd](/docs/office-api/usage-api/text-document-api/ApiDocument/Methods/MoveCursorToEnd.md), [CreateParagraph](/docs/office-api/usage-api/text-document-api/Api/Methods/CreateParagraph.md), [InsertContent](/docs/office-api/usage-api/text-document-api/ApiDocument/Methods/InsertContent.md), [CreateNumbering](/docs/office-api/usage-api/text-document-api/ApiDocument/Methods/CreateNumbering.md), [GetLevel](/docs/office-api/usage-api/text-document-api/ApiNumbering/Methods/GetLevel.md), [AddText](/docs/office-api/usage-api/text-document-api/ApiParagraph/Methods/AddText.md), [SetNumbering](/docs/office-api/usage-api/text-document-api/ApiParagraph/Methods/SetNumbering.md), [SetContextualSpacing](/docs/office-api/usage-api/text-document-api/ApiParagraph/Methods/SetContextualSpacing.md), [Asc.scope 对象](/docs/plugin-and-macros/interacting-with-editors/overview/how-to-call-commands.md#ascscope-object)

## 结果

<video className="light-video" controls style={{maxWidth: '848px'}}>
  <source src="/assets/images/plugins/functions-video/text-document-editor/insertList.webm" type="video/webm" />
  您的浏览器不支持 HTML5 视频。
</video>
<video className="dark-video" controls style={{maxWidth: '848px'}}>
  <source src="/assets/images/plugins/functions-video/text-document-editor/insertList.dark.webm" type="video/webm" />
  您的浏览器不支持 HTML5 视频。
</video>
