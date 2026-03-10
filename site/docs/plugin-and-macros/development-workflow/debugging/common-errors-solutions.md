---
sidebar_position: 2
---

# Common errors & solutions

## Overview

This guide covers the most common errors encountered during ONLYOFFICE plugin development and their solutions. Learn to quickly identify and fix issues to streamline your development process.

## Plugin initialization errors

### Plugin not appearing in menu

**Error name:** Plugin doesn't show in Plugins tab

**Symptoms:**
- Plugin installed but not visible
- No errors in console
- Config file present

:::warning[Common causes]
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

:::tip[Solutions]
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

**Verification steps:**
```bash
# 1. Validate JSON
cat config.json | python -m json.tool

# 2. Check file exists
ls index.html

# 3. Verify GUID format starts with "asc."

# 4. Restart ONLYOFFICE completely
```

### Plugin initializes but shows blank screen

**Error name:** White screen on plugin open

**Cause:** JavaScript errors preventing render

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

Error output: Check console for: "Uncaught TypeError: Cannot read property 'textContent' of null"

## API method errors

### executeMethod not working

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

Error output: Method executes but result is lost without callback.

### callCommand fails silently

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

Error output: Check console for API method errors or undefined returns.

## Configuration errors

### Icons not displaying

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

File structure:
```
my-plugin/
├── config.json
├── index.html
└── resources/
    ├── icon.png (48x48)
    └── icon@2x.png (96x96)
```
:::

Error output: Plugin shows generic icon, no error message.

### Modal/panel configuration issues

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

Error output: Plugin appears in unexpected mode or position.

## Event handling errors

### Events not firing

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

Error output: Event handler never executes, selection changes ignored.

### Button handler not responding

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

Error output: Clicking buttons has no effect.

## Data handling errors

### JSON parse errors

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

Error output: "SyntaxError: Unexpected token < in JSON at position 0"

### LocalStorage quota exceeded

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

Error output: "QuotaExceededError: Failed to execute 'setItem' on 'Storage'"

## Network errors

### CORS errors

**Error name:** Cross-origin request blocked

**Error in console:**
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

### Timeout errors

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

Error output: "TypeError: Failed to fetch" or "AbortError: The operation was aborted"

## UI/UX errors

### Elements not found

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

Error output: "Uncaught TypeError: Cannot read property 'addEventListener' of null"

## Conclusion

Understanding common errors and their solutions accelerates plugin development. Keep this guide handy as a reference when debugging issues, and remember to check the console first, validate your configuration, and handle errors gracefully.
