---
sidebar_position: 1
---

# Installing the AI plugin

The AI plugin is a background plugin that connects an AI provider to ONLYOFFICE editors. This page covers installation and initial configuration.

**Repository on GitHub:** [ai](https://github.com/ONLYOFFICE/onlyoffice.github.io/tree/master/sdkjs-plugins/content/ai)

## Installation

Starting from version 9.0.4, the AI plugin is included in server and desktop distributions built with ONLYOFFICE branding.

If you need to add it manually:

1. Download the plugin from the [ONLYOFFICE App Directory](https://www.onlyoffice.com/app-directory/en/ai).
2. Install it following the instructions for your deployment type:
   - [ONLYOFFICE Desktop Editors](../../tutorials/installing/onlyoffice-desktop-editors.md)
   - [ONLYOFFICE Docs (on-premises)](../../tutorials/installing/onlyoffice-docs-on-premises.md)
   - [ONLYOFFICE Cloud](../../tutorials/installing/onlyoffice-cloud.md)

The plugin GUID is `{9DC93CDB-B576-4F0C-B55E-FCC9C48DD007}`.

## Enabling the plugin

1. Open the **Plugins** tab and click the **Plugin Manager** icon.
2. Find the **AI** plugin and click **Install** (or **Update** if already installed).
3. Click **Background Plugins** and activate the **AI** switch.

## Configuring an AI provider

1. Go to the **AI** tab and click **Settings**.
2. Select **Edit AI models** and click ![Plus icon](/assets/images/plugins/plus.svg#gh-light-mode-only)![Plus icon](/assets/images/plugins/plus.dark.svg#gh-dark-mode-only).
3. Choose a provider from the list or enter a custom provider's details with your API key.
4. In the row of icons, select what the model is used for: *Text*, *Images*, *Embeddings*, *Audio Processing*, *Content Moderation*, *Realtime Tasks*, *Coding Help*, *Visual Analysis*.
5. Click **OK** to save.

For custom providers, see [Adding custom providers](../providers/adding-custom-providers.md).

## Support

To request a feature or report a bug, use the issues section on [GitHub](https://github.com/ONLYOFFICE/onlyoffice.github.io/issues).
