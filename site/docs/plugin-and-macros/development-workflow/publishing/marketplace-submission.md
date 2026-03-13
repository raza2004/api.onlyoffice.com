---
sidebar_position: -2
---

# Marketplace submission

## Overview

The ONLYOFFICE Plugin Marketplace is a public repository hosted on GitHub where users can discover and install community plugins directly from within ONLYOFFICE editors. This guide walks through the full process of submitting your plugin to the marketplace.

## Before you submit

Complete the [preparing for release](./preparing-for-release.md) checklist to ensure your plugin is ready. Key requirements:

- All required files present: `config.json`, `index.html`, `icon.png`
- Valid `guid` in `asc.{UUID}` format
- Version set to `1.0.0` (or higher)
- No debug code remaining
- Tested in the editors listed in `EditorsSupport`

## Submission steps

### Step 1 — Create a GitHub account

If you do not already have one, sign up at [github.com](https://github.com/).

### Step 2 — Fork the marketplace repository

Fork the official plugin marketplace repository:

[https://github.com/ONLYOFFICE/onlyoffice.github.io](https://github.com/ONLYOFFICE/onlyoffice.github.io)

Your fork will be available at `https://github.com/YOUR-USERNAME/onlyoffice.github.io`.

### Step 3 — Build a GitHub Pages site from your fork (recommended)

Building a GitHub Pages site from your fork lets you test your plugin in the web editor before submitting. Follow the official [GitHub Pages quickstart](https://docs.github.com/en/pages/quickstart) to enable it.

### Step 4 — Clone your fork locally

```bash
git clone https://github.com/YOUR-USERNAME/onlyoffice.github.io.git
cd onlyoffice.github.io
```

### Step 5 — Add your plugin folder

Copy your plugin folder into:

```
sdkjs-plugins/content/your-plugin-name/
```

The folder must contain at minimum:

```
your-plugin-name/
├── config.json
├── index.html
└── icon.png
```

**Folder naming:** Use lowercase letters and hyphens only (e.g., `my-translation-plugin`). The folder name becomes your plugin's identifier in the marketplace.

### Step 6 — Register your plugin in store/config.json

Open `store/config.json` at the root of the repository and add an entry for your plugin:

```json
{ "name": "your-plugin-name", "discussion": "" }
```

- `"name"` must exactly match your plugin folder name
- `"discussion"` can be left empty or set to a GitHub discussion ID

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

Error output: Marketplace cannot find your plugin folder — name mismatch between `store/config.json` and the actual folder name causes the plugin to not appear.

### Step 7 — Push your changes

```bash
git add sdkjs-plugins/content/your-plugin-name/
git add store/config.json
git commit -m "Add your-plugin-name plugin"
git push origin main
```

### Step 8 — Create a pull request

Create a pull request from your fork to the upstream repository:

[https://github.com/ONLYOFFICE/onlyoffice.github.io/pulls](https://github.com/ONLYOFFICE/onlyoffice.github.io/pulls)

You can also use the **Submit your own plugin** button in the Plugin Manager window inside ONLYOFFICE editors, which opens the pull request form directly.

Once submitted, the ONLYOFFICE team reviews your plugin. If it works correctly, the pull request is approved and your plugin appears in the marketplace.

## Testing before submission

Using your GitHub Pages fork, load your plugin in the ONLYOFFICE web editor by providing the URL to your `config.json`:

```
https://YOUR-USERNAME.github.io/sdkjs-plugins/content/your-plugin-name/config.json
```

Use this URL in the Plugin Manager (**Plugins → Plugin Manager → Add plugin from URL**) to verify the plugin works in the web environment before creating the pull request.

## After approval

Once your pull request is merged:

- Your plugin appears in the ONLYOFFICE Plugin Marketplace
- Users can install it directly from the Plugin Manager inside ONLYOFFICE editors
- It becomes visible in the [App Directory](https://www.onlyoffice.com/app-directory/en)

## Engaging with the community

- **Post issues** for feedback or bug reports: [https://github.com/ONLYOFFICE/onlyoffice.github.io/issues](https://github.com/ONLYOFFICE/onlyoffice.github.io/issues)
- **Join the forum**: [https://forum.onlyoffice.com](https://forum.onlyoffice.com)

## Next steps

- [Private distribution](./private-distribution.md) — Distribute without the marketplace
- [Versioning and updates](./versioning-and-updates.md) — Submit updates to your plugin

## Additional resources

- **Plugin Marketplace**: [https://www.onlyoffice.com/app-directory/en](https://www.onlyoffice.com/app-directory/en)
- **Marketplace repository**: [https://github.com/ONLYOFFICE/onlyoffice.github.io](https://github.com/ONLYOFFICE/onlyoffice.github.io)
- **Plugin examples**: [https://github.com/ONLYOFFICE/sdkjs-plugins](https://github.com/ONLYOFFICE/sdkjs-plugins)
