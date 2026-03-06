---
sidebar_position: 3
---

# Hot reload & live testing

## Overview

Hot reload and live testing capabilities significantly speed up plugin development by allowing you to see changes instantly without manually restarting ONLYOFFICE. This guide covers techniques for implementing hot reload, live testing workflows, and rapid iteration strategies.

## Why hot reload matters

### Traditional workflow problems

**Without hot reload:**
1. Make code changes
2. Save files
3. Close ONLYOFFICE completely
4. Restart ONLYOFFICE
5. Open document
6. Open plugin
7. Test changes
8. Repeat for every change ⏱️ **2-3 minutes per iteration**

**With hot reload:**
1. Make code changes
2. Save files
3. See changes immediately ⏱️ **2-3 seconds per iteration**

### Benefits

- **Faster development** - 50-100x faster iteration
- **Better focus** - Stay in flow state
- **More testing** - Test more scenarios quickly
- **Rapid debugging** - Fix issues faster
- **Improved quality** - Iterate until perfect

## Setting up hot reload

### Method 1: Using Live Server

**Install Live Server (VS Code):**
```bash
# Via VS Code Extensions
1. Open VS Code
2. Go to Extensions (Ctrl+Shift+X)
3. Search "Live Server"
4. Install by Ritwick Dey
```

**Setup plugin for Live Server:**

**config.json:**
```json
{
  "name": "Hot Reload Plugin",
  "guid": "asc.{hot-reload-001}",
  "version": "1.0.0",
  "variations": [
    {
      "url": "http://localhost:5500/index.html",
      "icons": ["icon.png"],
      "EditorsSupport": ["word", "cell", "slide"],
      "isModal": false,
      "isInsideMode": true,
      "type": "panel"
    }
  ]
}
```

**Start Live Server:**
```bash
# In VS Code:
1. Right-click on index.html
2. Select "Open with Live Server"
3. Plugin will now reload on file changes
```

### Method 2: Using Node.js server

**Create development server:**

**server.js:**
```javascript
const express = require('express');
const cors = require('cors');
const chokidar = require('chokidar');
const WebSocket = require('ws');
const path = require('path');

const app = express();
const PORT = 3000;
const WS_PORT = 3001;

// Enable CORS
app.use(cors());

// Serve static files
app.use(express.static(__dirname));

// WebSocket server for hot reload
const wss = new WebSocket.Server({ port: WS_PORT });

// Watch for file changes
const watcher = chokidar.watch([
  '*.js',
  '*.html',
  '*.css',
  '*.json'
], {
  ignored: /node_modules/,
  persistent: true
});

watcher.on('change', (filePath) => {
  console.log(`File changed: ${filePath}`);
  
  // Notify all connected clients
  wss.clients.forEach((client) => {
    if (client.readyState === WebSocket.OPEN) {
      client.send(JSON.stringify({
        type: 'reload',
        file: filePath,
        timestamp: Date.now()
      }));
    }
  });
});

app.listen(PORT, () => {
  console.log(`Server running at http://localhost:${PORT}`);
  console.log(`WebSocket running at ws://localhost:${WS_PORT}`);
});
```

**Add hot reload client to plugin:**

**index.html:**
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Hot Reload Plugin</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div id="app">
        <!-- Your plugin UI -->
    </div>
    
    <!-- Hot reload script -->
    <script src="hot-reload.js"></script>
    <script src="plugin.js"></script>
</body>
</html>
```

