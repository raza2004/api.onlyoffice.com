# Test environment setup

## Overview

A proper test environment is essential for efficient plugin development. This guide covers setting up a comprehensive testing environment that includes tools, workflows, and best practices for testing ONLYOFFICE plugins.

## Test environment components

A complete test environment includes:

1. **Development tools** - Code editors, version control
2. **Testing framework** - Automated and manual testing tools
3. **Debugging tools** - Browser DevTools, logging systems
4. **Documentation** - Test plans, test cases
5. **Sample data** - Test documents, test inputs

## Setting up development tools

### Code editor configuration

**VS Code (recommended):**

Install extensions:
```bash
# Via VS Code extensions marketplace
- ESLint
- Prettier
- Live Server
- GitLens
- Markdown All in One
```

**Workspace settings (.vscode/settings.json):**
```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "javascript.validate.enable": true,
  "files.associations": {
    "*.json": "jsonc"
  },
  "liveServer.settings.port": 5500
}
```

### Version control setup

**Initialize Git repository:**
```bash
# Navigate to your plugin directory
cd ~/onlyoffice-plugins/my-plugin

# Initialize Git
git init

# Create .gitignore
cat > .gitignore << EOL
node_modules/
.DS_Store
Thumbs.db
*.log
.vscode/
.idea/
dist/
*.zip
EOL

# Initial commit
git add .
git commit -m "Initial commit: Plugin structure"
```

### Package manager setup

**Initialize npm (optional but recommended):**
```bash
# Create package.json
npm init -y

# Install development dependencies
npm install --save-dev eslint prettier

# Create npm scripts
```

**package.json:**
```json
{
  "name": "my-onlyoffice-plugin",
  "version": "1.0.0",
  "description": "My ONLYOFFICE plugin",
  "scripts": {
    "test": "echo \"No tests yet\"",
    "lint": "eslint *.js",
    "format": "prettier --write **/*.{js,json,html,css}",
    "validate": "node validate-config.js"
  },
  "devDependencies": {
    "eslint": "^8.0.0",
    "prettier": "^2.0.0"
  }
}
```

## Creating test documents

### Document types

Create test documents for different scenarios:

**Basic test documents:**
```
test-docs/
├── empty.docx          # Empty document
├── simple-text.docx    # Plain text only
├── formatted.docx      # Bold, italic, colors
├── tables.docx         # Multiple tables
├── images.docx         # Images and shapes
├── large.docx          # 50+ pages
├── spreadsheet.xlsx    # Sample spreadsheet
└── presentation.pptx   # Sample slides
```

### Sample data for testing

**test-data.json:**
```json
{
  "sampleTexts": [
    "The quick brown fox jumps over the lazy dog",
    "Lorem ipsum dolor sit amet",
    "Test with special characters: @#$%^&*()",
    "Test with unicode: 你好世界 مرحبا العالم"
  ],
  "sampleUrls": [
    "https://www.example.com",
    "https://www.test.org/page?id=123"
  ],
  "sampleData": {
    "users": [
      {"id": 1, "name": "Alice"},
      {"id": 2, "name": "Bob"}
    ]
  }
}
```

## Setting up testing tools

### Manual testing checklist

**test-checklist.md:**
```markdown
# Plugin Testing Checklist

## Basic Functionality
- [ ] Plugin appears in Plugins menu
- [ ] Plugin opens without errors
- [ ] UI displays correctly
- [ ] All buttons are functional
- [ ] Plugin closes properly

## Core Features
- [ ] Feature 1 works as expected
- [ ] Feature 2 works as expected
- [ ] Error handling works
- [ ] Success messages display

## Browser Testing
- [ ] Chrome (latest)
- [ ] Firefox (latest)
- [ ] Safari (latest)
- [ ] Edge (latest)

## Editor Testing
- [ ] Document Editor
- [ ] Spreadsheet Editor
- [ ] Presentation Editor

## Edge Cases
- [ ] Empty document
- [ ] No selection
- [ ] Large document (50+ pages)
- [ ] Special characters
- [ ] Non-English text
```

### Automated testing setup

