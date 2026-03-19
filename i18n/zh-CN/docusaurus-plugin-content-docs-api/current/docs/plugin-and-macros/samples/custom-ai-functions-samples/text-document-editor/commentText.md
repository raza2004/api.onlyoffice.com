# commentText

此函数用于为文本添加说明或注释。如果未指定文本或段落编号，则默认使用当前段落。可指定说明以注释还是脚注的形式添加。AI 将根据您的提示生成内容，并以所选格式插入。

## 提示词

- 解释此文本
- 为此文本添加脚注
- 注释此文本

## 函数注册 {#function-registration}

```ts
let func = new RegisteredFunction({
    name: "commentText",
    description:
        "Use this function if you asked to comment or explain anything. If text or paragraph number is not specified assume that we are working with the current paragraph. Specify whether the explanation should be added as a comment or as a footnote. The AI will generate the content based on your prompt and insert it in the chosen format.",
    parameters: {
        type: "object",
        properties: {
            type: {
                type: "string",
                description:
                    "Whether to add as a 'comment' or as a 'footnote' (default is 'comment')",
            },
        },
        required: ["type"],
    },
    examples: [
        {
            prompt: "Explain this text",
            arguments: { type: "comment" },
        },
        {
            prompt: "Add a footnote to this text",
            arguments: { type: "footnote" },
        },
        {
            prompt: "Comment this text",
            arguments: { type: "comment" },
        },
        {
            prompt: "Explain this text as a footnote",
            arguments: { type: "footnote" },
        },
        {
            prompt: "Explain selected text as a comment",
            arguments: { type: "comment" },
        },
    ],
});
```

### 参数

| 名称 | 类型   | 示例      | 描述                                                                                   |
| ---- | ------ | --------- | -------------------------------------------------------------------------------------- |
| type | string | "comment" | 指定是以 "comment"（注释）还是 "footnote"（脚注）形式添加说明。默认值为 "comment"。   |

## 函数执行 {#function-execution}

```ts
(function () {
    let func = new RegisteredFunction({
        name: "commentText",
        description:
            "Use this function if you asked to comment or explain anything. If text or paragraph number is not specified assume that we are working with the current paragraph. Specify whether the explanation should be added as a comment or as a footnote. The AI will generate the content based on your prompt and insert it in the chosen format.",
        parameters: {
            type: "object",
            properties: {
                type: {
                    type: "string",
                    description:
                        "Whether to add as a 'comment' or as a 'footnote' (default is 'comment')",
                },
            },
            required: ["type"],
        },
        examples: [
            {
                prompt: "Explain this text",
                arguments: { type: "comment" },
            },
            {
                prompt: "Add a footnote to this text",
                arguments: { type: "footnote" },
            },
            {
                prompt: "Comment this text",
                arguments: { type: "comment" },
            },
            {
                prompt: "Explain this text as a footnote",
                arguments: { type: "footnote" },
            },
            {
                prompt: "Explain selected text as a comment",
                arguments: { type: "comment" },
            },
        ],
    });

    func.call = async function (params) {
        let type = params.type;
        let isFootnote = "footnote" === type;

        let text = await Asc.Editor.callCommand(function () {
            let doc = Api.GetDocument();
            let range = doc.GetRangeBySelect();
            let text = range ? range.GetText() : "";
            if (!text) {
                text = doc.GetCurrentWord();
                doc.SelectCurrentWord();
            }

            return text;
        });

        let argPromt = params.prompt + ":\n" + text;

        let requestEngine = AI.Request.create(AI.ActionType.Chat);
        if (!requestEngine) return;

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

        await Asc.Editor.callMethod("StartAction", [
            "Block",
            "AI (" + requestEngine.modelUI.name + ")",
        ]);
        await Asc.Editor.callMethod("StartAction", ["GroupActions"]);

        if (isFootnote) {
            let addFootnote = true;
            let result = await requestEngine.chatRequest(
                argPromt,
                false,
                async function (data) {
                    if (!data) return;

                    await checkEndAction();
                    Asc.scope.data = data;
                    Asc.scope.model = requestEngine.modelUI.name;

                    if (addFootnote) {
                        await Asc.Editor.callCommand(function () {
                            Api.GetDocument().AddFootnote();
                        });
                        addFootnote = false;
                    }
                    await Asc.Library.PasteText(data);
                },
            );
        } else {
            let commentId = null;
            let result = await requestEngine.chatRequest(
                argPromt,
                false,
                async function (data) {
                    if (!data) return;

                    await checkEndAction();
                    Asc.scope.data = data;
                    Asc.scope.model = requestEngine.modelUI.name;
                    Asc.scope.commentId = commentId;

                    commentId = await Asc.Editor.callCommand(function () {
                        let doc = Api.GetDocument();

                        let commentId = Asc.scope.commentId;
                        if (!commentId) {
                            let range = doc.GetRangeBySelect();
                            if (!range) return null;

                            let comment = range.AddComment(
                                Asc.scope.data,
                                Asc.scope.model,
                                "uid" + Asc.scope.model,
                            );
                            if (!comment) return null;
                            doc.ShowComment([comment.GetId()]);
                            return comment.GetId();
                        }

                        let comment = doc.GetCommentById(commentId);
                        if (!comment) return commentId;

                        comment.SetText(comment.GetText() + scope.data);
                        return commentId;
                    });
                },
            );
        }

        await checkEndAction();
        await Asc.Editor.callMethod("EndAction", ["GroupActions"]);
    };

    return func;
})();
```

