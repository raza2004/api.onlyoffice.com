---
sidebar_position: 2
---

# For desktop editors

## Overview

Developing plugins for ONLYOFFICE Desktop Editors provides unique opportunities and challenges compared to web-based development. Desktop editors offer more capabilities, better performance, and different testing workflows. This guide covers desktop-specific plugin development techniques.

## Desktop editor environment

### Key differences from web editors

**Advantages:**
- **File system access** - Read and write local files
- **Native integrations** - Interact with OS features
- **Better performance** - Direct hardware access
- **Offline capability** - No internet required
- **More storage** - No browser storage limits

**Considerations:**
- **Cross-platform compatibility** - Windows, macOS, Linux differences
- **Version fragmentation** - Users on different versions
- **Update distribution** - Manual or auto-update mechanisms
- **Testing complexity** - Test on multiple OS platforms

### Supported platforms

- **Windows** - Windows 7 SP1 and later
- **macOS** - macOS 10.10 (Yosemite) and later
- **Linux** - Ubuntu 16.04+, Debian 9+, Fedora, CentOS

## File system integration

### Accessing local files

**Error name:** Using web APIs for file access

:::warning[Wrong]
```javascript
// Web File API doesn't work in desktop context
const input = document.createElement('input');
input.type = 'file';
input.onchange = (e) => {
  const file = e.target.files[0];
  // Limited access in desktop
};
```
:::

:::tip[Correct]
```javascript
// Use ONLYOFFICE plugin methods for file operations
window.Asc.plugin.executeMethod("GetFileContent", [], function(content) {
  if (content) {
    processFileContent(content);
  }
});

// Or use callCommand for advanced file operations
window.Asc.plugin.callCommand(function() {
  const doc = Api.GetDocument();
  // Work with document directly
}, false);
```
:::

Error output: File API not available or behaves differently on desktop.

### Reading local files

```javascript
// Reading files in desktop environment
function readLocalFile(filePath) {
  // Note: Direct file system access is limited
  // Use ONLYOFFICE methods instead
  
  window.Asc.plugin.executeMethod("LoadFile", [filePath], function(result) {
    if (result && result.success) {
      console.log("File loaded:", result.data);
      processData(result.data);
    } else {
      console.error("Failed to load file");
    }
  });
}
```

### Saving files locally

```javascript
// Save plugin data to local storage
function saveToLocalStorage(key, data) {
  try {
    const serialized = JSON.stringify(data);
    localStorage.setItem(`plugin_${key}`, serialized);
    return true;
  } catch (error) {
    console.error("Save failed:", error);
    return false;
  }
}

// Load from local storage
function loadFromLocalStorage(key, defaultValue = null) {
  try {
    const item = localStorage.getItem(`plugin_${key}`);
    return item ? JSON.parse(item) : defaultValue;
  } catch (error) {
    console.error("Load failed:", error);
    return defaultValue;
  }
}
```

## Platform-specific development

### Detecting the platform

```javascript
// Platform detection utility
const Platform = {
  isWindows: navigator.platform.indexOf('Win') > -1,
  isMac: navigator.platform.indexOf('Mac') > -1,
  isLinux: navigator.platform.indexOf('Linux') > -1,
  
  getOS: function() {
    if (this.isWindows) return 'Windows';
    if (this.isMac) return 'macOS';
    if (this.isLinux) return 'Linux';
    return 'Unknown';
  },
  
  getPluginPath: function() {
    if (this.isWindows) {
      return 'C:\\Program Files\\ONLYOFFICE\\DesktopEditors\\editors\\sdkjs-plugins\\';
    } else if (this.isMac) {
      return '/Applications/ONLYOFFICE.app/Contents/Resources/editors/sdkjs-plugins/';
    } else if (this.isLinux) {
      return '/opt/onlyoffice/desktopeditors/editors/sdkjs-plugins/';
    }
    return null;
  }
};

// Use platform-specific code
window.Asc.plugin.init = function() {
  console.log('Running on:', Platform.getOS());
  
  if (Platform.isWindows) {
    initWindowsFeatures();
  } else if (Platform.isMac) {
    initMacFeatures();
  } else if (Platform.isLinux) {
    initLinuxFeatures();
  }
};
```

### Platform-specific UI considerations

**Error name:** Ignoring platform UI conventions

:::warning[Wrong]
```css
/* Using same styles for all platforms */
.button {
  border-radius: 4px;
  padding: 8px 16px;
}
```
:::