**hot-reload.js:**
```javascript
(function() {
  // Only enable in development
  const DEV_MODE = true;
  const WS_URL = 'ws://localhost:3001';
  
  if (!DEV_MODE) return;
  
  let ws;
  let reconnectInterval;
  
  function connect() {
    ws = new WebSocket(WS_URL);
    
    ws.onopen = () => {
      console.log('[Hot Reload] Connected');
      clearInterval(reconnectInterval);
    };
    
    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      
      if (data.type === 'reload') {
        console.log('[Hot Reload] Reloading...', data.file);
        
        // Reload CSS without full page reload
        if (data.file.endsWith('.css')) {
          reloadCSS();
        } else {
          // Full reload for JS/HTML changes
          window.location.reload();
        }
      }
    };
    
    ws.onclose = () => {
      console.log('[Hot Reload] Disconnected, reconnecting...');
      
      // Auto-reconnect
      reconnectInterval = setInterval(() => {
        connect();
      }, 2000);
    };
    
    ws.onerror = (error) => {
      console.error('[Hot Reload] Error:', error);
    };
  }
  
  function reloadCSS() {
    const links = document.querySelectorAll('link[rel="stylesheet"]');
    links.forEach(link => {
      const href = link.href.split('?')[0];
      link.href = href + '?t=' + Date.now();
    });
  }
  
  // Start connection
  connect();
  
  // Show hot reload indicator
  const indicator = document.createElement('div');
  indicator.id = 'hot-reload-indicator';
  indicator.textContent = '🔥';
  indicator.style.cssText = `
    position: fixed;
    bottom: 10px;
    right: 10px;
    width: 30px;
    height: 30px;
    background: #4caf50;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 16px;
    z-index: 9999;
    cursor: pointer;
  `;
  indicator.title = 'Hot Reload Active';
  document.body.appendChild(indicator);
})();
```

**Install dependencies and run:**
```bash
npm install express cors chokidar ws

node server.js
```

### Method 3: Using symbolic links

**Create symbolic link to plugin:**

**Windows (Run as Administrator):**
```cmd
mklink /D "C:\Program Files\ONLYOFFICE\DesktopEditors\editors\sdkjs-plugins\dev-plugin" "C:\dev\my-plugin"
```

**macOS/Linux:**
```bash
ln -s ~/dev/my-plugin /Applications/ONLYOFFICE.app/Contents/Resources/editors/sdkjs-plugins/dev-plugin
```

**Benefit:** Changes to files are immediately available without copying.

**Limitation:** Still requires ONLYOFFICE restart to see changes.

## Live testing workflows

### Auto-reload on save

**Error name:** Manual reload slowing down development

:::warning[Wrong]
```javascript
// No auto-reload - developer must manually refresh
window.Asc.plugin.init = function() {
  setupPlugin();
};
```
:::

:::tip[Correct]
```javascript
// Auto-reload with hot module replacement
(function() {
  let currentVersion = Date.now();
  
  window.Asc.plugin.init = function(data) {
    const initVersion = currentVersion;
    
    // Setup plugin
    setupPlugin(data);
    
    // Check for updates
    if (window.DEV_MODE) {
      setInterval(() => {
        checkForUpdates(initVersion);
      }, 1000);
    }
  };
  
  function checkForUpdates(version) {
    fetch('./version.json?t=' + Date.now())
      .then(r => r.json())
      .then(data => {
        if (data.version > version) {
          console.log('New version detected, reloading...');
          window.location.reload();
        }
      })
      .catch(() => {
        // Ignore errors in dev mode
      });
  }
})();
```

**version.json (auto-generated by build script):**
```json
{
  "version": 1678901234567
}
```
:::

Error output: Developer wastes time manually reloading after each change.

### Watch mode with automatic testing

**test-watch.js:**
```javascript
const chokidar = require('chokidar');
const { execSync } = require('child_process');

console.log('👀 Watching for changes...\n');

const watcher = chokidar.watch([
  '*.js',
  '*.html',
  '*.css'
], {
  ignored: /node_modules/,
  persistent: true,
  ignoreInitial: true
});

watcher.on('change', (path) => {
  console.log(`\n📝 Changed: ${path}`);
  console.log('🧪 Running tests...\n');
  
  try {
    execSync('npm test', { stdio: 'inherit' });
    console.log('\n✅ Tests passed!');
  } catch (error) {
    console.log('\n❌ Tests failed!');
  }
  
  console.log('\n👀 Watching for changes...\n');
});
```

**Run watch mode:**
```bash
node test-watch.js
```

## Live testing strategies

### Multi-device testing

**Setup for simultaneous testing:**

1. **Run plugin on local network:**
```javascript
// config.json - use local IP instead of localhost
{
  "variations": [{
    "url": "http://192.168.1.100:3000/index.html"
  }]
}
```

2. **Test on multiple devices:**
```
Desktop (Windows)  → http://192.168.1.100:3000
Desktop (Mac)      → http://192.168.1.100:3000
Laptop (Linux)     → http://192.168.1.100:3000
Mobile Browser     → http://192.168.1.100:3000
```

