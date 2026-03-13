---
sidebar_position: -1
---

# For web editors

To develop a plugin for ONLYOFFICE web editors, follow the steps below.

1. Create your plugin folder with [config.json](../../structure/configuration/configuration.md) and [index.html](../../structure/entry-point.md).

2. Serve the plugin folder from a local HTTP server so `config.json` is accessible at a URL:

   ```bash
   # Node.js
   npx http-server ./my-plugin --port 8080

   # Python
   python -m http.server 8080
   ```

3. In the web editor, go to **Plugins → Plugin Manager → Add plugin from URL** and enter:

   ```
   http://localhost:8080/config.json
   ```

4. Edit your plugin files and reload the plugin to see changes.

:::note
The web editor must be able to reach your development server. For ONLYOFFICE Cloud or a remote on-premises instance, use a publicly accessible URL or a tunneling tool such as `ngrok`.
:::

## Additional resources

- [Cloud/SaaS installation](../installing-and-testing/cloud-saas-installation.md)
- [Docs (on-premises) installation](../installing-and-testing/docs-on-premises-installation.md)
- [Hot reload & live testing](./hot-reload-live-testing.md)
- [Plugin examples](https://github.com/ONLYOFFICE/sdkjs-plugins)