:::tip[Correct]
```css
/* Platform-specific styles */
.button {
  padding: 8px 16px;
}

/* Windows - Squared corners */
.platform-windows .button {
  border-radius: 2px;
}

/* macOS - Rounded corners */
.platform-mac .button {
  border-radius: 6px;
}

/* Linux - Depends on DE */
.platform-linux .button {
  border-radius: 3px;
}
```

```javascript
// Apply platform class to body
document.body.classList.add(`platform-${Platform.getOS().toLowerCase()}`);
```
:::

Error output: UI looks out of place on different operating systems.

## Performance optimization for desktop

### Leveraging native performance

```javascript
// Use Web Workers for heavy computations
class WorkerPool {
  constructor(workerScript, poolSize = 4) {
    this.workers = [];
    this.queue = [];
    this.poolSize = poolSize;
    
    for (let i = 0; i < poolSize; i++) {
      const worker = new Worker(workerScript);
      worker.onmessage = this.handleMessage.bind(this);
      this.workers.push({ worker, busy: false });
    }
  }
  
  execute(data) {
    return new Promise((resolve, reject) => {
      const availableWorker = this.workers.find(w => !w.busy);
      
      if (availableWorker) {
        availableWorker.busy = true;
        availableWorker.resolve = resolve;
        availableWorker.reject = reject;
        availableWorker.worker.postMessage(data);
      } else {
        this.queue.push({ data, resolve, reject });
      }
    });
  }
  
  handleMessage(event) {
    const workerIndex = this.workers.findIndex(
      w => w.worker === event.target
    );
    
    if (workerIndex !== -1) {
      const worker = this.workers[workerIndex];
      worker.busy = false;
      
      if (worker.resolve) {
        worker.resolve(event.data);
        worker.resolve = null;
      }
      
      // Process queue
      if (this.queue.length > 0) {
        const task = this.queue.shift();
        this.execute(task.data).then(task.resolve).catch(task.reject);
      }
    }
  }
  
  terminate() {
    this.workers.forEach(w => w.worker.terminate());
    this.workers = [];
  }
}

// Usage
const pool = new WorkerPool('worker.js');

async function processLargeData(data) {
  try {
    const result = await pool.execute({ action: 'process', data });
    return result;
  } catch (error) {
    console.error('Processing failed:', error);
  }
}
```

### Memory management

**Error name:** Memory leaks in long-running desktop app

:::warning[Wrong]
```javascript
// Creating event listeners without cleanup
window.Asc.plugin.init = function() {
  document.addEventListener('click', handleClick);
  setInterval(updateData, 1000);
  
  // No cleanup - memory leak!
};
```
:::

:::tip[Correct]
```javascript
// Proper cleanup
let updateInterval = null;
let clickHandler = null;

window.Asc.plugin.init = function() {
  clickHandler = handleClick.bind(this);
  document.addEventListener('click', clickHandler);
  
  updateInterval = setInterval(updateData, 1000);
};

window.Asc.plugin.button = function(id) {
  if (id === -1) { // Cancel/Close
    // Clean up resources
    if (updateInterval) {
      clearInterval(updateInterval);
      updateInterval = null;
    }
    
    if (clickHandler) {
      document.removeEventListener('click', clickHandler);
      clickHandler = null;
    }
    
    // Close plugin
    window.Asc.plugin.executeCommand("close", "");
  }
};
```
:::

Error output: Desktop app memory usage grows over time, eventually slowing down or crashing.

## Desktop-specific features

### Keyboard shortcuts

```javascript
// Global keyboard shortcut handler
class KeyboardShortcuts {
  constructor() {
    this.shortcuts = new Map();
    document.addEventListener('keydown', this.handleKeyPress.bind(this));
  }
  
  register(keys, handler, description) {
    const normalizedKeys = this.normalizeKeys(keys);
    this.shortcuts.set(normalizedKeys, { handler, description });
  }
  
  normalizeKeys(keys) {
    return keys.toLowerCase()
      .split('+')
      .map(k => k.trim())
      .sort()
      .join('+');
  }
  
  getKeyCombo(event) {
    const keys = [];
    
    if (event.ctrlKey || event.metaKey) keys.push('ctrl');
    if (event.altKey) keys.push('alt');
    if (event.shiftKey) keys.push('shift');
    
    if (event.key && event.key !== 'Control' && 
        event.key !== 'Alt' && event.key !== 'Shift' && 
        event.key !== 'Meta') {
      keys.push(event.key.toLowerCase());
    }
    
    return keys.sort().join('+');
  }
  
  handleKeyPress(event) {
    const combo = this.getKeyCombo(event);
    const shortcut = this.shortcuts.get(combo);
    
    if (shortcut) {
      event.preventDefault();
      shortcut.handler(event);
    }
  }
  
  unregister(keys) {
    const normalizedKeys = this.normalizeKeys(keys);
    this.shortcuts.delete(normalizedKeys);
  }
}

// Usage
const shortcuts = new KeyboardShortcuts();

window.Asc.plugin.init = function() {
  shortcuts.register('Ctrl+Shift+H', () => {
    highlightSelection();
  }, 'Highlight selected text');
  
  shortcuts.register('Ctrl+Alt+C', () => {
    clearFormatting();
  }, 'Clear formatting');
};
```

