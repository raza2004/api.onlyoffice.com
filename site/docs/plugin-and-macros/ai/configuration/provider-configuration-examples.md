---
sidebar_position: -1
title: "Provider configuration examples"
---

# Provider configuration examples {#provider-configuration-examples}

This page provides practical setup instructions for the most common AI provider configurations: a local Ollama instance, a cloud provider (OpenAI), and how to verify that either configuration is working correctly.

## Section 1: Configuring Ollama with CORS (local provider) {#configuring-ollama-with-cors}

Ollama exposes a local HTTP API on `http://localhost:11434` by default. When ONLYOFFICE Docs runs in a browser, requests from the editor origin to Ollama are subject to the browser's same-origin policy. You must configure Ollama to emit permissive CORS headers so the browser allows these requests.

The environment variable that controls Ollama's allowed origins is `OLLAMA_ORIGINS`. Set it to:

```bash
OLLAMA_ORIGINS=http://*,https://*,onlyoffice://*
```

This permits requests from any HTTP or HTTPS origin, and also from the `onlyoffice://` scheme used by ONLYOFFICE Desktop Editors.

> For a full walkthrough including model selection and testing, see [Configuring Ollama with CORS](../../ai/configuring-ollama-with-cors.md).

### Linux (systemd) {#linux-systemd}

Edit the Ollama systemd service override:

```bash
sudo systemctl edit ollama
```

Add the following block inside the editor that opens:

```ini
[Service]
Environment="OLLAMA_ORIGINS=http://*,https://*,onlyoffice://*"
```

Save the file, then reload and restart the service:

```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

### macOS (launchctl) {#macos-launchctl}

Set the environment variable via `launchctl` so it is inherited by Ollama when it starts:

```bash
launchctl setenv OLLAMA_ORIGINS "http://*,https://*,onlyoffice://*"
```

Then restart Ollama:

```bash
osascript -e 'quit app "Ollama"'
open -a Ollama
```

### Windows (setx) {#windows-setx}

Open **Command Prompt** as administrator and run:

```bash
setx OLLAMA_ORIGINS "http://*,https://*,onlyoffice://*" /M
```

The `/M` flag writes the variable to the system-wide environment. After running the command, restart Ollama from the system tray or by stopping and re-starting the Ollama process.

## Section 2: Configuring a cloud provider (OpenAI example) {#configuring-a-cloud-provider}

### Obtain an API key {#obtain-an-api-key}

1. Sign in to the OpenAI developer platform at [https://platform.openai.com](https://platform.openai.com).
2. Navigate to **API keys** in the left sidebar.
3. Click **Create new secret key**, give it a name, and copy the key value. Store it securely — the full key is shown only once.

### Add the provider in ONLYOFFICE editors {#add-the-provider-in-onlyoffice-editors}

1. Open any document in ONLYOFFICE Docs or Desktop Editors.
2. Click the **AI** tab in the top toolbar.
3. Click **Settings**.
4. In the settings panel, click **Edit AI models**.
5. Click the **+** (plus) button to add a new provider.
6. From the provider list, select **OpenAI**.
7. Paste your API key into the **API key** field.
8. Select the **model capabilities** you want to enable (for example, **Text generation**, **Image generation**, **Embeddings**) and choose the specific model for each capability.
9. Click **OK** to save.

The OpenAI provider is now active. The AI plugin will route requests to OpenAI for the capabilities you enabled.

## Section 3: Verifying the configuration {#verifying-the-configuration}

### Quick functional test {#quick-functional-test}

The fastest way to confirm that a provider is active and reachable is to use the AI agent directly in the editor:

1. Open any document.
2. Press **Ctrl + /** to open the AI agent input.
3. Type a simple prompt such as `What is 2 + 2?` and press **Enter**.
4. If the provider is configured correctly, the agent returns a response within a few seconds.

### Checking for errors {#checking-for-errors}

If the request fails or produces no response:

1. Open the browser developer tools with **F12** (web editor) or the equivalent in Desktop Editors.
2. Go to the **Console** tab and look for error messages from the AI plugin. Common errors include:
   - `401 Unauthorized` — the API key is missing or incorrect.
   - `CORS error` — the provider (typically Ollama) is not configured to allow the editor's origin.
   - `net::ERR_CONNECTION_REFUSED` — a local provider (Ollama) is not running.
3. Go to the **Network** tab, filter by **Fetch/XHR**, and re-send the prompt. Inspect the request and response to see the exact HTTP status and response body returned by the provider.

Resolve the underlying error (wrong key, CORS misconfiguration, service not running) and repeat the functional test.