**Create test script (test.js):**
```javascript
// Simple test runner for plugin
const fs = require('fs');
const path = require('path');

class PluginTester {
  constructor(pluginPath) {
    this.pluginPath = pluginPath;
    this.results = {
      passed: 0,
      failed: 0,
      errors: []
    };
  }
  
  // Test 1: Config file exists and is valid JSON
  testConfigExists() {
    const configPath = path.join(this.pluginPath, 'config.json');
    
    try {
      if (!fs.existsSync(configPath)) {
        throw new Error('config.json not found');
      }
      
      const config = JSON.parse(fs.readFileSync(configPath, 'utf8'));
      
      if (!config.name || !config.guid || !config.version) {
        throw new Error('config.json missing required fields');
      }
      
      console.log('✓ config.json is valid');
      this.results.passed++;
    } catch (error) {
      console.error('✗ config.json test failed:', error.message);
      this.results.failed++;
      this.results.errors.push(error.message);
    }
  }
  
  // Test 2: Required files exist
  testRequiredFiles() {
    const requiredFiles = ['index.html', 'config.json'];
    
    requiredFiles.forEach(file => {
      const filePath = path.join(this.pluginPath, file);
      
      if (fs.existsSync(filePath)) {
        console.log(`✓ ${file} exists`);
        this.results.passed++;
      } else {
        console.error(`✗ ${file} missing`);
        this.results.failed++;
        this.results.errors.push(`Missing file: ${file}`);
      }
    });
  }
  
  // Test 3: HTML file structure
  testHtmlStructure() {
    const htmlPath = path.join(this.pluginPath, 'index.html');
    
    try {
      const html = fs.readFileSync(htmlPath, 'utf8');
      
      if (!html.includes('window.Asc.plugin')) {
        throw new Error('HTML missing window.Asc.plugin initialization');
      }
      
      console.log('✓ HTML structure is valid');
      this.results.passed++;
    } catch (error) {
      console.error('✗ HTML structure test failed:', error.message);
      this.results.failed++;
      this.results.errors.push(error.message);
    }
  }
  
  // Run all tests
  runAll() {
    console.log('\n=== Running Plugin Tests ===\n');
    
    this.testConfigExists();
    this.testRequiredFiles();
    this.testHtmlStructure();
    
    console.log('\n=== Test Results ===');
    console.log(`Passed: ${this.results.passed}`);
    console.log(`Failed: ${this.results.failed}`);
    
    if (this.results.errors.length > 0) {
      console.log('\nErrors:');
      this.results.errors.forEach(err => console.log(`  - ${err}`));
    }
    
    return this.results.failed === 0;
  }
}

// Run tests
const tester = new PluginTester('./');
const success = tester.runAll();

process.exit(success ? 0 : 1);
```

**Run tests:**
```bash
node test.js
```

## Debugging setup

### Enable browser DevTools

**In ONLYOFFICE Desktop Editors:**

Windows/Linux:
```
Press: Ctrl+Shift+Alt+F12
```

macOS:
```
Press: Cmd+Option+Shift+F12
```

### Console logging best practices

**logging.js:**
```javascript
// Debug logging utility
const DEBUG = true; // Set to false in production

const Logger = {
  info: function(message, data) {
    if (DEBUG) {
      console.log(`[INFO] ${message}`, data || '');
    }
  },
  
  warn: function(message, data) {
    if (DEBUG) {
      console.warn(`[WARN] ${message}`, data || '');
    }
  },
  
  error: function(message, error) {
    console.error(`[ERROR] ${message}`, error);
  },
  
  trace: function(message) {
    if (DEBUG) {
      console.trace(message);
    }
  }
};

// Usage in plugin
window.Asc.plugin.init = function(data) {
  Logger.info('Plugin initialized', data);
  
  try {
    // Plugin code
  } catch (error) {
    Logger.error('Initialization failed', error);
  }
};
```

### Error tracking

**error-handler.js:**
```javascript
// Global error handler
window.addEventListener('error', function(event) {
  const errorLog = {
    timestamp: new Date().toISOString(),
    message: event.message,
    filename: event.filename,
    line: event.lineno,
    column: event.colno,
    stack: event.error ? event.error.stack : 'No stack trace'
  };
  
  console.error('Caught error:', errorLog);
  
  // Save to local storage for debugging
  saveErrorLog(errorLog);
});

function saveErrorLog(errorLog) {
  try {
    const errors = JSON.parse(localStorage.getItem('pluginErrors') || '[]');
    errors.push(errorLog);
    
    // Keep last 50 errors
    if (errors.length > 50) {
      errors.shift();
    }
    
    localStorage.setItem('pluginErrors', JSON.stringify(errors));
  } catch (e) {
    console.error('Failed to save error log:', e);
  }
}

function getErrorLogs() {
  try {
    return JSON.parse(localStorage.getItem('pluginErrors') || '[]');
  } catch (e) {
    return [];
  }
}

function clearErrorLogs() {
  localStorage.removeItem('pluginErrors');
}
```

## Performance testing

### Performance monitoring