使用的方法：[GetDocument](/docs/office-api/usage-api/text-document-api/Api/Methods/GetDocument.md), [GetRangeBySelect](/docs/office-api/usage-api/text-document-api/ApiDocument/Methods/GetRangeBySelect.md), [GetText](/docs/office-api/usage-api/text-document-api/ApiRange/Methods/GetText.md), [AddComment](/docs/office-api/usage-api/text-document-api/ApiRange/Methods/AddComment.md), [GetCurrentWord](/docs/office-api/usage-api/text-document-api/ApiDocument/Methods/GetCurrentWord.md), [SelectCurrentWord](/docs/office-api/usage-api/text-document-api/ApiDocument/Methods/SelectCurrentWord.md), [AddFootnote](/docs/office-api/usage-api/text-document-api/ApiDocument/Methods/AddFootnote.md), [ShowComment](/docs/office-api/usage-api/text-document-api/ApiDocument/Methods/ShowComment.md), [GetCommentById](/docs/office-api/usage-api/text-document-api/ApiDocument/Methods/GetCommentById.md), [GetId](/docs/office-api/usage-api/text-document-api/ApiComment/Methods/GetId.md), [SetText](/docs/office-api/usage-api/text-document-api/ApiComment/Methods/SetText.md), [GetText](/docs/office-api/usage-api/text-document-api/ApiComment/Methods/GetText.md), [EndAction](/docs/plugin-and-macros/interacting-with-editors/api-by-editor-type/text-document-api/Methods/EndAction.md), [StartAction](/docs/plugin-and-macros/interacting-with-editors/api-by-editor-type/text-document-api/Methods/StartAction.md), [Asc.scope object](/docs/plugin-and-macros/interacting-with-editors/overview/how-to-call-commands.md#ascscope-object)

## 结果

<video className="light-video" controls style={{maxWidth: '848px'}}>

  <source src="/assets/images/plugins/functions-video/text-document-editor/commentText.webm" type="video/webm" />
  您的浏览器不支持 HTML5 视频。
</video>
<video className="dark-video" controls style={{maxWidth: '848px'}}>
  <source src="/assets/images/plugins/functions-video/text-document-editor/commentText.dark.webm" type="video/webm" />
  您的浏览器不支持 HTML5 视频。
</video>
