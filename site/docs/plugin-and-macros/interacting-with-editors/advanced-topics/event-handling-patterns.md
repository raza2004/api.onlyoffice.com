---
sidebar_position: -2
---

# Event handling patterns

## Overview

Event handling is central to creating interactive and responsive ONLYOFFICE plugins. This guide covers advanced patterns for managing events, implementing efficient event handlers, and creating robust event-driven architectures.

## Understanding ONLYOFFICE events

### Event types

ONLYOFFICE plugins can respond to several event types:

1. **Lifecycle events** - Plugin initialization and termination
2. **User interaction events** - Selection changes, button clicks
3. **Document events** - Content changes, save operations
4. **Editor events** - Theme changes, mode switches
5. **Custom events** - Plugin-specific events

### Event flow

```
User Action → Editor → Event Trigger → Plugin Handler → Response
```

## Core event handlers

### Initialization event

The init event is called when the plugin loads:

```javascript
window.Asc.plugin.init = function (data) {
  console.log("Plugin initialized with data:", data);

  // Setup event listeners
  setupEventHandlers();

  // Initialize UI
  initializeInterface();

  // Process initial data
  if (data) {
    processInitialData(data);
  }
};
```

### Button click event

Handle clicks on buttons defined in config.json:

```javascript
window.Asc.plugin.button = function (id) {
  switch (id) {
    case 0: // First button (usually OK/Apply)
      handleApply();
      break;
    case 1: // Second button
      handleSecondary();
      break;
    case -1: // Cancel button
      handleCancel();
      break;
    default:
      console.log("Unknown button:", id);
  }
};

function handleApply() {
  // Perform action
  const result = processData();

  // Close plugin if successful
  if (result.success) {
    window.Asc.plugin.executeCommand("close", "");
  }
}
```

### Selection change event

Respond to user selection changes:

```javascript
window.Asc.plugin.attachEvent("onSelectionChanged", function (selection) {
  console.log("Selection changed:", selection);

  if (selection && selection.text) {
    handleNewSelection(selection.text);
  } else {
    handleEmptySelection();
  }
});

function handleNewSelection(text) {
  // Update UI based on selection
  document.getElementById("selectedText").textContent = text;

  // Enable/disable actions
  document.getElementById("processBtn").disabled = false;
}

function handleEmptySelection() {
  document.getElementById("selectedText").textContent = "No selection";
  document.getElementById("processBtn").disabled = true;
}
```

## Advanced event patterns

### Event delegation

Use event delegation for better performance with multiple similar elements:

**Error name:** Multiple individual event listeners

:::warning[Wrong]

```javascript
document.querySelectorAll(".action-btn").forEach((btn) => {
  btn.addEventListener("click", handleClick);
});
```

:::

:::tip[Correct]

```javascript
document.getElementById("container").addEventListener("click", function (e) {
  if (e.target.classList.contains("action-btn")) {
    const action = e.target.dataset.action;
    handleAction(action, e.target);
  }
});

function handleAction(action, element) {
  switch (action) {
    case "format":
      applyFormatting(element.dataset.format);
      break;
    case "insert":
      insertContent(element.dataset.content);
      break;
  }
}
```

:::

Error output: Performance issue - Creating many listeners consumes memory and slows down the page.

### Debouncing events

Prevent excessive event handler calls:

```javascript
function debounce(func, wait) {
  let timeout;
  return function executedFunction(...args) {
    const later = () => {
      clearTimeout(timeout);
      func(...args);
    };
    clearTimeout(timeout);
    timeout = setTimeout(later, wait);
  };
}

// Usage with selection changes
const debouncedHandler = debounce(function (selection) {
  updatePreview(selection.text);
}, 300);

window.Asc.plugin.attachEvent("onSelectionChanged", debouncedHandler);
```

### Throttling events

Limit event handler execution frequency:

```javascript
function throttle(func, limit) {
  let inThrottle;
  return function (...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;
      setTimeout(() => (inThrottle = false), limit);
    }
  };
}

// Usage with scroll events
const throttledScroll = throttle(function (e) {
  updateScrollPosition(e.target.scrollTop);
}, 100);

document.getElementById("content").addEventListener("scroll", throttledScroll);
```

### Event queueing

Queue events for batch processing:

