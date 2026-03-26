---
sidebar_position: -1
title: Privacy and data handling
description: Security and data practices.
---

# Privacy and data handling {#privacy-and-data-handling}

AI tools in ONLYOFFICE plugins can send document content to external services. Understanding what data leaves the editor — and how to minimize exposure — is essential for building plugins that can be used safely in sensitive environments.

## What data is sent to the AI model {#what-data-is-sent}

The only data transmitted to the AI provider is the `prompt` string passed to `chatRequest`. This string is constructed by your plugin code, so you have full control over what it contains.

Common sources of user data that end up in the prompt:

- Selected text retrieved via `Asc.Editor.callMethod("GetSelectedText")`
- Cell values from a spreadsheet
- Document metadata such as the file name or author
- Any other content your code explicitly embeds in the prompt string

No document content is sent to the provider automatically or implicitly. If your `func.call` implementation does not embed document content in the prompt, none reaches the provider.

## Minimize data exposure {#minimize-data-exposure}

Extract only the content necessary for the task. Do not send the entire document when a single selection or cell value is sufficient.

**Avoid:**

```ts
const fullText = await Asc.Editor.callMethod("GetDocumentText");
const prompt = `Analyze this document:\n\n${fullText}`;
```

**Prefer:**

```ts
const selectedText = await Asc.Editor.callMethod("GetSelectedText");
const prompt = `Summarize the following paragraph:\n\n${selectedText}`;
```

Additional data minimization practices:

- Strip personally identifiable information (names, email addresses, phone numbers) from the prompt if the task does not require it.
- Truncate overly long selections to a reasonable maximum length before embedding them.
- Avoid logging prompt strings to the browser console or to any external service beyond the AI provider.

## Local providers keep data on-premises {#local-providers}

When an ONLYOFFICE installation is configured to use a local AI provider such as [Ollama](../../ai/configuring-ollama-with-cors.md), all `chatRequest` calls are sent to a server within the organization's own infrastructure. No data reaches a third-party cloud service.

This approach is recommended for:

- Organizations handling confidential documents, financial records, or legal content
- Environments subject to data residency regulations (GDPR, HIPAA, and similar)
- Development and testing workflows where sending real document content to a cloud provider is undesirable

See [Configuring Ollama with CORS](../../ai/configuring-ollama-with-cors.md) for setup instructions.

## API key storage {#api-key-storage}

Never hard-code API keys in plugin source code. Plugin files are distributed to users and may be inspected or extracted.

The correct approach is to use the AI plugin's built-in key management:

1. Open the **AI** tab in the ONLYOFFICE editor.
2. Click **Settings**.
3. Enter the API key in the designated field.

The plugin runtime makes the configured key available to `chatRequest` automatically. Your plugin code does not need to handle the key at all — it is injected by the runtime, not by your `func.call` implementation.

> A hard-coded API key in a distributed plugin is a credential leak. Any user who installs the plugin can extract and use the key, incurring charges under your account.

## CORS and network security {#cors-and-network-security}

When using a self-hosted AI provider, the browser enforces CORS (Cross-Origin Resource Sharing) restrictions on requests made from the editor's origin. Misconfigured CORS settings will cause `chatRequest` to fail silently or with an opaque network error.

Checklist for self-hosted providers:

- Configure the provider to emit the correct `Access-Control-Allow-Origin` header. For Ollama, see [Configuring Ollama with CORS](../../ai/configuring-ollama-with-cors.md).
- Use HTTPS with a valid certificate. Browsers block mixed-content requests (HTTP endpoints called from an HTTPS page).
- Consider placing the provider behind a reverse proxy (nginx, Caddy, or similar) to handle TLS termination and CORS headers centrally.
- Restrict the allowed origins to only the ONLYOFFICE server hostname — do not use `*` in production environments.

## Inform users when document content is transmitted {#inform-users}

If your plugin sends document content to a third-party AI provider, disclose this clearly:

- In the plugin's **description** field shown in the ONLYOFFICE marketplace or admin panel
- In the plugin's settings UI, with a note such as: "Selected text is sent to [Provider Name] to generate a response. Review [Provider Name]'s privacy policy before using this feature with confidential documents."
- In the plugin's README or documentation

Transparency builds user trust and helps administrators make informed decisions about which plugins to approve for their organization.

## Data retention {#data-retention}

Cloud AI providers differ in how long they retain prompts and completions. Some providers store request data for abuse monitoring; others offer zero-retention API plans for enterprise customers.

For compliance-sensitive environments:

- Review the data retention policy of every provider your plugin supports.
- Prefer providers that offer zero-retention or short-retention plans when handling regulated data.
- For the highest assurance, use a local model — retained data is then governed entirely by your own infrastructure policies.
- Document the retention policy in your plugin's privacy disclosure so that administrators can make informed deployment decisions.
