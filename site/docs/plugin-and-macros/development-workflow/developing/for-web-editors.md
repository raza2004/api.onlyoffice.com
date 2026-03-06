---
sidebar_position: 1
---

# For web editors

## Overview

Developing plugins for ONLYOFFICE web editors (browser-based) requires understanding web technologies, browser APIs, and ONLYOFFICE-specific considerations. This guide covers best practices, patterns, and techniques for creating robust web-based plugins.

## Web editor environment

### Understanding the web context

ONLYOFFICE web editors run entirely in the browser with these characteristics:

- **Browser-based** - No access to local file system
- **Sandboxed** - Iframe isolation for security
- **Network-dependent** - Requires internet connection
- **Cross-browser** - Must work in multiple browsers
- **Responsive** - Adapts to different screen sizes

### Supported browsers

ONLYOFFICE web editors support:

- **Chrome/Edge** (Chromium) - Latest 2 versions
- **Firefox** - Latest 2 versions
- **Safari** - Latest 2 versions
- **Opera** - Latest version
- **Mobile browsers** - Chrome Mobile, Safari Mobile

## Plugin structure for web editors

### Basic web plugin structure

```
my-web-plugin/
├── config.json
├── index.html
├── plugin.js
├── styles.css
├── assets/
│   ├── icons/
│   │   ├── icon.svg
│   │   └── icon@2x.svg
│   └── images/
└── translations/
    ├── en.json
    └── ru.json
```

### Configuration for web editors

**config.json:**
```json
{
  "name": "Web Plugin Example",
  "guid": "asc.{web-plugin-001}",
  "version": "1.0.0",
  "variations": [
    {
      "description": "Example plugin for web editors",
      "url": "index.html",
      
      "icons": [
        "assets/icons/icon.svg",
        "assets/icons/icon@2x.svg"
      ],
      
      "isViewer": false,
      "EditorsSupport": ["word", "cell", "slide"],
      "isVisual": true,
      "isModal": false,
      "isInsideMode": true,
      "type": "panel",
      "size": [350, 600],
      
      "initDataType": "text",
      "initOnSelectionChanged": true
    }
  ]
}
```

## Optimizing for web performance

### Minimize initial load time

**Error name:** Large bundle causing slow plugin load

:::warning[Wrong]
```html
<!-- Loading large libraries synchronously -->
<script src="https://cdn.example.com/large-library-5mb.js"></script>
<script src="https://cdn.example.com/another-library-3mb.js"></script>
<script src="plugin.js"></script>
```
:::

:::tip[Correct]
```html
<!-- Load only essential scripts, defer others -->
<script src="https://onlyoffice.github.io/sdkjs-plugins/v1/plugins.js"></script>
<script src="plugin.js" defer></script>

<!-- Lazy load heavy libraries only when needed -->
<script>
  function loadHeavyLibrary() {
    return new Promise((resolve, reject) => {
      const script = document.createElement('script');
      script.src = 'https://cdn.example.com/large-library.js';
      script.onload = resolve;
      script.onerror = reject;
      document.head.appendChild(script);
    });
  }
  
  // Load only when user triggers specific feature
  async function useAdvancedFeature() {
    await loadHeavyLibrary();
    // Now use the library
  }
</script>
```
:::

Error output: Plugin takes 5-10 seconds to load, poor user experience.

### Optimize asset loading

```javascript
// Lazy load images
const lazyLoadImages = () => {
  const images = document.querySelectorAll('img[data-src]');
  
  const imageObserver = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const img = entry.target;
        img.src = img.dataset.src;
        img.removeAttribute('data-src');
        imageObserver.unobserve(img);
      }
    });
  });
  
  images.forEach(img => imageObserver.observe(img));
};

window.Asc.plugin.init = function() {
  lazyLoadImages();
};
```

### Use efficient data structures

**Error name:** Inefficient data lookup causing lag

