---
sidebar_position: -2
---

# 市场提交

## 概述

ONLYOFFICE 插件市场是托管在 GitHub 上的公共仓库，用户可以直接在 ONLYOFFICE 编辑器内发现并安装社区插件。本指南介绍将插件提交到市场的完整流程。

## 提交前准备

请先完成[发布准备](./preparing-for-release.md)清单，确保插件已就绪。主要要求：

- 所有必需文件齐全：`config.json`、`index.html`、`icon.png`
- `guid` 使用 `asc.{UUID}` 格式且有效
- 版本设置为 `1.0.0`（或更高）
- 无调试代码残留
- 已在 `EditorsSupport` 中列出的编辑器中完成测试

## 提交步骤

### 第 1 步 — 创建 GitHub 账号

如果尚未注册，请前往 [github.com](https://github.com/) 注册。

### 第 2 步 — Fork 市场仓库

Fork 官方插件市场仓库：

[https://github.com/ONLYOFFICE/onlyoffice.github.io](https://github.com/ONLYOFFICE/onlyoffice.github.io)

您的 Fork 将位于 `https://github.com/YOUR-USERNAME/onlyoffice.github.io`。

### 第 3 步 — 从 Fork 构建 GitHub Pages 站点（推荐）

从 Fork 构建 GitHub Pages 站点可让您在提交前于网页编辑器中测试插件。请参阅官方 [GitHub Pages 快速入门](https://docs.github.com/en/pages/quickstart)启用此功能。

### 第 4 步 — 将 Fork 克隆到本地

```bash
git clone https://github.com/YOUR-USERNAME/onlyoffice.github.io.git
cd onlyoffice.github.io
```

### 第 5 步 — 添加插件文件夹

将插件文件夹复制到：

```
sdkjs-plugins/content/your-plugin-name/
```

文件夹至少需包含：

```
your-plugin-name/
├── config.json
├── index.html
└── icon.png
```

**文件夹命名：** 仅使用小写字母和连字符（例如 `my-translation-plugin`）。文件夹名称将成为插件在市场中的标识符。

### 第 6 步 — 在 store/config.json 中注册插件

打开仓库根目录的 `store/config.json`，为您的插件添加条目：

```json
{ "name": "your-plugin-name", "discussion": "" }
```

- `"name"` 必须与插件文件夹名称完全一致
- `"discussion"` 可留空或设置为 GitHub 讨论 ID

**Error name:** Plugin name mismatch

:::warning[Wrong]
```json
{ "name": "My Translation Plugin", "discussion": "" }
```
:::

:::tip[Correct]
```json
{ "name": "my-translation-plugin", "discussion": "" }
```
:::

Error output: 市场找不到插件文件夹——`store/config.json` 与实际文件夹名称不匹配导致插件无法显示。

### 第 7 步 — 推送更改

```bash
git add sdkjs-plugins/content/your-plugin-name/
git add store/config.json
git commit -m "Add your-plugin-name plugin"
git push origin main
```

### 第 8 步 — 创建 Pull Request

从您的 Fork 向上游仓库创建 Pull Request：

[https://github.com/ONLYOFFICE/onlyoffice.github.io/pulls](https://github.com/ONLYOFFICE/onlyoffice.github.io/pulls)

您也可以使用 ONLYOFFICE 编辑器中插件管理器窗口的**提交您自己的插件**按钮，该按钮将直接打开 Pull Request 表单。

提交后，ONLYOFFICE 团队将审核您的插件。如果运行正常，Pull Request 将被批准，插件出现在市场中。

## 提交前测试

使用您的 GitHub Pages Fork，通过提供 `config.json` 的 URL 在 ONLYOFFICE 网页编辑器中加载插件：

```
https://YOUR-USERNAME.github.io/sdkjs-plugins/content/your-plugin-name/config.json
```

在创建 Pull Request 之前，在插件管理器（**插件 → 插件管理器 → 通过 URL 添加插件**）中使用此 URL 验证插件在网页环境中是否正常运行。

## 批准后

Pull Request 合并后：

- 您的插件出现在 ONLYOFFICE 插件市场中
- 用户可直接从 ONLYOFFICE 编辑器内的插件管理器安装
- 插件在 [App Directory](https://www.onlyoffice.com/app-directory/en) 中可见

## 参与社区

- **提交问题**以获取反馈或报告 Bug：[https://github.com/ONLYOFFICE/onlyoffice.github.io/issues](https://github.com/ONLYOFFICE/onlyoffice.github.io/issues)
- **加入论坛**：[https://forum.onlyoffice.com](https://forum.onlyoffice.com)

## 后续步骤

- [私有分发](./private-distribution.md) — 不通过市场进行分发
- [版本控制与更新](./versioning-and-updates.md) — 提交插件更新

## 其他资源

- **插件市场**：[https://www.onlyoffice.com/app-directory/en](https://www.onlyoffice.com/app-directory/en)
- **市场仓库**：[https://github.com/ONLYOFFICE/onlyoffice.github.io](https://github.com/ONLYOFFICE/onlyoffice.github.io)
- **插件示例**：[https://github.com/ONLYOFFICE/sdkjs-plugins](https://github.com/ONLYOFFICE/sdkjs-plugins)
