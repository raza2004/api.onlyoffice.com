---
sidebar_position: 5
---

# Your first plugin

## Overview

In this guide, you'll create a complete, functional OnlyOffice plugin from scratch. We'll build a "Text Transformer" plugin that demonstrates core concepts like user interface, API interaction, and document manipulation. By the end, you'll have a working plugin and understand how to expand it with your own features.

## What We'll Build

**Text Transformer Plugin** - A tool that:

- Gets selected text from the document
- Transforms it (uppercase, lowercase, title case, reverse)
- Inserts the transformed text back into the document
- Provides a clean, user-friendly interface

## Prerequisites

Before starting, make sure you have:

- OnlyOffice Desktop Editors installed
- A text editor (VS Code recommended)
- Basic knowledge of HTML, CSS, and JavaScript
- Your development environment set up (see "Development Environment Setup" guide)

## Step-by-Step Tutorial

### Step 1: Create the Plugin Directory Structure

First, create a folder for your plugin in your development workspace:

```bash
# Navigate to your development directory
cd ~/onlyoffice-plugins-dev

# Create plugin folder and structure
mkdir text-transformer
cd text-transformer
mkdir assets
mkdir assets/icons
```

Your folder structure should look like this:

```
text-transformer/
├── assets/
│   └── icons/
│       ├── icon.png
│       └── icon@2x.png
├── config.json
├── index.html
├── plugin.js
└── styles.css
```

### Step 2: Create the Plugin Icon

For this tutorial, you'll need a plugin icon. Create a 48x48 pixel PNG image named `icon.png` and save it in `assets/icons/`.

**Quick Options:**