:::warning[Wrong]
```javascript
// Using array for frequent lookups - O(n) complexity
const items = ['item1', 'item2', 'item3', /*...1000 items*/];

function findItem(id) {
  // Slow for large arrays
  return items.find(item => item.id === id);
}
```
:::

:::tip[Correct]
```javascript
// Using Map for O(1) lookups
const itemsMap = new Map();
items.forEach(item => itemsMap.set(item.id, item));

function findItem(id) {
  // Fast constant-time lookup
  return itemsMap.get(id);
}
```
:::

Error output: Plugin becomes unresponsive with large datasets.

## Handling network requests

### Making API calls

```javascript
// Best practices for network requests
class APIClient {
  constructor(baseURL) {
    this.baseURL = baseURL;
    this.timeout = 10000; // 10 seconds
  }
  
  async fetch(endpoint, options = {}) {
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), this.timeout);
    
    try {
      const response = await fetch(`${this.baseURL}${endpoint}`, {
        ...options,
        signal: controller.signal,
        headers: {
          'Content-Type': 'application/json',
          ...options.headers
        }
      });
      
      clearTimeout(timeoutId);
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      return await response.json();
    } catch (error) {
      if (error.name === 'AbortError') {
        throw new Error('Request timeout');
      }
      throw error;
    }
  }
  
  async get(endpoint) {
    return this.fetch(endpoint, { method: 'GET' });
  }
  
  async post(endpoint, data) {
    return this.fetch(endpoint, {
      method: 'POST',
      body: JSON.stringify(data)
    });
  }
}

// Usage
const api = new APIClient('https://api.example.com');

async function loadData() {
  try {
    showLoading();
    const data = await api.get('/data');
    processData(data);
  } catch (error) {
    console.error('Failed to load data:', error);
    showError('Unable to load data. Please try again.');
  } finally {
    hideLoading();
  }
}
```

### Handling CORS

**Error name:** CORS blocking API requests

:::warning[Wrong]
```javascript
// Direct fetch to API without CORS headers
fetch('https://external-api.com/data')
  .then(response => response.json())
  .then(data => console.log(data));
```
:::

:::tip[Correct]
```javascript
// Option 1: Use proxy server
fetch('https://your-proxy.com/api/data')
  .then(response => response.json())
  .then(data => console.log(data));

// Option 2: Ensure API supports CORS
// Server must include headers:
// Access-Control-Allow-Origin: *
// Access-Control-Allow-Methods: GET, POST

// Option 3: Use JSONP for GET requests (legacy)
function loadJSONP(url, callback) {
  const script = document.createElement('script');
  const callbackName = 'jsonp_' + Date.now();
  
  window[callbackName] = function(data) {
    callback(data);
    delete window[callbackName];
    document.head.removeChild(script);
  };
  
  script.src = `${url}?callback=${callbackName}`;
  document.head.appendChild(script);
}
```
:::

Error output: "CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource."

## Browser storage

### Using localStorage

```javascript
// Storage utility for web plugins
const Storage = {
  set: function(key, value) {
    try {
      const data = JSON.stringify(value);
      localStorage.setItem(`plugin_${key}`, data);
      return true;
    } catch (error) {
      console.error('Storage error:', error);
      return false;
    }
  },
  
  get: function(key, defaultValue = null) {
    try {
      const data = localStorage.getItem(`plugin_${key}`);
      return data ? JSON.parse(data) : defaultValue;
    } catch (error) {
      console.error('Storage error:', error);
      return defaultValue;
    }
  },
  
  remove: function(key) {
    localStorage.removeItem(`plugin_${key}`);
  },
  
  clear: function() {
    Object.keys(localStorage).forEach(key => {
      if (key.startsWith('plugin_')) {
        localStorage.removeItem(key);
      }
    });
  }
};

// Usage
window.Asc.plugin.init = function() {
  // Load saved settings
  const settings = Storage.get('settings', { theme: 'light' });
  applySettings(settings);
};

function saveSettings(settings) {
  Storage.set('settings', settings);
}
```

### Storage quota management

**Error name:** Exceeding storage quota

