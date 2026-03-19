---
sidebar_position: 7
---

# UI 组件库

## 概述

ONLYOFFICE 提供了一套全面的 UI 组件库，使插件开发者能够创建与编辑器原生外观和风格相匹配的一致、专业的界面。该库包含针对常见 UI 模式的预构建组件，可减少开发时间并确保视觉一致性。

## 什么是 UI 组件库

UI 组件库是专为 ONLYOFFICE 插件设计的一组即用型界面组件。这些组件具有以下特点：

- **预设样式** - 与 ONLYOFFICE 设计语言相匹配
- **主题感知** - 自动适应浅色和深色模式
- **无障碍** - 遵循 Web 无障碍标准
- **响应式** - 适用于不同屏幕尺寸
- **可定制** - 可根据需求调整样式

## 可用组件

### 上下文菜单

在插件界面中创建右键上下文菜单。

**使用场景：**

- 对选定项目执行自定义操作
- 快速访问常用操作
- 根据用户交互提供上下文选项

**示例：**

```javascript
window.Asc.plugin.contextMenuShow({
  items: [
    { id: "copy", text: "Copy", icon: "copy.png" },
    { id: "paste", text: "Paste", icon: "paste.png" },
    { separator: true },
    { id: "delete", text: "Delete", icon: "delete.png" },
  ],
  x: 100,
  y: 150,
});

window.Asc.plugin.onContextMenuClick = function (id) {
  if (id === "copy") {
    // Handle copy action
  }
};
```

### 工具栏按钮

向插件工具栏添加自定义按钮以执行快速操作。

**使用场景：**

- 插件主要操作
- 模式切换
- 工具选择
- 快速设置访问

**示例：**

```html
<div class="toolbar">
  <button class="toolbar-button" id="boldBtn">
    <img src="icons/bold.svg" alt="Bold" />
    <span>Bold</span>
  </button>
  <button class="toolbar-button active" id="italicBtn">
    <img src="icons/italic.svg" alt="Italic" />
    <span>Italic</span>
  </button>
</div>
```

```css
.toolbar-button {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 6px 12px;
  border: 1px solid #ccc;
  background: #fff;
  cursor: pointer;
  border-radius: 4px;
}

.toolbar-button:hover {
  background: #f5f5f5;
}

.toolbar-button.active {
  background: #e3f2fd;
  border-color: #2196f3;
}
```

### 窗口与面板

为插件内容创建模态对话框和侧边面板。

**模态窗口示例：**

```json
{
  "isModal": true,
  "isVisual": true,
  "size": [400, 300],
  "buttons": [
    { "text": "OK", "primary": true },
    { "text": "Cancel", "primary": false }
  ]
}
```

**侧边面板示例：**

```json
{
  "isModal": false,
  "isInsideMode": true,
  "type": "panel",
  "size": [350, 600]
}
```

**主要特性：**

- 可调整大小的窗口
- 可拖动的对话框
- 持久化面板状态
- 主题感知样式

### 输入辅助

带有验证和样式的表单输入组件。

**文本输入：**

```html
<div class="input-group">
  <label for="username">Username:</label>
  <input
    type="text"
    id="username"
    class="input-field"
    placeholder="Enter username"
  />
  <span class="input-hint">Must be 3-20 characters</span>
</div>
```

**复选框：**

```html
<label class="checkbox-label">
  <input type="checkbox" id="rememberMe" />
  <span>Remember me</span>
</label>
```

**下拉选择：**

```html
<div class="input-group">
  <label for="language">Language:</label>
  <select id="language" class="select-field">
    <option value="en">English</option>
    <option value="ru">Русский</option>
    <option value="de">Deutsch</option>
  </select>
</div>
```

**样式：**

```css
.input-group {
  margin-bottom: 16px;
}

.input-group label {
  display: block;
  margin-bottom: 6px;
  font-weight: 500;
  font-size: 14px;
}

.input-field,
.select-field {
  width: 100%;
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

.input-field:focus,
.select-field:focus {
  outline: none;
  border-color: #2196f3;
  box-shadow: 0 0 0 2px rgba(33, 150, 243, 0.1);
}

.input-hint {
  display: block;
  margin-top: 4px;
  font-size: 12px;
  color: #666;
}
```

### 自定义按钮

