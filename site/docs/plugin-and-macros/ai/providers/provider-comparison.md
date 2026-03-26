---
sidebar_position: 4
---

# Provider comparison

The ONLYOFFICE AI plugin supports any OpenAI-compatible provider. This page compares the most commonly used options to help you choose the right one for your use case.

## Comparison table

| Provider | Pricing | Privacy | Setup complexity | Best for |
|----------|---------|---------|-----------------|----------|
| **OpenAI** (GPT-4o, GPT-4o mini) | Pay-per-token | Data sent to OpenAI servers | Low | General-purpose tasks, best quality |
| **Anthropic Claude** | Pay-per-token | Data sent to Anthropic servers | Low | Long documents, nuanced reasoning |
| **DeepSeek** | Pay-per-token | Data sent to DeepSeek servers | Low | Cost-effective, strong coding tasks |
| **Ollama** (self-hosted) | Free | Data stays on your machine | Medium | Privacy-sensitive environments |
| **Custom endpoint** | Varies | Depends on deployment | Medium–High | Enterprise or specialized models |

## Choosing a provider

**Choose OpenAI or Anthropic** if you need the highest quality output for general writing, summarization, and analysis tasks, and you are comfortable with data being processed on external servers.

**Choose DeepSeek** if cost efficiency is a priority and your tasks involve technical or coding-related content.

**Choose Ollama** if you are working in a privacy-sensitive environment, are on a restricted network, or want to avoid per-request costs. See [Configuring Ollama with CORS](../configuration/configuring-ollama-with-cors.md) for setup instructions.

**Choose a custom endpoint** if your organization runs a private AI infrastructure or requires a specific model not available through standard providers. See [Adding custom providers](adding-custom-providers.md) for implementation details.

## Model selection within a provider

Most providers offer multiple models at different capability and cost levels. As a general rule:

- Use **smaller/faster models** (e.g., GPT-4o mini) for formatting, short rewrites, and simple completions.
- Use **larger/more capable models** (e.g., GPT-4o, Claude Opus) for complex reasoning, long document summarization, and analysis tasks.

You can assign different models to different task types within the AI plugin settings (**Settings → Edit AI models**).
