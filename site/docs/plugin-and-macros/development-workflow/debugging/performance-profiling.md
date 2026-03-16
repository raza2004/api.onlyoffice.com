---
sidebar_position: -4
---

# Performance profiling

## Overview

Performance profiling helps identify bottlenecks, optimize code, and ensure your ONLYOFFICE plugin runs smoothly. This guide covers tools and techniques for measuring and improving plugin performance.

## Why performance matters

### Performance impact

**Poor performance causes:**
- Slow plugin loading
- Unresponsive UI
- High memory usage
- Battery drain on laptops
- Poor user experience

**Good performance provides:**
- Fast, responsive interactions
- Smooth animations
- Efficient resource usage
- Better user satisfaction

## Performance measurement

### Using Performance API

```javascript
// Measure specific operations
performance.mark('operation-start');

// Perform operation
performHeavyTask();

performance.mark('operation-end');
performance.measure('operation', 'operation-start', 'operation-end');

// Get results
const measure = performance.getEntriesByName('operation')[0];
console.log(`Operation took: ${measure.duration}ms`);
```

### Performance observer

```javascript
// Monitor performance entries
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log(`${entry.name}: ${entry.duration}ms`);
    
    if (entry.duration > 100) {
      console.warn(`Slow operation detected: ${entry.name}`);
    }
  }
});

observer.observe({ entryTypes: ['measure', 'navigation'] });
```

## Browser profiling tools

### Chrome DevTools profiler

**Recording performance:**
```
1. Open DevTools (F12)
2. Go to Performance tab
3. Click Record (●)
4. Perform actions in plugin
5. Click Stop
6. Analyze timeline
```

**What to look for:**
- **Long tasks** - Yellow blocks over 50ms
- **Layout shifts** - Unexpected reflows
- **Script execution** - JavaScript bottlenecks
- **Rendering** - Paint and composite time

### Memory profiler

**Detecting memory leaks:**
```
1. Open DevTools → Memory tab
2. Take heap snapshot (Baseline)
3. Use plugin normally
4. Take another snapshot (After use)
5. Compare snapshots
6. Look for retained objects
```

**Common memory issues:**

**Error name:** Memory leak from event listeners

:::warning[Wrong]
```javascript
// Creating listeners without cleanup
setInterval(() => {
  const handler = () => console.log('click');
  document.addEventListener('click', handler);
  // handler is never removed - memory leak!
}, 1000);
```
:::

:::tip[Correct]
```javascript
// Proper cleanup
let handlers = [];

function addHandler() {
  const handler = () => console.log('click');
  document.addEventListener('click', handler);
  handlers.push(handler);
}

function cleanup() {
  handlers.forEach(handler => {
    document.removeEventListener('click', handler);
  });
  handlers = [];
}

window.Asc.plugin.button = function(id) {
  if (id === -1) {
    cleanup();
    window.Asc.plugin.executeCommand("close", "");
  }
};
```
:::

Error output: Memory grows continuously, eventually causing slowdown or crashes.

## Optimization techniques

### Minimize DOM operations

**Error name:** Excessive DOM manipulation

:::warning[Wrong]
```javascript
// Multiple DOM updates
for (let i = 0; i < 100; i++) {
  const div = document.createElement('div');
  div.textContent = `Item ${i}`;
  document.body.appendChild(div);  // Reflow on each append!
}
```
:::

:::tip[Correct]
```javascript
// Batch DOM updates
const fragment = document.createDocumentFragment();

for (let i = 0; i < 100; i++) {
  const div = document.createElement('div');
  div.textContent = `Item ${i}`;
  fragment.appendChild(div);
}

// Single reflow
document.body.appendChild(fragment);
```
:::

Error output: Slow rendering, janky animations.

### Debounce expensive operations

```javascript
// Debounce helper
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

// Usage
const expensiveSearch = debounce(function(query) {
  // Expensive API call or computation
  searchAPI(query);
}, 300);

// Attach to input
document.getElementById('search').addEventListener('input', (e) => {
  expensiveSearch(e.target.value);
});
```

### Use requestAnimationFrame

**Error name:** Choppy animations

:::warning[Wrong]
```javascript
// Using setInterval for animations
setInterval(() => {
  element.style.left = position + 'px';
  position += 5;
}, 16);  // Doesn't sync with display refresh
```
:::

