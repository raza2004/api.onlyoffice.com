---
sidebar_position: -1
---

# 浏览器 DevTools 指南

## 概述

浏览器 DevTools 是调试 ONLYOFFICE 插件的必备工具。本指南介绍如何访问、使用和掌握调试工具，以便快速定位并修复插件中的问题。

## 访问 DevTools

### 在 ONLYOFFICE 桌面编辑器中

**Windows/Linux：**
```
Press: Ctrl+Shift+Alt+F12
```

**macOS：**
```
Press: Cmd+Option+Shift+F12
```

**替代方法：**
1. 在插件的任意位置右键单击
2. 选择"检查元素"或"检查"

### 在 ONLYOFFICE Web 编辑器中

**所有浏览器：**
```
Press F12 or Ctrl+Shift+I (Cmd+Option+I on Mac)
```

**右键单击方法：**
1. 在插件区域右键单击
2. 选择"检查"或"检查元素"

## DevTools 面板概览

### Console 面板

Console 是调试插件的主要工具。

**常见用途：**
- 查看错误信息和警告
- 记录调试信息
- 执行 JavaScript 代码
- 测试 API 调用

**基本控制台方法：**
```javascript
// Log messages
console.log('Plugin initialized');
console.log('User data:', userData);

// Warnings
console.warn('API key missing');

// Errors
console.error('Failed to load data:', error);

// Tables for structured data
console.table([
  { name: 'John', age: 30 },
  { name: 'Jane', age: 25 }
]);

// Grouping logs
console.group('API Calls');
console.log('Call 1: Success');
console.log('Call 2: Failed');
console.groupEnd();

// Timing operations
console.time('data-processing');
processData();
console.timeEnd('data-processing');
```

### Sources 面板

逐行调试 JavaScript 代码。

**主要功能：**
- 设置断点
- 逐步执行代码
- 监视变量
- 查看调用堆栈

**设置断点：**
```javascript
function processSelection(text) {
  // Click line number in Sources panel to set breakpoint here
  const words = text.split(' ');

  // Execution pauses at breakpoint
  const count = words.length;

  return count;
}
```

**条件断点：**
```javascript
// Right-click line number → Add conditional breakpoint
function processItem(item) {
  // Break only when item.id === 5
  const result = transform(item);
  return result;
}
```

### Network 面板

监控插件发出的所有网络请求。

**需要检查的内容：**
- API 请求/响应
- 加载时间
- 失败的请求
- 请求头

**过滤请求：**
```
- XHR: API calls
- JS: JavaScript files
- CSS: Stylesheets
- Img: Images
```

### Elements 面板

实时检查和修改 HTML/CSS。

**常见任务：**
- 检查元素结构
- 实时编辑 HTML
- 修改 CSS 样式
- 调试布局问题

**实时 CSS 编辑：**
```css
/* In Styles pane, edit CSS directly */
.button {
  background: #2196f3;  /* Change color */
  padding: 10px 20px;    /* Adjust spacing */
}
```

## 调试技术

### 使用断点

**Error name:** No visibility into code execution

:::warning[Wrong]
```javascript
// No debugging - just console.log everywhere
function calculateTotal(items) {
  console.log('items:', items);
  const total = items.reduce((sum, item) => {
    console.log('item:', item);
    return sum + item.price;
  }, 0);
  console.log('total:', total);
  return total;
}
```
:::

:::tip[Correct]
```javascript
// Use breakpoints instead
function calculateTotal(items) {
  // Set breakpoint here in DevTools
  const total = items.reduce((sum, item) => {
    // Inspect variables in Scope panel
    return sum + item.price;
  }, 0);
  return total;
}
```
:::

错误输出：控制台被日志堆满，难以跟踪执行流程。

### 监视表达式

在逐步执行代码时监视特定值：

```javascript
// In DevTools Watch panel, add expressions:
items.length
currentItem.price
total
items[0].name
```

### 调用堆栈分析

了解代码的调用方式：

```javascript
function level1() {
  level2();
}

function level2() {
  level3();
}

function level3() {
  debugger; // Execution pauses here
  // Check Call Stack panel to see:
  // level3
  // level2
  // level1
}
```

## 常见调试场景

### 调试 API 调用

```javascript
async function fetchData() {
  try {
    // Set breakpoint here
    const response = await fetch('https://api.example.com/data');

    // Check Network panel for:
    // - Request URL
    // - Response status
    // - Response body

    const data = await response.json();

    // Inspect data in console
    console.log('Fetched data:', data);

    return data;
  } catch (error) {
    // Check error details
    console.error('Fetch failed:', error);
    throw error;
  }
}
```