创建与 ONLYOFFICE 设计模式相匹配的样式按钮。

**按钮类型：**

**主要按钮：**

```html
<button class="btn btn-primary">Save Changes</button>
```

**次要按钮：**

```html
<button class="btn btn-secondary">Cancel</button>
```

**危险按钮：**

```html
<button class="btn btn-danger">Delete</button>
```

**图标按钮：**

```html
<button class="btn btn-icon">
  <img src="icons/settings.svg" alt="Settings" />
</button>
```

**样式：**

```css
.btn {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-primary {
  background: #2196f3;
  color: white;
}

.btn-primary:hover {
  background: #1976d2;
}

.btn-secondary {
  background: #f5f5f5;
  color: #333;
  border: 1px solid #ddd;
}

.btn-secondary:hover {
  background: #e0e0e0;
}

.btn-danger {
  background: #f44336;
  color: white;
}

.btn-danger:hover {
  background: #d32f2f;
}

.btn-icon {
  padding: 8px;
  background: transparent;
  border: 1px solid transparent;
}

.btn-icon:hover {
  background: #f5f5f5;
  border-color: #ddd;
}
```

### 内容控制按钮

用于内容插入和操作的特殊按钮。

**使用场景：**

- 插入预格式化内容
- 应用模板
- 添加结构化数据
- 创建内容块

**示例：**

```html
<div class="content-controls">
  <button class="content-btn" data-type="table">
    <img src="icons/table.svg" alt="Insert Table" />
    <span>Insert Table</span>
  </button>
  <button class="content-btn" data-type="image">
    <img src="icons/image.svg" alt="Insert Image" />
    <span>Insert Image</span>
  </button>
  <button class="content-btn" data-type="list">
    <img src="icons/list.svg" alt="Insert List" />
    <span>Insert List</span>
  </button>
</div>
```

```javascript
document.querySelectorAll(".content-btn").forEach((btn) => {
  btn.addEventListener("click", function () {
    const type = this.dataset.type;
    insertContent(type);
  });
});

function insertContent(type) {
  if (type === "table") {
    window.Asc.plugin.executeMethod("InsertTable", [3, 3]);
  } else if (type === "image") {
    // Handle image insertion
  } else if (type === "list") {
    window.Asc.plugin.executeMethod("PasteHtml", [
      "<ul><li>Item 1</li><li>Item 2</li></ul>",
    ]);
  }
}
```

## 使用组件库

### 安装

组件库可通过 ONLYOFFICE 插件 SDK 获取：

```html
<script src="https://onlyoffice.github.io/sdkjs-plugins/v1/plugins.js"></script>
<script src="https://onlyoffice.github.io/sdkjs-plugins/v1/plugins-ui.js"></script>
```

### 基本设置

在插件的 `index.html` 中引入 UI 库：

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>My Plugin</title>
    <link
      rel="stylesheet"
      href="https://onlyoffice.github.io/sdkjs-plugins/v1/plugins-ui.css"
    />
    <script src="https://onlyoffice.github.io/sdkjs-plugins/v1/plugins.js"></script>
    <script src="https://onlyoffice.github.io/sdkjs-plugins/v1/plugins-ui.js"></script>
    <link rel="stylesheet" href="styles.css" />
  </head>
  <body>
    <!-- Your plugin UI here -->
    <script src="plugin.js"></script>
  </body>
</html>
```

### 主题支持

组件库会自动适应 ONLYOFFICE 主题：

```javascript
window.Asc.plugin.onThemeChanged = function (theme) {
  // Apply theme-specific styles
  document.body.classList.toggle("dark-mode", theme.type === "dark");

  // Update component colors
  if (theme.type === "dark") {
    document.documentElement.style.setProperty("--bg-color", "#1e1e1e");
    document.documentElement.style.setProperty("--text-color", "#e0e0e0");
  } else {
    document.documentElement.style.setProperty("--bg-color", "#ffffff");
    document.documentElement.style.setProperty("--text-color", "#333333");
  }
};
```

## 最佳实践

### 一致性

- **使用标准组件** - 优先使用库组件而非自定义组件
- **遵循命名规范** - 使用一致的类名和 ID
- **保持间距** - 使用标准内边距和外边距（4px、8px、12px、16px）

### 无障碍

- **键盘导航** - 确保所有交互元素均可通过键盘访问
- **ARIA 标签** - 为屏幕阅读器添加适当的 ARIA 属性
- **焦点指示器** - 提供清晰的可视焦点状态
- **颜色对比度** - 确保足够的对比度（最低符合 WCAG AA 标准）

**示例：**

```html
<button class="btn btn-primary" aria-label="Save document" tabindex="0">
  Save
