# **Memory Management, Memory Lifecycle & Garbage Collection**

## **Concept Overview (70%)**

### **1. Memory Management in JavaScript**

#### **The Challenge**
JavaScript has **automatic memory management**, unlike languages like C/C++ where developers manually allocate and free memory. This convenience comes with complexity under the hood.

**Why Memory Management Matters:**
- **Performance**: Poor memory usage slows down applications
- **Memory Leaks**: Can crash browsers/servers
- **Predictability**: Consistent performance requires good memory habits
- **Scalability**: Applications must handle increasing data loads

#### **JavaScript Memory Model**
JavaScript uses a **heap-based memory model** with several distinct regions:

1. **Stack**: Fixed size, stores primitive values and references
   - Fast allocation/deallocation (LIFO)
   - Limited size (~1MB in browsers)
   - Stores function calls, local variables, primitive values

2. **Heap**: Dynamic size, stores objects, functions, closures
   - Slower allocation (requires garbage collection)
   - Much larger capacity
   - Complex structure with multiple generations

3. **External Memory**: WebAssembly memory, GPU buffers, etc.

### **2. Memory Lifecycle**

#### **Three Fundamental Phases:**

```javascript
// 1. ALLOCATION
let primitive = 42;               // Stack allocation
let object = { x: 10, y: 20 };    // Heap allocation
let array = new Array(1000000);   // Large heap allocation

// 2. USAGE
object.x = 100;                    // Read/write operations
array[0] = 'value';               // Memory access
function process() {               // Function creates execution context
    let local = object.x + 10;    // More allocations during usage
    return local;
}

// 3. RELEASE (Garbage Collection)
object = null;                    // Mark for collection
// array automatically collected when no longer reachable
```

#### **Detailed Lifecycle Stages:**

**Stage 1: Allocation Strategies**
- **Inline caching**: V8 optimizes object property access
- **Hidden classes**: For object shape optimization
- **Array pre-allocation**: `new Array(size)` vs `[]`
- **String interning**: Reuse of identical strings

**Stage 2: Usage Patterns**
- **Temporal locality**: Recently used memory likely to be used again
- **Spatial locality**: Nearby memory locations likely to be used together
- **Reference patterns**: How objects reference each other

**Stage 3: Deallocation Triggers**
- Variable goes out of scope
- Object reference set to null/undefined
- Reassignment to new value
- Function execution completes (for local variables)

### **3. Garbage Collection (GC)**

#### **The Mark-and-Sweep Algorithm (Modern Implementation)**
**Phase 1: Marking**
- Start from **roots** (global object, currently executing functions, etc.)
- Traverse all reachable objects, marking them as "alive"
- Use **depth-first search** or **breadth-first search**

**Phase 2: Sweeping**
- Scan all memory
- Free memory occupied by unmarked objects
- Add freed blocks to free list for reuse

**Phase 3: Compacting** (Optional but important)
- Move live objects together
- Eliminate fragmentation
- Improve cache locality

#### **Generational Collection**
Modern JavaScript engines use **generational hypothesis**:
- **Most objects die young**
- **Old objects likely to survive**

**Heap Structure:**
```
┌─────────────────────────────────────────┐
│            NEW SPACE (Young)            │
│  • Small (1-8MB)                        │
│  • Frequent, fast collections           │
│  • Scavenger algorithm                  │
├─────────────────────────────────────────┤
│            OLD SPACE (Old)              │
│  • Large                                │
│  • Infrequent, slow collections         │
│  • Mark-sweep-compact                   │
├─────────────────────────────────────────┤
│          LARGE OBJECT SPACE             │
│  • Very large objects (>1MB)            │
│  • Treated specially                    │
├─────────────────────────────────────────┤
│            CODE SPACE                   │
│  • Compiled code                        │
│  • Separate management                  │
└─────────────────────────────────────────┘
```

#### **Scavenger (Young Generation GC)**
**Semi-space algorithm**:
- Divide young generation into **from-space** and **to-space**
- Copy live objects from from-space to to-space
- Discard from-space entirely
- Promotes long-lived objects to old generation

**Advantages**:
- Very fast for young objects
- Excellent spatial locality (objects copied contiguously)
- Automatic compaction

