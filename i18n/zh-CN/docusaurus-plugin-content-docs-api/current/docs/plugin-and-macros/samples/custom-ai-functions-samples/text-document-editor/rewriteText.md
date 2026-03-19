# rewriteText

此函数用于改写或替换文本。如果未指定文本或段落编号，则默认使用当前段落。

## 提示词

- 改写
- 重新表述句子
- 使文本更具情感色彩
- 重新措辞
- 以正式风格改写

## 函数注册 {#function-registration}

```ts
let func = new RegisteredFunction({
    name: "rewriteText",
    description:
        "Use this function when you asked to rewrite or replace some text. If text or paragraph number is not specified assume that we are working with the current paragraph.",
    parameters: {
        type: "object",
        properties: {
            parNumber: {
                type: "number",
                description: "The paragraph number to change",
            },
            prompt: {
                type: "string",
                description: "Instructions on how to change the text",
            },
            showDifference: {
                type: "boolean",
                description:
                    "Whether to show the difference between the original and new text, or just replace it",
            },
            type: {
                type: "string",
                description:
                    "Which part of the text to be rewritten (e.g., 'sentence' or 'paragraph')",
            },
        },
        required: [],
    },
    examples: [
        {
            prompt: "Rewrite",
            arguments: { type: "paragraph" },
        },
        {
            prompt: "Rephrase sentence",
            arguments: { type: "sentence" },
        },
        {
            prompt: "Make the text more emotional",
            arguments: { parNumber: 2, type: "paragraph" },
        },
        {
            prompt: "Rewrite in official style",
            arguments: { type: "paragraph" },
        },
        {
            prompt: "Rewrite the first paragraph",
            arguments: { parNumber: 1, type: "paragraph" },
        },
        {
            prompt: "Rewrite the current paragraph to be more official",
            arguments: { type: "paragraph" },
        },
    ],
});
```

### 参数

| 名称           | 类型    | 示例        | 描述                                                                   |
|----------------|---------|-------------|------------------------------------------------------------------------|
| parNumber      | number  | 1           | 要更改的段落编号。                                                     |
| prompt         | string  | "Rewrite"   | 关于如何更改文本的指令。                                               |
| showDifference | boolean | true        | 指定是否显示原文与新文本之间的差异，或直接替换。                       |
| type           | string  | "paragraph" | 要改写的文本部分（例如 "sentence" 或 "paragraph"）。                   |

## 函数执行 {#function-execution}

