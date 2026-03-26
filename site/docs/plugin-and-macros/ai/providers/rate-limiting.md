---
sidebar_position: 3
---

# Rate limiting

AI providers enforce rate limits to control how many requests can be made within a given time window. This page explains how rate limiting works and how to handle it in the ONLYOFFICE AI plugin.

## How rate limits work

Rate limits are enforced by the AI provider, not by the ONLYOFFICE AI plugin. They typically apply to:

- **Requests per minute (RPM)** — the number of API calls allowed per minute.
- **Tokens per minute (TPM)** — the total number of input and output tokens processed per minute.
- **Requests per day (RPD)** — a daily cap on total requests (common on free tiers).

When a limit is exceeded, the provider returns a `429 Too Many Requests` response.

## Handling rate limit errors

When the AI plugin receives a `429` response, the current request fails. To reduce the likelihood of hitting limits:

- **Use lighter models for simple tasks.** Small models process faster and consume fewer tokens.
- **Avoid sending large text blocks unnecessarily.** Select only the relevant portion of text before triggering an AI action.
- **Use a self-hosted model.** Local models like [Ollama](../configuration/configuring-ollama-with-cors.md) have no provider-enforced rate limits.

## Provider-specific limits

Rate limits vary significantly between providers and plan tiers. Check your provider's documentation for current limits:

| Provider | Limits reference |
|----------|-----------------|
| OpenAI | [platform.openai.com/docs/guides/rate-limits](https://platform.openai.com/docs/guides/rate-limits) |
| Anthropic | [docs.anthropic.com/en/api/rate-limits](https://docs.anthropic.com/en/api/rate-limits) |
| DeepSeek | Provider dashboard |
| Ollama (self-hosted) | No rate limits |

## Throttling strategy

If your use case involves high-frequency AI requests (e.g., processing multiple documents), consider:

1. **Batching requests** — process documents sequentially rather than in parallel.
2. **Adding delays between requests** — introduce a short wait between consecutive calls.
3. **Upgrading your plan** — higher-tier plans offer significantly higher limits.
4. **Using a self-hosted model** — eliminates provider-side rate limiting entirely.