```javascript
class EventQueue {
  constructor() {
    this.queue = [];
    this.processing = false;
  }

  add(event) {
    this.queue.push(event);
    this.process();
  }

  async process() {
    if (this.processing || this.queue.length === 0) return;

    this.processing = true;

    while (this.queue.length > 0) {
      const event = this.queue.shift();
      await this.handleEvent(event);
    }

    this.processing = false;
  }

  async handleEvent(event) {
    // Process event
    console.log("Processing event:", event.type);
    // Simulate async operation
    await new Promise((resolve) => setTimeout(resolve, 100));
  }
}

// Usage
const eventQueue = new EventQueue();

window.Asc.plugin.attachEvent("onSelectionChanged", function (selection) {
  eventQueue.add({
    type: "selection",
    data: selection,
  });
});
```

## Custom event system

### Creating custom events

Implement a custom event emitter for plugin-specific events:

```javascript
class EventEmitter {
  constructor() {
    this.events = {};
  }

  on(event, listener) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(listener);
  }

  off(event, listenerToRemove) {
    if (!this.events[event]) return;

    this.events[event] = this.events[event].filter(
      (listener) => listener !== listenerToRemove,
    );
  }

  emit(event, ...args) {
    if (!this.events[event]) return;

    this.events[event].forEach((listener) => {
      listener(...args);
    });
  }

  once(event, listener) {
    const onceWrapper = (...args) => {
      listener(...args);
      this.off(event, onceWrapper);
    };
    this.on(event, onceWrapper);
  }
}

// Usage
const pluginEvents = new EventEmitter();

// Register listeners
pluginEvents.on("dataProcessed", function (result) {
  console.log("Data processed:", result);
  updateUI(result);
});

pluginEvents.on("error", function (error) {
  console.error("Error occurred:", error);
  showErrorMessage(error.message);
});

// Emit events
function processData(data) {
  try {
    const result = transform(data);
    pluginEvents.emit("dataProcessed", result);
  } catch (error) {
    pluginEvents.emit("error", error);
  }
}
```

### Event namespacing

Organize events with namespaces:

```javascript
class NamespacedEventEmitter extends EventEmitter {
  on(event, listener) {
    const [namespace, eventName] = event.split(":");
    const fullEvent = namespace && eventName ? event : `global:${event}`;
    super.on(fullEvent, listener);
  }

  emit(event, ...args) {
    const [namespace, eventName] = event.split(":");
    const fullEvent = namespace && eventName ? event : `global:${event}`;
    super.emit(fullEvent, ...args);
  }
}

// Usage
const events = new NamespacedEventEmitter();

events.on("ui:update", function (data) {
  updateInterface(data);
});

events.on("data:loaded", function (data) {
  processLoadedData(data);
});

events.on("data:saved", function () {
  showSuccessMessage("Data saved");
});

// Emit namespaced events
events.emit("ui:update", { state: "ready" });
events.emit("data:loaded", { items: [] });
```

## Asynchronous event handling

### Promise-based events

Handle async operations in events:

```javascript
class AsyncEventEmitter extends EventEmitter {
  async emitAsync(event, ...args) {
    if (!this.events[event]) return;

    const promises = this.events[event].map((listener) =>
      Promise.resolve(listener(...args)),
    );

    return Promise.all(promises);
  }
}

// Usage
const asyncEvents = new AsyncEventEmitter();

asyncEvents.on("saveData", async function (data) {
  await saveToAPI(data);
  return { success: true };
});

asyncEvents.on("saveData", async function (data) {
  await updateLocalCache(data);
  return { cached: true };
});

// Emit and wait for all handlers
async function handleSave(data) {
  try {
    showLoading();
    const results = await asyncEvents.emitAsync("saveData", data);
    console.log("All save handlers completed:", results);
    hideLoading();
  } catch (error) {
    console.error("Save failed:", error);
    showError(error);
  }
}
```

### Event chains

Create event chains for sequential processing:

