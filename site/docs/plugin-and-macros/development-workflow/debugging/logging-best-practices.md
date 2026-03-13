---
sidebar_position: -3
---

# Logging best practices

## Overview

Effective logging is crucial for debugging and monitoring ONLYOFFICE plugins. This guide covers best practices for implementing strategic, useful logging that helps you quickly identify and resolve issues.

## Logging principles

### Log levels

Use appropriate log levels to categorize messages:

```javascript
const LogLevel = {
  ERROR: 'error',    // Critical issues
  WARN: 'warn',      // Warning conditions
  INFO: 'info',      // Informational messages
  DEBUG: 'debug',    // Debug information
  TRACE: 'trace'     // Detailed trace
};

class Logger {
  constructor(context) {
    this.context = context;
    this.enabled = true;
  }
  
  error(message, data) {
    console.error(`[${this.context}] ERROR:`, message, data || '');
  }
  
  warn(message, data) {
    console.warn(`[${this.context}] WARN:`, message, data || '');
  }
  
  info(message, data) {
    console.log(`[${this.context}] INFO:`, message, data || '');
  }
  
  debug(message, data) {
    if (this.isDebugMode()) {
      console.log(`[${this.context}] DEBUG:`, message, data || '');
    }
  }
  
  isDebugMode() {
    return window.location.hostname === 'localhost' ||
           localStorage.getItem('debug') === 'true';
  }
}

// Usage
const logger = new Logger('MyPlugin');
logger.info('Plugin initialized');
logger.debug('Config loaded', configData);
logger.error('Failed to save', error);
```

### When to log

**DO log:**
- Plugin initialization
- API calls and responses
- User actions
- Error conditions
- State changes
- Performance metrics

**DON'T log:**
- Sensitive data (passwords, tokens)
- Excessive loop iterations
- Normal operations in production
- Complete large objects

## Structured logging

### Format for readability

**Error name:** Unstructured console spam

:::warning[Wrong]
```javascript
// Messy, unstructured logs
console.log('data');
console.log(data);
console.log('processing');
console.log('done');
```
:::

:::tip[Correct]
```javascript
// Structured, meaningful logs
console.group('Data Processing');
console.log('Input:', data);
console.time('processing-time');

// Process data
processData(data);

console.timeEnd('processing-time');
console.log('Result:', result);
console.groupEnd();
```
:::

Error output: Console cluttered, hard to follow execution flow.

### Log context and timestamps

```javascript
class ContextLogger {
  log(level, message, data) {
    const entry = {
      timestamp: new Date().toISOString(),
      level,
      message,
      data,
      context: {
        plugin: 'MyPlugin',
        version: '1.0.0',
        user: this.getUserId(),
        session: this.getSessionId()
      }
    };
    
    console[level](entry);
    
    // Optionally save to storage
    this.saveLog(entry);
  }
  
  saveLog(entry) {
    const logs = JSON.parse(localStorage.getItem('logs') || '[]');
    logs.push(entry);
    
    // Keep last 100 logs
    if (logs.length > 100) {
      logs.shift();
    }
    
    localStorage.setItem('logs', JSON.stringify(logs));
  }
  
  getUserId() {
    return 'user-' + Math.random().toString(36).substr(2, 9);
  }
  
  getSessionId() {
    let sessionId = sessionStorage.getItem('sessionId');
    if (!sessionId) {
      sessionId = 'session-' + Date.now();
      sessionStorage.setItem('sessionId', sessionId);
    }
    return sessionId;
  }
}
```

## API logging

### Log API interactions

```javascript
function apiLogger(method, params, result, error) {
  const log = {
    method,
    params,
    result: result !== undefined ? result : null,
    error: error ? error.message : null,
    timestamp: new Date().toISOString()
  };
  
  if (error) {
    console.error('API Error:', log);
  } else {
    console.log('API Call:', log);
  }
  
  return log;
}

// Usage with ONLYOFFICE API
window.Asc.plugin.executeMethod("GetSelectedText", [], function(text) {
  apiLogger('GetSelectedText', [], text, null);
  
  if (text) {
    processText(text);
  }
});

// With error handling
window.Asc.plugin.callCommand(function() {
  try {
    const doc = Api.GetDocument();
    const result = doc.GetAllParagraphs();
    apiLogger('GetAllParagraphs', [], result.length, null);
    return result;
  } catch (error) {
    apiLogger('GetAllParagraphs', [], null, error);
    throw error;
  }
}, false);
```

## Performance logging

### Track execution time