**performance.js:**
```javascript
class PerformanceMonitor {
  constructor() {
    this.metrics = [];
  }
  
  start(label) {
    performance.mark(`${label}-start`);
  }
  
  end(label) {
    performance.mark(`${label}-end`);
    performance.measure(label, `${label}-start`, `${label}-end`);
    
    const measure = performance.getEntriesByName(label)[0];
    const duration = measure.duration;
    
    this.metrics.push({
      label: label,
      duration: duration,
      timestamp: new Date().toISOString()
    });
    
    console.log(`${label}: ${duration.toFixed(2)}ms`);
    
    return duration;
  }
  
  getMetrics() {
    return this.metrics;
  }
  
  getSummary() {
    const summary = {};
    
    this.metrics.forEach(metric => {
      if (!summary[metric.label]) {
        summary[metric.label] = {
          count: 0,
          total: 0,
          avg: 0,
          min: Infinity,
          max: 0
        };
      }
      
      const s = summary[metric.label];
      s.count++;
      s.total += metric.duration;
      s.avg = s.total / s.count;
      s.min = Math.min(s.min, metric.duration);
      s.max = Math.max(s.max, metric.duration);
    });
    
    return summary;
  }
}

// Usage
const perfMon = new PerformanceMonitor();

window.Asc.plugin.init = function() {
  perfMon.start('initialization');
  
  // Plugin initialization code
  
  perfMon.end('initialization');
};
```

## Common testing issues

### Plugin doesn't reload after changes

**Error name:** Changes not reflected after modification

:::warning[Wrong]
```javascript
// Making changes without restarting ONLYOFFICE
// File: plugin.js
function oldFunction() {
  // Old code still running
}
```
:::

:::tip[Correct]
```bash
# After making changes:
# 1. Save all files
# 2. Completely quit ONLYOFFICE (not just close windows)
# 3. Relaunch ONLYOFFICE
# 4. Open document and test plugin

# Or use symbolic link for faster iteration
ln -s ~/dev/my-plugin /path/to/plugins/my-plugin
```
:::

Error output: Old version of plugin still running despite code changes.

### Cannot access DevTools

**Error name:** DevTools not appearing

:::warning[Wrong]
```
# Pressing wrong key combination
Pressing: Ctrl+Shift+I (this is for browser, not ONLYOFFICE)
```
:::

:::tip[Correct]
```
# Correct key combination for ONLYOFFICE
Windows/Linux: Ctrl+Shift+Alt+F12
macOS: Cmd+Option+Shift+F12

# If still not working:
1. Check ONLYOFFICE version (older versions may not support)
2. Try right-clicking on plugin and select "Inspect"
3. Update ONLYOFFICE to latest version
```
:::

Error output: Developer tools don't open.

### Test data not loading

**Error name:** Sample data fails to load in plugin

:::warning[Wrong]
```javascript
// Loading data with incorrect path
fetch('./test-data.json')
  .then(response => response.json())
  .then(data => {
    // Data not loading...
  });
```
:::

:::tip[Correct]
```javascript
// Use correct relative path from plugin root
fetch('test-data/samples.json')
  .then(response => {
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    return response.json();
  })
  .then(data => {
    console.log('Data loaded:', data);
  })
  .catch(error => {
    console.error('Failed to load test data:', error);
  });
```
:::

Error output: "Failed to fetch" or 404 error in console.

## Test documentation

### Creating test reports

**test-report-template.md:**
```markdown
# Test Report: [Plugin Name] v[Version]

**Date:** [Date]
**Tester:** [Name]
**Environment:** [ONLYOFFICE version, OS, Browser]

## Test Summary
- Total Tests: X
- Passed: Y
- Failed: Z
- Blocked: A

## Test Results

### Feature: [Feature Name]
**Test Case:** [Description]
**Status:** Pass/Fail
**Notes:** [Any observations]

### Bugs Found
1. **Bug Title**
   - Severity: High/Medium/Low
   - Steps to reproduce:
   - Expected result:
   - Actual result:
   - Screenshot: [if applicable]

## Recommendations
[Any suggestions for improvement]
```

## Next steps

- Learn about [Desktop Editors installation](./desktop-editors-installation.md)
- Set up your plugin development workflow
- Test your plugins thoroughly before deployment

## Additional resources

- **Testing best practices**: [https://testingjavascript.com](https://testingjavascript.com)
- **Browser DevTools**: [https://developers.google.com/web/tools/chrome-devtools](https://developers.google.com/web/tools/chrome-devtools)
- **ONLYOFFICE API**: [https://api.onlyoffice.com](https://api.onlyoffice.com)

## Conclusion

A well-configured test environment is crucial for efficient plugin development. By setting up proper tools, creating comprehensive test cases, and establishing good testing practices, you can ensure your ONLYOFFICE plugins are robust, performant, and user-friendly.