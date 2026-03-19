---
sidebar_position: -2
---

# 常见错误与解决方案

## 概述

本指南涵盖了 ONLYOFFICE 插件开发过程中最常见的错误及其解决方案。学会快速定位和修复问题，以简化您的开发流程。

## 插件初始化错误

### 插件未出现在菜单中

**Error name:** Plugin doesn't show in Plugins tab

**症状：**
- 插件已安装但不可见
- 控制台中无错误
- 配置文件存在

:::warning[Wrong]
```json
// Wrong GUID format
{
  "guid": "{12345678-ABCD}"  // Missing asc. prefix
}

// Invalid JSON
{
  "name": "My Plugin",
  "version": "1.0.0",  // Extra comma
}

// Wrong file path
{
  "variations": [{
    "url": "plugin.html"  // File doesn't exist
  }]
}
```
:::

:::tip[Correct]
```json
// Correct GUID format
{
  "guid": "asc.{12345678-1234-1234-1234-123456789ABC}"
}

// Valid JSON
{
  "name": "My Plugin",
  "version": "1.0.0"
}

// Correct file path
{
  "variations": [{
    "url": "index.html"  // Verify file exists
  }]
}
```
:::

**验证步骤：**
```bash
# 1. Validate JSON
cat config.json | python -m json.tool

# 2. Check file exists
ls index.html

# 3. Verify GUID format starts with "asc."

# 4. Restart ONLYOFFICE completely
```

### 插件初始化但显示空白屏幕

**Error name:** White screen on plugin open

**原因：** JavaScript 错误导致渲染失败

:::warning[Wrong]
```javascript
// Error in init function
window.Asc.plugin.init = function(data) {
  document.getElementById('nonexistent').textContent = data;
  // Throws error, plugin shows blank
};
```
:::

:::tip[Correct]
```javascript
// Proper error handling
window.Asc.plugin.init = function(data) {
  const element = document.getElementById('output');

  if (!element) {
    console.error('Element not found');
    return;
  }

  element.textContent = data || 'No data';
};
```
:::

错误输出：请在控制台中检查："Uncaught TypeError: Cannot read property 'textContent' of null"

## API 方法错误

### executeMethod 不起作用

**Error name:** API method returns undefined

:::warning[Wrong]
```javascript
// Missing callback function
window.Asc.plugin.executeMethod("GetSelectedText");

// Result is undefined
```
:::

:::tip[Correct]
```javascript
// Include callback to receive result
window.Asc.plugin.executeMethod("GetSelectedText", [], function(text) {
  console.log('Selected text:', text);

  if (text) {
    processText(text);
  } else {
    showMessage('No text selected');
  }
});
```
:::

错误输出：方法已执行，但由于缺少回调函数，结果丢失。

### callCommand 静默失败

**Error name:** callCommand doesn't execute

:::warning[Wrong]
```javascript
// Syntax error in callCommand
window.Asc.plugin.callCommand(function() {
  const doc = Api.GetDocument();
  doc.nonExistentMethod();  // Method doesn't exist
}, true);  // Wrong second parameter
```
:::

:::tip[Correct]
```javascript
// Correct syntax and error handling
window.Asc.plugin.callCommand(function() {
  try {
    const doc = Api.GetDocument();

    if (!doc) {
      throw new Error('Document not available');
    }

    // Use valid API method
    const paragraphs = doc.GetAllParagraphs();
    return paragraphs.length;
  } catch (error) {
    console.error('callCommand error:', error);
    return null;
  }
}, false);  // false for async execution
```
:::

错误输出：请在控制台中检查 API 方法错误或未定义的返回值。

## 配置错误

### 图标未显示

**Error name:** Plugin shows default icon instead of custom

:::warning[Wrong]
```json
{
  "variations": [{
    "icons": ["icon.png"]  // File in wrong location
  }]
}
```
:::

:::tip[Correct]
```json
{
  "variations": [{
    "icons": [
      "resources/icon.png",
      "resources/icon@2x.png"
    ]
  }]
}
```

文件结构：
```
my-plugin/
├── config.json
├── index.html
└── resources/
    ├── icon.png (48x48)
    └── icon@2x.png (96x96)
```
:::

错误输出：插件显示通用图标，无错误信息。

### 模态框/面板配置问题

**Error name:** Plugin opens in wrong mode

:::warning[Wrong]
```json
{
  "isModal": true,
  "isInsideMode": true,  // Conflicting settings
  "type": "panel"
}
```
:::

:::tip[Correct]
```json
// For modal dialog
{
  "isModal": true,
  "isInsideMode": false,
  "buttons": [
    {"text": "OK", "primary": true},
    {"text": "Cancel"}
  ]
}

// For side panel
{
  "isModal": false,
  "isInsideMode": true,
  "type": "panel"
}
```
:::

错误输出：插件以意外的模式或位置出现。

## 事件处理错误

### 事件未触发

**Error name:** onSelectionChanged not triggering

:::warning[Wrong]
```javascript
// Wrong event attachment
window.Asc.plugin.onSelectionChanged = function(selection) {
  console.log('Selection changed');
};
```
:::