### System tray integration (if supported)

```javascript
// Check if system tray features are available
function checkSystemFeatures() {
  return {
    hasTray: typeof window.Asc.plugin.tray !== 'undefined',
    hasNotifications: 'Notification' in window,
    hasClipboard: navigator.clipboard !== undefined
  };
}

// Request notification permission
async function enableNotifications() {
  if ('Notification' in window) {
    const permission = await Notification.requestPermission();
    return permission === 'granted';
  }
  return false;
}

// Show desktop notification
function showNotification(title, body, options = {}) {
  if (Notification.permission === 'granted') {
    return new Notification(title, {
      body,
      icon: 'assets/icons/icon.png',
      ...options
    });
  }
}

// Usage
window.Asc.plugin.init = async function() {
  const features = checkSystemFeatures();
  console.log('System features:', features);
  
  if (features.hasNotifications) {
    const granted = await enableNotifications();
    if (granted) {
      showNotification('Plugin Ready', 'Your plugin is now active');
    }
  }
};
```

## Debugging desktop plugins

### Enable developer tools

**Windows/Linux:**
```
Press: Ctrl+Shift+Alt+F12
```

**macOS:**
```
Press: Cmd+Option+Shift+F12
```

### Console logging best practices

```javascript
// Desktop-specific logging
const DesktopLogger = {
  enabled: true,
  logFile: [],
  maxLogs: 1000,
  
  log: function(level, message, data) {
    if (!this.enabled) return;
    
    const entry = {
      timestamp: new Date().toISOString(),
      level,
      message,
      data,
      platform: Platform.getOS()
    };
    
    // Console output
    console[level](message, data || '');
    
    // Store in memory
    this.logFile.push(entry);
    
    // Keep only recent logs
    if (this.logFile.length > this.maxLogs) {
      this.logFile.shift();
    }
    
    // Persist to localStorage
    this.saveLogs();
  },
  
  info: function(message, data) {
    this.log('log', message, data);
  },
  
  warn: function(message, data) {
    this.log('warn', message, data);
  },
  
  error: function(message, data) {
    this.log('error', message, data);
  },
  
  saveLogs: function() {
    try {
      localStorage.setItem('plugin_logs', JSON.stringify(this.logFile));
    } catch (e) {
      console.error('Failed to save logs:', e);
    }
  },
  
  getLogs: function() {
    return this.logFile;
  },
  
  exportLogs: function() {
    const blob = new Blob([JSON.stringify(this.logFile, null, 2)], {
      type: 'application/json'
    });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `plugin-logs-${Date.now()}.json`;
    a.click();
    URL.revokeObjectURL(url);
  },
  
  clearLogs: function() {
    this.logFile = [];
    localStorage.removeItem('plugin_logs');
  }
};

// Usage
window.Asc.plugin.init = function() {
  DesktopLogger.info('Plugin initialized', {
    version: '1.0.0',
    platform: Platform.getOS()
  });
};
```

## Testing desktop plugins

### Multi-platform testing checklist

```markdown
## Desktop Testing Checklist

### Windows
- [ ] Windows 10
- [ ] Windows 11
- [ ] Different display scales (100%, 125%, 150%)
- [ ] Light and dark themes

### macOS
- [ ] macOS 11 (Big Sur)
- [ ] macOS 12 (Monterey)
- [ ] macOS 13 (Ventura)
- [ ] Retina and non-Retina displays

### Linux
- [ ] Ubuntu 20.04/22.04
- [ ] Debian 11
- [ ] Fedora latest
- [ ] Different desktop environments (GNOME, KDE, XFCE)

### All Platforms
- [ ] Plugin loads correctly
- [ ] UI scales properly
- [ ] Keyboard shortcuts work
- [ ] File operations work
- [ ] Memory usage stable
- [ ] No crashes after extended use
```

### Automated testing setup