### Browser DevTools live editing

**Enable live CSS editing:**

```javascript
// Inject style changes without reload
function updateStyles(css) {
  let style = document.getElementById('live-styles');
  
  if (!style) {
    style = document.createElement('style');
    style.id = 'live-styles';
    document.head.appendChild(style);
  }
  
  style.textContent = css;
}

// Usage in DevTools console
updateStyles(`
  .button {
    background: #2196f3;
    color: white;
  }
`);
```

### Live debugging

**Add debug panel:**

```javascript
class DebugPanel {
  constructor() {
    this.panel = null;
    this.logs = [];
    this.createPanel();
  }
  
  createPanel() {
    this.panel = document.createElement('div');
    this.panel.id = 'debug-panel';
    this.panel.style.cssText = `
      position: fixed;
      bottom: 0;
      left: 0;
      right: 0;
      height: 200px;
      background: #1e1e1e;
      color: #d4d4d4;
      font-family: monospace;
      font-size: 12px;
      padding: 10px;
      overflow-y: auto;
      border-top: 1px solid #333;
      z-index: 10000;
      display: none;
    `;
    
    document.body.appendChild(this.panel);
    
    // Toggle with Ctrl+D
    document.addEventListener('keydown', (e) => {
      if (e.ctrlKey && e.key === 'd') {
        e.preventDefault();
        this.toggle();
      }
    });
  }
  
  toggle() {
    this.panel.style.display = 
      this.panel.style.display === 'none' ? 'block' : 'none';
  }
  
  log(message, data) {
    const entry = {
      timestamp: new Date().toLocaleTimeString(),
      message,
      data
    };
    
    this.logs.push(entry);
    
    const line = document.createElement('div');
    line.textContent = `[${entry.timestamp}] ${message}`;
    
    if (data) {
      line.textContent += ': ' + JSON.stringify(data);
    }
    
    this.panel.appendChild(line);
    this.panel.scrollTop = this.panel.scrollHeight;
  }
  
  clear() {
    this.logs = [];
    this.panel.innerHTML = '';
  }
}

// Global debug panel
const debug = new DebugPanel();

// Usage
window.Asc.plugin.init = function(data) {
  debug.log('Plugin initialized', data);
};
```

## Performance monitoring during development

### Real-time performance metrics

```javascript
class PerformanceMonitor {
  constructor() {
    this.metrics = {};
    this.observers = [];
    this.setupObservers();
    this.createDisplay();
  }
  
  setupObservers() {
    // Monitor long tasks
    if ('PerformanceObserver' in window) {
      const observer = new PerformanceObserver((list) => {
        for (const entry of list.getEntries()) {
          if (entry.duration > 50) {
            console.warn('Long task detected:', entry.duration + 'ms');
          }
        }
      });
      
      observer.observe({ entryTypes: ['longtask'] });
      this.observers.push(observer);
    }
  }
  
  createDisplay() {
    const display = document.createElement('div');
    display.id = 'perf-monitor';
    display.style.cssText = `
      position: fixed;
      top: 10px;
      right: 10px;
      background: rgba(0, 0, 0, 0.8);
      color: #0f0;
      padding: 10px;
      font-family: monospace;
      font-size: 11px;
      border-radius: 4px;
      z-index: 9999;
    `;
    
    document.body.appendChild(display);
    
    setInterval(() => {
      this.updateDisplay(display);
    }, 1000);
  }
  
  updateDisplay(display) {
    const memory = performance.memory;
    const fps = this.calculateFPS();
    
    display.innerHTML = `
      <div>FPS: ${fps}</div>
      <div>Memory: ${(memory.usedJSHeapSize / 1048576).toFixed(2)} MB</div>
      <div>Limit: ${(memory.jsHeapSizeLimit / 1048576).toFixed(2)} MB</div>
    `;
  }
  
  calculateFPS() {
    // Simple FPS calculation
    if (!this.lastTime) {
      this.lastTime = performance.now();
      this.frames = 0;
      return 0;
    }
    
    this.frames++;
    const now = performance.now();
    const delta = now - this.lastTime;
    
    if (delta >= 1000) {
      const fps = Math.round((this.frames * 1000) / delta);
      this.frames = 0;
      this.lastTime = now;
      return fps;
    }
    
    return this.lastFPS || 0;
  }
}

// Enable in development
if (window.DEV_MODE) {
  new PerformanceMonitor();
}
```

