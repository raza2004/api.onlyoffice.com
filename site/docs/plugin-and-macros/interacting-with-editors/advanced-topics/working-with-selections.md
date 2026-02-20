---
sidebar_position: -4
---

# Working with selections

## Overview

Working with text and object selections is a fundamental aspect of ONLYOFFICE plugin development. This guide covers how to detect, manipulate, and respond to user selections across different editor types.

## Understanding selections

Selections represent the content currently highlighted by the user in the document, spreadsheet, or presentation. Plugins can:

- **Read selection content** - Get the text, format, or objects the user has selected
- **Modify selections** - Change the selected content programmatically
- **React to changes** - Respond when the user changes their selection
- **Replace selections** - Insert new content in place of the selected content

## Getting selected content

### Getting selected text

The most common operation is retrieving the text the user has selected:

```javascript
window.Asc.plugin.executeMethod("GetSelectedText", [], function (text) {
  if (text && text.trim() !== "") {
    console.log("Selected text:", text);
    // Process the selected text
  } else {
    console.log("No text selected");
  }
});
```

### Getting selection range

To get more detailed information about the selection:

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();
  const range = doc.GetRangeBySelect();

  if (range) {
    const text = range.GetText();
    const length = text.length;
    console.log(`Selected: ${length} characters`);
    return { text: text, length: length };
  }
  return null;
}, false);
```

### Getting formatted content

To retrieve selection with formatting information:

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();
  const range = doc.GetRangeBySelect();

  if (range) {
    // Get text properties
    const textPr = range.GetTextPr();

    return {
      text: range.GetText(),
      isBold: textPr.GetBold(),
      isItalic: textPr.GetItalic(),
      fontSize: textPr.GetFontSize(),
    };
  }
  return null;
}, false);
```

## Responding to selection changes

### Using the onSelectionChanged event

Monitor when the user changes their selection:

```javascript
window.Asc.plugin.attachEvent("onSelectionChanged", function (selection) {
  if (selection && selection.text) {
    console.log("New selection:", selection.text);
    updatePluginUI(selection.text);
  } else {
    console.log("Selection cleared");
    clearPluginUI();
  }
});
```

### Auto-opening on selection

Configure your plugin to open automatically when text is selected:

**config.json:**

```json
{
  "initOnSelectionChanged": true,
  "initDataType": "text"
}
```

**plugin.js:**

```javascript
window.Asc.plugin.init = function (selectedText) {
  if (selectedText && selectedText.trim()) {
    // Plugin opened with selected text
    document.getElementById("input").value = selectedText;
    processText(selectedText);
  }
};
```

## Modifying selections

### Replacing selected text

Replace the current selection with new content:

```javascript
function replaceSelection(newText) {
  window.Asc.plugin.executeMethod("PasteText", [newText], function () {
    console.log("Selection replaced successfully");
  });
}

// Example: Convert selection to uppercase
window.Asc.plugin.executeMethod("GetSelectedText", [], function (text) {
  if (text) {
    const upperText = text.toUpperCase();
    window.Asc.plugin.executeMethod("PasteText", [upperText]);
  }
});
```

### Replacing with formatted content

Replace selection with HTML or formatted text:

```javascript
function replaceWithFormatted(htmlContent) {
  window.Asc.plugin.executeMethod("PasteHtml", [htmlContent], function () {
    console.log("Formatted content inserted");
  });
}

// Example: Bold the selected text
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();
  const range = doc.GetRangeBySelect();

  if (range) {
    const textPr = Api.CreateTextPr();
    textPr.SetBold(true);
    range.SetTextPr(textPr);
  }
}, false);
```

### Wrapping selections

Wrap selected text with additional content:

```javascript
window.Asc.plugin.executeMethod("GetSelectedText", [], function (text) {
  if (text) {
    const wrapped = `**${text}**`; // Wrap in bold markdown
    window.Asc.plugin.executeMethod("PasteText", [wrapped]);
  }
});
```

## Working with different editor types

### Document editor selections

In word processor documents, selections can span:

- Text characters
- Multiple paragraphs
- Tables and cells
- Images and shapes

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();
  const range = doc.GetRangeBySelect();

  if (range) {
    // Check if selection contains a table
    const parent = range.GetParent();
    if (parent && parent.GetClassType() === "docTable") {
      console.log("Selection is in a table");
    }
  }
}, false);
```

### Spreadsheet selections

In spreadsheets, selections work with cells and ranges:

```javascript
window.Asc.plugin.callCommand(function () {
  const sheet = Api.GetActiveSheet();
  const selection = sheet.GetSelection();

  // Get selected range address (e.g., "A1:C5")
  const address = selection.GetAddress();

  // Get values from selected cells
  const values = selection.GetValue();

  return { address: address, values: values };
}, false);
```

### Presentation selections

In presentations, selections can include:

- Text in shapes
- Entire shapes
- Multiple objects
- Slide content

```javascript
window.Asc.plugin.callCommand(function () {
  const presentation = Api.GetPresentation();
  const currentSlide = presentation.GetCurrentSlide();

  // Get all shapes on the current slide
  const shapes = currentSlide.GetAllShapes();

  // Work with selected shapes
  shapes.forEach(function (shape) {
    const content = shape.GetDocContent();
    // Process shape content
  });
}, false);
```

## Advanced selection techniques

### Multi-selection handling

Handle multiple non-contiguous selections:

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();
  const allRanges = doc.GetAllRanges();

  const results = [];
  allRanges.forEach(function (range) {
    results.push({
      text: range.GetText(),
      start: range.GetStart(),
      end: range.GetEnd(),
    });
  });

  return results;
}, false);
```

