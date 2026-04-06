---
sidebar_position: 4
---

# Use cases and examples

This page shows practical scenarios for the AI agent across different editor types.

## Text document editor

![Text document editor use case](/assets/images/plugins/ai-agent-prompt.png#gh-light-mode-only)![Text document editor use case](/assets/images/plugins/ai-agent-prompt.dark.png#gh-dark-mode-only)

**Annotating content**

Select a paragraph and type:
> "Explain this text and add it as a comment"

The agent calls the `commentText` tool, sends the selected text to the AI model, and inserts the response as a document comment.

**Rewriting for tone**

Select a formal paragraph and type:
> "Rewrite this in a casual, friendly tone"

The agent replaces the selected text with the rewritten version.

**Spell and grammar check**

Type:
> "Check the spelling and grammar of this paragraph"

The agent reviews the text and applies corrections using track changes so you can review each change before accepting.

## Spreadsheet editor

![Spreadsheet editor use case](/assets/images/plugins/ai-use-case-spreadsheet.png#gh-light-mode-only)![Spreadsheet editor use case](/assets/images/plugins/ai-use-case-spreadsheet.dark.png#gh-dark-mode-only)

**Chart generation**

Select a data range and type:
> "Create a bar chart from this data"

The agent calls the chart tool with the selected range and inserts a chart into the sheet.

**Formula explanation**

Click a cell with a complex formula and type:
> "Explain what this formula does"

The agent adds a comment to the cell explaining the formula's logic in plain language.

**Sorting and filtering**

Type:
> "Sort this table by the Revenue column in descending order"

The agent applies the sort without you needing to open any menus.

## Presentation editor

![Presentation editor use case](/assets/images/plugins/ai-use-case-presentation.png#gh-light-mode-only)![Presentation editor use case](/assets/images/plugins/ai-use-case-presentation.dark.png#gh-dark-mode-only)

**Adding slides**

Type:
> "Add a new slide at the end of the presentation"

The agent inserts a blank slide with the default layout.

**Adding content to slides**

Type:
> "Add a table with 3 columns and 4 rows to the current slide"

The agent creates and positions the table on the active slide.

**Changing slide backgrounds**

Type:
> "Change the background of this slide to a dark blue gradient"

The agent applies the background change using the Office API.

## Multi-step conversation example

The agent maintains conversation history, enabling iterative refinement:

1. "Summarize the selected text" → agent inserts a summary
2. "Make it shorter" → agent revises the summary
3. "Add it as a footnote instead" → agent moves the summary to a footnote

Press `Ctrl + Alt + /` to reset the history and start a new conversation.
