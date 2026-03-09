# **Modules & Debugging in JavaScript**

## **Modules in JavaScript (40%)**

### **1. CommonJS (Node.js Modules)**
- **Format**: Synchronous, designed for server-side
- **Syntax**: `require()` / `module.exports`
- **Key Features**: 
  - Runtime loading
  - Caching of modules
  - Dynamic imports possible
  - Used in Node.js and bundlers like Webpack

### **2. ESM (ES Modules)**
- **Format**: Asynchronous, native browser support
- **Syntax**: `import` / `export`
- **Key Features**:
  - Static analysis (tree-shaking possible)
  - Top-level await support
  - Better for browsers (async loading)
  - Strict mode by default
  - Named and default exports

### **Comparison**:
- **CommonJS**: Dynamic, synchronous, Node.js standard
- **ESM**: Static, asynchronous, modern standard
- **Interop**: Can use both with tools, but mixing requires care

## **Debugging in JavaScript (60%)**

### **1. Using Browser DevTools**
- **Sources Tab**: Set breakpoints, step through code
- **Console**: Execute JS, view logs with `console.log()`, `.warn()`, `.error()`
- **Elements Tab**: Inspect and modify DOM/CSS
- **Network Tab**: Monitor HTTP requests, timing, status codes
- **Application Tab**: Check storage, service workers, manifest

### **2. Debugging Issues**
- **Console Methods**: `console.table()`, `console.dir()`, `console.trace()`
- **Breakpoints**: Line, conditional, DOM, event listener
- **Watch Expressions**: Monitor variables in real-time
- **Call Stack**: Trace function execution order
- **Scope Inspection**: View local/global/closure variables

### **3. Debugging Memory Leaks**
- **Memory Tab**: Take heap snapshots, compare snapshots
- **Performance Monitor**: Track memory usage over time
- **Allocation Timeline**: See object allocation over time
- **Detached DOM Trees**: Elements removed from DOM but still referenced
- **Common Leak Patterns**:
  - Forgotten timers/intervals
  - Event listeners not removed
  - Closure references
  - Global variables

### **4. Debugging Performance**
- **Performance Tab**: Record and analyze runtime performance
- **Lighthouse**: Audit for performance, accessibility, SEO
- **Coverage Tab**: Find unused JavaScript/CSS
- **Rendering Tools**: FPS meter, paint flashing, layout shifts
- **Bottleneck Identification**:
  - Long tasks (>50ms blocking main thread)
  - Excessive reflows/repaints
  - Memory-intensive operations
  - Network request waterfalls

## **Quick Reference Code Snippets**

```javascript
// MODULES
// CommonJS
const module = require('./module');
module.exports = { value: 42 };

// ESM
import { value } from './module.js';
export const value = 42;

// DEBUGGING
console.table(data); // Tabular data
console.trace(); // Call stack trace
console.time('label'); console.timeEnd('label'); // Timing

// Performance monitoring
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log(`${entry.name}: ${entry.duration}ms`);
  }
});
observer.observe({ entryTypes: ['longtask', 'measure'] });

// Memory leak detection pattern
const weakSet = new WeakSet(); // Doesn't prevent GC
let data = new Array(1000000).fill('data');
weakSet.add(data); // Can be collected if no other references
```

## **Best Practices**
1. **Use ESM for new projects** when possible
2. **Profile before optimizing** - measure first
3. **Regular memory checks** in long-running apps
4. **Use production builds** for performance testing
5. **Monitor Core Web Vitals** (LCP, FID, CLS)