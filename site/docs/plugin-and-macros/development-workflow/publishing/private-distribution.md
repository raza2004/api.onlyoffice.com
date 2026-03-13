---
sidebar_position: -3
---

# Private distribution

## Overview

Not every plugin needs to be published to the ONLYOFFICE Plugin Marketplace. You may want to distribute a plugin privately — for internal use within a company, to share with a specific team, or as a beta release before a public launch. This page covers the available methods for distributing plugins outside the marketplace.

## Distribution methods

| Method | Best for |
|--------|----------|
| Archive (`.zip` / `.plugin`) | Sharing with individuals, manual installs |
| Direct folder copy | On-premises deployments, IT-managed setups |
| Self-hosted URL | Team or organization-wide distribution |
| On-premises admin panel | ONLYOFFICE Docs on-premises enterprise deployment |

## Packaging a plugin for distribution

Before distributing, package your plugin as an archive:

1. Open your plugin folder
2. Select all files and subfolders **inside** the plugin folder (not the folder itself)
3. Create a ZIP archive from the selection
4. Optionally rename the extension from `.zip` to `.plugin`

**Error name:** Incorrect archive structure

:::warning[Wrong]
```
my-plugin.zip
└── my-plugin/          ← folder is inside the archive
    ├── config.json
    ├── index.html
    └── icon.png
```
:::

:::tip[Correct]
```
my-plugin.zip
├── config.json          ← files are at the archive root
├── index.html
└── icon.png
```
:::

Error output: Plugin fails to install — all plugin files must be at the archive root, not inside a subfolder.

## Installing via archive

Recipients can install the plugin archive through the Plugin Manager:

1. Open an ONLYOFFICE editor
2. Go to **Plugins** → **Plugin Manager**
3. Click **Add plugin from file** (or drag and drop the `.zip`/`.plugin` file)
4. The plugin appears in the Plugins tab

This method works for both Desktop Editors and web editors where the Plugin Manager is accessible.

## Installing via folder copy (Desktop Editors)

For Desktop Editors, a plugin can be installed by placing the plugin folder directly in the plugins directory. This is useful for IT-managed deployments where administrators push plugins to user machines.

### Plugins directory locations

**Windows:**

```
C:\Program Files\ONLYOFFICE\DesktopEditors\editors\sdkjs-plugins\
```

Or for per-user installations:

```
C:\Users\USERNAME\AppData\Local\ONLYOFFICE\DesktopEditors\editors\sdkjs-plugins\
```

**macOS:**

```
/Applications/ONLYOFFICE.app/Contents/Resources/editors/sdkjs-plugins/
```

**Linux:**

```
/opt/onlyoffice/desktopeditors/editors/sdkjs-plugins/
```

After placing the plugin folder, restart ONLYOFFICE Desktop Editors. The plugin will appear in the Plugins tab.

## Self-hosted distribution

You can host your plugin on any web server and share it with users via a URL. This allows a team to always use the latest version of a plugin without manual re-installation.

### Step 1 — Host the plugin files

Upload your plugin folder to a web server so that `config.json` is accessible at a public or internal URL:

```
https://your-server.example.com/plugins/your-plugin-name/config.json
```

All relative paths in `config.json` (icons, HTML files) must also be accessible from the same server.

### Step 2 — Set the baseUrl field

In `config.json`, set `baseUrl` to the URL of your plugin folder:

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

### Step 3 — Share the config.json URL

Users install the plugin by providing the `config.json` URL in the Plugin Manager:

1. Go to **Plugins** → **Plugin Manager**
2. Click **Add plugin from URL**
3. Paste the `config.json` URL
4. Click **OK**

:::tip
For on-premises installations, you can serve plugins from an internal server (e.g., an intranet or NAS) so that plugins are available to the whole organization without internet access.
:::

## On-premises deployment via admin panel

For ONLYOFFICE Docs on-premises, administrators can deploy plugins for all users through the admin panel. This bypasses per-user installation and makes plugins available to everyone connected to the instance.

### Adding a plugin for all users

1. Open the ONLYOFFICE Docs admin panel
2. Navigate to **Plugins** settings
3. Add the path or URL to your plugin's `config.json`
4. Save the configuration

The plugin becomes available in the editors for all users on that ONLYOFFICE Docs instance without individual installation steps.

For detailed on-premises setup steps, see [Docs (on-premises) installation](../installing-and-testing/docs-on-premises-installation.md).

## Cloud/SaaS distribution

For ONLYOFFICE Cloud and SaaS editions, plugin installation is managed per-portal. See [Cloud/SaaS installation](../installing-and-testing/cloud-saas-installation.md) for instructions on enabling plugins in a cloud environment.

Private plugins can be made available within a specific portal without being published to the public marketplace.

## Security considerations

When distributing plugins privately:

- Only distribute plugins from trusted sources
- Serve self-hosted plugins over HTTPS to prevent tampering in transit
- Review plugin code before deploying organization-wide — plugins have access to document content and can make network requests
- For on-premises deployments, restrict plugin installation to approved sources through admin panel settings

## Next steps

- [Versioning and updates](./versioning-and-updates.md) — Plan future releases of your distributed plugin
- [Marketplace submission](./marketplace-submission.md) — Publish to the public marketplace when ready

## Additional resources

- **Desktop Editors installation**: [Desktop Editors installation](../installing-and-testing/desktop-editors-installation.md)
- **On-premises installation**: [Docs (on-premises) installation](../installing-and-testing/docs-on-premises-installation.md)
- **Cloud installation**: [Cloud/SaaS installation](../installing-and-testing/cloud-saas-installation.md)
- **Plugin configuration**: [Configuration](../../structure/configuration/configuration.md)
