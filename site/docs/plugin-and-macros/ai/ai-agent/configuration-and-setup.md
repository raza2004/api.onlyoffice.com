---
sidebar_position: 2
---

# Configuration and setup

The AI agent is part of the AI plugin. Before using the agent, the plugin must be installed and an AI provider must be configured.

## Prerequisites

- ONLYOFFICE Docs version 9.0.4 or later (plugin is included by default).
- An AI provider account with an API key, or a self-hosted model (e.g., Ollama).

If you have not yet installed the plugin, see [Installing the AI plugin](../getting-started/installing-ai-plugin.md).

## Enabling the AI agent

1. Open the **Plugins** tab and click the **Plugin Manager** icon.
2. Find the **AI** plugin and click **Install** or **Update** if already installed.

   ![AI plugin](/assets/images/plugins/install-ai-plugin.png#gh-light-mode-only)![AI plugin](/assets/images/plugins/install-ai-plugin.dark.png#gh-dark-mode-only)

3. Click the **Background Plugins** button and activate the **AI** switch.

   ![Activate AI](/assets/images/plugins/activate-ai.png#gh-light-mode-only)![Activate AI](/assets/images/plugins/activate-ai.dark.png#gh-dark-mode-only)

4. Find the new **AI** tab in the top toolbar.
5. Click **Settings** to open the configuration window.
6. Select **Edit AI models** and click ![Plus icon](/assets/images/plugins/plus.svg#gh-light-mode-only)![Plus icon](/assets/images/plugins/plus.dark.svg#gh-dark-mode-only).
7. Choose an AI provider from the list or add a custom one with your API key.
8. In the row of icons, select the task types the model handles.
9. Click **OK** to save.

   ![AI settings](/assets/images/plugins/ai-settings.png#gh-light-mode-only)![AI settings](/assets/images/plugins/ai-settings.dark.png#gh-dark-mode-only)

10. Go back to **Settings** and set the model for the **Chatbot** (this is the model the AI agent uses).

The AI agent is now active and ready to use.

## Agent behavior settings

The agent uses the model assigned to the **Chatbot** task type. To change the model:

1. Open **Settings → Edit AI models**.
2. Select the model row and click the edit icon.
3. Ensure the **Chatbot** icon is selected in the task type row.
4. Click **OK**.
