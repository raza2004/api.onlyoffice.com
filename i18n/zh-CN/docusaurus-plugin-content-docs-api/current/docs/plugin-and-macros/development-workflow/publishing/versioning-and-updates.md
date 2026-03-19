---
sidebar_position: -4
---

# 版本控制与更新

## 概述

插件发布到 ONLYOFFICE 插件市场后，您需要发布更新来修复 Bug、添加功能或维护与新 ONLYOFFICE 版本的兼容性。本页介绍如何为插件设置版本号以及如何提交更新。

## 版本控制方案

ONLYOFFICE 插件使用语义版本控制（`MAJOR.MINOR.PATCH`）：

| 部分 | 何时递增 | 示例 |
|------|-------------------|---------|
| `MAJOR` | 破坏性变更或重大重写 | `1.0.0` → `2.0.0` |
| `MINOR` | 新功能、向后兼容 | `1.0.0` → `1.1.0` |
| `PATCH` | Bug 修复、小幅修正 | `1.0.0` → `1.0.1` |

版本在 `config.json` 中设置：

```json
{
  "name": "My Plugin",
  "guid": "asc.{FFE1F462-1EA2-4391-990D-4CC84940B754}",
  "version": "1.2.0"
}
```

**提交更新时始终递增版本号。** 保持相同的版本号使用户和审核人员无法区分不同发布。

**Error name:** Version not incremented on update

:::warning[Wrong]
```json
{ "version": "1.0.0" }
```
*（提交更新时未更改版本号）*
:::

:::tip[Correct]
```json
{ "version": "1.0.1" }
```
:::

Error output: 市场和插件管理器无法检测到有可用更新——用户继续运行旧版本。

## 指定最低 ONLYOFFICE 版本

使用 `minVersion` 字段声明运行插件所需的最低 ONLYOFFICE 编辑器版本：

```json
{
  "minVersion": "7.0.0"
}
```

当插件开始使用特定 ONLYOFFICE 版本中引入的 API 方法或功能时，请更新 `minVersion`。参阅[更新日志](../../more-information/changelog.md)确认特定方法是何时添加的。

:::tip
设置准确的 `minVersion` 可防止旧版本用户安装不兼容的插件并遇到静默失败。
:::

## 提交更新

更新遵循与初始提交相同的流程，使用您现有的市场仓库 Fork。

### 第 1 步 — 将 Fork 与上游同步

```bash
git remote add upstream https://github.com/ONLYOFFICE/onlyoffice.github.io.git
git fetch upstream
git checkout main
git merge upstream/main
```

### 第 2 步 — 更新插件文件

替换以下目录中的已更新文件：

```
sdkjs-plugins/content/your-plugin-name/
```

### 第 3 步 — 在 config.json 中递增版本号

```json
{
  "version": "1.1.0"
}
```

### 第 4 步 — 提交并推送

```bash
git add sdkjs-plugins/content/your-plugin-name/
git commit -m "Update your-plugin-name to v1.1.0"
git push origin main
```

### 第 5 步 — 创建 Pull Request

从您的 Fork 向上游仓库创建 Pull Request。在 Pull Request 说明中包含：

- 此版本中的变更内容
- 任何新的 ONLYOFFICE 版本要求
- Bug 修复或破坏性变更

## 保持向后兼容性

更新插件时，尽量保持向后兼容性：

**避免删除用户可能依赖的现有功能。** 如果某个功能需要重大变更，请考虑使用 `MAJOR` 版本号并记录迁移说明。

**不要更改插件 GUID。** GUID 是永久性的，用于在所有 ONLYOFFICE 安装中标识您的插件。更改 GUID 会在市场中创建重复的插件条目。

**Error name:** GUID changed on update

:::warning[Wrong]
```json
{ "guid": "asc.{NEW-GUID-FOR-UPDATE}" }
```
:::

:::tip[Correct]
```json
{ "guid": "asc.{ORIGINAL-GUID-UNCHANGED}" }
```
:::

Error output: GUID 变更导致市场将此更新视为全新插件——现有用户不会收到更新，旧版本继续安装。

## 处理 ONLYOFFICE 版本兼容性

发布新 ONLYOFFICE 版本后，验证您的插件是否仍然正常运行。主要测试内容：

- 所有 `executeMethod` 调用返回预期结果
- 事件处理程序（`onDocumentReady`、`button` 等）正常触发
- UI 元素在插件面板或窗口中正确渲染

如果新的 ONLYOFFICE 版本引入的变更破坏了您的插件，请发布 `PATCH` 更新并在必要时更新 `minVersion`。

每次 ONLYOFFICE 发布后，查看[更新日志](../../more-information/changelog.md)以识别与您的插件相关的 API 变更。

## 为插件维护更新日志

维护更新日志有助于用户了解版本间的变化。在插件文件夹根目录保存 `CHANGELOG.md`（可选但推荐）：

```markdown
# Changelog

## 1.1.0 — 2025-06-01
- Added support for Spreadsheet Editor
- Improved performance for large documents

## 1.0.1 — 2025-03-15
- Fixed icon not appearing on high-DPI displays
- Fixed plugin not closing when Cancel is clicked

## 1.0.0 — 2025-01-10
- Initial release
```

## 从市场移除插件

如果您需要从市场移除插件（例如，不再维护），请在市场仓库中提交一个 Issue 请求移除：

[https://github.com/ONLYOFFICE/onlyoffice.github.io/issues](https://github.com/ONLYOFFICE/onlyoffice.github.io/issues)

请注明插件名称和移除原因。

## 后续步骤

- 查看完整的[市场提交](./marketplace-submission.md)指南
- 准备新发布时查看[发布准备](./preparing-for-release.md)
- 了解[私有分发](./private-distribution.md)作为市场更新的替代方案

## 其他资源

- **插件更新日志**：[更新日志](../../more-information/changelog.md)
- **配置参考**：[配置](../../structure/configuration/configuration.md)
- **市场仓库**：[https://github.com/ONLYOFFICE/onlyoffice.github.io](https://github.com/ONLYOFFICE/onlyoffice.github.io)
- **插件示例**：[https://github.com/ONLYOFFICE/sdkjs-plugins](https://github.com/ONLYOFFICE/sdkjs-plugins)
