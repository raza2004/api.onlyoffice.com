---
sidebar_position: -1
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# 桌面编辑器安装

从 [onlyoffice.com/download-desktop.aspx](https://www.onlyoffice.com/download-desktop.aspx) 下载 ONLYOFFICE 桌面编辑器，并运行对应平台的安装程序。

## 添加测试插件

有两种方式可以在桌面编辑器中安装插件。

### 方式一：插件管理器（推荐）

1. 打开**插件**选项卡。
2. 单击**插件管理器 → 我的插件**。
3. 单击**手动安装插件**并选择您的 `.plugin` 压缩包。

### 方式二：插件文件夹

将插件文件夹直接放入 `sdkjs-plugins` 目录：

<Tabs>
  <TabItem value="win" label="Windows">
    ```bash
    %ProgramFiles%\ONLYOFFICE\DesktopEditors\editors\sdkjs-plugins\
    ```
  </TabItem>
  <TabItem value="mac" label="macOS">
    ```bash
    /Applications/ONLYOFFICE.app/Contents/Resources/editors/sdkjs-plugins/
    ```
  </TabItem>
  <TabItem value="lin" label="Linux">
    ```bash
    /opt/onlyoffice/desktopeditors/editors/sdkjs-plugins/
    ```
  </TabItem>
</Tabs>

使用 `config.json` 中的插件 **GUID** 作为文件夹名称（例如，`{91EAC419-EF8B-440C-A960-B451C7DF3A37}`）。放置文件夹后，重启桌面编辑器。

## 移除插件

1. 打开**插件 → 插件管理器**。
2. 单击插件下方的**删除**。

:::note
通过插件文件夹添加的插件必须手动删除，即从 `sdkjs-plugins` 中删除对应文件夹。
:::

## 其他资源

- [适用于桌面编辑器](../developing/for-desktop-editors.md) — 开发工作流
- [测试环境搭建](./test-environment-setup.md)