## Automated testing during development

### Unit tests with hot reload

**test-runner.html:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Plugin Tests</title>
    <style>
        body { font-family: Arial, sans-serif; padding: 20px; }
        .test { padding: 10px; margin: 5px 0; }
        .pass { background: #d4edda; color: #155724; }
        .fail { background: #f8d7da; color: #721c24; }
    </style>
</head>
<body>
    <h1>Plugin Test Suite</h1>
    <div id="results"></div>
    
    <script src="plugin.js"></script>
    <script>
        class TestRunner {
          constructor() {
            this.tests = [];
            this.results = [];
          }
          
          test(name, fn) {
            this.tests.push({ name, fn });
          }
          
          async run() {
            const resultsDiv = document.getElementById('results');
            resultsDiv.innerHTML = '';
            
            for (const test of this.tests) {
              const resultDiv = document.createElement('div');
              resultDiv.className = 'test';
              
              try {
                await test.fn();
                resultDiv.className += ' pass';
                resultDiv.textContent = `✓ ${test.name}`;
                this.results.push({ name: test.name, passed: true });
              } catch (error) {
                resultDiv.className += ' fail';
                resultDiv.textContent = `✗ ${test.name}: ${error.message}`;
                this.results.push({
                  name: test.name,
                  passed: false,
                  error: error.message
                });
              }
              
              resultsDiv.appendChild(resultDiv);
            }
            
            this.printSummary();
          }
          
          printSummary() {
            const passed = this.results.filter(r => r.passed).length;
            const failed = this.results.filter(r => !r.passed).length;
            
            console.log('\n=== Test Summary ===');
            console.log(`Total: ${this.results.length}`);
            console.log(`Passed: ${passed}`);
            console.log(`Failed: ${failed}`);
          }
        }
        
        // Define tests
        const runner = new TestRunner();
        
        runner.test('Plugin API available', () => {
          if (!window.Asc || !window.Asc.plugin) {
            throw new Error('Plugin API not found');
          }
        });
        
        runner.test('Config is valid', () => {
          // Add validation logic
        });
        
        // Run tests
        runner.run();
        
        // Auto-reload on changes
        setInterval(() => {
          fetch('version.json?t=' + Date.now())
            .then(r => r.json())
            .then(() => {
              runner.run();
            });
        }, 2000);
    </script>
</body>
</html>
```

## Best practices

### Development vs production mode

**Error name:** Debug code running in production

:::warning[Wrong]
```javascript
// Debug code always running
console.log('Debug: User clicked button');
debugPanel.show();
performanceMonitor.track();
```
:::

:::tip[Correct]
```javascript
// Use environment flag
const DEV_MODE = window.location.hostname === 'localhost' || 
                 window.location.hostname === '127.0.0.1';

if (DEV_MODE) {
  console.log('Debug: User clicked button');
  debugPanel.show();
  performanceMonitor.track();
}

// Or use environment variable from build
if (process.env.NODE_ENV === 'development') {
  // Development-only code
}
```
:::

Error output: Production plugin slower due to debug code, console cluttered with debug messages.

### Efficient file watching

```javascript
// Debounce file change events
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

// Watch with debounce
watcher.on('change', debounce((path) => {
  console.log('File changed:', path);
  reloadPlugin();
}, 300));
```

## Troubleshooting

### Hot reload not working

**Common causes:**
1. CORS blocking requests
2. WebSocket connection failed
3. Wrong URL in config.json
4. Firewall blocking port

**Solutions:**
```javascript
// Check WebSocket connection
const ws = new WebSocket('ws://localhost:3001');
ws.onopen = () => console.log('Connected');
ws.onerror = (e) => console.error('Connection failed:', e);
```

## Conclusion

Hot reload and live testing dramatically improve plugin development speed and quality. By setting up proper development workflows, monitoring tools, and automated testing, you can iterate quickly and build better ONLYOFFICE plugins.