</button>
```

### 响应式设计

使组件在不同插件尺寸下均能正常工作：

```css
.plugin-container {
  padding: 16px;
}

@media (max-width: 350px) {
  .plugin-container {
    padding: 12px;
  }

  .btn {
    width: 100%;
    margin-bottom: 8px;
  }
}

@media (min-width: 600px) {
  .toolbar-button span {
    display: inline; /* Show button labels on larger screens */
  }
}
```

### 性能

- **最小化 DOM 操作** - 尽可能批量更新
- **使用事件委托** - 将监听器附加到父元素
- **延迟加载资源** - 仅在需要时加载大型组件
- **缓存 DOM 引用** - 存储频繁访问的元素

**错误名称：** 重复 DOM 查询导致性能问题

:::warning[错误]
```javascript
document.getElementById('container').querySelector('.btn').addEventListener('click', ...);
document.getElementById('container').querySelector('.btn').classList.add('active');
```
:::

:::tip[正确]
```javascript
// Cache DOM references
const container = document.getElementById('container');
const buttons = container.querySelectorAll('.btn');

// Use event delegation
container.addEventListener('click', function(e) {
  if (e.target.classList.contains('btn')) {
    handleButtonClick(e.target);
  }
});
```
:::

错误输出：重复的 DOM 查询速度较慢；缓存引用和事件委托可提升性能。

## 组件示例

### 完整表单示例

```html
<div class="plugin-container">
  <h2>Settings</h2>

  <div class="input-group">
    <label for="apiKey">API Key:</label>
    <input
      type="text"
      id="apiKey"
      class="input-field"
      placeholder="Enter your API key"
    />
    <span class="input-hint">Required for external service integration</span>
  </div>

  <div class="input-group">
    <label for="language">Language:</label>
    <select id="language" class="select-field">
      <option value="en">English</option>
      <option value="ru">Русский</option>
      <option value="de">Deutsch</option>
    </select>
  </div>

  <label class="checkbox-label">
    <input type="checkbox" id="autoSync" />
    <span>Enable automatic synchronization</span>
  </label>

  <div class="button-group">
    <button class="btn btn-primary" onclick="saveSettings()">Save</button>
    <button class="btn btn-secondary" onclick="resetSettings()">Reset</button>
  </div>
</div>
```

### 带操作的工具栏

```html
<div class="toolbar">
  <div class="toolbar-group">
    <button class="toolbar-button" title="Bold" data-action="bold">
      <img src="icons/bold.svg" alt="Bold" />
    </button>
    <button class="toolbar-button" title="Italic" data-action="italic">
      <img src="icons/italic.svg" alt="Italic" />
    </button>
    <button class="toolbar-button" title="Underline" data-action="underline">
      <img src="icons/underline.svg" alt="Underline" />
    </button>
  </div>

  <div class="toolbar-separator"></div>

  <div class="toolbar-group">
    <button class="toolbar-button" title="Insert Link" data-action="link">
      <img src="icons/link.svg" alt="Link" />
    </button>
    <button class="toolbar-button" title="Insert Image" data-action="image">
      <img src="icons/image.svg" alt="Image" />
    </button>
  </div>
</div>
```

## 附加资源

- **ONLYOFFICE 插件 SDK**：[https://github.com/ONLYOFFICE/sdkjs-plugins](https://github.com/ONLYOFFICE/sdkjs-plugins)
- **UI 组件示例**：[https://onlyoffice.github.io/sdkjs-plugins/](https://onlyoffice.github.io/sdkjs-plugins/)
- **插件市场**：[https://www.onlyoffice.com/app-directory](https://www.onlyoffice.com/app-directory)

## 结论

ONLYOFFICE UI 组件库提供了创建专业、一致的插件界面所需的一切。通过使用这些预构建组件并遵循最佳实践，您可以专注于插件功能的开发，同时确保与 ONLYOFFICE 编辑器无缝集成的精致用户体验。
