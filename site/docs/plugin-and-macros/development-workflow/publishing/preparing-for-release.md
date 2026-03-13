---
sidebar_position: -1
---

# Preparing for release

## Overview

Before distributing your plugin — whether through the ONLYOFFICE Plugin Marketplace or privately — you need to verify that it is complete, stable, and ready for other users. This page covers the required files, configuration checks, and quality steps to complete before releasing.

## Required files

Every plugin must include the following files at the root of the plugin folder:

| File | Required | Description |
|------|----------|-------------|
| `config.json` | Yes | Plugin configuration and metadata |
| `index.html` | Yes | Plugin entry point (main UI or background page) |
| `icon.png` | Yes | Default plugin icon (40×40 pixels) |
| `icon@2x.png` | Recommended | High-DPI icon (80×80 pixels) |

### Additional files

Depending on plugin type and functionality:

- **Localization files**: `translations/` folder with language JSON files (see [Localization](../../structure/localization.md))
- **Additional HTML files**: For plugins with multiple variations (e.g., an About window)
- **Assets**: Images, stylesheets, or scripts referenced by `index.html`

## config.json checklist

### Required fields

```json
{
  "name": "My Plugin",
  "guid": "asc.{FFE1F462-1EA2-4391-990D-4CC84940B754}",
  "version": "1.0.0",
  "variations": [
    {
      "description": "What the plugin does",
      "url": "index.html",
      "icons": ["icon.png", "icon@2x.png"],
      "isViewer": false,
      "EditorsSupport": ["word"]
    }
  ]
}
```

**`name`** — Must be unique and clearly describe what the plugin does.

**`guid`** — Must follow the `asc.{UUID}` format. The UUID must be unique — do not reuse GUIDs from other plugins. Generate a new UUID at [uuidgenerator.net](https://www.uuidgenerator.net/).

**Error name:** Invalid GUID format

:::warning[Wrong]
```json
{ "guid": "my-plugin-guid" }
```
:::

:::tip[Correct]
```json
{ "guid": "asc.{FFE1F462-1EA2-4391-990D-4CC84940B754}" }
```
:::

Error output: Plugin fails to register in the editors — invalid GUID format causes a silent load failure.

**`version`** — Must follow semantic versioning (`MAJOR.MINOR.PATCH`). Start at `1.0.0` for a first release.

**`variations[].EditorsSupport`** — Only list editors where the plugin has been tested: `"word"`, `"cell"`, `"slide"`, `"pdf"`.

### Optional but recommended fields

```json
{
  "minVersion": "7.0.0",
  "help": "https://example.com/plugin-help"
}
```

- **`minVersion`**: Prevents installation on ONLYOFFICE versions that lack required API methods
- **`help`**: A link to documentation or a help page shown in the plugin window

## Icon requirements

| Property | Requirement |
|----------|-------------|
| Format | PNG |
| Standard size | 40×40 pixels |
| High-DPI size | 80×80 pixels (`icon@2x.png`) |
| Background | Transparent or white |

**Error name:** Unsupported icon format

:::warning[Wrong]
```json
{ "icons": ["icon.svg"] }
```
:::

:::tip[Correct]
```json
{ "icons": ["icon.png", "icon@2x.png"] }
```
:::

Error output: Plugin icon does not appear in the Plugins tab — SVG icons are not supported.

## Code quality

### Remove debug code

Remove all debugging artifacts before release:

**Error name:** Debug code in production

:::warning[Wrong]
```javascript
window.Asc.plugin.init = function() {
  debugger;
  console.log('DEBUG: init called', arguments);
  console.table(window.Asc.plugin);
};
```
:::

:::tip[Correct]
```javascript
window.Asc.plugin.init = function() {
  loadData();
};
```
:::

Error output: Production plugin runs slowly and exposes internal state in the browser console.

### Plugin must close correctly

Every plugin variation must handle the `button` callback and close properly:

```javascript
window.Asc.plugin.button = function(id) {
  if (id === 0) {
    window.Asc.plugin.callCommand(function() {
      // apply changes
    });
  } else {
    window.Asc.plugin.executeMethod("CloseWindow");
  }
};
```

### External resources

- All external resources must be loaded over HTTPS
- Avoid loading from unreliable CDNs
- Bundle critical dependencies locally when possible

## Final folder structure

```
your-plugin-name/
├── config.json          ✓ Required
├── index.html           ✓ Required
├── icon.png             ✓ Required (40×40 PNG)
├── icon@2x.png          ✓ Recommended (80×80 PNG)
├── plugin.css           Optional
└── translations/        Optional
    ├── en.json
    └── fr.json
```

## Pre-release checklist

- [ ] Plugin loads without errors in ONLYOFFICE Desktop Editors
- [ ] Plugin loads without errors in the web editor
- [ ] `config.json`, `index.html`, and `icon.png` are all present
- [ ] GUID uses the correct `asc.{UUID}` format and is unique
- [ ] Version follows `MAJOR.MINOR.PATCH`
- [ ] `EditorsSupport` only lists tested editors
- [ ] No `debugger` statements in the code
- [ ] No excessive `console.log` statements
- [ ] Icons are PNG at correct dimensions
- [ ] External resources load over HTTPS
- [ ] Plugin closes correctly when dismissed

## Next steps

- [Marketplace submission](./marketplace-submission.md) — Publish to the ONLYOFFICE Plugin Marketplace
- [Private distribution](./private-distribution.md) — Distribute without going through the marketplace
- [Versioning and updates](./versioning-and-updates.md) — Plan future releases

## Additional resources

- **Plugin configuration reference**: [Configuration](../../structure/configuration/configuration.md)
- **Plugin variations**: [Variations](../../structure/configuration/variations.md)
- **Localization guide**: [Localization](../../structure/localization.md)
- **Plugin examples**: [https://github.com/ONLYOFFICE/sdkjs-plugins](https://github.com/ONLYOFFICE/sdkjs-plugins)
