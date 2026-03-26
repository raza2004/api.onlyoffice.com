---
sidebar_position: -4
title: "Prerequisites"
---

# Prerequisites {#prerequisites}

This page describes everything you need before you can start building AI features with the ONLYOFFICE AI plugin.

## Required software {#required-software}

Before you begin, make sure the following are in place:

- **ONLYOFFICE Docs** (server or cloud) or **ONLYOFFICE Desktop Editors** installed and running.
- The **AI plugin** installed. The plugin is available from:
  - ONLYOFFICE Docs or Desktop Editors **v9.0.4** and later (bundled or App Directory).
  - The ONLYOFFICE App Directory: [https://www.onlyoffice.com/app-directory/en/ai](https://www.onlyoffice.com/app-directory/en/ai).
- At least one **AI provider account** configured with a valid API key or local endpoint.

> The plugin GUID is `{9DC93CDB-B576-4F0C-B55E-FCC9C48DD007}`. You will need this value when referencing the plugin programmatically or when locating its configuration files on disk.

## Supported AI providers {#supported-ai-providers}

The AI plugin supports the following provider types out of the box:

| Provider | Type | Notes |
| --- | --- | --- |
| OpenAI | Cloud | Requires an API key from [platform.openai.com](https://platform.openai.com) |
| DeepSeek | Cloud | Requires an API key from the DeepSeek developer dashboard |
| Ollama | Local | Runs entirely on your machine; no API key required |
| Other OpenAI-compatible providers | Cloud or local | Any provider that exposes an OpenAI-compatible REST API |

Additional providers may be added in future plugin releases. Check the App Directory listing for the current full list.

## API key requirements {#api-key-requirements}

### Cloud providers {#cloud-providers}

For cloud providers such as OpenAI or DeepSeek:

1. Create an account on the provider's developer platform.
2. Navigate to the **API keys** section of your account dashboard.
3. Generate a new secret key and copy it immediately — most providers do not show the full key again after creation.
4. Keep the key secure. Do not commit it to source control or expose it in client-side code.

### Local providers {#local-providers}

For local providers such as Ollama:

- No API key is required.
- You need a running Ollama instance accessible from the machine or browser where ONLYOFFICE is running.
- CORS must be configured so that the editor's origin is permitted to reach the Ollama HTTP endpoint. See [Provider configuration examples](./provider-configuration-examples.md#configuring-ollama-with-cors) for setup instructions.

## Environment requirements {#environment-requirements}

### Supported editor versions {#supported-editor-versions}

| Component | Minimum version |
| --- | --- |
| ONLYOFFICE Docs | 7.5 (plugin API); AI plugin features require v9.0.4+) |
| ONLYOFFICE Desktop Editors | v9.0.4+ |
| AI plugin | v2.4.2+ for AI Agent (beta) support |

### Browser support for the web editor {#browser-support}

When using ONLYOFFICE Docs in a browser, the AI plugin runs inside a sandboxed iframe. The following browsers are supported:

- **Chromium-based browsers** (Chrome, Edge, Opera) — fully supported.
- **Firefox** — supported.
- **Safari** — supported with the same caveats that apply to other ONLYOFFICE Docs features.

If a request to a cloud provider fails in the browser, open the browser developer console (**F12**) and check the **Network** tab for blocked requests, and the **Console** tab for CORS or mixed-content errors.

### Desktop Editors {#desktop-editors}

In ONLYOFFICE Desktop Editors the plugin runs in a Chromium-based embedded browser. Network requests to cloud providers go out directly from the desktop application, so no additional browser configuration is needed. For local providers, make sure the Ollama (or equivalent) service is running before launching the editor.
