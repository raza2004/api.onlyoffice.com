---
sidebar_position: 1
---

# Browser DevTools guide

## Overview

Browser DevTools are essential for debugging ONLYOFFICE plugins. This guide covers how to access, use, and master debugging tools to quickly identify and fix issues in your plugins.

## Accessing DevTools

### In ONLYOFFICE Desktop Editors

**Windows/Linux:**
```
Press: Ctrl+Shift+Alt+F12
```

**macOS:**
```
Press: Cmd+Option+Shift+F12
```

**Alternative method:**
1. Right-click anywhere in the plugin
2. Select "Inspect Element" or "Inspect"

### In ONLYOFFICE web editors

**All browsers:**
```
Press F12 or Ctrl+Shift+I (Cmd+Option+I on Mac)
```

**Right-click method:**
1. Right-click on plugin area
2. Select "Inspect" or "Inspect Element"

## DevTools panels overview

### Console panel

The Console is your primary debugging tool for plugins.

**Common uses:**
- View error messages and warnings
- Log debugging information
- Execute JavaScript code
- Test API calls

**Basic console methods:**
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

### Sources panel

Debug JavaScript code line by line.

**Key features:**
- Set breakpoints
- Step through code
- Watch variables
- View call stack

**Setting breakpoints:**
```javascript
function processSelection(text) {
  // Click line number in Sources panel to set breakpoint here
  const words = text.split(' ');
  
  // Execution pauses at breakpoint
  const count = words.length;
  
  return count;
}
```

**Conditional breakpoints:**
```javascript
// Right-click line number → Add conditional breakpoint
function processItem(item) {
  // Break only when item.id === 5
  const result = transform(item);
  return result;
}
```

### Network panel

Monitor all network requests from your plugin.

**What to check:**
- API request/response
- Load times
- Failed requests
- Request headers

**Filtering requests:**
```
- XHR: API calls
- JS: JavaScript files
- CSS: Stylesheets
- Img: Images
```

### Elements panel

Inspect and modify HTML/CSS in real-time.

**Common tasks:**
- Inspect element structure
- Edit HTML live
- Modify CSS styles
- Debug layout issues

**Live CSS editing:**
```css
/* In Styles pane, edit CSS directly */
.button {
  background: #2196f3;  /* Change color */
  padding: 10px 20px;    /* Adjust spacing */
}
```

## Debugging techniques

### Using breakpoints

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

Error output: Console cluttered with logs, hard to track execution flow.

### Watch expressions

Monitor specific values as you step through code:

```javascript
// In DevTools Watch panel, add expressions:
items.length
currentItem.price
total
items[0].name
```

### Call stack analysis

Understand how your code was called:

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

## Common debugging scenarios

### Debugging API calls

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

### Debugging event handlers

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

Error output: Can't tell if event listener is attached or firing.

### Debugging ONLYOFFICE API calls

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

## Advanced debugging

### Using debugger statement

```javascript
function complexFunction(data) {
  // Code pauses here when DevTools is open
  debugger;
  
  const processed = processData(data);
  return processed;
}
```

### Debugging async code

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

### Performance profiling

**Record performance:**
```
1. Open Performance tab
2. Click Record (red circle)
3. Perform actions in plugin
4. Click Stop
5. Analyze timeline
```

**What to look for:**
- Long tasks (yellow blocks)
- Layout thrashing
- Memory spikes
- Script execution time

### Memory profiling

**Detect memory leaks:**
```
1. Open Memory tab
2. Take heap snapshot
3. Use plugin for a while
4. Take another snapshot
5. Compare snapshots
```

**Common leak sources:**
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

## DevTools tips and tricks

### Quick commands

**Console shortcuts:**
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

### Preserve log

Keep console logs when navigating:
```
Console → Settings (gear icon) → ✓ Preserve log
```

### Filter console messages

```
Console → Filter box:
- /error/i    (regex for errors)
- -warning    (exclude warnings)
- method:POST (filter by type)
```

### Live expressions

Monitor values in real-time:
```
Console → Create live expression
→ Enter: myVariable
→ Value updates automatically
```

## Best practices

### Strategic logging

```javascript
// Log important events
window.Asc.plugin.init = function(data) {
  console.log('🚀 Plugin initialized', data);
};

window.Asc.plugin.button = function(id) {
  console.log('🔘 Button clicked:', id);
};

// Log errors prominently
console.error('❌ Critical error:', error);

// Log warnings
console.warn('⚠️ Deprecated method used');
```

### Clean up debug code

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

Error output: Production plugin slow and console cluttered.

## Conclusion

Mastering Browser DevTools is essential for efficient plugin debugging. Use breakpoints strategically, monitor network requests, profile performance, and leverage advanced features to quickly identify and fix issues in your ONLYOFFICE plugins.
