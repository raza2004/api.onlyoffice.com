---
sidebar_position: -1
---

# 插件架构

## 概述

ONLYOFFICE 插件遵循模块化架构，将插件界面与编辑器核心功能分离。理解此架构对于构建健壮且高效的插件至关重要。

## 架构组件

### 1. 插件容器（iframe）

每个插件都在 ONLYOFFICE 编辑器内的隔离 iframe 环境中运行。这提供了：

- **安全隔离** - 插件无法直接访问编辑器的内部状态
- **跨平台兼容性** - 插件在不同平台上保持一致的运行效果
- **独立执行** - 插件错误不会导致主编辑器崩溃

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

### 2. 通信层（API 桥接）

插件通过 `window.Asc.plugin` API 接口与编辑器进行通信：

```javascript
// Plugin → Editor
window.Asc.plugin.executeMethod("PasteText", ["Hello"]);

// Editor → Plugin
window.Asc.plugin.init = function (data) {
  // Initialization code
};
```

### 3. 文件系统结构

一个典型的插件由以下核心组件构成：

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

## 关键架构原则

### 关注点分离

ONLYOFFICE 插件遵循 MVC 模式：

- **视图（HTML/CSS）** - 用户界面与展示层
- **控制器（JavaScript）** - 业务逻辑与 API 交互
- **模型（Config）** - 插件元数据与配置

### 事件驱动通信

插件基于事件驱动模型运行：

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

### 无状态设计

插件应设计为在会话之间保持无状态：

- **默认无持久化存储** - 如有需要，请使用 localStorage
- **干净的初始化** - 不要假设存在先前的状态
- **自包含** - 包含所有依赖项

## 插件类型

### 模态插件

模态插件会阻止与文档的交互：

```json
{
  "isModal": true,
  "isVisual": true,
  "buttons": [{ "text": "OK", "primary": true }, { "text": "Cancel" }]
}
```

**适用场景：** 表单、配置对话框、一次性操作

### 面板插件

面板插件与文档并排运行：

```json
{
  "isModal": false,
  "isInsideMode": true,
  "type": "panel"
}
```

**适用场景：** 持续性工作流、工具、参考面板

### 后台插件

后台插件在没有界面的情况下运行：

```json
{
  "isVisual": false,
  "isModal": false
}
```

**适用场景：** 自动化任务、事件驱动的操作

## 数据流

### 输入流

```
User Action → Plugin UI → JavaScript Handler → API Call → Editor
```

示例：

```javascript
<button onclick="insertText()">Insert</button>;

function insertText() {
  const text = document.getElementById("input").value;
  window.Asc.plugin.executeMethod("PasteText", [text]);
}
```

### 输出流

```
Editor Event → API Callback → Plugin Handler → UI Update
```

示例：

```javascript
window.Asc.plugin.onSelectionChanged = function (selection) {
  if (selection && selection.text) {
    document.getElementById("preview").textContent = selection.text;
  }
};
```

## 安全架构

### 沙箱机制

插件被沙箱化以防止：

- 直接访问文件系统
- 未经授权的网络请求
- 访问其他插件或编辑器内部组件
- 对主编辑器的 XSS 攻击

### 权限模型

插件只能执行 API 明确允许的操作：

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

错误输出：直接 DOM 访问被插件沙箱阻止，对编辑器内容不起任何作用。

## 性能最佳实践

### 高效初始化

```javascript
window.Asc.plugin.init = function (data) {
  // Quick initialization
  setupUI();

  // Defer heavy operations
  setTimeout(loadHeavyResources, 100);
};
```

### 内存管理

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

## 多编辑器支持

插件可以支持多种编辑器类型：

```json
{
  "EditorsSupport": ["word", "cell", "slide"]
}
```

检测并适配当前编辑器：

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

## 扩展点

ONLYOFFICE 提供了多个扩展点：

1. **API 方法** - 用于操作内容的预定义函数
2. **事件** - 响应编辑器和用户操作
3. **UI 集成** - 工具栏、面板、模态框
4. **Document Builder API** - 直接操作文档

## 代码组织最佳实践

### 模块化设计

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

### 错误处理

```javascript
window.Asc.plugin.executeMethod("PasteText", [text], function (result) {
  if (result === undefined || result === null) {
    console.error("Failed to paste text");
    showErrorMessage("Operation failed");
  }
});
```

### 响应式设计

```css
@media (max-width: 400px) {
  .plugin-container {
    padding: 10px;
    font-size: 12px;
  }
}
```

## 总结

理解 ONLYOFFICE 插件架构使您能够构建安全、高性能且用户友好的插件。通过遵循这些架构最佳实践并充分利用所提供的 API，您可以创建功能强大的扩展，与编辑器生态系统无缝集成。
