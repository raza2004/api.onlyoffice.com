# AI

将 AI 提供商（例如 OpenAI、DeepSeek）连接到 ONLYOFFICE 编辑器，实现智能文本生成、编辑、摘要和自动宏创建。

**插件类型：** 后台。

**支持的编辑器：** 文档、电子表格、演示文稿、PDF。

```mdx-code-block
import YoutubeVideo from '@site/src/components/YoutubeVideo/YoutubeVideo';

<YoutubeVideo videoId="oQbH8JIe3eE"/>
```

## 安装

默认情况下，ONLYOFFICE 企业版、社区版（Docs + Workspace）和 ONLYOFFICE 云端均提供此插件。

您也可以从 [ONLYOFFICE 应用目录](https://www.onlyoffice.com/app-directory/en/ai)下载该插件，并按照[桌面端](/docs/plugin-and-macros/tutorials/installing/onlyoffice-desktop-editors.md)安装说明进行安装。

从版本 9.0.4 起，AI 插件已添加到采用 ONLYOFFICE 品牌构建的服务器版和桌面版发行版中。

插件 guid：`{9DC93CDB-B576-4F0C-B55E-FCC9C48DD007}`。

## 使用方法

1. 打开 **插件** 选项卡，点击 **插件管理器** 图标。
2. 点击 **后台插件** 按钮，激活 **AI** 开关。
3. 在 ONLYOFFICE 编辑器的顶部工具栏中找到新增的 **AI** 选项卡。
4. 选择其中一个按钮：
   - **设置**：配置面板，用于选择 AI 提供商、输入 API 密钥并选择其中一个模型；
   - **聊天机器人**：与 AI 展开对话，提问、改写文本、头脑风暴等；
   - **摘要**：自动对输入的文本生成摘要，并选择插入结果的方式；
   - **翻译**：使用已配置的 AI 服务翻译所选文本。
5. 您可以对选中的文本使用该插件。为此，选中文本后右键单击，在 AI 菜单中选择以下选项之一：**摘要**、**文本分析**、**翻译**、**图像** 或 **聊天机器人**。
6. 插件将根据已配置的 AI 模型作出响应。
7. 将响应插入文档或根据需要使用。

## 配置

要开始使用插件，需要设置 AI 提供商：

1. 转到 **AI** 选项卡，点击 **设置** 打开配置窗口。
2. 选择 **编辑 AI 模型**，然后点击 **添加**。
3. 从列表中选择 AI 提供商，或通过输入 API 密钥添加新的 AI 模型。
4. 在图标行中，选择该模型的用途：*文本*、*图像*、*嵌入*、*音频处理*、*内容审核*、*实时任务*、*编程帮助*、*视觉分析*。
5. 点击 **确定** 保存设置并完成连接流程。

有关添加自定义提供商的详细信息，请参阅此[博客文章](https://www.onlyoffice.com/blog/2025/03/how-to-add-a-custom-provider-to-the-onlyoffice-ai-plugin)。

## 插件结构

GitHub 上的仓库：[ai](https://github.com/ONLYOFFICE/onlyoffice.github.io/tree/master/sdkjs-plugins/content/ai)。

1. *config.json*、*index.html* 和 *code.js*
2. 图标
3. translations 文件夹包含多种语言的翻译。
4. 第三方服务：
   该插件支持 OpenAI 及自定义 AI 提供商，需要配置 API 密钥和模型。许可证和条款取决于所使用的提供商。

## 配置文件

``` json
{
    "name" : "AI",
    "nameLocale": {
        "fr": "AI",
        "es": "AI",
        "de": "AI",
        "cs": "AI",
        "zh": "AI",
        "pt-BR": "AI",
        "sr-Cyrl-RS": "AI",
        "sr-Latn-RS": "AI",
        "ja-JA": "AI",
        "sq-AL": "AI",
        "it": "IA",
        "ar-SA": "AI"
    },

    "guid" : "asc.{9DC93CDB-B576-4F0C-B55E-FCC9C48DD007}",
    "version": "2.4.2",
    "minVersion" : "8.2.0",

    "variations" : [
        {
            "description": "Use the AI chatbot to perform tasks which involve understanding or generating natural language or code.",
            "descriptionLocale": {
                "fr": "Utilisez le chatbot AI pour effectuer des tâches qui impliquent la compréhension ou la génération de langage naturel ou de code.",
                "es": "Utilice el chatbot AI para realizar tareas que impliquen la comprensión o generación de lenguaje natural o de código.",
                "pt-BR": "Use o chatbot AI para realizar tarefas que envolvam compreensão ou geração de linguagem ou código natural.",
                "de": "Verwenden Sie den AI-Chatbot, um Aufgaben auszuführen, die das Verstehen oder Generieren von natürlicher Sprache oder Code beinhalten.",
                "cs": "Použijte chatbota AI k provádění úkolů, který zahrnuje porozumění nebo generování přirozeného jazyka nebo kódu.",
                "zh": "使用 AI 聊天机器人完成有关理解、生成自然语言或代码的任务。",
                "sr-Cyrl-RS": "Користите AI чет робота за обављање задатака који укључују разумевање или генерисање природног језика или кода.",
                "sr-Latn-RS": "Koristite AI čet robota za obavljanje zadataka koji uključuju razumevanje ili generisanje prirodnog jezika ili koda.",
                "ja-JA": "自然言語やコードの理解または生成が必要なタスクを行うには、AIチャットボットを使用できます。",
                "sq-AL": "Shtoni dhe selektoni modele AI për detyra të ndryshme.",
                "it": "Utilizza il chatbot dell'IA per eseguire attività che implicano la comprensione o la generazione di codice o linguaggio naturale.",
                "ar-SA": "استخدموا روبوت المحادثة الذكي لتنفيذ المهام التي تتطلب فهمًا أو إنتاجًا للغة الطبيعية أو البرمجة."
            },

            "url"         : "index.html",

            "icons": "resources/%theme-type%(light|dark)/icon%scale%(default).%extension%(png)",

            "isViewer"            : false,
            "EditorsSupport"      : ["word", "cell", "slide", "pdf"],
            "type"                : "background",
            "initDataType"        : "none",
            "buttons"             : [],
            "events"              : ["onAIPluginSettings", "onContextMenuShow", "onContextMenuClick", "onToolbarMenuClick"],

            "store": {
                "background": {
                    "light" : "linear-gradient(90deg, #F9B6FF 0%, #E370EE 102.01%)",
                    "dark" : "linear-gradient(90deg, #F9B6FF 0%, #E370EE 102.01%)"
                },
                "screenshots" :
                [
                    "resources/store/screenshots/screen_1.png",
                    "resources/store/screenshots/screen_2.png",
                    "resources/store/screenshots/screen_3.png",
                    "resources/store/screenshots/screen_4.png",
                    "resources/store/screenshots/screen_5.png",
                    "resources/store/screenshots/screen_6.png"
                ],
                "icons" : {
                    "light" : "resources/store/icons",
                    "dark"  : "resources/store/icons"
                },
                "categories": ["specAbilities", "work", "recommended"]
            }
        }
    ],

    "onlyofficeScheme": true
}
```

## 方法和事件

- init
- [button](/docs/plugin-and-macros/customization/buttons.md)
- [onTranslate](/docs/plugin-and-macros/structure/localization.md#applying-translations-to-plugin)
- [attachEditorEvent](/docs/plugin-and-macros/interacting-with-editors/overview/how-to-attach-events.md#option-1-using-the-attacheditorevent-method)
- onThemeChanged
- onThemeChangedBase
- [executeMethod ("CloseWindow")](/docs/plugin-and-macros/customization/windows-and-panels.md#closing-a-window)
- [executeMethod ("PasteText")](/docs/plugin-and-macros/interacting-with-editors/api-by-editor-type/text-document-api/Methods/PasteText.md)
- info.aiPluginSettings
- [info.editorType](/docs/plugin-and-macros/interacting-with-editors/overview/how-to-call-commands.md#editorType)
- [info.data](/docs/plugin-and-macros/interacting-with-editors/overview/how-to-call-commands.md#data)
- [info.guid](/docs/plugin-and-macros/interacting-with-editors/overview/how-to-call-commands.md#guid)
- [info.width](/docs/plugin-and-macros/interacting-with-editors/overview/how-to-call-commands.md#width)

## 支持

如果您想就此插件请求功能或报告错误，请使用 [GitHub](https://github.com/ONLYOFFICE/onlyoffice.github.io/issues) 上的 issues 区。
