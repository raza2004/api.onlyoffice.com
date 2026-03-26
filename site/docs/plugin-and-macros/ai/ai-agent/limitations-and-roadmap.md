---
sidebar_position: 5
---

# Limitations and roadmap

The AI agent is currently in **beta**. This page documents known constraints and planned improvements.

## Current limitations

**Adding a custom AI tool requires modifying the plugin source code.**
There is no plugin-level UI for registering custom tools. Developers must clone the [AI plugin repository](https://github.com/ONLYOFFICE/onlyoffice.github.io/tree/master/sdkjs-plugins/content/ai), add their tool to the helpers folder, rebuild, and deploy via a custom store link. See [Creating a custom AI tool](../custom-ai-tools/creating-a-custom-ai-tool.md) for the full process.

**Tool selection depends on the AI model's judgment.**
The agent relies on the configured model to select the correct tool based on the user's request. Ambiguous or poorly worded requests may result in the wrong tool being called. Providing clear, specific instructions improves accuracy.

**Conversation history is session-based.**
The agent's conversation history is not persisted between sessions. Closing the editor or pressing `Ctrl + Alt + /` resets the history.

**Not all editor features are accessible via the agent.**
The agent can only call tools that have been registered. Features without a corresponding tool (e.g., advanced table styling, complex chart formatting) must still be applied manually.

**Beta availability.**
The AI agent feature is available starting from AI plugin version 2.4.2. Some behaviors may change in future releases.

## Planned improvements

The AI agent functionality continues to evolve. Areas under active development include:

- A plugin-level interface for registering and managing custom tools without modifying source code.
- Persistent conversation history across editor sessions.
- Expanded built-in tool coverage for all editor types.
- Improved tool selection accuracy through better prompt engineering and example sets.

To request a feature or report a bug, use the issues section on [GitHub](https://github.com/ONLYOFFICE/onlyoffice.github.io/issues).