:::tip[Correct]
```javascript
// Use requestAnimationFrame
function animate() {
  element.style.left = position + 'px';
  position += 5;
  
  if (position < 500) {
    requestAnimationFrame(animate);
  }
}

requestAnimationFrame(animate);
```
:::

Error output: Stuttery, inconsistent animation.

### Lazy loading

```javascript
// Lazy load heavy components
class LazyLoader {
  constructor() {
    this.loaded = new Set();
  }
  
  async load(componentName, loader) {
    if (this.loaded.has(componentName)) {
      return;
    }
    
    console.time(`load-${componentName}`);
    
    try {
      await loader();
      this.loaded.add(componentName);
      console.timeEnd(`load-${componentName}`);
    } catch (error) {
      console.error(`Failed to load ${componentName}:`, error);
    }
  }
}

// Usage
const lazy = new LazyLoader();

document.getElementById('advancedBtn').addEventListener('click', async () => {
  await lazy.load('chart-library', async () => {
    const script = document.createElement('script');
    script.src = 'https://cdn.jsdelivr.net/npm/chart.js';
    document.head.appendChild(script);
    
    await new Promise(resolve => {
      script.onload = resolve;
    });
  });
  
  // Now use Chart.js
  renderChart();
});
```

## Performance budgets

### Set performance targets

```javascript
class PerformanceBudget {
  constructor() {
    this.budgets = {
      initTime: 1000,      // 1 second
      apiCall: 500,        // 500ms
      render: 100,         // 100ms
      interaction: 50      // 50ms
    };
  }
  
  check(operation, duration) {
    const budget = this.budgets[operation];
    
    if (!budget) {
      console.warn(`No budget defined for: ${operation}`);
      return true;
    }
    
    if (duration > budget) {
      console.error(`Performance budget exceeded for ${operation}:`, {
        budget: budget,
        actual: duration,
        exceeded: duration - budget
      });
      return false;
    }
    
    console.log(`✓ Within budget for ${operation}: ${duration}ms / ${budget}ms`);
    return true;
  }
  
  measure(operation, fn) {
    const start = performance.now();
    const result = fn();
    const duration = performance.now() - start;
    
    this.check(operation, duration);
    
    return result;
  }
}

// Usage
const budget = new PerformanceBudget();

window.Asc.plugin.init = function() {
  budget.measure('initTime', () => {
    setupUI();
    loadData();
  });
};
```

## Automated performance testing

### Performance test suite

```javascript
class PerformanceTest {
  constructor() {
    this.tests = [];
    this.results = [];
  }
  
  add(name, fn, maxDuration) {
    this.tests.push({ name, fn, maxDuration });
  }
  
  async run() {
    console.log('Running performance tests...\n');
    
    for (const test of this.tests) {
      const start = performance.now();
      
      try {
        await test.fn();
        const duration = performance.now() - start;
        
        const passed = duration <= test.maxDuration;
        const status = passed ? '✓ PASS' : '✗ FAIL';
        
        this.results.push({
          name: test.name,
          duration,
          maxDuration: test.maxDuration,
          passed
        });
        
        console.log(`${status} ${test.name}: ${duration.toFixed(2)}ms (max: ${test.maxDuration}ms)`);
      } catch (error) {
        console.error(`✗ ERROR ${test.name}:`, error.message);
        this.results.push({
          name: test.name,
          error: error.message,
          passed: false
        });
      }
    }
    
    this.printSummary();
  }
  
  printSummary() {
    const passed = this.results.filter(r => r.passed).length;
    const failed = this.results.filter(r => !r.passed).length;
    
    console.log(`\n=== Summary ===`);
    console.log(`Total: ${this.results.length}`);
    console.log(`Passed: ${passed}`);
    console.log(`Failed: ${failed}`);
  }
}

// Usage
const perfTest = new PerformanceTest();

perfTest.add('Load data', async () => {
  await loadData();
}, 500);

perfTest.add('Render UI', () => {
  renderUI();
}, 100);

perfTest.add('Process selection', () => {
  processSelection('test text');
}, 50);

// Run tests
perfTest.run();
```

## Real-world optimization

### Optimize data structures

**Error name:** Wrong data structure causing slowness