```javascript
class EventChain {
  constructor() {
    this.steps = [];
  }

  addStep(name, handler) {
    this.steps.push({ name, handler });
    return this;
  }

  async execute(data) {
    let result = data;

    for (const step of this.steps) {
      console.log(`Executing step: ${step.name}`);
      try {
        result = await step.handler(result);
      } catch (error) {
        console.error(`Step ${step.name} failed:`, error);
        throw error;
      }
    }

    return result;
  }
}

// Usage
const processingChain = new EventChain()
  .addStep("validate", async function (data) {
    if (!data || !data.text) {
      throw new Error("Invalid data");
    }
    return data;
  })
  .addStep("transform", async function (data) {
    return {
      ...data,
      text: data.text.toUpperCase(),
    };
  })
  .addStep("save", async function (data) {
    await saveData(data);
    return { ...data, saved: true };
  });

// Execute chain
async function processUserInput(input) {
  try {
    const result = await processingChain.execute({ text: input });
    console.log("Processing complete:", result);
  } catch (error) {
    console.error("Processing failed:", error);
  }
}
```

## Error handling in events

### Try-catch wrappers

Wrap event handlers with error handling:

```javascript
function safeHandler(handler) {
  return function (...args) {
    try {
      return handler.apply(this, args);
    } catch (error) {
      console.error("Event handler error:", error);
      showErrorNotification(error.message);
    }
  };
}

// Usage
window.Asc.plugin.button = safeHandler(function (id) {
  if (id === 0) {
    // This error will be caught
    throw new Error("Something went wrong");
  }
});
```

### Error event pattern

Emit error events for centralized handling:

```javascript
const eventBus = new EventEmitter();

// Register global error handler
eventBus.on("error", function (error, context) {
  console.error(`Error in ${context}:`, error);
  logErrorToServer(error, context);
  showUserFriendlyError(error);
});

// Use in handlers
function processSelection(selection) {
  try {
    // Processing logic
    const result = transform(selection);
    eventBus.emit("success", result);
  } catch (error) {
    eventBus.emit("error", error, "processSelection");
  }
}
```

## Event testing patterns

### Mock event system

Create mock events for testing:

```javascript
class MockEventSystem {
  constructor() {
    this.handlers = {};
    this.emittedEvents = [];
  }

  attachEvent(event, handler) {
    this.handlers[event] = handler;
  }

  triggerEvent(event, data) {
    this.emittedEvents.push({ event, data, timestamp: Date.now() });

    if (this.handlers[event]) {
      this.handlers[event](data);
    }
  }

  getEmittedEvents() {
    return this.emittedEvents;
  }

  clearEvents() {
    this.emittedEvents = [];
  }
}

// Usage in tests
const mockEvents = new MockEventSystem();

// Replace real event system with mock
window.Asc = {
  plugin: {
    attachEvent: mockEvents.attachEvent.bind(mockEvents),
  },
};

// Test event handling
mockEvents.triggerEvent("onSelectionChanged", { text: "test" });
console.log("Events emitted:", mockEvents.getEmittedEvents());
```

## Best practices

### Clean up event listeners

```javascript
let selectionHandler = null;

window.Asc.plugin.init = function () {
  // Create handler reference for cleanup
  selectionHandler = function (selection) {
    handleSelection(selection);
  };

  window.Asc.plugin.attachEvent("onSelectionChanged", selectionHandler);
};

window.Asc.plugin.button = function (id) {
  if (id === -1) {
    // Clean up before closing
    if (selectionHandler) {
      window.Asc.plugin.detachEvent("onSelectionChanged", selectionHandler);
    }
    window.Asc.plugin.executeCommand("close", "");
  }
};
```

### Use meaningful event names

**Error name:** Generic event names

:::warning[Wrong]

```javascript
eventBus.emit("update", data);
eventBus.emit("change", value);
```

:::

:::tip[Correct]

```javascript
eventBus.emit("user:profile:updated", userProfile);
eventBus.emit("document:selection:changed", selection);
eventBus.emit("api:data:loaded", apiResponse);
```

:::

Error output: Maintainability issue - Generic names make it unclear what event does, leading to confusion in large codebases.

### Document event contracts

```javascript
/**
 * Event: selection:processed
 * Emitted when text selection has been processed
 *
 * @event selection:processed
 * @param {Object} result - Processing result
 * @param {string} result.original - Original selected text
 * @param {string} result.transformed - Transformed text
 * @param {number} result.processingTime - Time taken in ms
 */
eventBus.emit("selection:processed", {
  original: originalText,
  transformed: transformedText,
  processingTime: endTime - startTime,
});
```

## Conclusion

Mastering event handling patterns is essential for creating responsive and maintainable ONLYOFFICE plugins. By implementing proper event management, you can build plugins that react efficiently to user actions and system events while maintaining clean, testable code.
