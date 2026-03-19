---
sidebar_position: -1
---

# 发布准备

## 概述

在分发插件之前——无论是通过 ONLYOFFICE 插件市场还是私有方式——您需要确认插件完整、稳定且已为其他用户做好准备。本页介绍发布前需完成的必需文件、配置检查和质量步骤。

## 必需文件

每个插件必须在插件文件夹根目录包含以下文件：

| 文件 | 是否必需 | 说明 |
|------|----------|-------------|
| `config.json` | 是 | 插件配置和元数据 |
| `index.html` | 是 | 插件入口点（主界面或后台页面） |
| `icon.png` | 是 | 默认插件图标（40×40 像素） |
| `icon@2x.png` | 推荐 | 高 DPI 图标（80×80 像素） |

### 附加文件

根据插件类型和功能：

- **本地化文件**：带有语言 JSON 文件的 `translations/` 文件夹（参见[本地化](../../structure/localization.md)）
- **附加 HTML 文件**：适用于具有多个变体的插件（例如，关于窗口）
- **资源文件**：`index.html` 引用的图片、样式表或脚本

## config.json 检查清单

### 必需字段

```json
{
  "name": "My Plugin",
  "guid": "asc.{FFE1F462-1EA2-4391-990D-4CC84940B754}",
  "version": "1.0.0",
  "variations": [
    {
      "description": "What the plugin does",
      "url": "index.html",
      "icons": ["icon.png", "icon@2x.png"],
      "isViewer": false,
      "EditorsSupport": ["word"]
    }
  ]
}
```

**`name`** — 必须唯一且清晰描述插件功能。

**`guid`** — 必须遵循 `asc.{UUID}` 格式。UUID 必须唯一——不要复用其他插件的 GUID。在 [uuidgenerator.net](https://www.uuidgenerator.net/) 生成新 UUID。

**Error name:** Invalid GUID format

:::warning[Wrong]
```json
{ "guid": "my-plugin-guid" }
```
:::

:::tip[Correct]
```json
{ "guid": "asc.{FFE1F462-1EA2-4391-990D-4CC84940B754}" }
```
:::

Error output: 插件无法在编辑器中注册——无效的 GUID 格式导致静默加载失败。

**`version`** — 必须遵循语义版本控制（`MAJOR.MINOR.PATCH`）。首次发布从 `1.0.0` 开始。

**`variations[].EditorsSupport`** — 仅列出已测试过插件的编辑器：`"word"`、`"cell"`、`"slide"`、`"pdf"`。

### 可选但推荐的字段

```json
{
  "minVersion": "7.0.0",
  "help": "https://example.com/plugin-help"
}
```

- **`minVersion`**：防止在缺少所需 API 方法的 ONLYOFFICE 版本上安装
- **`help`**：指向文档或插件窗口中显示的帮助页面的链接

## 图标要求

| 属性 | 要求 |
|----------|-------------|
| 格式 | PNG |
| 标准尺寸 | 40×40 像素 |
| 高 DPI 尺寸 | 80×80 像素（`icon@2x.png`） |
| 背景 | 透明或白色 |

**Error name:** Unsupported icon format

:::warning[Wrong]
```json
{ "icons": ["icon.svg"] }
```
:::

:::tip[Correct]
```json
{ "icons": ["icon.png", "icon@2x.png"] }
```
:::

Error output: 插件图标不显示在插件选项卡中——不支持 SVG 图标。

## 代码质量

### 移除调试代码

发布前移除所有调试工件：

**Error name:** Debug code in production

:::warning[Wrong]
```javascript
window.Asc.plugin.init = function() {
  debugger;
  console.log('DEBUG: init called', arguments);
  console.table(window.Asc.plugin);
};
```
:::

:::tip[Correct]
```javascript
window.Asc.plugin.init = function() {
  loadData();
};
```
:::

Error output: 生产插件运行缓慢并在浏览器控制台中暴露内部状态。

### 插件必须正确关闭

每个插件变体必须处理 `button` 回调并正确关闭：

```javascript
window.Asc.plugin.button = function(id) {
  if (id === 0) {
    window.Asc.plugin.callCommand(function() {
      // apply changes
    });
  } else {
    window.Asc.plugin.executeMethod("CloseWindow");
  }
};
```

### 外部资源

- 所有外部资源必须通过 HTTPS 加载
- 避免从不可靠的 CDN 加载
- 尽可能在本地打包关键依赖项

## 最终文件夹结构

```
your-plugin-name/
├── config.json          ✓ Required
├── index.html           ✓ Required
├── icon.png             ✓ Required (40×40 PNG)
├── icon@2x.png          ✓ Recommended (80×80 PNG)
├── plugin.css           Optional
└── translations/        Optional
    ├── en.json
    └── fr.json
```

## 发布前检查清单

- [ ] 插件在 ONLYOFFICE 桌面编辑器中加载无误
- [ ] 插件在网页编辑器中加载无误
- [ ] `config.json`、`index.html` 和 `icon.png` 均已存在
- [ ] GUID 使用正确的 `asc.{UUID}` 格式且唯一
- [ ] 版本遵循 `MAJOR.MINOR.PATCH`
- [ ] `EditorsSupport` 仅列出已测试的编辑器
- [ ] 代码中无 `debugger` 语句
- [ ] 无过多 `console.log` 语句
- [ ] 图标为 PNG 格式且尺寸正确
- [ ] 外部资源通过 HTTPS 加载
- [ ] 插件关闭时行为正确

## 后续步骤

- [市场提交](./marketplace-submission.md) — 发布到 ONLYOFFICE 插件市场
- [私有分发](./private-distribution.md) — 不通过市场进行分发
- [版本控制与更新](./versioning-and-updates.md) — 规划未来发布

## 其他资源

- **插件配置参考**：[配置](../../structure/configuration/configuration.md)
- **插件变体**：[变体](../../structure/configuration/variations.md)
- **本地化指南**：[本地化](../../structure/localization.md)
- **插件示例**：[https://github.com/ONLYOFFICE/sdkjs-plugins](https://github.com/ONLYOFFICE/sdkjs-plugins)
