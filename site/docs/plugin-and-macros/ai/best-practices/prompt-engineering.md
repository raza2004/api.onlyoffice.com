---
sidebar_position: -5
title: Prompt engineering
description: Writing effective prompts.
---

# Prompt engineering {#prompt-engineering}

Writing effective prompts is one of the most important skills when building custom AI tools with ONLYOFFICE plugins. The quality of the `params`, `examples`, and `description` fields you provide directly determines how reliably the AI model will invoke your tools and how accurate the results will be.

## What prompt engineering means in this context {#what-prompt-engineering-means}

When you register a function using `RegisteredFunction`, the `description`, `params`, and `examples` fields are not just documentation — they are part of the prompt sent to the AI model at runtime. The model reads these fields to decide:

- Which tool to call for a given user request
- What argument values to extract from natural language input
- How to handle edge cases and optional parameters

Every word in these fields influences model behavior. Treating them as a prompt engineering task — not just metadata — is the key difference between a tool that works reliably and one that misfires.

## Writing clear `params` descriptions {#params-descriptions}

Each parameter description should answer three questions for the model:

- What type of value is expected?
- What are the valid or recommended values?
- What is the default value when the parameter is omitted?

**Error name:** Vague parameter description

:::warning[Wrong]
```ts
{
  name: "language",
  description: "The language to use."
}
```
:::

:::tip[Correct]
```ts
{
  name: "language",
  description: "The target language for translation. Must be a BCP 47 language tag such as 'en', 'fr', 'de', or 'zh-CN'. Defaults to 'en' if not specified."
}
```
:::

Error output: The AI model may misinterpret the parameter, infer incorrect values, or fail to extract the intended input from natural language.

Additional guidelines:

- Be explicit about whether a parameter is required or optional.
- List valid values inline when the set is small (fewer than eight options).
- Avoid generic phrases like "the value" or "the input" — name the concept directly.
- If a numeric parameter has a meaningful range, state it: `"An integer between 1 and 10"`.

## Writing effective `examples` {#examples}

Examples teach the model how to map natural language phrases to structured tool calls. Provide at least two or three examples per tool, and make sure they cover distinct scenarios.

**Cover these cases:**

- The most common usage with all required parameters
- A usage where one or more optional parameters are omitted
- A usage that exercises an edge case or a less obvious phrasing

**Example set for a "translate selection" tool:**

```ts
examples: [
  {
    prompt: "Translate the selected text to French",
    params: { language: "fr" }
  },
  {
    prompt: "Translate this to simplified Chinese",
    params: { language: "zh-CN" }
  },
  {
    prompt: "Translate the highlighted passage",
    params: { language: "en" }  // defaults to English when not stated
  }
]
```

**Common mistakes to avoid:**

- Providing only one example — the model may over-fit to that phrasing.
- Writing two examples that are nearly identical — they add no new information.
- Using ambiguous phrasing such as "do the thing" — examples should reflect realistic user input.
- Forgetting to show optional parameter omission — without it, the model may always try to fill every field.

## Writing a good `description` {#description}

The `description` field is the primary signal the model uses to decide which tool to invoke. Keep it short (one or two sentences), distinctive, and action-oriented.

A good description answers:

- What does this tool do?
- When should the model choose this tool over others?

**Error name:** Non-distinctive tool description

:::warning[Wrong]
```ts
description: "Does text stuff."
```
:::

:::tip[Correct]
```ts
description: "Translates the currently selected text into the specified language and replaces the selection with the translated result. Use this tool when the user asks to translate, convert, or localize selected text."
```
:::

Error output: The AI model cannot distinguish between tools and may invoke the wrong one, or fail to invoke any tool at all.

If your plugin registers multiple tools, make sure each description clearly differentiates that tool from the others. The model will read all descriptions simultaneously when deciding which tool to call.

## Writing good `prompt` values in `func.call` {#prompt-in-func-call}

Inside `func.call`, you construct the prompt string that is passed as the first argument to `chatRequest`. This prompt is different from the registration metadata — it is the actual instruction sent to the language model for a specific invocation.

**Include relevant context in the prompt.** If your tool operates on selected text, embed that text directly:

```ts
async call(params) {
  const selectedText = await Asc.Editor.callMethod("GetSelectedText");
  const prompt = `Translate the following text into ${params.language}:\n\n${selectedText}`;
  await chatRequest(prompt, null, (chunk) => {
    // handle streaming response
  });
}
```

**Guidelines for constructing the prompt string:**

- Provide the task instruction first, then the content to act on.
- Label sections clearly: use prefixes like `"Text to translate:"` or `"Cell value:"` before inserting variable content.
- Keep the prompt as short as possible while still providing all necessary context — every token costs money and adds latency.
- Avoid repeating instructions that are already covered in the `description` field; the model sees both.
- If the task involves multiple steps, use numbered instructions in the prompt string to guide the model.

**Example — summarize a paragraph:**

```ts
const text = await Asc.Editor.callMethod("GetSelectedText");
const prompt = [
  "Summarize the following paragraph in one sentence.",
  "Return only the summary with no additional explanation.",
  "",
  `Paragraph:\n${text}`
].join("\n");
```

## Common mistakes {#common-mistakes}

- **Vague descriptions causing wrong tool selection.** If two tools have similar descriptions, the model may call the wrong one. Make each description distinctly different.
- **Too few examples.** A single example leaves the model with little signal about how to handle variations in phrasing.
- **Prompts without context.** Sending `"Translate this"` without the actual selected text forces the model to guess or produce an error.
- **Over-long prompts.** Embedding entire documents when only one paragraph is relevant inflates token usage and can degrade response quality.
- **Ignoring optional parameters in examples.** If you never show the model that a parameter can be omitted, it may always try to infer a value even when none was provided.
