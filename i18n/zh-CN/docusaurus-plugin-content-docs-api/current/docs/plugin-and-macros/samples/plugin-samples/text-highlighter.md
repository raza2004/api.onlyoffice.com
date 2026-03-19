# 文本高亮

在文档中搜索文本并应用高亮、颜色和格式样式。该插件支持查找特定单词或短语，并使用各种颜色、文本格式选项和高亮效果自定义其外观。

**插件类型：** 可视化、面板、非系统。

**支持的编辑器：** 文档、PDF。

## 结果

<!-- ![TextHighlighter](/assets/images/plugins/text-highlighter-image.png#gh-light-mode-only)
![TextHighlighter](/assets/images/plugins/text-highlighter-image-dark-mode.png#gh-dark-mode-only) -->

## 安装

从 [GitHub](https://github.com/ONLYOFFICE/onlyoffice.github.io/tree/master/sdkjs-plugins/content/texthighlighter) 下载此插件，并按照[桌面端](/docs/plugin-and-macros/tutorials/installing/onlyoffice-desktop-editors.md)、[本地部署](/docs/plugin-and-macros/tutorials/installing/onlyoffice-docs-on-premises.md)或[云端](/docs/plugin-and-macros/tutorials/installing/onlyoffice-cloud.md)安装说明进行安装。

插件 GUID：`{07FD8DFA-DFE0-4089-A124-0730933CC804}`。

## 使用方法

1. 在 **插件** 选项卡中找到该插件。
2. 插件以侧边面板形式打开。
3. 输入要搜索的文本，或直接在文档中选择文本（选中内容将自动填入搜索框）。
4. 配置高亮偏好设置：
   - **忽略大小写**：勾选此选项以进行不区分大小写的搜索。
   - **高亮颜色**：从黄色、绿色、蓝色、红色或无填充中选择。
   - **文字颜色**：点击下拉菜单打开颜色选择器，选择自定义文字颜色。
   - **文字格式**：展开此部分以应用加粗、斜体、下划线或删除线格式。
5. 点击 **应用** 按钮以高亮文档中所有匹配文本。
6. 应用后：
   - 插件显示找到的匹配数量。
   - 点击 **撤销** 以取消高亮。
   - 点击 **继续高亮** 以搜索其他词语。

## 插件结构

GitHub 仓库：[texthighlighter](https://github.com/ONLYOFFICE/onlyoffice.github.io/tree/master/sdkjs-plugins/content/texthighlighter)。

1. *config.json*、*index.html* 和 *code.js* - 核心插件文件

2. 图标

3. *translations* 文件夹包含俄语、德语、西班牙语、捷克语和法语翻译。

4. **styles.css** - 插件界面的自定义样式，包括深色模式支持和响应式布局。

5. **第三方库**：
   - [Pickr](https://simonwep.github.io/pickr/) - 现代轻量级颜色选择器库。许可证：[MIT License](https://github.com/Simonwep/pickr/blob/master/LICENSE)。


## 配置

```json
{
  "name": "Text Highlighter",
  "nameLocale": {
    "en-US": "Text Highlighter",
    "ru": "Выделение текста",
    "de": "Texthervorhebung",
    "fr": "Surligneur de texte",
  },
  "version": "1.0.0",
  "baseUrl": "https://onlyoffice.github.io/sdkjs-plugins/content/texthighlighter/",
  "guid": "asc.{07FD8DFA-DFE0-4089-A124-0730933CC804}",
  "manifestVersion": "7.3.0",
  "variations": [
    {
      "description": "This plugin allows you to search for text and apply highlighting, color, and formatting styles in the document.",
      "descriptionLocale": {
        "en": "This plugin allows you to search for text and apply highlighting, color, and formatting styles in the document.",
        "ru": "Этот плагин позволяет вам искать текст и применять выделение, цвет и стили форматирования в документе.",
        "de": "Dieses Plugin ermöglicht das Suchen von Text und das Anwenden von Hervorhebungen, Farben und Formatierungen im Dokument.",
        "fr": "Ce plugin vous permet de rechercher du texte et d'appliquer des styles de surlignage, de couleur et de formatage dans le document.",

      },
      "url": "index.html",
      "type": "panel",
      "size": [300, 600],
      "isViewer": true,
      "EditorsSupport": ["word", "pdf"],
      "isVisual": true,
      "isModal": false,
      "isInsideMode": true,
      "initDataType": "text",
      "isUpdateOleOnResize": true,
      "initOnSelectionChanged": true,
      "events": [
        "onSelectionChanged",
        "onClick",
        "onApply",
        "onGetSelectedText"
      ],
      "methods": ["ApplyHighlight", "GetSelectedText"],
      "buttons": [],
      "icons": ["resources/light/icon.svg", "resources/light/icon@2x.svg"],
      "icons2": [
        {
          "theme": "flat",
          "style": "light",
          "100%": { "normal": "resources/light/icon.svg" },
          "200%": { "normal": "resources/light/icon@2x.svg" },
          "default": { "normal": "resources/light/icon.svg" }
        },
        {
          "theme": "flatDark",
          "style": "dark",
          "100%": { "normal": "resources/dark/icon.svg" },
          "200%": { "normal": "resources/dark/icon@2x.svg" }
        }
      ],
      "store": {
        "background": {
          "light": "#107cbc",
          "dark": "#5f55af"
        },
        "screenshots": [
          "resources/store/screenshots/screen_1.png",
          "resources/store/screenshots/screen_2.png",
        ],
        "categories": ["work", "specAbilities"],
        "icons": {
          "light": "resources/store/icons",
          "dark": "resources/store/icons"
        }
      }
    }
  ]
}
```

## 方法与事件

### 方法

- [callCommand](/docs/plugin-and-macros/interacting-with-editors/overview/#callcommand) - 执行文档操作
- [executeMethod ("Search")](/docs/office-api/usage-api/text-document-api/ApiParagraph/Methods/Search) - 在文档中搜索文本
- [executeMethod ("GetRangeBySelect")](/docs/plugin-and-macros/interacting-with-editors/common-api/Methods/GetRangeBySelect) - 获取选中范围
- [executeMethod ("Undo")](/docs/plugin-and-macros/interacting-with-editors/api-by-editor-type/text-document-api/Methods/Undo/) - 撤销上一次操作

### 事件

- [init](/docs/plugin-and-macros/interacting-with-editors/overview/#how-it-works) - 初始化插件
- [onThemeChanged](/docs/plugin-and-macros/interacting-with-editors/overview/) - 处理主题切换（深色/浅色模式）
- [onTranslate](/docs/plugin-and-macros/structure/localization.md#applying-translations-to-plugin) - 应用翻译
- [onSelectionChanged](/docs/plugin-and-macros/interacting-with-editors/overview/) - 响应文本选择变化
- [onCommandCallback](/docs/plugin-and-macros/interacting-with-editors/overview/#oncommandcallback) - 处理 callCommand 的返回结果

## 支持

如需请求功能或报告此插件的错误，请在 [GitHub](https://github.com/ONLYOFFICE/onlyoffice.github.io/issues) 的 issues 区提交。
