---
sidebar_position: -2
---

# 事件处理模式

## 概述

事件处理是创建交互式、响应式 ONLYOFFICE 插件的核心。本指南涵盖了管理事件、实现高效事件处理程序以及构建健壮的事件驱动架构的高级模式。

## 理解 ONLYOFFICE 事件

### 事件类型

ONLYOFFICE 插件可以响应以下几种事件类型：

1. **生命周期事件** - 插件初始化与终止
2. **用户交互事件** - 选区变化、按钮点击
3. **文档事件** - 内容变化、保存操作
4. **编辑器事件** - 主题变化、模式切换
5. **自定义事件** - 插件特定的事件

### 事件流

```
用户操作 → 编辑器 → 触发事件 → 插件处理程序 → 响应
```

## 核心事件处理程序

### 初始化事件

插件加载时会调用 init 事件：

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

### 按钮点击事件

处理 config.json 中定义的按钮点击：

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

### 选区变化事件

响应用户选区的变化：

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

## 高级事件模式

### 事件委托

使用事件委托提升多个同类元素的性能：

**错误名称：** 多个独立事件监听器

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

错误输出：性能问题 - 创建过多监听器会消耗内存并降低页面速度。

### 防抖事件

防止事件处理程序被过度调用：

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

### 节流事件

限制事件处理程序的执行频率：

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

### 事件队列

将事件加入队列以进行批量处理：

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

## 自定义事件系统

### 创建自定义事件

为插件特定事件实现自定义事件发射器：

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

### 事件命名空间

使用命名空间对事件进行组织：

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

## 异步事件处理

### 基于 Promise 的事件

在事件中处理异步操作：

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

### 事件链

创建事件链以进行顺序处理：

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

## 事件中的错误处理

### Try-catch 包装器

使用错误处理对事件处理程序进行包装：

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

### 错误事件模式

发射错误事件以进行集中处理：

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

## 事件测试模式

### 模拟事件系统

创建模拟事件以进行测试：

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

## 最佳实践

### 清理事件监听器

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

### 使用有意义的事件名称

**错误名称：** 泛型事件名称

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

错误输出：可维护性问题 - 泛型名称无法清晰表达事件的含义，在大型代码库中容易引发混淆。

### 记录事件契约

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

## 结论

掌握事件处理模式对于创建响应式、可维护的 ONLYOFFICE 插件至关重要。通过实施正确的事件管理，您可以构建能够高效响应用户操作和系统事件的插件，同时保持代码整洁且易于测试。
