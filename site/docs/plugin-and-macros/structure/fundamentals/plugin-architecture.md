---
sidebar_position: -1
---

# Plugin architecture

## Overview

ONLYOFFICE plugins follow a modular architecture that separates the plugin interface from the core editor functionality. Understanding this architecture is essential for building robust and efficient plugins.

## Architectural components

### 1. Plugin container (iframe)

Each plugin runs in an isolated iframe environment within the ONLYOFFICE editor. This provides:

- **Security isolation** - Plugins cannot directly access the editor's internal state
- **Cross-platform compatibility** - Plugins work consistently across different platforms
- **Independent execution** - Plugin errors don't crash the main editor

```
┌─────────────────────────────────────┐
│   ONLYOFFICE Editor (Main Window)  │
│  ┌───────────────────────────────┐ │
│  │   Plugin Iframe (Sandboxed)   │ │
│  │   ┌─────────────────────────┐ │ │
│  │   │  Plugin UI (HTML/CSS)   │ │ │
│  │   │  Plugin Logic (JS)      │ │ │
│  │   └─────────────────────────┘ │ │
│  └───────────────────────────────┘ │
└─────────────────────────────────────┘
```

### 2. Communication layer (api bridge)

Plugins communicate with the editor through the `window.Asc.plugin` API interface:

```javascript
// Plugin → Editor
window.Asc.plugin.executeMethod("PasteText", ["Hello"]);

// Editor → Plugin
window.Asc.plugin.init = function (data) {
  // Initialization code
};
```

### 3. File system structure

A typical plugin consists of these core components:

```
my-plugin/
├── config.json          # Plugin configuration and metadata
├── index.html           # User interface
├── plugin.js            # Business logic
├── styles.css           # Styling
└── assets/              # Icons, images, resources
    └── icons/
        ├── icon.png
        └── icon@2x.png
```

## Key architectural principles

### Separation of concerns

ONLYOFFICE plugins follow the MVC pattern:

- **View (HTML/CSS)** - User interface and presentation
- **Controller (JavaScript)** - Business logic and API interaction
- **Model (Config)** - Plugin metadata and configuration

### Event-driven communication

Plugins operate on an event-driven model:

```javascript
// Editor loads plugin
window.Asc.plugin.init = function (data) {
  console.log("Plugin initialized");
};

// User interacts with editor
window.Asc.plugin.onSelectionChanged = function (selection) {
  console.log("Selection changed:", selection);
};

// Plugin responds to button clicks
window.Asc.plugin.button = function (id) {
  if (id === 0) {
    // Handle OK button
  }
};
```

### Stateless design

Plugins should be designed to be stateless between sessions:

- **No persistent storage by default** - Use localStorage if needed
- **Clean initialization** - Don't assume previous state
- **Self-contained** - Include all dependencies

## Plugin types

### Modal plugins

Modal plugins block interaction with the document:

```json
{
  "isModal": true,
  "isVisual": true,
  "buttons": [{ "text": "OK", "primary": true }, { "text": "Cancel" }]
}
```

**Use for:** Forms, configuration dialogs, one-time actions

### Panel plugins

Panel plugins run alongside the document:

```json
{
  "isModal": false,
  "isInsideMode": true,
  "type": "panel"
}
```

**Use for:** Continuous workflows, tools, reference panels

### Background plugins

Background plugins run without UI:

```json
{
  "isVisual": false,
  "isModal": false
}
```

**Use for:** Automated tasks, event-driven operations

## Data flow

### Input flow

```
User Action → Plugin UI → JavaScript Handler → API Call → Editor
```

Example:

```javascript
<button onclick="insertText()">Insert</button>;

function insertText() {
  const text = document.getElementById("input").value;
  window.Asc.plugin.executeMethod("PasteText", [text]);
}
```

### Output flow

```
Editor Event → API Callback → Plugin Handler → UI Update
```

Example:

```javascript
window.Asc.plugin.onSelectionChanged = function (selection) {
  if (selection && selection.text) {
    document.getElementById("preview").textContent = selection.text;
  }
};
```

## Security architecture

### Sandboxing

Plugins are sandboxed to prevent:

- Direct file system access
- Unauthorized network requests
- Access to other plugins or editor internals
- XSS attacks on the main editor

### Permissions model

Plugins can only perform actions explicitly allowed by the API:

**Error name:** Direct DOM manipulation of editor content

:::warning[Wrong]
```javascript
document.querySelector(".editor-content").innerHTML = "Unsafe";
```
:::

:::tip[Correct]
```javascript
window.Asc.plugin.executeMethod("PasteText", ["Safe content"]);
```
:::

Error output: Direct DOM access is blocked by the plugin sandbox and has no effect on the editor content.

## Performance best practices

### Efficient initialization

```javascript
window.Asc.plugin.init = function (data) {
  // Quick initialization
  setupUI();

  // Defer heavy operations
  setTimeout(loadHeavyResources, 100);
};
```

### Memory management

```javascript
function cleanup() {
  clearInterval(updateInterval);
  document.removeEventListener("click", handler);
}

window.Asc.plugin.button = function (id) {
  if (id === -1) {
    cleanup();
  }
};
```

## Multi-editor support

Plugins can support multiple editor types:

```json
{
  "EditorsSupport": ["word", "cell", "slide"]
}
```

Detect and adapt to the current editor:

```javascript
window.Asc.plugin.init = function (data) {
  const editorType = getEditorType(); // word, cell, or slide

  if (editorType === "word") {
    enableWordFeatures();
  } else if (editorType === "cell") {
    enableSpreadsheetFeatures();
  }
};
```

## Extension points

ONLYOFFICE provides several extension points:

1. **API Methods** - Predefined functions to manipulate content
2. **Events** - React to editor and user actions
3. **UI Integration** - Toolbars, panels, modals
4. **Document Builder API** - Direct document manipulation

## Code organization best practices

### Modular design

```javascript
// Separate concerns into modules
const UI = {
  init: function () {
    /* UI setup */
  },
  update: function () {
    /* UI updates */
  },
};

const API = {
  insertText: function (text) {
    /* API calls */
  },
  getSelection: function () {
    /* API calls */
  },
};

const State = {
  data: {},
  update: function (newData) {
    /* State management */
  },
};
```

### Error handling

```javascript
window.Asc.plugin.executeMethod("PasteText", [text], function (result) {
  if (result === undefined || result === null) {
    console.error("Failed to paste text");
    showErrorMessage("Operation failed");
  }
});
```

### Responsive design

```css
@media (max-width: 400px) {
  .plugin-container {
    padding: 10px;
    font-size: 12px;
  }
}
```

## Conclusion

Understanding the ONLYOFFICE plugin architecture enables you to build secure, performant, and user-friendly plugins. By following these architectural best practices and leveraging the provided API, you can create powerful extensions that seamlessly integrate with the editor ecosystem.