```javascript
class PerformanceLogger {
  constructor() {
    this.timers = new Map();
  }
  
  start(label) {
    this.timers.set(label, performance.now());
  }
  
  end(label) {
    const startTime = this.timers.get(label);
    
    if (!startTime) {
      console.warn(`Timer "${label}" not found`);
      return;
    }
    
    const duration = performance.now() - startTime;
    this.timers.delete(label);
    
    console.log(`[PERF] ${label}: ${duration.toFixed(2)}ms`);
    
    if (duration > 1000) {
      console.warn(`[PERF] Slow operation: ${label}`);
    }
    
    return duration;
  }
  
  measure(fn, label) {
    this.start(label);
    const result = fn();
    this.end(label);
    return result;
  }
  
  async measureAsync(fn, label) {
    this.start(label);
    const result = await fn();
    this.end(label);
    return result;
  }
}

// Usage
const perf = new PerformanceLogger();

window.Asc.plugin.init = function() {
  perf.start('initialization');
  
  setupUI();
  loadSettings();
  attachEvents();
  
  perf.end('initialization');
};

// Measure function execution
const result = perf.measure(() => {
  return processLargeData(data);
}, 'data-processing');
```

## Error logging

### Comprehensive error tracking

```javascript
class ErrorLogger {
  constructor() {
    this.errors = [];
    this.setupGlobalHandlers();
  }
  
  setupGlobalHandlers() {
    window.addEventListener('error', (event) => {
      this.logError({
        type: 'runtime',
        message: event.message,
        filename: event.filename,
        line: event.lineno,
        column: event.colno,
        stack: event.error ? event.error.stack : null
      });
    });
    
    window.addEventListener('unhandledrejection', (event) => {
      this.logError({
        type: 'promise',
        message: event.reason.message || event.reason,
        stack: event.reason.stack || null
      });
    });
  }
  
  logError(error) {
    const errorEntry = {
      ...error,
      timestamp: new Date().toISOString(),
      userAgent: navigator.userAgent,
      url: window.location.href
    };
    
    this.errors.push(errorEntry);
    console.error('Error logged:', errorEntry);
    
    // Save to localStorage
    this.persistErrors();
    
    // Optionally send to server
    this.reportError(errorEntry);
  }
  
  persistErrors() {
    try {
      const errors = this.errors.slice(-50); // Keep last 50
      localStorage.setItem('plugin_errors', JSON.stringify(errors));
    } catch (e) {
      console.error('Failed to persist errors');
    }
  }
  
  reportError(error) {
    // Send to error tracking service
    if (window.navigator.onLine) {
      fetch('/api/log-error', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(error)
      }).catch(() => {
        // Silently fail
      });
    }
  }
  
  getErrors() {
    return this.errors;
  }
  
  clearErrors() {
    this.errors = [];
    localStorage.removeItem('plugin_errors');
  }
}

// Initialize globally
const errorLogger = new ErrorLogger();
```

## Environment-based logging

### Development vs production

**Error name:** Debug logs in production

:::warning[Wrong]
```javascript
// Always logging everything
function processData(data) {
  console.log('Data:', data);
  console.log('Processing...');
  console.log('Step 1');
  console.log('Step 2');
  console.log('Complete');
}
```
:::

:::tip[Correct]
```javascript
// Environment-aware logging
const ENV = {
  isDevelopment: window.location.hostname === 'localhost',
  isProduction: window.location.hostname !== 'localhost'
};

function processData(data) {
  if (ENV.isDevelopment) {
    console.log('Data:', data);
    console.log('Processing...');
  }
  
  // Process data
  
  if (ENV.isDevelopment) {
    console.log('Complete');
  }
}

// Or use logger class
class EnvLogger {
  log(...args) {
    if (ENV.isDevelopment) {
      console.log(...args);
    }
  }
  
  // Always log errors
  error(...args) {
    console.error(...args);
  }
}
```
:::

Error output: Production console cluttered with debug logs.

## Advanced logging patterns

### Conditional logging

```javascript
// Log only specific conditions
function logIf(condition, message, data) {
  if (condition) {
    console.log(message, data);
  }
}

// Usage
logIf(items.length > 100, 'Large dataset:', items.length);
logIf(response.status !== 200, 'Unexpected status:', response.status);
```

### Rate-limited logging

```javascript
// Prevent log spam
class RateLimitedLogger {
  constructor(limit = 10, windowMs = 1000) {
    this.limit = limit;
    this.windowMs = windowMs;
    this.counts = new Map();
  }
  
  log(key, message, data) {
    const now = Date.now();
    const record = this.counts.get(key) || { count: 0, resetAt: now + this.windowMs };
    
    if (now > record.resetAt) {
      record.count = 0;
      record.resetAt = now + this.windowMs;
    }
    
    if (record.count < this.limit) {
      console.log(message, data);
      record.count++;
      this.counts.set(key, record);
    } else if (record.count === this.limit) {
      console.warn(`Log rate limit reached for: ${key}`);
      record.count++;
      this.counts.set(key, record);
    }
  }
}

// Usage
const rateLimited = new RateLimitedLogger(5, 1000);

// This will only log 5 times per second
setInterval(() => {
  rateLimited.log('loop', 'Loop iteration');
}, 10);
```

