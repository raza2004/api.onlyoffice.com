---
sidebar_position: 2
---

# Provider configuration

This page covers how to manage API keys, understand the request/response flow, and handle errors when working with AI providers in the ONLYOFFICE AI plugin.

## API key management

Each AI provider requires an API key for authentication. Keys are stored locally within the plugin settings and are never sent anywhere other than the provider's API endpoint.

**Best practices:**

- Use separate API keys for development and production environments.
- Rotate keys periodically according to your provider's security recommendations.
- For self-hosted models (e.g., Ollama), no API key is required — leave the key field empty.

To update a key:

1. Open the **AI** tab and click **Settings**.
2. Select **Edit AI models** and click ![Edit icon](/assets/images/plugins/edit.svg#gh-light-mode-only)![Edit icon](/assets/images/plugins/edit.dark.svg#gh-dark-mode-only).
3. Update the API key field and click **OK**.

## Request and response flow

When a user action triggers the AI plugin, the following happens:

1. The plugin collects the relevant document content (selected text, paragraph, or range).
2. It constructs a request with the user's prompt and the collected content.
3. The request is sent to the configured provider's API endpoint using `AI.Request.create()`.
4. The provider streams or returns a complete response.
5. The plugin inserts the response into the document via the Office API.

Streaming responses are handled chunk by chunk, allowing text to appear incrementally in the editor without blocking the UI.

## Error handling

Common errors and how to resolve them:

| Error | Cause | Resolution |
|-------|-------|------------|
| `401 Unauthorized` | Invalid or missing API key | Check and update the API key in Settings |
| `429 Too Many Requests` | Rate limit exceeded | Wait and retry, or switch to a higher-tier plan |
| `503 Service Unavailable` | Provider is temporarily down | Retry after a short delay |
| CORS error | Provider does not allow browser requests | Use a server-side proxy or configure CORS (see [Configuring Ollama with CORS](../configuration/configuring-ollama-with-cors.md)) |
| Empty response | Model returned no output | Rephrase the prompt or check the model's context window limit |

The plugin uses `StartAction` and `EndAction` to ensure that even if a request fails mid-stream, the editor state is cleanly rolled back.