:::warning[Wrong]
```javascript
// Storing large data without checking quota
function saveData(data) {
  localStorage.setItem('largeData', JSON.stringify(data));
}
```
:::

:::tip[Correct]
```javascript
function saveData(data) {
  try {
    const dataString = JSON.stringify(data);
    const sizeInBytes = new Blob([dataString]).size;
    const sizeInMB = sizeInBytes / (1024 * 1024);
    
    if (sizeInMB > 5) {
      console.warn('Data too large for localStorage');
      // Use IndexedDB instead
      return saveToIndexedDB(data);
    }
    
    localStorage.setItem('largeData', dataString);
  } catch (error) {
    if (error.name === 'QuotaExceededError') {
      console.error('Storage quota exceeded');
      // Clear old data or use compression
      clearOldData();
      return saveData(data);
    }
    throw error;
  }
}
```
:::

Error output: "QuotaExceededError: Failed to execute 'setItem' on 'Storage': Setting the value exceeded the quota."

## Responsive design

### Mobile-friendly plugins

```css
/* Responsive plugin styles */
.plugin-container {
  padding: 16px;
  max-width: 100%;
}

/* Tablet and below */
@media (max-width: 768px) {
  .plugin-container {
    padding: 12px;
  }
  
  .button-group {
    flex-direction: column;
  }
  
  .button-group button {
    width: 100%;
    margin-bottom: 8px;
  }
}

/* Mobile */
@media (max-width: 480px) {
  .plugin-container {
    padding: 8px;
    font-size: 14px;
  }
  
  .header {
    font-size: 16px;
  }
  
  input, select, textarea {
    font-size: 16px; /* Prevent zoom on iOS */
  }
}

/* Touch-friendly tap targets */
@media (hover: none) and (pointer: coarse) {
  button, .clickable {
    min-height: 44px;
    min-width: 44px;
  }
}
```

### Touch events

```javascript
// Handle both mouse and touch events
class TouchHandler {
  constructor(element) {
    this.element = element;
    this.startX = 0;
    this.startY = 0;
    
    // Use passive listeners for better scroll performance
    element.addEventListener('touchstart', this.handleStart.bind(this), { passive: true });
    element.addEventListener('touchmove', this.handleMove.bind(this), { passive: false });
    element.addEventListener('touchend', this.handleEnd.bind(this));
  }
  
  handleStart(e) {
    const touch = e.touches[0];
    this.startX = touch.clientX;
    this.startY = touch.clientY;
  }
  
  handleMove(e) {
    if (!this.startX || !this.startY) return;
    
    const touch = e.touches[0];
    const diffX = touch.clientX - this.startX;
    const diffY = touch.clientY - this.startY;
    
    // Detect swipe
    if (Math.abs(diffX) > 50) {
      if (diffX > 0) {
        this.onSwipeRight();
      } else {
        this.onSwipeLeft();
      }
      e.preventDefault();
    }
  }
  
  handleEnd(e) {
    this.startX = 0;
    this.startY = 0;
  }
  
  onSwipeLeft() {
    console.log('Swiped left');
  }
  
  onSwipeRight() {
    console.log('Swiped right');
  }
}
```

## Security considerations

### Content Security Policy (CSP)

```javascript
// Safe content insertion
function insertSafeHTML(container, html) {
  // Create a sandbox iframe for untrusted HTML
  const iframe = document.createElement('iframe');
  iframe.sandbox = 'allow-same-origin';
  iframe.style.display = 'none';
  document.body.appendChild(iframe);
  
  const doc = iframe.contentDocument;
  doc.open();
  doc.write(html);
  doc.close();
  
  // Copy sanitized content
  container.innerHTML = doc.body.innerHTML;
  
  // Remove iframe
  document.body.removeChild(iframe);
}
```

### XSS prevention

**Error name:** XSS vulnerability from user input

:::warning[Wrong]
```javascript
// Directly inserting user input - DANGEROUS
function displayUserInput(input) {
  document.getElementById('output').innerHTML = input;
}
```
:::

