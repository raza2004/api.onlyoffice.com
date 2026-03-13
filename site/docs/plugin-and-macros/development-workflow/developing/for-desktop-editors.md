---
sidebar_position: -2
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# For desktop editors

To develop a plugin for ONLYOFFICE Desktop Editors, follow the steps below.

1. Create your plugin folder with [config.json](../../structure/configuration/configuration.md) and [index.html](../../structure/entry-point.md).

2. Pack all plugin files into a `.zip` archive and change the extension to `.plugin` (all files must be at the archive root, not inside a subfolder).

3. Install the plugin via [Desktop Editors installation](../installing-and-testing/desktop-editors-installation.md).

   After installation, the plugin folder appears in the `sdkjs-plugins` directory. The path depends on your operating system:

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

   The folder name is the plugin GUID from `config.json` (for example, `{07FD8DFA-DFE0-4089-AL24-0730933CC80A}`).

4. Edit your plugin files in the `sdkjs-plugins` folder and reload the plugin to see changes.

## Additional resources

- [Desktop Editors installation](../installing-and-testing/desktop-editors-installation.md)
- [Hot reload & live testing](./hot-reload-live-testing.md)
- [Plugin examples](https://github.com/ONLYOFFICE/sdkjs-plugins)