:::tip[Correct]
```javascript
// Use attachEvent method
window.Asc.plugin.attachEvent("onSelectionChanged", function(selection) {
  console.log('Selection changed:', selection);

  if (selection && selection.text) {
    updateUI(selection.text);
  }
});
```
:::

错误输出：事件处理程序从不执行，选择变更被忽略。

### 按钮处理程序无响应

**Error name:** Button clicks do nothing

:::warning[Wrong]
```javascript
// No button handler defined
window.Asc.plugin.init = function() {
  // Initialize plugin
};
// Buttons don't work
```
:::

:::tip[Correct]
```javascript
// Define button handler
window.Asc.plugin.button = function(id) {
  if (id === 0) {
    // OK button
    handleOK();
  } else if (id === 1) {
    // Cancel button
    handleCancel();
  } else if (id === -1) {
    // Close button
    window.Asc.plugin.executeCommand("close", "");
  }
};
```
:::

错误输出：点击按钮没有任何效果。

## 数据处理错误

### JSON 解析错误

**Error name:** Cannot parse JSON response

:::warning[Wrong]
```javascript
// No error handling for invalid JSON
fetch('/api/data')
  .then(response => response.json())
  .then(data => {
    processData(data);
  });
```
:::

:::tip[Correct]
```javascript
// Handle JSON parse errors
fetch('/api/data')
  .then(response => {
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    return response.text();
  })
  .then(text => {
    try {
      const data = JSON.parse(text);
      processData(data);
    } catch (error) {
      console.error('Invalid JSON:', text);
      showError('Server returned invalid data');
    }
  })
  .catch(error => {
    console.error('Fetch error:', error);
    showError('Failed to load data');
  });
```
:::

错误输出："SyntaxError: Unexpected token < in JSON at position 0"

### LocalStorage 配额超出

**Error name:** Cannot store data locally

:::warning[Wrong]
```javascript
// Storing without size check
localStorage.setItem('largeData', JSON.stringify(hugeData));
```
:::

:::tip[Correct]
```javascript
// Check size before storing
function safeSave(key, data) {
  try {
    const serialized = JSON.stringify(data);
    const size = new Blob([serialized]).size;
    const sizeMB = size / (1024 * 1024);

    if (sizeMB > 5) {
      console.warn(`Data too large: ${sizeMB.toFixed(2)}MB`);
      return false;
    }

    localStorage.setItem(key, serialized);
    return true;
  } catch (error) {
    if (error.name === 'QuotaExceededError') {
      console.error('Storage quota exceeded');
      clearOldData();
    }
    return false;
  }
}
```
:::

错误输出："QuotaExceededError: Failed to execute 'setItem' on 'Storage'"

## 网络错误

### CORS 错误

**Error name:** Cross-origin request blocked

**控制台中的错误：**
```
Access to fetch at 'https://api.example.com/data' from origin
'http://localhost:3000' has been blocked by CORS policy
```

:::warning[Wrong]
```javascript
// Direct fetch to external API
fetch('https://external-api.com/data')
  .then(response => response.json());
```
:::

:::tip[Correct]
```javascript
// Use proxy server
fetch('/api/proxy?url=' + encodeURIComponent('https://external-api.com/data'))
  .then(response => response.json());

// Or configure server CORS headers
// Server must include:
// Access-Control-Allow-Origin: *
```
:::

### 超时错误

**Error name:** Request times out

:::warning[Wrong]
```javascript
// No timeout handling
fetch('https://slow-api.com/data')
  .then(response => response.json());
// Waits forever
```
:::

:::tip[Correct]
```javascript
// Add timeout
async function fetchWithTimeout(url, timeout = 5000) {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeout);

  try {
    const response = await fetch(url, {
      signal: controller.signal
    });
    clearTimeout(timeoutId);
    return await response.json();
  } catch (error) {
    if (error.name === 'AbortError') {
      throw new Error('Request timeout');
    }
    throw error;
  }
}
```
:::

错误输出："TypeError: Failed to fetch" 或 "AbortError: The operation was aborted"

## UI/UX 错误

### 元素未找到

**Error name:** DOM element not found

:::warning[Wrong]
```javascript
// Accessing elements too early
window.Asc.plugin.init = function() {
  const btn = document.getElementById('myButton');
  btn.addEventListener('click', handleClick);  // btn is null
};
```
:::

:::tip[Correct]
```javascript
// Wait for DOM ready
window.Asc.plugin.init = function() {
  if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', setupUI);
  } else {
    setupUI();
  }
};

function setupUI() {
  const btn = document.getElementById('myButton');

  if (btn) {
    btn.addEventListener('click', handleClick);
  } else {
    console.error('Button not found in DOM');
  }
}
```
:::

错误输出："Uncaught TypeError: Cannot read property 'addEventListener' of null"

## 总结

了解常见错误及其解决方案可加快插件开发速度。在调试问题时，请将本指南作为参考，并记住首先检查控制台、验证配置，以及优雅地处理错误。
