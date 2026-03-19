---
sidebar_position: -4
---

# 使用外部库

## 概述

外部库通过提供常见任务的预构建功能，可以显著增强您的 ONLYOFFICE 插件的能力。本指南介绍如何在插件中集成、管理和优化外部库。

## 为何使用外部库

### 优点

- **节省开发时间** - 避免重复造轮子
- **经过测试的代码** - 经过实战检验的可靠解决方案
- **丰富的功能** - 复杂功能开箱即用
- **社区支持** - 有文档和帮助可查
- **定期更新** - 持续修复漏洞并改进功能

### 注意事项

- **体积问题** - 体积较大的库会拖慢插件加载速度
- **兼容性** - 确保浏览器兼容性
- **许可证** - 检查许可证兼容性
- **依赖关系** - 管理依赖链
- **安全性** - 审查库是否存在漏洞

## 引入外部库

### 方法一：CDN（内容分发网络）

**适用场景：** 公开的、流行的库

**优点：**
- 无需本地存储
- 加载速度快（浏览器缓存）
- 始终保持最新（如使用 latest 版本）

**index.html：**
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>My Plugin</title>

    <!-- jQuery from CDN -->
    <script src="https://code.jquery.com/jquery-3.7.0.min.js"></script>

    <!-- Lodash from CDN -->
    <script src="https://cdn.jsdelivr.net/npm/lodash@4.17.21/lodash.min.js"></script>

    <!-- Chart.js from CDN -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
</head>
<body>
    <script src="plugin.js"></script>
</body>
</html>
```

**Error name:** CDN unavailable or blocked

:::warning[Wrong]
```html
<!-- Only CDN - fails without internet -->
<script src="https://cdn.example.com/library.js"></script>
<script>
  // Assumes library loaded
  myLibrary.doSomething();
</script>
```
:::

:::tip[Correct]
```html
<!-- CDN with fallback to local -->
<script src="https://cdn.example.com/library.js"></script>
<script>
  // Check if library loaded from CDN
  if (typeof myLibrary === 'undefined') {
    // Load local fallback
    const script = document.createElement('script');
    script.src = 'libs/library.min.js';
    document.head.appendChild(script);
  }
</script>
```
:::

错误输出：当 CDN 被屏蔽或不可用时，出现 "myLibrary is not defined"。

### 方法二：本地文件

**适用场景：** 自定义库、离线插件、大型文件

**插件结构：**
```
my-plugin/
├── config.json
├── index.html
├── plugin.js
├── libs/
│   ├── jquery.min.js
│   ├── lodash.min.js
│   └── chart.min.js
└── assets/
```

**index.html：**
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>My Plugin</title>

    <!-- Local libraries -->
    <script src="libs/jquery.min.js"></script>
    <script src="libs/lodash.min.js"></script>
    <script src="libs/chart.min.js"></script>
</head>
<body>
    <script src="plugin.js"></script>
</body>
</html>
```

### 方法三：动态加载

**适用场景：** 条件性功能、懒加载

```javascript
// Load library only when needed
class LibraryLoader {
  constructor() {
    this.loaded = new Set();
  }

  load(name, url) {
    if (this.loaded.has(name)) {
      return Promise.resolve();
    }

    return new Promise((resolve, reject) => {
      const script = document.createElement('script');
      script.src = url;

      script.onload = () => {
        this.loaded.add(name);
        resolve();
      };

      script.onerror = () => {
        reject(new Error(`Failed to load ${name}`));
      };

      document.head.appendChild(script);
    });
  }

  async loadMultiple(libraries) {
    const promises = libraries.map(lib =>
      this.load(lib.name, lib.url)
    );

    return Promise.all(promises);
  }
}

// Usage
const loader = new LibraryLoader();

async function enableChartFeature() {
  try {
    await loader.load('chart', 'https://cdn.jsdelivr.net/npm/chart.js');
    createChart();
  } catch (error) {
    console.error('Failed to load Chart.js:', error);
  }
}
```

## 插件常用库

### UI 框架

#### Bootstrap

**适用场景：** 响应式布局、组件

```html
<!-- Bootstrap CSS -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">

<!-- Bootstrap JS -->
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
```

**使用示例：**
```html
<div class="container">
  <div class="row">
    <div class="col-md-6">
      <button class="btn btn-primary">Action</button>
    </div>
  </div>
</div>
```

#### Tailwind CSS

**适用场景：** 原子化样式

```html
<script src="https://cdn.tailwindcss.com"></script>
```

**使用示例：**
```html
<div class="flex items-center justify-center p-4">
  <button class="bg-blue-500 text-white px-4 py-2 rounded">
    Click Me
  </button>
</div>
```

### 数据处理

#### Lodash

**适用场景：** 数组/对象工具函数

```html
<script src="https://cdn.jsdelivr.net/npm/lodash@4.17.21/lodash.min.js"></script>
```

**使用示例：**
```javascript
// Deep clone object
const cloned = _.cloneDeep(originalObject);

// Debounce function
const debouncedSearch = _.debounce(searchFunction, 300);

// Group array by property
const grouped = _.groupBy(users, 'role');
```

