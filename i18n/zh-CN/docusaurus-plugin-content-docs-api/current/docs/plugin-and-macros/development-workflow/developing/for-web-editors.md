---
sidebar_position: -1
---

# 适用于网页编辑器

如需为 ONLYOFFICE 网页编辑器开发插件，请按照以下步骤操作。

1. 创建包含 [config.json](../../structure/configuration/configuration.md) 和 [index.html](../../structure/entry-point.md) 的插件文件夹。

2. 通过本地 HTTP 服务器提供插件文件夹服务，使 `config.json` 可通过 URL 访问：

   ```bash
   # Node.js
   npx http-server ./my-plugin --port 8080

   # Python
   python -m http.server 8080
   ```

3. 在网页编辑器中，转到**插件 → 插件管理器 → 通过 URL 添加插件**，并输入：

   ```
   http://localhost:8080/config.json
   ```

4. 编辑插件文件并重新加载插件以查看更改。

:::note
网页编辑器必须能够访问您的开发服务器。对于 ONLYOFFICE Cloud 或远程本地实例，请使用可公开访问的 URL 或 `ngrok` 等隧道工具。
:::

## 其他资源

- [Cloud/SaaS 安装](../installing-and-testing/cloud-saas-installation.md)
- [本地部署安装](../installing-and-testing/docs-on-premises-installation.md)
- [热重载与实时测试](./hot-reload-live-testing.md)
- [插件示例](https://github.com/ONLYOFFICE/sdkjs-plugins)