:::warning[Wrong]
```javascript
// Using array for lookups - O(n) complexity
const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' },
  // ... 1000 more users
];

function findUser(id) {
  return users.find(u => u.id === id);  // Slow for large arrays
}
```
:::

:::tip[Correct]
```javascript
// Using Map for O(1) lookups
const usersMap = new Map();
users.forEach(user => usersMap.set(user.id, user));

function findUser(id) {
  return usersMap.get(id);  // Fast constant-time lookup
}
```
:::

Error output: Plugin becomes unresponsive with large datasets.

### Optimize loops

**Error name:** Recalculating array length on every iteration

:::warning[Wrong]
```javascript
for (let i = 0; i < array.length; i++) {
  // array.length recalculated every iteration
}
```
:::

:::tip[Correct]
```javascript
const len = array.length;
for (let i = 0; i < len; i++) {
  // Length cached
}

// Or use for...of
for (const item of array) {
  // Cleaner and just as fast
}
```
:::

Error output: Unnecessary repeated property access on large arrays slows loop performance.

### Virtual scrolling for large lists

```javascript
// Handle large lists efficiently
class VirtualList {
  constructor(container, items, itemHeight) {
    this.container = container;
    this.items = items;
    this.itemHeight = itemHeight;
    this.visibleCount = Math.ceil(container.clientHeight / itemHeight);
    this.startIndex = 0;
    
    this.render();
    
    container.addEventListener('scroll', () => {
      this.onScroll();
    });
  }
  
  render() {
    const endIndex = Math.min(
      this.startIndex + this.visibleCount + 5,
      this.items.length
    );
    
    this.container.innerHTML = '';
    this.container.style.height = this.items.length * this.itemHeight + 'px';
    
    for (let i = this.startIndex; i < endIndex; i++) {
      const item = document.createElement('div');
      item.style.position = 'absolute';
      item.style.top = i * this.itemHeight + 'px';
      item.style.height = this.itemHeight + 'px';
      item.textContent = this.items[i];
      this.container.appendChild(item);
    }
  }
  
  onScroll() {
    const newStartIndex = Math.floor(this.container.scrollTop / this.itemHeight);
    
    if (newStartIndex !== this.startIndex) {
      this.startIndex = newStartIndex;
      this.render();
    }
  }
}

// Render 10,000 items smoothly
const list = new VirtualList(
  document.getElementById('list'),
  Array.from({ length: 10000 }, (_, i) => `Item ${i}`),
  30
);
```

## Performance monitoring in production

### Real user monitoring

```javascript
class RealUserMonitoring {
  constructor() {
    this.metrics = [];
  }
  
  track(metric, value) {
    this.metrics.push({
      metric,
      value,
      timestamp: Date.now(),
      userAgent: navigator.userAgent
    });
    
    // Send to analytics
    this.send();
  }
  
  send() {
    if (this.metrics.length >= 10) {
      navigator.sendBeacon('/api/metrics', JSON.stringify(this.metrics));
      this.metrics = [];
    }
  }
  
  trackPageLoad() {
    window.addEventListener('load', () => {
      const perfData = performance.getEntriesByType('navigation')[0];
      
      this.track('page-load', perfData.loadEventEnd - perfData.fetchStart);
      this.track('dom-interactive', perfData.domInteractive - perfData.fetchStart);
    });
  }
}

const rum = new RealUserMonitoring();
rum.trackPageLoad();
```

## Best practices

### Performance checklist

```markdown
## Plugin Performance Checklist

### Load Time
- [ ] Plugin loads in under 2 seconds
- [ ] Initial render under 1 second
- [ ] No blocking scripts
- [ ] Assets optimized (minified, compressed)

### Runtime
- [ ] No memory leaks
- [ ] Smooth scrolling (60 FPS)
- [ ] Fast interactions (<100ms)
- [ ] Efficient event handlers

### Network
- [ ] Minimal API calls
- [ ] Requests cached when appropriate
- [ ] Failed requests handled gracefully
- [ ] No unnecessary polling

### Code
- [ ] Loops optimized
- [ ] DOM updates batched
- [ ] Heavy operations debounced
- [ ] Large lists virtualized
```

## Conclusion

Performance profiling and optimization are ongoing processes. Measure first, optimize strategically, and always test the impact of your changes. A fast, responsive plugin provides a superior user experience and reflects professional development quality.
