---
sidebar_position: 3
---

# Plugin lifecycle

## Overview

Understanding the plugin lifecycle is crucial for developing robust and efficient OnlyOffice plugins. The lifecycle defines how a plugin is loaded, initialized, executed, and terminated within the OnlyOffice editor environment.

## Lifecycle Stages

The OnlyOffice plugin lifecycle consists of several distinct stages:

```
Registration → Loading → Initialization → Active State → Termination
```

## 1. Registration Stage

### Plugin Discovery

When OnlyOffice starts, it scans the plugins directory and reads each plugin's `config.json` file. During registration:

- The editor parses the configuration file
- Plugin metadata is validated
- Icons and UI elements are prepared
- The plugin appears in the Plugins menu

### Configuration Validation

OnlyOffice checks for required fields:

- `name`: Plugin display name
- `guid`: Unique identifier
- `version`: Plugin version number
- `variations`: Array of plugin variations
- Editor support requirements

If validation fails, the plugin won't be registered and won't appear in the interface.

## 2. Loading Stage

### User Activation

The loading stage begins when a user clicks on the plugin in the Plugins menu or tab. During this stage:

1. **Resource Loading**: OnlyOffice loads the plugin's HTML file specified in `config.json`
2. **Iframe Creation**: An iframe is created to host the plugin's interface
3. **Script Execution**: JavaScript files are loaded and executed
4. **Style Application**: CSS files are applied to the plugin interface

### Plugin Variation Selection

If multiple variations exist, OnlyOffice selects the appropriate one based on:

- Current editor type (word, cell, or slide)
- User preferences
- System capabilities
- Document state (editing vs. viewing)

## 3. Initialization Stage

### The init() Method

The core of the initialization stage is the `window.Asc.plugin.init()` method, which is automatically called by OnlyOffice when the plugin is ready:

```javascript
window.Asc.plugin.init = function (data) {
  // data contains initialization parameters

  // Access init data if specified in config
  var initData = data;

  // Set up event listeners
  setupEventListeners();

  // Initialize plugin state
  initializePluginState();

  // Prepare UI elements
  setupUserInterface();

  // Notify that initialization is complete
  console.log("Plugin initialized successfully");
};
```

### Initialization Data

The `data` parameter passed to `init()` contains information specified in the plugin's configuration:

```json
{
  "variations": [
    {
      "initDataType": "text",
      "initData": "Initial configuration data"
    }
  ]
}
```

Available `initDataType` values:

- `"none"`: No initialization data
- `"text"`: Plain text data
- `"html"`: HTML content
- `"ole"`: OLE object data

### Setup Tasks During Initialization

Common initialization tasks include:

1. **UI Setup**: Render forms, buttons, and interactive elements
2. **State Management**: Initialize variables and data structures
3. **Event Binding**: Attach event listeners to UI elements
4. **API Preparation**: Set up communication with the editor
5. **External Resources**: Load data from APIs or local storage

## 4. Active State

### Plugin Operation

During the active state, the plugin is fully functional and can:

#### Interact with the Document

```javascript
// Insert text at cursor position
window.Asc.plugin.executeMethod("PasteText", ["Hello World!"]);

// Get current selection
window.Asc.plugin.executeMethod("GetSelectedText", [], function (data) {
  console.log("Selected text:", data);
});

// Insert an image
window.Asc.plugin.executeMethod("PasteImage", [imageUrl]);
```

#### Respond to User Actions

```javascript
// Handle button clicks in the plugin UI
document.getElementById("submitBtn").addEventListener("click", function () {
  var userInput = document.getElementById("textInput").value;
  processUserInput(userInput);
});

// Handle button clicks defined in config.json
window.Asc.plugin.button = function (id) {
  if (id === 0) {
    // First button clicked (usually OK/Submit)
    handleSubmit();
  } else if (id === -1) {
    // Cancel button clicked
    window.Asc.plugin.executeCommand("close", "");
  }
};
```

#### Execute Editor Methods

The plugin can call various API methods:

```javascript
// Document manipulation
window.Asc.plugin.executeMethod("Method", [params], callback);

// Common methods:
// - PasteText: Insert text
// - PasteHtml: Insert HTML content
// - GetCurrentWord: Get word at cursor
// - GetSelectedText: Get selected text
// - InsertAndReplace: Replace selection with new content
// - PasteImage: Insert image
```

#### Handle Events

```javascript
// Listen for editor events
window.Asc.plugin.event_onTargetPositionChanged = function () {
  // Cursor position changed
  updatePluginState();
};

window.Asc.plugin.event_onDocumentContentReady = function () {
  // Document is ready for manipulation
  performDocumentOperations();
};
```

### Communication Patterns

#### Synchronous Operations

Some operations execute immediately:

```javascript
window.Asc.plugin.executeCommand("close", "");
window.Asc.plugin.executeCommand("resize", { width: 400, height: 300 });
```

#### Asynchronous Operations

Many operations require callbacks:

```javascript
window.Asc.plugin.executeMethod("GetSelectedText", [], function (data) {
  // Process the returned data
  if (data) {
    displayText(data);
  }
});
```

### State Management

Plugins should manage their own state throughout the active phase:

```javascript
var pluginState = {
  initialized: false,
  currentData: null,
  settings: {},
  history: [],
};

function updateState(newData) {
  pluginState.currentData = newData;
  pluginState.history.push({
    timestamp: Date.now(),
    data: newData,
  });
}
```

## 5. Termination Stage