#### Moment.js（或 Day.js）

**适用场景：** 日期处理

```html
<!-- Day.js (lighter alternative to Moment.js) -->
<script src="https://cdn.jsdelivr.net/npm/dayjs@1.11.9/dayjs.min.js"></script>
```

**使用示例：**
```javascript
// Format date
const formatted = dayjs().format('YYYY-MM-DD HH:mm:ss');

// Add days
const future = dayjs().add(7, 'day');

// Compare dates
const isBefore = dayjs('2024-01-01').isBefore(dayjs());
```

### 数据可视化

#### Chart.js

**适用场景：** 图表和图形

```html
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
```

**使用示例：**
```javascript
const ctx = document.getElementById('myChart').getContext('2d');
const chart = new Chart(ctx, {
  type: 'bar',
  data: {
    labels: ['Jan', 'Feb', 'Mar', 'Apr', 'May'],
    datasets: [{
      label: 'Sales',
      data: [12, 19, 3, 5, 2],
      backgroundColor: 'rgba(54, 162, 235, 0.5)'
    }]
  }
});
```

#### D3.js

**适用场景：** 复杂可视化

```html
<script src="https://d3js.org/d3.v7.min.js"></script>
```

**使用示例：**
```javascript
// Create SVG chart
d3.select('#chart')
  .selectAll('rect')
  .data([4, 8, 15, 16, 23, 42])
  .enter()
  .append('rect')
  .attr('width', 20)
  .attr('height', d => d * 5)
  .attr('x', (d, i) => i * 25);
```

### HTTP 请求

#### Axios

**适用场景：** API 调用

```html
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
```

**使用示例：**
```javascript
// GET request
axios.get('https://api.example.com/data')
  .then(response => {
    console.log(response.data);
  })
  .catch(error => {
    console.error('Error:', error);
  });

// POST request
axios.post('https://api.example.com/save', {
  title: 'My Document',
  content: 'Document content'
})
  .then(response => {
    console.log('Saved:', response.data);
  });
```

### 文本处理

#### Marked.js

**适用场景：** Markdown 解析

```html
<script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
```

**使用示例：**
```javascript
const markdown = '# Hello\n\nThis is **bold** text.';
const html = marked.parse(markdown);
document.getElementById('output').innerHTML = html;
```

#### Highlight.js

**适用场景：** 代码语法高亮

```html
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.8.0/styles/default.min.css">
<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.8.0/highlight.min.js"></script>
```

**使用示例：**
```javascript
// Highlight all code blocks
document.querySelectorAll('pre code').forEach((block) => {
  hljs.highlightBlock(block);
});
```

### 表单验证

#### Validator.js

**适用场景：** 输入验证

```html
<script src="https://cdn.jsdelivr.net/npm/validator@13.11.0/validator.min.js"></script>
```

**使用示例：**
```javascript
// Validate email
const isValidEmail = validator.isEmail('test@example.com');

// Validate URL
const isValidURL = validator.isURL('https://example.com');

// Validate credit card
const isValidCard = validator.isCreditCard('4242424242424242');
```

## 管理库版本

### 版本锁定

**Error name:** Breaking changes from automatic updates

:::warning[Wrong]
```html
<!-- Using "latest" - breaks when library updates -->
<script src="https://cdn.example.com/library/latest/library.js"></script>
```
:::

:::tip[Correct]
```html
<!-- Lock to specific version -->
<script src="https://cdn.example.com/library/1.2.3/library.js"></script>

<!-- Or use semantic versioning -->
<script src="https://cdn.example.com/library@^1.2.0/library.js"></script>
```
:::

错误输出：当库自动更新并引入破坏性变更时，插件意外崩溃。

### 版本追踪

**package.json（用于文档记录）：**
```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "externalDependencies": {
    "jquery": "3.7.0",
    "lodash": "4.17.21",
    "chart.js": "4.4.0"
  }
}
```

## 优化库的使用

### 手动 Tree Shaking

**只加载所需内容：**

**Error name:** Loading entire library for one function

:::warning[Wrong]
```html
<!-- Loading entire Lodash (70KB) for one function -->
<script src="https://cdn.jsdelivr.net/npm/lodash@4.17.21/lodash.min.js"></script>
<script>
  const result = _.debounce(myFunction, 300);
</script>
```
:::

:::tip[Correct]
```html
<!-- Load only debounce function (2KB) -->
<script src="https://cdn.jsdelivr.net/npm/lodash.debounce@4.0.8/index.min.js"></script>
<script>
  const result = debounce(myFunction, 300);
</script>
```
:::

错误输出：由于加载了不必要的库代码，导致插件加载缓慢。

### 懒加载

```javascript
// Load libraries only when needed
const LibraryCache = {
  libraries: {},

  async load(name, url) {
    if (this.libraries[name]) {
      return this.libraries[name];
    }

    return new Promise((resolve, reject) => {
      const script = document.createElement('script');
      script.src = url;

      script.onload = () => {
        this.libraries[name] = true;
        resolve();
      };

      script.onerror = reject;

      document.head.appendChild(script);
    });
  }
};

// Usage
document.getElementById('chartBtn').addEventListener('click', async () => {
  if (!LibraryCache.libraries.chartjs) {
    showLoading();
    await LibraryCache.load('chartjs', 'https://cdn.jsdelivr.net/npm/chart.js');
    hideLoading();
  }

  createChart();
});
```