```ts
(function () {
    let func = new RegisteredFunction({
        name: "rewriteText",
        description:
            "Use this function when you asked to rewrite or replace some text. If text or paragraph number is not specified assume that we are working with the current paragraph.",
        parameters: {
            type: "object",
            properties: {
                parNumber: {
                    type: "number",
                    description: "The paragraph number to change",
                },
                prompt: {
                    type: "string",
                    description: "Instructions on how to change the text",
                },
                showDifference: {
                    type: "boolean",
                    description:
                        "Whether to show the difference between the original and new text, or just replace it",
                },
                type: {
                    type: "string",
                    description:
                        "Which part of the text to be rewritten (e.g., 'sentence' or 'paragraph')",
                },
            },
            required: [],
        },
        examples: [
            {
                prompt: "Rewrite",
                arguments: { type: "paragraph" },
            },
            {
                prompt: "Rephrase sentence",
                arguments: { type: "sentence" },
            },
            {
                prompt: "Make the text more emotional",
                arguments: { parNumber: 2, type: "paragraph" },
            },
            {
                prompt: "Rewrite in official style",
                arguments: { type: "paragraph" },
            },
            {
                prompt: "Rewrite the first paragraph",
                arguments: { parNumber: 1, type: "paragraph" },
            },
            {
                prompt: "Rewrite the current paragraph to be more official",
                arguments: { type: "paragraph" },
            },
        ],
    });

    func.call = async function (params) {
        let text = "";
        if ("paragraph" === params.type) {
            Asc.scope.parNumber = params.parNumber;
            text = await Asc.Editor.callCommand(function () {
                let doc = Api.GetDocument();
                let par =
                    undefined === Asc.scope.parNumber
                        ? doc.GetCurrentParagraph()
                        : doc.GetElement(Asc.scope.parNumber - 1);
                if (!par) return "";
                par.Select();
                return par.GetText();
            });
        } else // if ("sentence" === params.type)
        {
            text = await Asc.Editor.callCommand(function () {
                return Api.GetDocument().GetCurrentSentence();
            });
        }

        let argPromt =
            params.prompt +
            ":\n" +
            text +
            "\n Answer with only the new one sentence, no need of any explanations";

        let requestEngine = AI.Request.create(AI.ActionType.Chat);
        if (!requestEngine) return;

        await Asc.Editor.callMethod("StartAction", ["GroupActions"]);

        let turnOffTrackChanges = false;
        if (params.showDifference) {
            let isTrackChanges = await Asc.Editor.callCommand(function () {
                return Api.GetDocument().IsTrackRevisions();
            });

            if (!isTrackChanges) {
                await Asc.Editor.callCommand(function () {
                    Api.GetDocument().SetTrackRevisions(true);
                });
                turnOffTrackChanges = true;
            }
        }

        await Asc.Editor.callMethod("StartAction", [
            "Block",
            "AI (" + requestEngine.modelUI.name + ")",
        ]);

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

        let result = await requestEngine.chatRequest(
            argPromt,
            false,
            async function (data) {
                if (!data) return;
                await checkEndAction();

                if (text && "sentence" === params.type) {
                    Asc.scope.data = data;
                    await Asc.Editor.callCommand(function () {
                        let doc = Api.GetDocument();
                        doc.ReplaceCurrentSentence("");
                    });
                    text = null;
                }

                await Asc.Library.PasteText(data);
            },
        );

        await checkEndAction();

        if (turnOffTrackChanges)
            await Asc.Editor.callCommand(function () {
                return Api.GetDocument().SetTrackRevisions(false);
            });

        await Asc.Editor.callMethod("EndAction", ["GroupActions"]);
    };

    return func;
})();
```

使用的方法：[GetDocument](/docs/office-api/usage-api/text-document-api/Api/Methods/GetDocument.md), [GetCurrentParagraph](/docs/office-api/usage-api/text-document-api/ApiDocument/Methods/GetCurrentParagraph.md), [GetElement](/docs/office-api/usage-api/text-document-api/ApiDocument/Methods/GetElement.md), [GetText](/docs/office-api/usage-api/text-document-api/ApiParagraph/Methods/GetText.md), [Select](/docs/office-api/usage-api/text-document-api/ApiParagraph/Methods/Select.md), [GetCurrentSentence](/docs/office-api/usage-api/text-document-api/ApiDocument/Methods/GetCurrentSentence.md), [IsTrackRevisions](/docs/office-api/usage-api/text-document-api/ApiDocument/Methods/IsTrackRevisions.md), [SetTrackRevisions](/docs/office-api/usage-api/text-document-api/ApiDocument/Methods/SetTrackRevisions.md), [ReplaceCurrentSentence](/docs/office-api/usage-api/text-document-api/ApiDocument/Methods/ReplaceCurrentSentence.md), [EndAction](/docs/plugin-and-macros/interacting-with-editors/api-by-editor-type/text-document-api/Methods/EndAction.md), [StartAction](/docs/plugin-and-macros/interacting-with-editors/api-by-editor-type/text-document-api/Methods/StartAction.md), [Asc.scope object](/docs/plugin-and-macros/interacting-with-editors/overview/how-to-call-commands.md#ascscope-object)

## 结果

<video className="light-video" controls style={{maxWidth: '848px'}}>
  <source src="/assets/images/plugins/functions-video/text-document-editor/rewriteText.webm" type="video/webm" />
  您的浏览器不支持 HTML5 视频。
</video>
<video className="dark-video" controls style={{maxWidth: '848px'}}>
  <source src="/assets/images/plugins/functions-video/text-document-editor/rewriteText.dark.webm" type="video/webm" />
  您的浏览器不支持 HTML5 视频。
</video>