### Selection boundaries

Get information about selection start and end positions:

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();
  const range = doc.GetRangeBySelect();

  if (range) {
    return {
      start: range.GetStart(),
      end: range.GetEnd(),
      length: range.GetEnd() - range.GetStart(),
    };
  }
  return null;
}, false);
```

### Expanding selections

Programmatically expand or modify the selection:

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();
  const range = doc.GetRangeBySelect();

  if (range) {
    // Select the entire paragraph containing the selection
    const paragraph = range.GetParagraph();
    if (paragraph) {
      const paraRange = paragraph.GetRange();
      doc.Select(paraRange);
    }
  }
}, false);
```

## Practical examples

### Word counter for selection

```javascript
function countWordsInSelection() {
  window.Asc.plugin.executeMethod("GetSelectedText", [], function (text) {
    if (text && text.trim()) {
      const words = text.trim().split(/\s+/).length;
      const characters = text.length;
      const charactersNoSpaces = text.replace(/\s/g, "").length;

      displayStats({
        words: words,
        characters: characters,
        charactersNoSpaces: charactersNoSpaces,
      });
    }
  });
}
```

### Text transformation tool

```javascript
function transformSelection(transformType) {
  window.Asc.plugin.executeMethod("GetSelectedText", [], function (text) {
    if (!text) return;

    let transformed;
    switch (transformType) {
      case "uppercase":
        transformed = text.toUpperCase();
        break;
      case "lowercase":
        transformed = text.toLowerCase();
        break;
      case "titlecase":
        transformed = text
          .toLowerCase()
          .replace(/\b\w/g, (c) => c.toUpperCase());
        break;
      case "reverse":
        transformed = text.split("").reverse().join("");
        break;
    }

    window.Asc.plugin.executeMethod("PasteText", [transformed]);
  });
}
```

### Selection highlighter

```javascript
function highlightSelection(color) {
  window.Asc.plugin.callCommand(function () {
    const doc = Api.GetDocument();
    const range = doc.GetRangeBySelect();

    if (range) {
      const textPr = Api.CreateTextPr();
      textPr.SetHighlight(color);
      range.SetTextPr(textPr);
    }
  }, false);
}

// Usage
highlightSelection("yellow");
```

### Format painter

```javascript
let copiedFormat = null;

function copyFormat() {
  window.Asc.plugin.callCommand(function () {
    const doc = Api.GetDocument();
    const range = doc.GetRangeBySelect();

    if (range) {
      copiedFormat = range.GetTextPr();
      return "Format copied";
    }
    return "No selection";
  }, false);
}

function pasteFormat() {
  if (!copiedFormat) {
    console.log("No format copied");
    return;
  }

  window.Asc.plugin.callCommand(function () {
    const doc = Api.GetDocument();
    const range = doc.GetRangeBySelect();

    if (range && Asc.scope.copiedFormat) {
      range.SetTextPr(Asc.scope.copiedFormat);
    }
  }, false);
}
```

## Best practices

### Always check for empty selections

```javascript
window.Asc.plugin.executeMethod("GetSelectedText", [], function (text) {
  if (!text || text.trim() === "") {
    showMessage("Please select some text first");
    return;
  }

  // Process the selection
  processText(text);
});
```

### Provide user feedback

```javascript
window.Asc.plugin.executeMethod("GetSelectedText", [], function (text) {
  if (text) {
    showLoadingIndicator();
    processSelection(text).then(() => {
      hideLoadingIndicator();
      showSuccessMessage("Selection processed successfully");
    });
  }
});
```

### Handle selection changes gracefully

```javascript
let lastSelection = null;

window.Asc.plugin.attachEvent("onSelectionChanged", function (selection) {
  // Debounce to avoid excessive processing
  clearTimeout(window.selectionTimeout);

  window.selectionTimeout = setTimeout(() => {
    if (selection && selection.text !== lastSelection) {
      lastSelection = selection.text;
      updatePluginForSelection(selection.text);
    }
  }, 300);
});
```

### Clean up on plugin close

```javascript
window.Asc.plugin.button = function (id) {
  if (id === -1) {
    // Clear any stored selection data
    lastSelection = null;
    copiedFormat = null;

    // Close the plugin
    window.Asc.plugin.executeCommand("close", "");
  }
};
```

## Troubleshooting

### Selection not detected

**Problem:** `GetSelectedText` returns empty or null

**Solutions:**

- Ensure text is actually selected in the document
- Check if the plugin has proper permissions
- Verify the editor is in edit mode (not viewer mode)

### Selection changes not triggering events

**Problem:** `onSelectionChanged` event not firing

**Solutions:**

- Verify event is attached using `attachEvent` method
- Check if `initOnSelectionChanged` is set correctly in config
- Ensure the plugin is properly initialized

### Formatting lost when replacing

**Problem:** Text loses formatting when replaced

**Solutions:**

- Use `PasteHtml` instead of `PasteText` for formatted content
- Use `callCommand` with `SetTextPr` to preserve formatting
- Store original format before replacement and reapply

## Conclusion

Working with selections is essential for creating interactive and useful ONLYOFFICE plugins. By understanding how to detect, read, and manipulate selections, you can build plugins that seamlessly integrate with user workflows and enhance document editing capabilities.
