---
sidebar_position: -4
---

# Versioning and updates

## Overview

Once your plugin is published in the ONLYOFFICE Plugin Marketplace, you will need to release updates to fix bugs, add features, or maintain compatibility with new ONLYOFFICE versions. This page explains how to version your plugin and how to submit updates.

## Versioning scheme

ONLYOFFICE plugins use semantic versioning (`MAJOR.MINOR.PATCH`):

| Part | When to increment | Example |
|------|-------------------|---------|
| `MAJOR` | Breaking changes or major rewrites | `1.0.0` → `2.0.0` |
| `MINOR` | New features, backwards-compatible | `1.0.0` → `1.1.0` |
| `PATCH` | Bug fixes, small corrections | `1.0.0` → `1.0.1` |

The version is set in `config.json`:

```json
{
  "name": "My Plugin",
  "guid": "asc.{FFE1F462-1EA2-4391-990D-4CC84940B754}",
  "version": "1.2.0"
}
```

**Always increment the version when submitting an update.** Keeping the same version number makes it impossible for users and reviewers to distinguish releases.

**Error name:** Version not incremented on update

:::warning[Wrong]
```json
{ "version": "1.0.0" }
```
*(submitting an update without changing the version)*
:::

:::tip[Correct]
```json
{ "version": "1.0.1" }
```
:::

Error output: Marketplace and Plugin Manager cannot detect that an update is available — users continue running the old version.

## Specifying minimum ONLYOFFICE version

Use the `minVersion` field to declare the minimum ONLYOFFICE editor version required to run your plugin:

```json
{
  "minVersion": "7.0.0"
}
```

Update `minVersion` when your plugin starts using API methods or features introduced in a specific ONLYOFFICE version. Refer to the [Changelog](../../more-information/changelog.md) to identify when specific methods were added.

:::tip
Setting an accurate `minVersion` prevents users on older versions from installing an incompatible plugin and encountering silent failures.
:::

## Submitting an update

Updates follow the same process as the initial submission, using your existing fork of the marketplace repository.

### Step 1 — Sync your fork with upstream

```bash
git remote add upstream https://github.com/ONLYOFFICE/onlyoffice.github.io.git
git fetch upstream
git checkout main
git merge upstream/main
```

### Step 2 — Update your plugin files

Replace the updated files in:

```
sdkjs-plugins/content/your-plugin-name/
```

### Step 3 — Increment the version in config.json

```json
{
  "version": "1.1.0"
}
```

### Step 4 — Commit and push

```bash
git add sdkjs-plugins/content/your-plugin-name/
git commit -m "Update your-plugin-name to v1.1.0"
git push origin main
```

### Step 5 — Create a pull request

Open a pull request from your fork to the upstream repository. In the pull request description, include:

- What changed in this version
- Any new ONLYOFFICE version requirements
- Bug fixes or breaking changes

## Maintaining backwards compatibility

When updating a plugin, try to preserve backwards compatibility:

**Avoid removing existing functionality** that users may depend on. If a feature needs to change significantly, consider using a `MAJOR` version bump and documenting the migration.

**Do not change the plugin GUID.** The GUID is permanent and identifies your plugin across all ONLYOFFICE installations. Changing it creates a duplicate plugin entry in the marketplace.

**Error name:** GUID changed on update

:::warning[Wrong]
```json
{ "guid": "asc.{NEW-GUID-FOR-UPDATE}" }
```
:::

:::tip[Correct]
```json
{ "guid": "asc.{ORIGINAL-GUID-UNCHANGED}" }
```
:::

Error output: GUID change causes the marketplace to treat the update as a brand new plugin — existing users do not receive the update and the old version remains installed.

## Handling ONLYOFFICE version compatibility

When a new ONLYOFFICE version is released, verify that your plugin still works correctly. Key things to test:

- All `executeMethod` calls return expected results
- Event handlers (`onDocumentReady`, `button`, etc.) fire correctly
- UI elements render correctly within the plugin panel or window

If the new ONLYOFFICE version introduces a change that breaks your plugin, release a `PATCH` update and update `minVersion` if needed.

Check the [Changelog](../../more-information/changelog.md) after each ONLYOFFICE release to identify any API changes relevant to your plugin.

## Keeping a changelog for your plugin

Maintaining a changelog helps users understand what changed between versions. Keep it in a `CHANGELOG.md` at the root of your plugin folder (optional but recommended):

```markdown
# Changelog

## 1.1.0 — 2025-06-01
- Added support for Spreadsheet Editor
- Improved performance for large documents

## 1.0.1 — 2025-03-15
- Fixed icon not appearing on high-DPI displays
- Fixed plugin not closing when Cancel is clicked

## 1.0.0 — 2025-01-10
- Initial release
```

## Removing a plugin from the marketplace

If you need to remove your plugin from the marketplace (e.g., it is no longer maintained), open an issue in the marketplace repository requesting removal:

[https://github.com/ONLYOFFICE/onlyoffice.github.io/issues](https://github.com/ONLYOFFICE/onlyoffice.github.io/issues)

Include the plugin name and reason for removal.

## Next steps

- Review the full [marketplace submission](./marketplace-submission.md) guide
- Check [preparing for release](./preparing-for-release.md) when preparing a new release
- Learn about [private distribution](./private-distribution.md) as an alternative to marketplace updates

## Additional resources

- **Plugin Changelog**: [Changelog](../../more-information/changelog.md)
- **Configuration reference**: [Configuration](../../structure/configuration/configuration.md)
- **Marketplace repository**: [https://github.com/ONLYOFFICE/onlyoffice.github.io](https://github.com/ONLYOFFICE/onlyoffice.github.io)
- **Plugin examples**: [https://github.com/ONLYOFFICE/sdkjs-plugins](https://github.com/ONLYOFFICE/sdkjs-plugins)