#### **Incremental and Concurrent GC**
- **Incremental GC**: Break GC into small chunks interleaved with execution
- **Concurrent GC**: Run GC in parallel with JavaScript execution
- **Idle-time GC**: Run during browser idle periods (requestIdleCallback)

## **Code Explanation (30%)**

```javascript
// ====== 1. MEMORY ALLOCATION PATTERNS ======

// A. STACK ALLOCATION (Primitives)
function stackExample() {
    let a = 10;            // Number → Stack
    let b = 'hello';       // String → Stack (small strings)
    let c = true;          // Boolean → Stack
    let d = null;          // Null → Stack
    let e = undefined;     // Undefined → Stack
    
    // These are copied by value
    let copy = a;          // New stack allocation (value copy)
    a = 20;                // Does NOT affect copy
    
    return a + copy;       // Stack variables freed when function exits
}

// B. HEAP ALLOCATION (Objects)
function heapExample() {
    // Objects allocated in heap
    let obj = {            // Object → Heap, reference → Stack
        name: 'John',
        age: 30,
        scores: [95, 87, 92]  // Array → Heap
    };
    
    // Functions are objects too
    let func = function() {    // Function → Heap
        console.log('Hello');
    };
    
    // Large strings go to heap
    let largeString = 'x'.repeat(1000000);  // Heap
    
    // References are copied, not objects
    let objRef = obj;           // Same heap object, new stack reference
    objRef.age = 31;            // Affects original obj
    
    return obj;                 // Object persists if returned
}

// C. MEMORY FRAGMENTATION EXAMPLE
function fragmentationDemo() {
    const objects = [];
    
    // Create alternating large/small objects
    for (let i = 0; i < 1000; i++) {
        if (i % 2 === 0) {
            objects.push(new Array(1000).fill('x'));  // Large
        } else {
            objects.push({ tiny: 'object' });        // Small
        }
    }
    
    // Delete all large objects
    for (let i = 0; i < objects.length; i += 2) {
        objects[i] = null;
    }
    
    // Heap now has holes (fragmentation)
    // GC compaction will eventually fix this
}

// ====== 2. COMMON MEMORY LEAKS ======

// A. ACCIDENTAL GLOBAL VARIABLES
function createGlobal() {
    // Missing var/let/const creates global
    globalVar = new Array(1000000).fill('leak');  // LEAK!
    
    // 'this' in non-strict mode
    this.anotherLeak = 'global';  // LEAK in non-strict mode!
}

// B. FORGOTTEN TIMERS/CALLBACKS
function timerLeak() {
    const data = new Array(1000).fill('data');
    
    // Timer keeps reference to data
    setInterval(() => {
        console.log(data.length);  // data never collected
    }, 1000);
    
    // Solution: clear interval when done
    // const timerId = setInterval(...);
    // clearInterval(timerId);
}

// C. CLOSURE LEAKS
function closureLeak() {
    const hugeData = new Array(1000000).fill('secret');
    
    return function() {
        // Closure captures hugeData even if unused
        console.log('Doing nothing with hugeData');
        // hugeData stays in memory as long as returned function exists
    };
}

// D. DOM REFERENCES
function domReferenceLeak() {
    const elements = [];
    
    // Store DOM elements in array
    for (let i = 0; i < 1000; i++) {
        const element = document.createElement('div');
        document.body.appendChild(element);
        elements.push(element);  // Reference prevents GC
        
        // Even after removal from DOM
        // document.body.removeChild(element);
        // Still in array → LEAK!
    }
    
    // Solution: nullify references
    // elements.length = 0;
}

// E. DETACHED DOM TREES
function detachedDOM() {
    // Create DOM element
    const parent = document.createElement('div');
    const child = document.createElement('span');
    parent.appendChild(child);
    
    // Remove from document but keep reference
    document.body.appendChild(parent);
    document.body.removeChild(parent);
    
    // parent still references child
    // child still references parent (circular)
    // Entire tree is detached but not collected
    
    // Solution: nullify references
    // parent = null;
    // child = null;
}

// ====== 3. GARBAGE COLLECTION IN ACTION ======

// A. REFERENCE COUNTING (Old, problematic)
function referenceCounting() {
    let objA = { data: 'A' };
    let objB = { data: 'B' };
    
    // Circular reference
    objA.ref = objB;  // objB ref count = 2
    objB.ref = objA;  // objA ref count = 2
    
    // Even after removing external references
    objA = null;
    objB = null;
    
    // Reference count never reaches 0
    // MEMORY LEAK in reference counting
    // Modern JS uses mark-and-sweep instead
}

// B. MARK-AND-SWEEP DEMONSTRATION
// Simulating GC algorithm conceptually
class MemoryHeap {
    constructor() {
        this.objects = new Map();  // Object ID → Object
        this.references = new Map(); // Object ID → [Referenced IDs]
        this.roots = new Set();    // Root object IDs
    }
    
    allocate(data) {
        const id = Symbol();
        this.objects.set(id, data);
        this.references.set(id, new Set());
        return id;
    }
    
    addReference(fromId, toId) {
        this.references.get(fromId).add(toId);
    }
    
    markFromRoots() {
        const marked = new Set();
        const stack = [...this.roots];
        
        while (stack.length > 0) {
            const id = stack.pop();
            if (!marked.has(id)) {
                marked.add(id);
                const refs = this.references.get(id) || new Set();
                stack.push(...refs);
            }
        }
        
        return marked;
    }
    
    sweep(marked) {
        const toDelete = [];
        
        for (const [id, obj] of this.objects) {
            if (!marked.has(id) && !this.roots.has(id)) {
                toDelete.push(id);
            }
        }
        
        // Free memory
        for (const id of toDelete) {
            this.objects.delete(id);
            this.references.delete(id);
        }
        
        return toDelete.length;
    }
    
    collectGarbage() {
        console.log('Starting GC...');
        const marked = this.markFromRoots();
        const freed = this.sweep(marked);
        console.log(`Freed ${freed} objects`);
        return freed;
    }
}

// C. GENERATIONAL GC BEHAVIOR
function generationalGCDemo() {
    // Young generation behavior
    let temporary = [];
    
    for (let i = 0; i < 10000; i++) {
        // These objects die quickly (scavenger collects them)
        temporary.push({ id: i });
    }
    
    // Long-lived object (promoted to old generation)
    const permanent = { 
        data: new Array(1000).fill('important'),
        createdAt: Date.now()
    };
    
    // Mix of young and old
    const mixed = [];
    
    // Young objects
    for (let i = 0; i < 100; i++) {
        mixed.push({ temp: i });
    }
    
    // Old object reference
    mixed.push(permanent);
    
    // When young GC runs:
    // - temporary objects collected quickly
    // - permanent survives multiple collections
    // - Eventually promoted to old generation
}

// ====== 4. MEMORY OPTIMIZATION TECHNIQUES ======

// A. OBJECT POOLING
class ObjectPool {
    constructor(createFn, resetFn, initialSize = 100) {
        this.createFn = createFn;
        this.resetFn = resetFn;
        this.pool = [];
        this.active = new Set();
        
        // Pre-allocate objects
        for (let i = 0; i < initialSize; i++) {
            this.pool.push(createFn());
        }
    }
    
    acquire() {
        let obj;
        
        if (this.pool.length > 0) {
            obj = this.pool.pop();
        } else {
            obj = this.createFn();
        }
        
        this.resetFn(obj);
        this.active.add(obj);
        return obj;
    }
    
    release(obj) {
        this.resetFn(obj);
        this.active.delete(obj);
        this.pool.push(obj);
    }
    
    get size() {
        return this.pool.length + this.active.size;
    }
}

// Usage: Particle system
class Particle {
    constructor() {
        this.x = 0;
        this.y = 0;
        this.vx = 0;
        this.vy = 0;
        this.life = 0;
    }
    
    reset() {
        this.x = 0;
        this.y = 0;
        this.vx = 0;
        this.vy = 0;
        this.life = 100;
    }
}

const particlePool = new ObjectPool(
    () => new Particle(),
    (p) => p.reset(),
    1000
);

// B. ARRAY REUSE
class ReusableArray {
    constructor() {
        this.array = [];
        this.length = 0;
    }
    
    push(value) {
        if (this.length >= this.array.length) {
            this.array.push(value);
        } else {
            this.array[this.length] = value;
        }
        this.length++;
    }
    
    clear() {
        this.length = 0;
        // Keep allocated memory
    }
    
    toArray() {
        return this.array.slice(0, this.length);
    }
}

// C. LAZY INITIALIZATION
class LazyResource {
    constructor() {
        this._data = null;
        this._initialized = false;
    }
    
    get data() {
        if (!this._initialized) {
            this._data = this.loadExpensiveData();
            this._initialized = true;
        }
        return this._data;
    }
    
    loadExpensiveData() {
        // Simulate expensive operation
        console.log('Loading expensive data...');
        return new Array(1000000).fill('data');
    }
    
    clear() {
        this._data = null;
        this._initialized = false;
        // Suggest GC to collect
        if (global.gc) global.gc();
    }
}

// ====== 5. MEMORY PROFILING & DEBUGGING ======

// A. MEMORY USAGE MEASUREMENT
class MemoryMonitor {
    static get usedJSHeapSize() {
        if (performance.memory) {
            return performance.memory.usedJSHeapSize;
        }
        return null;
    }
    
    static get totalJSHeapSize() {
        if (performance.memory) {
            return performance.memory.totalJSHeapSize;
        }
        return null;
    }
    
    static get heapSizeLimit() {
        if (performance.memory) {
            return performance.memory.jsHeapSizeLimit;
        }
        return null;
    }
    
    static measure(description, fn) {
        const before = MemoryMonitor.usedJSHeapSize;
        const result = fn();
        const after = MemoryMonitor.usedJSHeapSize;
        
        if (before && after) {
            const diff = after - before;
            console.log(`${description}: ${diff} bytes`);
        }
        
        return result;
    }
    
    static checkForLeaks(testFn, iterations = 100) {
        const initial = MemoryMonitor.usedJSHeapSize;
        const snapshots = [];
        
        for (let i = 0; i < iterations; i++) {
            testFn();
            
            // Force GC if available (Node.js)
            if (global.gc) global.gc();
            
            // Take snapshot
            snapshots.push(MemoryMonitor.usedJSHeapSize);
        }
        
        const final = MemoryMonitor.usedJSHeapSize;
        const growth = final - initial;
        
        console.log(`Memory growth after ${iterations} iterations: ${growth} bytes`);
        
        // Analyze trend
        const trend = this.analyzeTrend(snapshots);
        console.log(`Memory trend: ${trend}`);
        
        return growth;
    }
    
    static analyzeTrend(snapshots) {
        // Simple trend analysis
        const first = snapshots[0];
        const last = snapshots[snapshots.length - 1];
        
        if (last > first * 1.5) return 'LEAKING';
        if (last > first) return 'INCREASING';
        if (last < first) return 'DECREASING';
        return 'STABLE';
    }
}

// B. LEAK DETECTION PATTERNS
function createLeakyObjects() {
    const leaks = [];
    
    return {
        addLeak: (size) => {
            leaks.push(new Array(size).fill('leak'));
        },
        
        clearLeaks: () => {
            leaks.length = 0;
            // In real code, might need to nullify references
        },
        
        check: () => {
            return leaks.length;
        }
    };
}

// C. MEMORY PRESSURE SIMULATION
function createMemoryPressure() {
    const chunks = [];
    const chunkSize = 1024 * 1024; // 1MB
    
    console.log('Creating memory pressure...');
    
    try {
        while (true) {
            chunks.push(new Array(chunkSize).fill('x'));
            console.log(`Allocated ${chunks.length}MB`);
            
            // Slow down to observe
            if (chunks.length % 10 === 0) {
                console.log('Current memory:', 
                    (performance.memory?.usedJSHeapSize || 0) / 1024 / 1024, 'MB');
            }
        }
    } catch (error) {
        console.log('Memory limit reached:', error.message);
        return chunks.length;
    }
}

// ====== 6. ADVANCED MEMORY PATTERNS ======

// A. WEAK REFERENCES (ES2021)
function weakReferenceDemo() {
    // WeakRef doesn't prevent garbage collection
    let largeObject = { data: new Array(1000000).fill('data') };
    const weakRef = new WeakRef(largeObject);
    
    // Check if object still exists
    console.log('Initial:', weakRef.deref() !== undefined);
    
    // Remove strong reference
    largeObject = null;
    
    // Suggest GC
    if (global.gc) {
        global.gc();
        console.log('After GC:', weakRef.deref() !== undefined);
    }
    
    // FinalizationRegistry for cleanup
    const registry = new FinalizationRegistry((heldValue) => {
        console.log(`Object with value "${heldValue}" was garbage collected`);
    });
    
    const obj = { id: 'test' };
    registry.register(obj, 'important data');
    
    // When obj is GC'd, callback runs
}

// B. SHARED ARRAY BUFFERS (Advanced)
function sharedMemoryDemo() {
    // Create shared memory (for Web Workers)
    const sharedBuffer = new SharedArrayBuffer(1024); // 1KB shared memory
    const sharedArray = new Int32Array(sharedBuffer);
    
    // Multiple threads can access this memory
    // Synchronization needed with Atomics
    
    // Atomics for thread-safe operations
    Atomics.add(sharedArray, 0, 5); // Atomic addition
    Atomics.compareExchange(sharedArray, 0, 5, 10); // Atomic compare-and-swap
    
    return sharedBuffer;
}

// C. MANUAL MEMORY MANAGEMENT SIMULATION
class ManualMemoryManager {
    constructor() {
        this.memory = new ArrayBuffer(1024 * 1024); // 1MB
        this.heap = new Uint8Array(this.memory);
        this.freeList = [{ start: 0, size: this.heap.length }];
        this.allocations = new Map(); // ID → {start, size}
        this.nextId = 1;
    }
    
    allocate(size) {
        // Find first fit
        for (let i = 0; i < this.freeList.length; i++) {
            const block = this.freeList[i];
            if (block.size >= size) {
                // Use this block
                const allocation = {
                    start: block.start,
                    size: size
                };
                
                const id = this.nextId++;
                this.allocations.set(id, allocation);
                
                // Update free list
                if (block.size === size) {
                    this.freeList.splice(i, 1);
                } else {
                    block.start += size;
                    block.size -= size;
                }
                
                return { id, pointer: allocation.start };
            }
        }
        
        throw new Error('Out of memory');
    }
    
    free(id) {
        const allocation = this.allocations.get(id);
        if (!allocation) return false;
        
        // Add to free list and merge adjacent blocks
        this.freeList.push({ 
            start: allocation.start, 
            size: allocation.size 
        });
        this.freeList.sort((a, b) => a.start - b.start);
        
        // Merge adjacent free blocks
        for (let i = 0; i < this.freeList.length - 1; i++) {
            const current = this.freeList[i];
            const next = this.freeList[i + 1];
            
            if (current.start + current.size === next.start) {
                current.size += next.size;
                this.freeList.splice(i + 1, 1);
                i--;
            }
        }
        
        this.allocations.delete(id);
        return true;
    }
    
    write(id, data) {
        const allocation = this.allocations.get(id);
        if (!allocation) return false;
        
        const view = new Uint8Array(this.memory, allocation.start, allocation.size);
        for (let i = 0; i < Math.min(data.length, allocation.size); i++) {
            view[i] = data.charCodeAt(i);
        }
        
        return true;
    }
    
    read(id) {
        const allocation = this.allocations.get(id);
        if (!allocation) return null;
        
        const view = new Uint8Array(this.memory, allocation.start, allocation.size);
        let result = '';
        for (let i = 0; i < allocation.size; i++) {
            if (view[i] === 0) break; // Null terminator
            result += String.fromCharCode(view[i]);
        }
        
        return result;
    }
}

// ====== 7. REAL-WORLD SCENARIOS ======

// A. IMAGE PROCESSING MEMORY MANAGEMENT
class ImageProcessor {
    constructor() {
        this.canvas = document.createElement('canvas');
        this.ctx = this.canvas.getContext('2d');
        this.imageDataCache = new WeakMap(); // Auto-cleaned by GC
    }
    
    processImage(image) {
        // Reuse canvas instead of creating new one
        this.canvas.width = image.width;
        this.canvas.height = image.height;
        this.ctx.drawImage(image, 0, 0);
        
        // Get image data (large memory allocation)
        const imageData = this.ctx.getImageData(0, 0, image.width, image.height);
        
        // Process pixels
        const data = imageData.data;
        for (let i = 0; i < data.length; i += 4) {
            // Simple grayscale
            const avg = (data[i] + data[i + 1] + data[i + 2]) / 3;
            data[i] = avg;     // R
            data[i + 1] = avg; // G
            data[i + 2] = avg; // B
            // Alpha stays the same
        }
        
        // Put back
        this.ctx.putImageData(imageData, 0, 0);
        
        // imageData automatically GC'd when no longer referenced
        
        return this.canvas.toDataURL();
    }
    
    // Use WeakMap for caching without preventing GC
    getCachedOperation(image, operation) {
        const key = { image, operation };
        if (this.imageDataCache.has(key)) {
            return this.imageDataCache.get(key);
        }
        
        const result = this.processImage(image);
        this.imageDataCache.set(key, result);
        return result;
    }
}

// B. REAL-TIME DATA STREAM PROCESSING
class StreamProcessor {
    constructor(bufferSize = 1000) {
        this.buffer = new Array(bufferSize);
        this.head = 0;
        this.tail = 0;
        this.count = 0;
        
        // Reusable object pool for data points
        this.pointPool = new ObjectPool(
            () => ({ timestamp: 0, value: 0 }),
            (p) => { p.timestamp = 0; p.value = 0; },
            1000
        );
    }
    
    addData(value) {
        if (this.count >= this.buffer.length) {
            // Buffer full, overwrite oldest
            this.pointPool.release(this.buffer[this.tail]);
            this.tail = (this.tail + 1) % this.buffer.length;
            this.count--;
        }
        
        const point = this.pointPool.acquire();
        point.timestamp = Date.now();
        point.value = value;
        
        this.buffer[this.head] = point;
        this.head = (this.head + 1) % this.buffer.length;
        this.count++;
        
        return point;
    }
    
    processWindow(windowSize) {
        const result = [];
        let current = this.tail;
        
        for (let i = 0; i < Math.min(this.count, windowSize); i++) {
            result.push(this.buffer[current]);
            current = (current + 1) % this.buffer.length;
        }
        
        return result;
    }
    
    clear() {
        // Return all points to pool
        let current = this.tail;
        for (let i = 0; i < this.count; i++) {
            this.pointPool.release(this.buffer[current]);
            current = (current + 1) % this.buffer.length;
        }
        
        this.head = 0;
        this.tail = 0;
        this.count = 0;
    }
}

// C. LARGE DATASET PAGINATION
class VirtualScrollDataSource {
    constructor(totalItems, pageSize = 100) {
        this.totalItems = totalItems;
        this.pageSize = pageSize;
        this.cache = new Map(); // page → data
        this.maxCacheSize = 10; // Keep only 10 pages in memory
        
        // WeakRef for least recently used pages
        this.weakRefs = new Map();
    }
    
    async getPage(page) {
        // Check cache
        if (this.cache.has(page)) {
            return this.cache.get(page);
        }
        
        // Clean cache if too large
        if (this.cache.size >= this.maxCacheSize) {
            this.evictOldPages();
        }
        
        // Load page
        const data = await this.loadPageFromSource(page);
        
        // Cache with WeakRef
        this.cache.set(page, data);
        this.weakRefs.set(page, new WeakRef(data));
        
        return data;
    }
    
    async loadPageFromSource(page) {
        // Simulate API call
        await new Promise(resolve => setTimeout(resolve, 50));
        
        const start = page * this.pageSize;
        const end = Math.min(start + this.pageSize, this.totalItems);
        const data = [];
        
        for (let i = start; i < end; i++) {
            data.push({
                id: i,
                value: `Item ${i}`,
                data: new Array(100).fill('x').join('') // Simulate large item
            });
        }
        
        return data;
    }
    
    evictOldPages() {
        // Simple LRU: remove oldest page
        const oldest = this.cache.keys().next().value;
        if (oldest !== undefined) {
            this.cache.delete(oldest);
            this.weakRefs.delete(oldest);
        }
    }
    
    clear() {
        this.cache.clear();
        this.weakRefs.clear();
        
        // Suggest GC
        if (global.gc) global.gc();
    }
}

// ====== 8. BEST PRACTICES SUMMARY ======

/*
MEMORY MANAGEMENT BEST PRACTICES:

1. ALLOCATION:
   - Use object pools for frequently created/destroyed objects
   - Pre-allocate arrays when size is known
   - Avoid large temporary objects in loops

2. USAGE:
   - Minimize closure scope
   - Use const/let instead of var
   - Declare variables in smallest possible scope
   - Avoid creating functions inside loops

3. CLEANUP:
   - Nullify references when done
   - Clear intervals/timeouts
   - Remove event listeners
   - Delete DOM references
   - Use WeakMap/WeakSet when appropriate

4. MONITORING:
   - Use Chrome DevTools Memory profiler
   - Check performance.memory API
   - Look for increasing heap size
   - Test with different data volumes

5. AVOID COMMON LEAKS:
   - Accidental globals (use strict mode)
   - Forgotten timers/callbacks
   - Circular references in closures
   - Detached DOM trees
   - Cached data without expiration

6. PERFORMANCE:
   - Use requestIdleCallback for non-critical work
   - Batch DOM updates
   - Use virtual scrolling for large lists
   - Implement pagination/lazy loading

7. ADVANCED TECHNIQUES:
   - Web Workers for heavy computation
   - SharedArrayBuffer for inter-thread communication
   - WeakRef for caching without preventing GC
   - FinalizationRegistry for cleanup callbacks
*/

// Final example: Comprehensive memory-aware component
class MemoryAwareComponent {
    constructor() {
        this.data = null;
        this.listeners = new Set();
        this.timers = new Set();
        this.domRefs = new WeakMap();
        
        // Memory monitoring
        this.memoryCheckInterval = setInterval(() => {
            this.checkMemory();
        }, 60000); // Check every minute
    }
    
    async loadData() {
        // Clear previous data if exists
        this.clearData();
        
        // Load new data
        this.data = await this.fetchData();
        
        // Process with memory awareness
        return this.processWithMemoryLimit(this.data);
    }
    
    processWithMemoryLimit(data) {
        const chunkSize = 1000;
        const results = [];
        
        for (let i = 0; i < data.length; i += chunkSize) {
            const chunk = data.slice(i, i + chunkSize);
            results.push(...this.processChunk(chunk));
            
            // Check memory every few chunks
            if (i % (chunkSize * 10) === 0) {
                this.checkMemory();
            }
        }
        
        return results;
    }
    
    processChunk(chunk) {
        // Process chunk
        return chunk.map(item => ({ ...item, processed: true }));
    }
    
    checkMemory() {
        if (!performance.memory) return;
        
        const used = performance.memory.usedJSHeapSize;
        const limit = performance.memory.jsHeapSizeLimit;
        const ratio = used / limit;
        
        if (ratio > 0.8) {
            console.warn('High memory usage:', Math.round(ratio * 100) + '%');
            this.cleanupNonEssential();
        }
        
        if (ratio > 0.9) {
            console.error('Critical memory usage!');
            this.emergencyCleanup();
        }
    }
    
    cleanupNonEssential() {
        // Clear caches
        // Cancel non-critical operations
        // Reduce data retention
    }
    
    emergencyCleanup() {
        // Clear all non-essential data
        // Stop all timers
        // Cancel pending operations
        // Notify user
    }
    
    clearData() {
        this.data = null;
        
        // Clear listeners
        for (const listener of this.listeners) {
            listener.target.removeEventListener(listener.type, listener.handler);
        }
        this.listeners.clear();
        
        // Clear timers
        for (const timer of this.timers) {
            clearTimeout(timer);
        }
        this.timers.clear();
        
        // DOM references are weak, don't need manual cleanup
        
        // Suggest GC
        if (global.gc) {
            setTimeout(() => global.gc(), 0);
        }
    }
    
    destroy() {
        this.clearData();
        clearInterval(this.memoryCheckInterval);
        
        // Final cleanup
        this.data = null;
        this.listeners = null;
        this.timers = null;
    }
}
```

### **Key Takeaways:**

1. **Memory Lifecycle**: Allocation → Usage → Release (GC)
2. **Garbage Collection**: Modern JS uses mark-and-sweep with generational collection
3. **Memory Leaks**: Caused by unwanted persistent references
4. **Optimization**: Object pooling, lazy loading, and proper cleanup
5. **Monitoring**: Use browser tools and `performance.memory` API
6. **Modern Features**: WeakRef, FinalizationRegistry, SharedArrayBuffer
7. **Best Practices**: Scope management, reference cleanup, and memory-aware coding

Understanding memory management is crucial for building performant, reliable JavaScript applications that scale effectively.