### Closing the Plugin

The termination stage begins when:

- The user closes the plugin manually
- The plugin executes a close command
- The document is closed
- OnlyOffice is shut down

### Cleanup Operations

Before termination, plugins should:

```javascript
window.Asc.plugin.onExternalMouseUp = function () {
  // Clean up when user clicks outside plugin
};

// Perform cleanup before closing
function cleanup() {
  // Save state if needed
  savePluginState();

  // Remove event listeners
  removeEventListeners();

  // Clear intervals and timeouts
  clearInterval(intervalId);

  // Release resources
  releaseResources();
}

// Close the plugin
function closePlugin() {
  cleanup();
  window.Asc.plugin.executeCommand("close", "");
}
```

### Modal vs. Modeless Lifecycle

#### Modal Plugins

```json
{
  "isModal": true,
  "buttons": [
    { "text": "OK", "primary": true },
    { "text": "Cancel", "primary": false }
  ]
}
```

- Block interaction with the document
- Typically have OK/Cancel buttons
- Terminate when a button is clicked

#### Modeless Plugins

```json
{
  "isModal": false,
  "isInsideMode": false
}
```

- Allow concurrent document editing
- Can remain open indefinitely
- User explicitly closes them

## Lifecycle Best Practices

### 1. Efficient Initialization

```javascript
window.Asc.plugin.init = function (data) {
  // Quick initialization
  loadCriticalResources();

  // Defer non-critical tasks
  setTimeout(function () {
    loadOptionalResources();
  }, 100);
};
```

### 2. Proper Resource Management

```javascript
var resourceHandles = [];

function loadResource(url) {
  var handle = createResourceHandle(url);
  resourceHandles.push(handle);
  return handle;
}

function cleanup() {
  resourceHandles.forEach(function (handle) {
    handle.release();
  });
  resourceHandles = [];
}
```

### 3. Error Handling

```javascript
window.Asc.plugin.init = function (data) {
  try {
    initializePlugin(data);
  } catch (error) {
    console.error("Plugin initialization failed:", error);
    displayErrorMessage("Failed to initialize plugin");
  }
};

function safeExecuteMethod(method, params, callback) {
  try {
    window.Asc.plugin.executeMethod(method, params, function (result) {
      if (callback) {
        callback(result);
      }
    });
  } catch (error) {
    console.error("Method execution failed:", error);
  }
}
```

### 4. State Persistence

```javascript
// Save state before closing
window.addEventListener("beforeunload", function () {
  localStorage.setItem("pluginState", JSON.stringify(pluginState));
});

// Restore state on init
window.Asc.plugin.init = function (data) {
  var savedState = localStorage.getItem("pluginState");
  if (savedState) {
    pluginState = JSON.parse(savedState);
  }
};
```

### 5. Performance Optimization

```javascript
// Debounce frequent operations
function debounce(func, wait) {
  var timeout;
  return function () {
    var context = this,
      args = arguments;
    clearTimeout(timeout);
    timeout = setTimeout(function () {
      func.apply(context, args);
    }, wait);
  };
}

var debouncedUpdate = debounce(function (text) {
  updatePreview(text);
}, 300);

document.getElementById("input").addEventListener("input", function (e) {
  debouncedUpdate(e.target.value);
});
```

## Lifecycle Events Timeline

Here's a visual representation of the complete plugin lifecycle:

```
User clicks plugin icon
        ↓
[REGISTRATION] - Plugin discovered and validated
        ↓
[LOADING] - Resources loaded, iframe created
        ↓
[INITIALIZATION] - init() called, setup complete
        ↓
[ACTIVE STATE]
├─→ User interactions
├─→ Document modifications
├─→ API calls
└─→ Event handling
        ↓
User closes plugin OR plugin executes close command
        ↓
[TERMINATION] - Cleanup, resources released
        ↓
Plugin removed from memory
```

## Common Lifecycle Issues and Solutions

### Issue 1: Plugin Doesn't Initialize

**Possible causes:**

- Invalid `config.json`
- JavaScript errors in initialization code
- Missing required files

**Solution:**

```javascript
window.Asc.plugin.init = function (data) {
  console.log("Init called with data:", data);
  try {
    // Your initialization code
  } catch (error) {
    console.error("Init error:", error);
  }
};
```

### Issue 2: Methods Not Responding

**Cause:** Calling methods before initialization completes

**Solution:**

```javascript
var pluginReady = false;

window.Asc.plugin.init = function (data) {
  // Initialization code
  pluginReady = true;
};

function safeOperation() {
  if (!pluginReady) {
    console.warn("Plugin not ready yet");
    return;
  }
  window.Asc.plugin.executeMethod("PasteText", ["Hello"]);
}
```

### Issue 3: Memory Leaks

**Cause:** Not cleaning up event listeners and timers

**Solution:**

```javascript
var intervals = [];
var listeners = [];

function addInterval(callback, delay) {
  var id = setInterval(callback, delay);
  intervals.push(id);
  return id;
}

function cleanup() {
  intervals.forEach(clearInterval);
  listeners.forEach(function (listener) {
    listener.element.removeEventListener(listener.type, listener.handler);
  });
}
```

## Conclusion

The plugin lifecycle is a well-defined process that ensures plugins operate smoothly within OnlyOffice. By understanding each stage and following best practices, you can create plugins that are reliable, performant, and provide an excellent user experience. Remember to initialize properly, manage resources efficiently, and clean up thoroughly to create professional-quality plugins.
