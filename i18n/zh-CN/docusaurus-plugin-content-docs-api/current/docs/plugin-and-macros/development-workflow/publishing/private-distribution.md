---
sidebar_position: -3
---

# 私有分发

## 概述

并非每个插件都需要发布到 ONLYOFFICE 插件市场。您可能希望私下分发插件——用于公司内部使用、与特定团队共享，或在公开发布前作为测试版本。本页介绍在市场之外分发插件的可用方式。

## 分发方式

| 方式 | 最适合 |
|--------|----------|
| 压缩包（`.zip` / `.plugin`） | 个人分享、手动安装 |
| 直接复制文件夹 | 本地部署、IT 管理部署 |
| 自托管 URL | 团队或组织级分发 |
| 本地部署管理面板 | ONLYOFFICE Docs 本地部署企业版 |

## 打包插件以供分发

分发前，将插件打包为压缩包：

1. 打开插件文件夹
2. 选择插件文件夹**内部**的所有文件和子文件夹（而非文件夹本身）
3. 从所选内容创建 ZIP 压缩包
4. 可选择将扩展名从 `.zip` 改为 `.plugin`

**Error name:** Incorrect archive structure

:::warning[Wrong]
```
my-plugin.zip
└── my-plugin/          ← 文件夹在压缩包内
    ├── config.json
    ├── index.html
    └── icon.png
```
:::

:::tip[Correct]
```
my-plugin.zip
├── config.json          ← 文件在压缩包根目录
├── index.html
└── icon.png
```
:::

Error output: 插件安装失败——所有插件文件必须位于压缩包根目录，而非子文件夹内。

## 通过压缩包安装

接收方可通过插件管理器安装插件压缩包：

1. 打开 ONLYOFFICE 编辑器
2. 转到**插件** → **插件管理器**
3. 单击**从文件添加插件**（或拖放 `.zip`/`.plugin` 文件）
4. 插件出现在插件选项卡中

此方法适用于桌面编辑器和可访问插件管理器的网页编辑器。

## 通过复制文件夹安装（桌面编辑器）

对于桌面编辑器，可以通过将插件文件夹直接放入插件目录来安装插件。这对于管理员将插件推送到用户计算机的 IT 管理部署非常有用。

### 插件目录位置

**Windows：**

```
C:\Program Files\ONLYOFFICE\DesktopEditors\editors\sdkjs-plugins\
```

或用于每用户安装：

```
C:\Users\USERNAME\AppData\Local\ONLYOFFICE\DesktopEditors\editors\sdkjs-plugins\
```

**macOS：**

```
/Applications/ONLYOFFICE.app/Contents/Resources/editors/sdkjs-plugins/
```

**Linux：**

```
/opt/onlyoffice/desktopeditors/editors/sdkjs-plugins/
```

放置插件文件夹后，重启 ONLYOFFICE 桌面编辑器。插件将出现在插件选项卡中。

## 自托管分发

您可以在任意网络服务器上托管插件，并通过 URL 与用户共享。这样团队始终可以使用最新版插件，无需手动重新安装。

### 第 1 步 — 托管插件文件

将插件文件夹上传到网络服务器，使 `config.json` 可通过公共或内部 URL 访问：

```
https://your-server.example.com/plugins/your-plugin-name/config.json
```

`config.json` 中的所有相对路径（图标、HTML 文件）也必须可从同一服务器访问。

### 第 2 步 — 设置 baseUrl 字段

在 `config.json` 中，将 `baseUrl` 设置为插件文件夹的 URL：

```json
{
  "name": "My Plugin",
  "guid": "asc.{FFE1F462-1EA2-4391-990D-4CC84940B754}",
  "baseUrl": "https://your-server.example.com/plugins/your-plugin-name/",
  "version": "1.0.0",
  "variations": [
    {
      "url": "index.html",
      "icons": ["icon.png"],
      "EditorsSupport": ["word"]
    }
  ]
}
```

### 第 3 步 — 共享 config.json URL

用户通过在插件管理器中提供 `config.json` URL 来安装插件：

1. 转到**插件** → **插件管理器**
2. 单击**通过 URL 添加插件**
3. 粘贴 `config.json` URL
4. 单击**确定**

:::tip
对于本地部署，您可以从内部服务器（例如内网或 NAS）提供插件服务，这样无需互联网访问即可让整个组织使用插件。
:::

## 通过管理面板进行本地部署

对于 ONLYOFFICE Docs 本地部署，管理员可以通过管理面板为所有用户部署插件。这绕过了每用户安装步骤，使连接到该实例的所有人都能使用插件。

### 为所有用户添加插件

1. 打开 ONLYOFFICE Docs 管理面板
2. 转到**插件**设置
3. 添加插件 `config.json` 的路径或 URL
4. 保存配置

该插件对该 ONLYOFFICE Docs 实例的所有用户可用，无需单独安装。

有关本地部署的详细设置步骤，请参阅[本地部署安装](../installing-and-testing/docs-on-premises-installation.md)。

## Cloud/SaaS 分发

对于 ONLYOFFICE Cloud 和 SaaS 版本，插件安装按门户管理。有关在云环境中启用插件的说明，请参阅 [Cloud/SaaS 安装](../installing-and-testing/cloud-saas-installation.md)。

私有插件可在特定门户内使用，无需发布到公共市场。

## 安全注意事项

私有分发插件时：

- 仅从可信来源分发插件
- 通过 HTTPS 提供自托管插件服务，防止传输中被篡改
- 在全组织部署前审查插件代码——插件可以访问文档内容并发起网络请求
- 对于本地部署，通过管理面板设置将插件安装限制在已批准的来源

## 后续步骤

- [版本控制与更新](./versioning-and-updates.md) — 规划已分发插件的未来发布
- [市场提交](./marketplace-submission.md) — 准备就绪后发布到公共市场

## 其他资源

- **桌面编辑器安装**：[桌面编辑器安装](../installing-and-testing/desktop-editors-installation.md)
- **本地部署安装**：[本地部署安装](../installing-and-testing/docs-on-premises-installation.md)
- **Cloud 安装**：[Cloud/SaaS 安装](../installing-and-testing/cloud-saas-installation.md)
- **插件配置**：[配置](../../structure/configuration/configuration.md)