### Circular reference handling

```javascript
// Safely log objects with circular references
function safeLog(obj) {
  const seen = new WeakSet();
  
  const stringify = (value) => {
    if (value === null) return 'null';
    if (typeof value !== 'object') return JSON.stringify(value);
    
    if (seen.has(value)) return '"[Circular]"';
    
    seen.add(value);
    
    if (Array.isArray(value)) {
      return '[' + value.map(stringify).join(',') + ']';
    }
    
    const props = Object.keys(value)
      .map(key => `"${key}":${stringify(value[key])}`)
      .join(',');
      
    return '{' + props + '}';
  };
  
  try {
    console.log(JSON.parse(stringify(obj)));
  } catch (error) {
    console.log('[Unable to stringify object]');
  }
}
```

## Log aggregation

### Collecting logs for export

```javascript
class LogCollector {
  constructor() {
    this.logs = [];
    this.maxLogs = 1000;
  }
  
  add(level, message, data) {
    this.logs.push({
      timestamp: new Date().toISOString(),
      level,
      message,
      data
    });
    
    if (this.logs.length > this.maxLogs) {
      this.logs.shift();
    }
  }
  
  export() {
    const blob = new Blob([JSON.stringify(this.logs, null, 2)], {
      type: 'application/json'
    });
    
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `plugin-logs-${Date.now()}.json`;
    a.click();
    URL.revokeObjectURL(url);
  }
  
  clear() {
    this.logs = [];
  }
}

// Usage
const collector = new LogCollector();

// Add to your logger
console.log = new Proxy(console.log, {
  apply(target, thisArg, args) {
    collector.add('log', args[0], args.slice(1));
    return target.apply(thisArg, args);
  }
});
```

## Best practices summary
 
### Structured logging
 
**Error name:** Unstructured log messages
 
:::warning[Wrong]
```javascript
// Unstructured, unclear logs
console.log('click');
console.log(userId);
```
:::
 
:::tip[Correct]
```javascript
// Use structured logging with context
console.group('User Action');
console.log('Action:', 'button-click');
console.log('User:', userId);
console.groupEnd();
 
// Log with clear context
console.log('[Plugin] [Init] Starting initialization');
```
:::
 
Error output: Console cluttered, hard to trace events.
 
### Appropriate log levels
 
**Error name:** Using wrong log level
 
:::warning[Wrong]
```javascript
// Everything as console.log
console.log('Error occurred:', error);
console.log('Warning:', warning);
console.log('Info:', info);
```
:::
 
:::tip[Correct]
```javascript
// Use appropriate log levels
console.error('Critical error:', error);
console.warn('Warning:', warning);
console.log('Info:', info);
```
:::
 
Error output: Unable to filter by severity, critical issues missed.
 
### Sensitive data protection
 
**Error name:** Logging sensitive information
 
:::warning[Wrong]
```javascript
// Logging passwords and secrets
console.log('Password:', password);
console.log('API Key:', apiKey);
console.log('User data:', {name, email, creditCard});
```
:::
 
:::tip[Correct]
```javascript
// Never log sensitive data
console.log('Login attempt for user:', username);
console.log('API call authenticated');
console.log('User data:', {name, email}); // Exclude sensitive fields
```
:::
 
Error output: Security breach, exposed credentials in logs.
 
### Console spam prevention
 
**Error name:** Excessive logging in loops
 
:::warning[Wrong]
```javascript
// Logging every iteration
for (let i = 0; i < 1000; i++) {
  console.log('iteration:', i);
}
 
// Logging huge objects
console.log(hugeObject);
```
:::
 
:::tip[Correct]
```javascript
// Log summary instead
console.log('Processing 1000 items...');
processItems();
console.log('Processing complete');
 
// Log object size, not content
console.log(`Object size: ${Object.keys(hugeObject).length} properties`);
```
:::
 
Error output: Browser freezes, console becomes unusable.
 
### Production code cleanup
 
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
const DEBUG_MODE = window.location.hostname === 'localhost';
 
if (DEBUG_MODE) {
  console.log('Debug: Processing data');
  console.table(debugData);
}
 
// Remove all debugger statements
// debugger; ← Never commit this
```
:::
 
Error output: Production plugin slow, console cluttered, potential security issues.

## Conclusion

Effective logging is an art that balances informativeness with readability. Implement structured, environment-aware logging with appropriate levels, and your debugging process will become significantly more efficient and productive.