:::tip[Correct]
```javascript
// Properly escape user input
function displayUserInput(input) {
  const element = document.getElementById('output');
  element.textContent = input; // Automatically escapes
  
  // Or use a sanitization library
  element.innerHTML = DOMPurify.sanitize(input);
}

// Escape HTML entities
function escapeHTML(str) {
  const div = document.createElement('div');
  div.textContent = str;
  return div.innerHTML;
}
```
:::

Error output: Malicious script execution from user input.

## Testing web plugins

### Cross-browser testing

```javascript
// Browser detection for specific workarounds
const Browser = {
  isChrome: /Chrome/.test(navigator.userAgent),
  isFirefox: /Firefox/.test(navigator.userAgent),
  isSafari: /Safari/.test(navigator.userAgent) && !/Chrome/.test(navigator.userAgent),
  isEdge: /Edg/.test(navigator.userAgent),
  isMobile: /Android|webOS|iPhone|iPad|iPod/i.test(navigator.userAgent)
};

window.Asc.plugin.init = function() {
  if (Browser.isSafari) {
    // Safari-specific initialization
    applySafariWorkarounds();
  }
  
  if (Browser.isMobile) {
    // Mobile-specific UI
    enableMobileUI();
  }
};
```

### Performance monitoring

```javascript
// Monitor plugin performance in browser
const PerformanceTracker = {
  metrics: {},
  
  start: function(label) {
    this.metrics[label] = { start: performance.now() };
  },
  
  end: function(label) {
    if (!this.metrics[label]) return;
    
    const duration = performance.now() - this.metrics[label].start;
    this.metrics[label].duration = duration;
    
    // Log slow operations
    if (duration > 1000) {
      console.warn(`Slow operation "${label}": ${duration}ms`);
    }
    
    return duration;
  },
  
  report: function() {
    console.table(this.metrics);
  }
};

// Usage
window.Asc.plugin.init = function() {
  PerformanceTracker.start('initialization');
  
  // Plugin setup
  
  PerformanceTracker.end('initialization');
};
```

## Debugging web plugins

### Console logging

```javascript
// Structured logging for web debugging
const Logger = {
  prefix: '[MyPlugin]',
  
  log: function(...args) {
    console.log(this.prefix, ...args);
  },
  
  warn: function(...args) {
    console.warn(this.prefix, ...args);
  },
  
  error: function(...args) {
    console.error(this.prefix, ...args);
  },
  
  table: function(data) {
    console.log(this.prefix);
    console.table(data);
  },
  
  group: function(label) {
    console.group(this.prefix + ' ' + label);
  },
  
  groupEnd: function() {
    console.groupEnd();
  }
};

// Usage
window.Asc.plugin.init = function(data) {
  Logger.group('Initialization');
  Logger.log('Init data:', data);
  Logger.log('Browser:', navigator.userAgent);
  Logger.groupEnd();
};
```

## Best practices

### Code organization

```javascript
// Modular plugin structure
const MyPlugin = (function() {
  // Private variables
  let state = {
    initialized: false,
    data: null
  };
  
  // Private functions
  function initialize() {
    if (state.initialized) return;
    
    setupUI();
    attachEvents();
    loadSettings();
    
    state.initialized = true;
  }
  
  function setupUI() {
    // UI setup code
  }
  
  function attachEvents() {
    // Event listeners
  }
  
  function loadSettings() {
    // Load saved settings
  }
  
  // Public API
  return {
    init: initialize,
    
    getData: function() {
      return state.data;
    },
    
    setData: function(data) {
      state.data = data;
    }
  };
})();

// Initialize plugin
window.Asc.plugin.init = function(data) {
  MyPlugin.init();
  if (data) {
    MyPlugin.setData(data);
  }
};
```

## Conclusion

Developing plugins for ONLYOFFICE web editors requires attention to performance, compatibility, and security. By following these best practices and patterns, you can create robust, user-friendly plugins that work seamlessly across different browsers and devices.