1. Download a free icon from [icons8.com](https://icons8.com) or [flaticon.com](https://flaticon.com)
2. Use [canva.com](https://www.canva.com) to create a simple icon
3. For now, use any small PNG image as a placeholder

Save it as:

- `assets/icons/icon.png` (48x48 pixels for standard displays)
- `assets/icons/icon@2x.png` (96x96 pixels for retina displays - optional)

### Step 3: Create the Configuration File

Create `config.json` in the root of your plugin folder:

```json
{
  "name": "Text Transformer",
  "guid": "asc.{text-transformer-plugin-001}",
  "version": "1.0.0",
  "variations": [
    {
      "description": "Transform selected text in various ways",
      "url": "index.html",
      "icons": ["assets/icons/icon.png", "assets/icons/icon@2x.png"],
      "isViewer": false,
      "EditorsSupport": ["word", "cell", "slide"],
      "isVisual": true,
      "isModal": true,
      "isInsideMode": false,
      "initDataType": "none",
      "initData": "",
      "isUpdateOleOnResize": false,
      "buttons": [
        {
          "text": "Apply",
          "primary": true
        },
        {
          "text": "Cancel",
          "primary": false
        }
      ],
      "size": [350, 250]
    }
  ]
}
```

**Understanding the Configuration:**

- **name**: The plugin name shown to users
- **guid**: A unique identifier (change this for your own plugins!)
- **version**: Your plugin version (use semantic versioning like 1.0.0)
- **url**: The HTML file to load when the plugin starts
- **icons**: Array of icon paths (standard and retina versions)
- **isViewer**: `false` means the plugin only works in edit mode
- **EditorsSupport**: Which editors support this plugin (`word`, `cell`, `slide`)
- **isModal**: `true` means it blocks interaction with the document
- **buttons**: Buttons shown in the plugin footer
- **size**: Default width and height in pixels `[width, height]`

### Step 4: Create the Plugin Styles

Create `styles.css`:

```css
/* Reset and base styles */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family:
    -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue",
    Arial, sans-serif;
  font-size: 14px;
  color: #333;
  padding: 20px;
  background-color: #fff;
}

.container {
  max-width: 100%;
}

h1 {
  font-size: 18px;
  font-weight: 600;
  margin-bottom: 16px;
  color: #444;
}

.info {
  background-color: #e8f4fd;
  border-left: 3px solid #2d9ee0;
  padding: 12px;
  margin-bottom: 20px;
  font-size: 13px;
  line-height: 1.5;
}

.preview-section {
  margin-bottom: 20px;
}

.preview-label {
  font-size: 12px;
  font-weight: 600;
  color: #666;
  margin-bottom: 8px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.preview-box {
  background-color: #f5f5f5;
  border: 1px solid #ddd;
  border-radius: 4px;
  padding: 12px;
  min-height: 60px;
  max-height: 100px;
  overflow-y: auto;
  font-family: "Courier New", monospace;
  font-size: 13px;
  line-height: 1.6;
  word-wrap: break-word;
}

.preview-box.empty {
  color: #999;
  font-style: italic;
  font-family:
    -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
}

.button-group {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}

.transform-btn {
  padding: 10px 16px;
  background-color: #fff;
  border: 1px solid #ccc;
  border-radius: 4px;
  cursor: pointer;
  font-size: 13px;
  font-weight: 500;
  transition: all 0.2s ease;
  color: #333;
}

.transform-btn:hover {
  background-color: #f0f0f0;
  border-color: #999;
}

.transform-btn:active {
  background-color: #e0e0e0;
  transform: translateY(1px);
}

.transform-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.warning {
  background-color: #fff3cd;
  border-left: 3px solid #ffc107;
  padding: 12px;
  font-size: 13px;
  margin-top: 16px;
  display: none;
}

.warning.show {
  display: block;
}
```

### Step 5: Create the HTML Interface

Create `index.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Text Transformer</title>
    <link rel="stylesheet" href="styles.css" />
  </head>
  <body>
    <div class="container">
      <h1>Text Transformer</h1>

      <div class="info">
        Select text in your document, then choose a transformation below.
      </div>

      <!-- Preview section -->
      <div class="preview-section">
        <div class="preview-label">Selected Text</div>
        <div id="previewBox" class="preview-box empty">No text selected</div>
      </div>

      <!-- Transformation buttons -->
      <div class="transform-section">
        <div class="preview-label">Choose Transformation</div>
        <div class="button-group">
          <button class="transform-btn" id="uppercaseBtn" disabled>
            UPPERCASE
          </button>
          <button class="transform-btn" id="lowercaseBtn" disabled>
            lowercase
          </button>
          <button class="transform-btn" id="titlecaseBtn" disabled>
            Title Case
          </button>
          <button class="transform-btn" id="reverseBtn" disabled>
            esreveR
          </button>
        </div>
      </div>

      <!-- Warning for no selection -->
      <div id="warningBox" class="warning">
        Please select some text in the document first.
      </div>
    </div>

    <script src="plugin.js"></script>
  </body>
</html>
```

### Step 6: Create the Plugin Logic

Create `plugin.js`:

```javascript
/**
 * Text Transformer Plugin
 * Transforms selected text in various ways
 */

(function (window, undefined) {
  // Plugin state
  let selectedText = "";
  let transformedText = "";

  // UI Elements
  let previewBox;
  let warningBox;
  let uppercaseBtn;
  let lowercaseBtn;
  let titlecaseBtn;
  let reverseBtn;

  /**
   * Initialize the plugin
   * Called automatically by OnlyOffice when plugin loads
   */
  window.Asc.plugin.init = function (data) {
    console.log("Text Transformer plugin initialized");

    // Initialize UI elements
    initializeUIElements();

    // Set up event listeners
    setupEventListeners();

    // Get the currently selected text
    getSelectedText();
  };

  /**
   * Initialize references to UI elements
   */
  function initializeUIElements() {
    previewBox = document.getElementById("previewBox");
    warningBox = document.getElementById("warningBox");
    uppercaseBtn = document.getElementById("uppercaseBtn");
    lowercaseBtn = document.getElementById("lowercaseBtn");
    titlecaseBtn = document.getElementById("titlecaseBtn");
    reverseBtn = document.getElementById("reverseBtn");
  }

  /**
   * Set up event listeners for transformation buttons
   */
  function setupEventListeners() {
    uppercaseBtn.addEventListener("click", function () {
      transformText("uppercase");
    });

    lowercaseBtn.addEventListener("click", function () {
      transformText("lowercase");
    });

    titlecaseBtn.addEventListener("click", function () {
      transformText("titlecase");
    });

    reverseBtn.addEventListener("click", function () {
      transformText("reverse");
    });
  }

  /**
   * Get the selected text from the document
   */
  function getSelectedText() {
    window.Asc.plugin.executeMethod("GetSelectedText", [], function (text) {
      if (text && text.trim() !== "") {
        selectedText = text;
        displaySelectedText(text);
        enableTransformButtons();
        hideWarning();
      } else {
        selectedText = "";
        displayNoSelection();
        disableTransformButtons();
        showWarning();
      }
    });
  }

  /**
   * Display the selected text in the preview box
   */
  function displaySelectedText(text) {
    previewBox.textContent = text;
    previewBox.classList.remove("empty");
  }

  /**
   * Display message when no text is selected
   */
  function displayNoSelection() {
    previewBox.textContent = "No text selected";
    previewBox.classList.add("empty");
  }

  /**
   * Enable all transformation buttons
   */
  function enableTransformButtons() {
    uppercaseBtn.disabled = false;
    lowercaseBtn.disabled = false;
    titlecaseBtn.disabled = false;
    reverseBtn.disabled = false;
  }

  /**
   * Disable all transformation buttons
   */
  function disableTransformButtons() {
    uppercaseBtn.disabled = true;
    lowercaseBtn.disabled = true;
    titlecaseBtn.disabled = true;
    reverseBtn.disabled = true;
  }

  /**
   * Show/hide warning message
   */
  function showWarning() {
    warningBox.classList.add("show");
  }

  function hideWarning() {
    warningBox.classList.remove("show");
  }

  /**
   * Transform text based on the selected type
   */
  function transformText(type) {
    if (!selectedText) {
      showWarning();
      return;
    }

    switch (type) {
      case "uppercase":
        transformedText = selectedText.toUpperCase();
        break;
      case "lowercase":
        transformedText = selectedText.toLowerCase();
        break;
      case "titlecase":
        transformedText = toTitleCase(selectedText);
        break;
      case "reverse":
        transformedText = reverseString(selectedText);
        break;
      default:
        transformedText = selectedText;
    }

    console.log("Text transformed:", type);
    previewBox.textContent = transformedText;
  }

  /**
   * Convert string to title case
   */
  function toTitleCase(str) {
    return str
      .toLowerCase()
      .split(" ")
      .map(function (word) {
        if (word.length === 0) return word;
        return word.charAt(0).toUpperCase() + word.slice(1);
      })
      .join(" ");
  }

  /**
   * Reverse a string
   */
  function reverseString(str) {
    return str.split("").reverse().join("");
  }

  /**
   * Handle button clicks from config.json
   * Button 0 = Apply, Button 1 = Cancel
   */
  window.Asc.plugin.button = function (id) {
    if (id === 0) {
      applyTransformation();
    } else {
      closePlugin();
    }
  };

  /**
   * Apply the transformation to the document
   */
  function applyTransformation() {
    if (transformedText && transformedText !== selectedText) {
      window.Asc.plugin.executeMethod(
        "PasteText",
        [transformedText],
        function () {
          console.log("Text successfully transformed and inserted");
          closePlugin();
        },
      );
    } else {
      closePlugin();
    }
  }

  /**
   * Close the plugin
   */
  function closePlugin() {
    window.Asc.plugin.executeCommand("close", "");
  }
})(window, undefined);
```

## Step 7: Deploy and Test Your Plugin

### Option 1: Using Symbolic Link (Recommended)

**macOS/Linux:**

```bash
ln -s ~/onlyoffice-plugins-dev/text-transformer /Applications/ONLYOFFICE.app/Contents/Resources/editors/sdkjs-plugins/text-transformer
```

**Windows (Run as Administrator):**

```cmd
mklink /D "C:\Program Files\ONLYOFFICE\DesktopEditors\editors\sdkjs-plugins\text-transformer" "C:\dev\text-transformer"
```

### Option 2: Copy Files Manually

Copy the entire `text-transformer` folder to your OnlyOffice plugins directory:

**Windows:**

```
C:\Program Files\ONLYOFFICE\DesktopEditors\editors\sdkjs-plugins\
```

**macOS:**

```
/Applications/ONLYOFFICE.app/Contents/Resources/editors/sdkjs-plugins/
```

**Linux:**

```
/opt/onlyoffice/desktopeditors/editors/sdkjs-plugins/
```

### Testing Steps

1. **Restart OnlyOffice** - Completely quit and relaunch OnlyOffice Desktop Editors
2. **Create a New Document** - Open Word, Spreadsheet, or Presentation
3. **Open Plugins Tab** - Look for the Plugins tab in the toolbar
4. **Find Your Plugin** - Look for "Text Transformer" in the plugins list
5. **Test the Plugin**:
   - Type some text in the document
   - Select the text
   - Click on "Text Transformer" plugin
   - Try different transformations
   - Click "Apply" to insert the transformed text

## Understanding the Code

### Key OnlyOffice API Methods Used

**1. window.Asc.plugin.init()**

```javascript
window.Asc.plugin.init = function (data) {
  // Called when plugin loads
  // Initialize your plugin here
};
```

**2. window.Asc.plugin.executeMethod()**

```javascript
window.Asc.plugin.executeMethod("GetSelectedText", [], function (result) {
  // Get data from the document
  console.log(result);
});
```

**3. window.Asc.plugin.button()**

```javascript
window.Asc.plugin.button = function (id) {
  // Handle button clicks from config.json
  // id 0 = first button, id -1 = cancel
};
```

**4. window.Asc.plugin.executeCommand()**

```javascript
window.Asc.plugin.executeCommand("close", "");
// Send commands to OnlyOffice
```

### Common API Methods

- `GetSelectedText` - Get currently selected text
- `PasteText` - Insert text at cursor
- `PasteHtml` - Insert HTML content
- `GetCurrentWord` - Get word at cursor
- `InsertAndReplace` - Replace selection

## Troubleshooting

### Plugin Doesn't Appear

**Check:**

- Config.json is valid JSON (use jsonlint.com)
- Plugin folder is in correct directory
- OnlyOffice was restarted
- GUID is unique

### Plugin Appears But Doesn't Load

**Check:**

- Browser console for errors (press `Ctrl+Shift+Alt+F12`)
- File paths in config.json are correct
- All files are present

### Transformations Don't Work

**Check:**

- Console for JavaScript errors
- Selected text is not empty
- Button event listeners are attached

## Extending Your Plugin

### Add More Transformations

```javascript
// Add to transformText function
case 'capitalize':
    transformedText = selectedText.charAt(0).toUpperCase() +
                     selectedText.slice(1).toLowerCase();
    break;

case 'alternating':
    transformedText = selectedText.split('').map((char, i) =>
        i % 2 === 0 ? char.toUpperCase() : char.toLowerCase()
    ).join('');
    break;
```

### Add Text Statistics

```javascript
function getTextStats(text) {
  return {
    characters: text.length,
    words: text.trim().split(/\s+/).length,
    sentences: text.split(/[.!?]+/).length - 1,
    paragraphs: text.split(/\n\n+/).length,
  };
}
```

### Save User Preferences

```javascript
// Save to localStorage
function savePreference(key, value) {
  localStorage.setItem("textTransformer_" + key, value);
}

// Load from localStorage
function loadPreference(key) {
  return localStorage.getItem("textTransformer_" + key);
}
```

## Next Steps

Congratulations! You've created your first OnlyOffice plugin. Here's what to explore next:

1. **Learn More API Methods** - Study the OnlyOffice API documentation
2. **Create More Complex UIs** - Try using frameworks like React or Vue
3. **Add External Integrations** - Connect to APIs and web services
4. **Publish Your Plugin** - Share it on the OnlyOffice Marketplace
5. **Join the Community** - Get help and share your work on the OnlyOffice forum

## Additional Resources

- **OnlyOffice Plugin API**: https://api.onlyoffice.com/plugin/basic
- **Example Plugins**: https://github.com/ONLYOFFICE/sdkjs-plugins
- **Developer Forum**: https://forum.onlyoffice.com/
- **Plugin Marketplace**: https://www.onlyoffice.com/app-directory

Happy plugin development!
