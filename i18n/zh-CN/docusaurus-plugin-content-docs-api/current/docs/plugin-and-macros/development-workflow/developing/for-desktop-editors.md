---
sidebar_position: -2
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# 适用于桌面编辑器

如需为 ONLYOFFICE 桌面编辑器开发插件，请按照以下步骤操作。

1. 创建包含 [config.json](../../structure/configuration/configuration.md) 和 [index.html](../../structure/entry-point.md) 的插件文件夹。

2. 将所有插件文件打包为 `.zip` 压缩文件，并将扩展名改为 `.plugin`（所有文件必须位于压缩文件的根目录，而非子文件夹中）。

3. 通过[桌面编辑器安装](../installing-and-testing/desktop-editors-installation.md)来安装插件。

   安装完成后，插件文件夹将出现在 `sdkjs-plugins` 目录中。路径因操作系统而异：

   <Tabs>
     <TabItem value="win" label="Windows">
       ```bash
       C:\Users\<username>\AppData\Local\ONLYOFFICE\DesktopEditors\data\sdkjs-plugins\
       ```
     </TabItem>
     <TabItem value="mac" label="macOS">
       ```bash
       ~/Library/Application\ Support/asc.onlyoffice.ONLYOFFICE/data/sdkjs-plugins/
       ```
     </TabItem>
     <TabItem value="lin" label="Linux">
       ```bash
       home/<username>/.local/share/onlyoffice/desktopeditors/sdkjs-plugins/
       ```
     </TabItem>
   </Tabs>

   文件夹名称为 `config.json` 中的插件 GUID（例如，`{07FD8DFA-DFE0-4089-A124-0730933CC80A}`）。

4. 在 `sdkjs-plugins` 文件夹中编辑插件文件并重新加载插件以查看更改。

## 其他资源

- [桌面编辑器安装](../installing-and-testing/desktop-editors-installation.md)
- [热重载与实时测试](./hot-reload-live-testing.md)
- [插件示例](https://github.com/ONLYOFFICE/sdkjs-plugins)
