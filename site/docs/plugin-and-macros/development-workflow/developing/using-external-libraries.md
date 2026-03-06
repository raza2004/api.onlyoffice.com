---
sidebar_position: 4
---

# Using external libraries

## Overview

External libraries can significantly enhance your ONLYOFFICE plugin's capabilities by providing pre-built functionality for common tasks. This guide covers how to integrate, manage, and optimize external libraries in your plugins.

## Why use external libraries?

### Benefits

- **Save development time** - Don't reinvent the wheel
- **Tested code** - Battle-tested, reliable solutions
- **Rich features** - Complex functionality ready to use
- **Community support** - Documentation and help available
- **Regular updates** - Bug fixes and improvements

### Considerations

- **Size matters** - Large libraries slow plugin loading
- **Compatibility** - Ensure browser compatibility
- **Licensing** - Check license compatibility
- **Dependencies** - Manage dependency chains
- **Security** - Vet libraries for vulnerabilities

## Including external libraries

### Method 1: CDN (Content Delivery Network)

**Best for:** Public, popular libraries

**Advantages:**
- No local storage needed
- Fast loading (cached by browsers)
- Always up-to-date (if using latest)

**index.html:**
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

Error output: "myLibrary is not defined" when CDN is blocked or unavailable.

### Method 2: Local files

**Best for:** Custom libraries, offline plugins, large files

**Plugin structure:**
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

**index.html:**
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

### Method 3: Dynamic loading

**Best for:** Conditional features, lazy loading

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

## Popular libraries for plugins

### UI frameworks

#### Bootstrap

**Use for:** Responsive layouts, components

```html
<!-- Bootstrap CSS -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">

<!-- Bootstrap JS -->
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
```

**Example usage:**
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

**Use for:** Utility-first styling

```html
<script src="https://cdn.tailwindcss.com"></script>
```

**Example usage:**
```html
<div class="flex items-center justify-center p-4">
  <button class="bg-blue-500 text-white px-4 py-2 rounded">
    Click Me
  </button>
</div>
```

### Data manipulation

#### Lodash

**Use for:** Array/object utilities

```html
<script src="https://cdn.jsdelivr.net/npm/lodash@4.17.21/lodash.min.js"></script>
```

**Example usage:**
```javascript
// Deep clone object
const cloned = _.cloneDeep(originalObject);

// Debounce function
const debouncedSearch = _.debounce(searchFunction, 300);

// Group array by property
const grouped = _.groupBy(users, 'role');
```

#### Moment.js (or Day.js)

**Use for:** Date manipulation

```html
<!-- Day.js (lighter alternative to Moment.js) -->
<script src="https://cdn.jsdelivr.net/npm/dayjs@1.11.9/dayjs.min.js"></script>
```

**Example usage:**
```javascript
// Format date
const formatted = dayjs().format('YYYY-MM-DD HH:mm:ss');

// Add days
const future = dayjs().add(7, 'day');

// Compare dates
const isBefore = dayjs('2024-01-01').isBefore(dayjs());
```

### Visualization

#### Chart.js

**Use for:** Charts and graphs

```html
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
```

**Example usage:**
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

**Use for:** Complex visualizations

```html
<script src="https://d3js.org/d3.v7.min.js"></script>
```

**Example usage:**
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

### HTTP requests

#### Axios

**Use for:** API calls

```html
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
```

**Example usage:**
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

### Text processing

#### Marked.js

**Use for:** Markdown parsing

```html
<script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
```

**Example usage:**
```javascript
const markdown = '# Hello\n\nThis is **bold** text.';
const html = marked.parse(markdown);
document.getElementById('output').innerHTML = html;
```

#### Highlight.js

**Use for:** Code syntax highlighting

```html
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.8.0/styles/default.min.css">
<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.8.0/highlight.min.js"></script>
```

**Example usage:**
```javascript
// Highlight all code blocks
document.querySelectorAll('pre code').forEach((block) => {
  hljs.highlightBlock(block);
});
```

### Form validation

#### Validator.js

**Use for:** Input validation

```html
<script src="https://cdn.jsdelivr.net/npm/validator@13.11.0/validator.min.js"></script>
```

**Example usage:**
```javascript
// Validate email
const isValidEmail = validator.isEmail('test@example.com');

// Validate URL
const isValidURL = validator.isURL('https://example.com');

// Validate credit card
const isValidCard = validator.isCreditCard('4242424242424242');
```

## Managing library versions

### Version locking

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

Error output: Plugin breaks unexpectedly when library auto-updates with breaking changes.

### Version tracking

**package.json (for documentation):**
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

## Optimizing library usage

### Tree shaking (manual)

**Load only what you need:**

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

Error output: Slow plugin load time due to unnecessary library code.

### Lazy loading

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

### Minification and compression

```javascript
// Prefer minified versions
// ✅ Good
<script src="library.min.js"></script>  // 50KB

// ❌ Bad
<script src="library.js"></script>      // 200KB
```

## Handling library conflicts

### Namespace conflicts

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

Error output: Unexpected behavior when libraries overwrite each other's global variables.

### Multiple library versions

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

## Security considerations

### Subresource Integrity (SRI)

```html
<!-- Add integrity hash to verify library hasn't been tampered with -->
<script 
  src="https://cdn.jsdelivr.net/npm/lodash@4.17.21/lodash.min.js"
  integrity="sha256-qXBd/EfAdjOA2FGrGAG+b3YBn2tn5A6bhz+LSgYD96k="
  crossorigin="anonymous">
</script>
```

**Generate SRI hash:**
```bash
# Using openssl
curl -s https://cdn.example.com/library.js | openssl dgst -sha256 -binary | openssl base64 -A
```

### Sanitizing library inputs

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

## Testing with external libraries

### Mock libraries in tests

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

## Best practices

### Choose wisely

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

### Keep updated

```json
// Track library versions
{
  "dependencies": {
    "chart.js": "4.4.0",
    "notes": "Check for updates monthly"
  }
}
```

### Document usage

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

## Common issues

### Library not loading

**Checklist:**
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

## Conclusion

External libraries can greatly enhance your ONLYOFFICE plugin's functionality when used wisely. Choose libraries carefully, optimize their loading, handle conflicts properly, and maintain security best practices to create powerful, efficient plugins.
