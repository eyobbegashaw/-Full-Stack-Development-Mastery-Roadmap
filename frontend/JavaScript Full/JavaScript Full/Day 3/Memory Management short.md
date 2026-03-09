# **Memory Management in JavaScript**

## **Memory Lifecycle**
JavaScript memory lifecycle follows three consistent stages:

1. **Allocation**: Memory is allocated when you create variables, objects, functions, etc.
2. **Usage**: Reading/writing to allocated memory during program execution
3. **Release**: Memory is freed when no longer needed, handled automatically by garbage collection

## **Garbage Collection**
JavaScript uses **automatic garbage collection** to manage memory. The main algorithms:

### **1. Reference Counting (Legacy)**
- Counts how many references point to each object
- Memory freed when count reaches zero
- **Problem**: Circular references create memory leaks
  ```javascript
  // Circular reference - would leak with reference counting
  let obj1 = { ref: null };
  let obj2 = { ref: obj1 };
  obj1.ref = obj2; // obj1 references obj2, obj2 references obj1
  ```

### **2. Mark-and-Sweep (Modern)**
- **Mark**: Starting from "roots" (global variables), marks all reachable objects
- **Sweep**: Removes unmarked (unreachable) objects
- Solves circular reference problem

### **3. Generational Collection**
- Objects divided into "young" (short-lived) and "old" (long-lived) generations
- Young generation cleaned frequently (Scavenge algorithm)
- Old generation cleaned less often (Mark-and-sweep/compact)

## **Common Memory Issues & Solutions**

### **Memory Leaks:**
1. **Accidental Globals**: Variables without `var`, `let`, `const`
   ```javascript
   function leak() {
       leakyVar = 'I am global!'; // Creates window.leakyVar
   }
   ```

2. **Forgotten Timers/Intervals**: 
   ```javascript
   // Keeps reference to element, preventing GC
   setInterval(() => {
       console.log(document.getElementById('element'));
   }, 1000);
   ```

3. **Closures Holding References**:
   ```javascript
   function outer() {
       const largeData = new Array(1000000);
       return function inner() {
           // inner function holds reference to largeData forever
       };
   }
   ```

4. **Detached DOM Elements**:
   ```javascript
   // Element removed from DOM but still referenced in JavaScript
   let element = document.getElementById('myDiv');
   document.body.removeChild(element);
   // element variable still holds reference
   ```

## **Best Practices**

### **Do:**
- Use `let` and `const` for block scoping
- Nullify references when done: `largeObject = null`
- Use weak references (`WeakMap`, `WeakSet`) for caches
- Clean up event listeners and timers
- Profile memory usage with DevTools

### **Don't:**
- Create unnecessary global variables
- Keep references to unused DOM elements
- Create circular references intentionally
- Cache everything indefinitely

## **Memory Profiling Tools**
1. **Chrome DevTools Memory Tab**
   - Heap Snapshots
   - Allocation Timeline
   - Allocation Sampling
2. **Node.js**: `--inspect` flag with Chrome DevTools
3. **Performance.memory API** (browser)

## **Example of Proper Memory Management**

```javascript
// BAD - Memory leak
class LeakyCache {
    constructor() {
        this.cache = new Map(); // Strong references
    }
    add(key, value) {
        this.cache.set(key, value);
    }
}

// GOOD - Uses WeakMap for automatic cleanup
class SafeCache {
    constructor() {
        this.cache = new WeakMap(); // Weak references
    }
    add(key, value) {
        this.cache.set(key, value); // Key must be object
    }
}

// Proper cleanup pattern
function processLargeData() {
    const data = loadHugeDataset(); // Large memory allocation
    
    // Process data...
    const result = process(data);
    
    // Explicitly release reference
    data = null; // Helps GC
    
    return result;
}
```

## **Key Takeaways**
1. JavaScript has **automatic memory management** but developers must still be mindful
2. **Mark-and-sweep** is the primary garbage collection algorithm
3. **Memory leaks** occur when unwanted references prevent garbage collection
4. **Weak references** (`WeakMap`, `WeakSet`) help avoid memory leaks
5. **Tools are available** to profile and debug memory issues
6. **Best practices** prevent most common memory problems