```javascript
// Simple test framework for desktop plugins
class PluginTester {
  constructor() {
    this.tests = [];
    this.results = [];
  }
  
  test(name, fn) {
    this.tests.push({ name, fn });
  }
  
  async runAll() {
    console.log('Running tests...');
    
    for (const test of this.tests) {
      try {
        await test.fn();
        this.results.push({
          name: test.name,
          status: 'PASS'
        });
        console.log(`✓ ${test.name}`);
      } catch (error) {
        this.results.push({
          name: test.name,
          status: 'FAIL',
          error: error.message
        });
        console.error(`✗ ${test.name}:`, error.message);
      }
    }
    
    return this.results;
  }
  
  getReport() {
    const passed = this.results.filter(r => r.status === 'PASS').length;
    const failed = this.results.filter(r => r.status === 'FAIL').length;
    
    return {
      total: this.results.length,
      passed,
      failed,
      results: this.results
    };
  }
}

// Usage
const tester = new PluginTester();

tester.test('Plugin initializes', () => {
  if (!window.Asc || !window.Asc.plugin) {
    throw new Error('Plugin API not available');
  }
});

tester.test('Platform detected correctly', () => {
  const os = Platform.getOS();
  if (os === 'Unknown') {
    throw new Error('Could not detect platform');
  }
});

tester.test('Local storage works', () => {
  const key = 'test_key';
  const value = 'test_value';
  
  localStorage.setItem(key, value);
  const retrieved = localStorage.getItem(key);
  localStorage.removeItem(key);
  
  if (retrieved !== value) {
    throw new Error('Local storage not working');
  }
});

// Run tests
window.Asc.plugin.init = async function() {
  await tester.runAll();
  const report = tester.getReport();
  console.log('Test Report:', report);
};
```

## Deployment and updates

### Version management

```javascript
// Version checking and update notification
const VersionManager = {
  current: '1.0.0',
  updateCheckURL: 'https://api.example.com/plugin/version',
  
  parseVersion: function(version) {
    const parts = version.split('.').map(Number);
    return {
      major: parts[0] || 0,
      minor: parts[1] || 0,
      patch: parts[2] || 0
    };
  },
  
  compareVersions: function(v1, v2) {
    const ver1 = this.parseVersion(v1);
    const ver2 = this.parseVersion(v2);
    
    if (ver1.major !== ver2.major) return ver1.major - ver2.major;
    if (ver1.minor !== ver2.minor) return ver1.minor - ver2.minor;
    return ver1.patch - ver2.patch;
  },
  
  checkForUpdates: async function() {
    try {
      const response = await fetch(this.updateCheckURL);
      const data = await response.json();
      
      if (this.compareVersions(data.latest, this.current) > 0) {
        return {
          updateAvailable: true,
          latest: data.latest,
          downloadURL: data.downloadURL,
          releaseNotes: data.releaseNotes
        };
      }
      
      return { updateAvailable: false };
    } catch (error) {
      console.error('Update check failed:', error);
      return { updateAvailable: false, error: error.message };
    }
  },
  
  notifyUpdate: function(updateInfo) {
    if (Notification.permission === 'granted') {
      const notification = new Notification('Update Available', {
        body: `Version ${updateInfo.latest} is now available`,
        icon: 'assets/icons/icon.png'
      });
      
      notification.onclick = () => {
        window.open(updateInfo.downloadURL);
      };
    }
  }
};

// Check for updates on init
window.Asc.plugin.init = async function() {
  const updateInfo = await VersionManager.checkForUpdates();
  
  if (updateInfo.updateAvailable) {
    VersionManager.notifyUpdate(updateInfo);
  }
};
```

## Best practices for desktop

### Optimize for offline use

```javascript
// Cache external resources for offline use
class ResourceCache {
  constructor() {
    this.cache = new Map();
    this.loadCache();
  }
  
  async fetch(url) {
    // Check cache first
    if (this.cache.has(url)) {
      return this.cache.get(url);
    }
    
    // Fetch from network
    try {
      const response = await fetch(url);
      const data = await response.text();
      
      // Save to cache
      this.cache.set(url, data);
      this.saveCache();
      
      return data;
    } catch (error) {
      console.error('Failed to fetch:', url, error);
      throw error;
    }
  }
  
  saveCache() {
    const cacheData = Array.from(this.cache.entries());
    localStorage.setItem('resource_cache', JSON.stringify(cacheData));
  }
  
  loadCache() {
    try {
      const cached = localStorage.getItem('resource_cache');
      if (cached) {
        const entries = JSON.parse(cached);
        this.cache = new Map(entries);
      }
    } catch (error) {
      console.error('Failed to load cache:', error);
    }
  }
}
```

## Conclusion

Developing plugins for ONLYOFFICE Desktop Editors offers powerful capabilities and direct system access. By following platform-specific best practices, optimizing for performance, and thorough testing across operating systems, you can create professional desktop plugins that enhance the ONLYOFFICE experience.