### 调试事件处理程序

**Error name:** Event handler not firing

:::warning[Wrong]
```javascript
// No way to see if event fires
document.getElementById('btn').addEventListener('click', function() {
  processClick();
});
```
:::

:::tip[Correct]
```javascript
// Add logging and breakpoint
document.getElementById('btn').addEventListener('click', function(event) {
  console.log('Button clicked', event);
  debugger; // Pause when clicked
  processClick();
});
```
:::

错误输出：无法判断事件监听器是否已附加或触发。

### 调试 ONLYOFFICE API 调用

```javascript
window.Asc.plugin.executeMethod("GetSelectedText", [], function(text) {
  // Set breakpoint in callback
  console.log('Selected text:', text);

  if (!text) {
    console.warn('No text selected');
    return;
  }

  // Process text
  processText(text);
});
```

## 高级调试

### 使用 debugger 语句

```javascript
function complexFunction(data) {
  // Code pauses here when DevTools is open
  debugger;

  const processed = processData(data);
  return processed;
}
```

### 调试异步代码

```javascript
async function loadUserData() {
  try {
    // Breakpoint here
    const user = await fetchUser();

    // Breakpoint here
    const profile = await fetchProfile(user.id);

    // Breakpoint here
    return { user, profile };
  } catch (error) {
    // Breakpoint here to catch errors
    console.error('Load failed:', error);
  }
}
```

### 性能分析

**录制性能：**
```
1. Open Performance tab
2. Click Record (red circle)
3. Perform actions in plugin
4. Click Stop
5. Analyze timeline
```

**需要关注的内容：**
- 耗时较长的任务（黄色区块）
- 布局抖动
- 内存峰值
- 脚本执行时间

### 内存分析

**检测内存泄漏：**
```
1. Open Memory tab
2. Take heap snapshot
3. Use plugin for a while
4. Take another snapshot
5. Compare snapshots
```

**常见泄漏来源：**
```javascript
// Leaked event listeners
let leakedHandlers = [];

function setupListener() {
  const handler = () => console.log('clicked');
  document.addEventListener('click', handler);
  leakedHandlers.push(handler); // Memory leak!
}

// Fix: Remove listeners
function cleanup() {
  leakedHandlers.forEach(handler => {
    document.removeEventListener('click', handler);
  });
  leakedHandlers = [];
}
```

## DevTools 使用技巧

### 快捷命令

**控制台快捷方式：**
```javascript
// $_ = last result
2 + 2
$_ // Returns 4

// $ = querySelector
$('#myElement')

// $$ = querySelectorAll
$$('.button')

// Clear console
clear()

// Copy to clipboard
copy(myObject)
```

### 保留日志

在导航时保留控制台日志：
```
Console → Settings (gear icon) → ✓ Preserve log
```

### 过滤控制台消息

```
Console → Filter box:
- /error/i    (regex for errors)
- -warning    (exclude warnings)
- method:POST (filter by type)
```

### 实时表达式

实时监控值：
```
Console → Create live expression
→ Enter: myVariable
→ Value updates automatically
```

## 最佳实践

### 策略性日志记录

```javascript
// Log important events
window.Asc.plugin.init = function(data) {
  console.log('Plugin initialized', data);
};

window.Asc.plugin.button = function(id) {
  console.log('Button clicked:', id);
};

// Log errors prominently
console.error('Critical error:', error);

// Log warnings
console.warn('Deprecated method used');
```

### 清理调试代码

**Error name:** Debug code in production

:::warning[Wrong]
```javascript
// Debug code always running
console.log('Debug: Processing data');
debugger;
console.table(debugData);
```
:::

:::tip[Correct]
```javascript
// Use development flag
const DEBUG = window.location.hostname === 'localhost';

if (DEBUG) {
  console.log('Debug: Processing data');
  console.table(debugData);
}

// Or use environment variable
if (process.env.NODE_ENV === 'development') {
  debugger;
}
```
:::

错误输出：生产环境插件运行缓慢，控制台被日志堆满。

## 总结

熟练掌握浏览器 DevTools 是高效调试插件的关键。合理使用断点、监控网络请求、分析性能，并充分利用高级功能，以便快速定位并修复 ONLYOFFICE 插件中的问题。