### 压缩与混淆

**Error name:** Using unminified library

:::warning[Wrong]
```html
<script src="library.js"></script>
```
:::

:::tip[Correct]
```html
<script src="library.min.js"></script>
```
:::

错误输出：由于引入了未压缩的大型文件，导致插件加载缓慢。

## 处理库冲突

### 命名空间冲突

**Error name:** Global variable collision

:::warning[Wrong]
```javascript
// Both libraries define window.$
<script src="jquery.js"></script>
<script src="another-library.js"></script>
<script>
  // $ is now from another-library, not jQuery!
  $('.element').hide();
</script>
```
:::

:::tip[Correct]
```javascript
// Use jQuery.noConflict()
<script src="jquery.js"></script>
<script>
  const $j = jQuery.noConflict();
</script>
<script src="another-library.js"></script>
<script>
  // Use $j for jQuery
  $j('.element').hide();

  // Use $ for another library
  $('.other').method();
</script>
```
:::

错误输出：多个库相互覆盖全局变量时，出现意外行为。

### 多版本库

```javascript
// Isolate libraries in IFEs
(function() {
  // Load version 1.0
  const script1 = document.createElement('script');
  script1.src = 'library-v1.0.js';

  script1.onload = () => {
    const LibV1 = window.Library;
    delete window.Library;

    // Use LibV1
  };

  document.head.appendChild(script1);
})();

(function() {
  // Load version 2.0
  const script2 = document.createElement('script');
  script2.src = 'library-v2.0.js';

  script2.onload = () => {
    const LibV2 = window.Library;

    // Use LibV2
  };

  document.head.appendChild(script2);
})();
```

## 安全注意事项

### 子资源完整性（SRI）

```html
<!-- Add integrity hash to verify library hasn't been tampered with -->
<script
  src="https://cdn.jsdelivr.net/npm/lodash@4.17.21/lodash.min.js"
  integrity="sha256-qXBd/EfAdjOA2FGrGAG+b3YBn2tn5A6bhz+LSgYD96k="
  crossorigin="anonymous">
</script>
```

**生成 SRI 哈希：**
```bash
# Using openssl
curl -s https://cdn.example.com/library.js | openssl dgst -sha256 -binary | openssl base64 -A
```

### 净化库输入

```javascript
// Sanitize data before passing to libraries
function sanitizeInput(input) {
  const div = document.createElement('div');
  div.textContent = input;
  return div.innerHTML;
}

// Use sanitized input
const userInput = document.getElementById('input').value;
const safe = sanitizeInput(userInput);
someLibrary.process(safe);
```

## 使用外部库进行测试

### 在测试中模拟库

```javascript
// Mock library for testing
const mockChartJS = {
  Chart: class {
    constructor(ctx, config) {
      this.ctx = ctx;
      this.config = config;
    }

    update() {
      console.log('Chart updated');
    }

    destroy() {
      console.log('Chart destroyed');
    }
  }
};

// Replace real library with mock in tests
if (window.TEST_MODE) {
  window.Chart = mockChartJS.Chart;
}
```

## 最佳实践

### 谨慎选择

```javascript
// Evaluate before adding
const libraryEvaluation = {
  size: '50KB minified',
  lastUpdate: '2 months ago',
  stars: '15K on GitHub',
  license: 'MIT',
  dependencies: 'None',
  browserSupport: 'IE11+',
  alternatives: ['library-a', 'library-b']
};
```

### 保持更新

```json
// Track library versions
{
  "dependencies": {
    "chart.js": "4.4.0",
    "notes": "Check for updates monthly"
  }
}
```

### 记录使用情况

```javascript
/**
 * External Libraries Used:
 *
 * 1. Chart.js (v4.4.0)
 *    Purpose: Data visualization
 *    License: MIT
 *    Size: 200KB
 *
 * 2. Lodash (v4.17.21)
 *    Purpose: Utility functions
 *    License: MIT
 *    Size: 70KB
 */
```

## 常见问题

### 库加载失败

**排查清单：**
```javascript
// Debug library loading
function checkLibrary(name, global) {
  if (typeof window[global] === 'undefined') {
    console.error(`${name} not loaded!`);
    console.log('Check:');
    console.log('1. URL is correct');
    console.log('2. CDN is accessible');
    console.log('3. No CORS errors');
    console.log('4. Script tag before usage');
    return false;
  }
  console.log(`${name} loaded successfully`);
  return true;
}

// Usage
window.addEventListener('load', () => {
  checkLibrary('jQuery', '$');
  checkLibrary('Lodash', '_');
  checkLibrary('Chart.js', 'Chart');
});
```

## 总结

合理使用外部库可以大幅增强您的 ONLYOFFICE 插件功能。谨慎选择库、优化加载方式、妥善处理冲突，并遵循安全最佳实践，从而构建强大、高效的插件。
