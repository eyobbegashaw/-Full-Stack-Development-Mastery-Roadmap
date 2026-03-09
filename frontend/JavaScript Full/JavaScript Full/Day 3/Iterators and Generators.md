# **Iterators and Generators - Comprehensive Guide**

## **Concept Overview (70%)**

### **1. Iterators: The Foundation of Iteration**

#### **What are Iterators?**
Iterators are **objects that implement the Iterator protocol** - a standardized way to produce a sequence of values. They provide a **consistent interface** for traversing different data structures.

**Key Characteristics:**
- **Stateful**: Remember their position in the sequence
- **Lazy evaluation**: Produce values on demand
- **Consumption**: Once consumed, cannot be reused without resetting
- **Protocol-based**: Follow specific rules defined by ECMAScript

#### **The Iterator Protocol**
An object is an iterator when it implements a **`next()`** method that returns an object with:
- **`value`**: The next value in the sequence
- **`done`**: Boolean indicating if iteration is complete

```javascript
// Iterator Protocol:
{
    next() {
        return {
            value: any,    // Current value or undefined when done
            done: boolean  // true when iteration complete
        };
    }
}
```

#### **Iterables vs Iterators**
- **Iterable**: Object that implements `[Symbol.iterator]()` method
- **Iterator**: Object returned by `[Symbol.iterator]()` that implements `next()`

**Built-in Iterables:**
- Arrays, Strings, Maps, Sets, NodeLists
- Arguments object
- TypedArrays

### **2. Generators: Simplified Iterators**

#### **What are Generators?**
Generators are **special functions** that can pause and resume execution. They provide **syntactic sugar** for creating iterators with much less code.

**Key Characteristics:**
- **Function with an asterisk**: `function* generator()`
- **Pausable**: Can yield multiple values
- **Resumable**: Maintain state between calls
- **Two-way communication**: Can receive values when resuming

#### **Generator Protocol**
Generators automatically implement both the **Iterable** and **Iterator** protocols:
- `next()`: Resume execution until next `yield` or `return`
- `return()`: Stop generator execution early
- `throw()`: Throw an error into generator

#### **Control Flow with Generators**
- **`yield`**: Pause execution and emit a value
- **`yield*`**: Delegate to another generator/iterable
- **`return`**: Final value and mark as done
- **`next(value)`**: Resume with a passed value

### **3. Advanced Concepts**

#### **Async Iterators & Generators**
ES2018 introduced async iteration for handling asynchronous sequences:

- **Async Iterators**: Implement `[Symbol.asyncIterator]()` and return promises
- **Async Generators**: `async function*` that can `yield` promises
- **`for await...of`**: Loop over async iterables

#### **Infinite Sequences**
Generators excel at creating infinite or very large sequences that would be impractical to store in memory.

#### **Coroutines & Cooperative Multitasking**
Generators can be used to implement coroutines - functions that can voluntarily yield control to other functions.

#### **State Machines**
Generators naturally model state machines with their ability to pause and resume at specific points.

## **Code Explanation (30%)**

```javascript
// ====== 1. ITERATORS - FROM BASIC TO ADVANCED ======

// A. MANUAL ITERATOR IMPLEMENTATION

// Basic custom iterator
const counterIterator = {
    current: 1,
    end: 5,
    
    next() {
        if (this.current <= this.end) {
            return { value: this.current++, done: false };
        }
        return { value: undefined, done: true };
    },
    
    // Optional: Make it iterable
    [Symbol.iterator]() {
        return this;
    }
};

// Using the iterator manually
let result = counterIterator.next();
while (!result.done) {
    console.log('Manual iteration:', result.value);
    result = counterIterator.next();
}

// B. ITERABLE OBJECTS
class Range {
    constructor(start, end, step = 1) {
        this.start = start;
        this.end = end;
        this.step = step;
    }
    
    // Make class iterable
    *[Symbol.iterator]() {
        let current = this.start;
        while (current <= this.end) {
            yield current;
            current += this.step;
        }
    }
    
    // Async version
    async *[Symbol.asyncIterator]() {
        let current = this.start;
        while (current <= this.end) {
            // Simulate async operation
            await new Promise(resolve => setTimeout(resolve, 100));
            yield current;
            current += this.step;
        }
    }
}

// Using the iterable
const range = new Range(1, 10, 2);
for (const num of range) {
    console.log('Range iteration:', num);
}

// C. ADVANCED ITERATOR WITH STATE
class Tree {
    constructor(value, children = []) {
        this.value = value;
        this.children = children;
    }
    
    // Depth-first traversal
    *[Symbol.iterator]() {
        yield this.value;
        for (const child of this.children) {
            yield* child; // Recursive delegation
        }
    }
    
    // Breadth-first traversal
    *bfs() {
        const queue = [this];
        while (queue.length > 0) {
            const node = queue.shift();
            yield node.value;
            queue.push(...node.children);
        }
    }
    
    // Reverse iteration
    *reverse() {
        for (let i = this.children.length - 1; i >= 0; i--) {
            yield* this.children[i].reverse();
        }
        yield this.value;
    }
}

// ====== 2. GENERATORS - COMPREHENSIVE EXAMPLES ======

// A. BASIC GENERATORS

// Simple number generator
function* numberGenerator(limit) {
    let num = 1;
    while (num <= limit) {
        yield num;
        num++;
    }
    return 'Finished'; // Return value when done
}

const gen = numberGenerator(3);
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 3, done: false }
console.log(gen.next()); // { value: 'Finished', done: true }

// B. GENERATOR WITH TWO-WAY COMMUNICATION
function* dialogGenerator() {
    const name = yield 'What is your name?';
    const age = yield `Hello ${name}, how old are you?`;
    const color = yield `${age} is a great age! What's your favorite color?`;
    return `${name}, ${age}, likes ${color}`;
}

const dialog = dialogGenerator();
console.log(dialog.next());           // { value: 'What is your name?', done: false }
console.log(dialog.next('Alice'));    // { value: 'Hello Alice, how old are you?', done: false }
console.log(dialog.next(30));         // { value: '30 is a great age! What's your favorite color?', done: false }
console.log(dialog.next('blue'));     // { value: 'Alice, 30, likes blue', done: true }

// C. INFINITE SEQUENCES
function* fibonacci() {
    let [prev, curr] = [0, 1];
    while (true) {
        yield curr;
        [prev, curr] = [curr, prev + curr];
    }
}

const fib = fibonacci();
console.log('Fibonacci:', fib.next().value); // 1
console.log('Fibonacci:', fib.next().value); // 1
console.log('Fibonacci:', fib.next().value); // 2
console.log('Fibonacci:', fib.next().value); // 3

// D. GENERATOR DELEGATION (yield*)
function* innerGenerator() {
    yield 'inner 1';
    yield 'inner 2';
}

function* outerGenerator() {
    yield 'outer start';
    yield* innerGenerator(); // Delegate to inner generator
    yield* [3, 4, 5];        // Delegate to array (any iterable)
    yield 'outer end';
}

// ====== 3. ADVANCED GENERATOR PATTERNS ======

// A. GENERATOR AS STATE MACHINE
function* trafficLight() {
    while (true) {
        yield 'GREEN - Go';
        yield 'YELLOW - Slow down';
        yield 'RED - Stop';
        yield 'YELLOW - Get ready';
    }
}

const light = trafficLight();
setInterval(() => {
    console.log('Traffic light:', light.next().value);
}, 2000);

// B. GENERATOR FOR LAZY EVALUATION
function* lazyFilter(iterable, predicate) {
    for (const item of iterable) {
        if (predicate(item)) {
            yield item;
        }
    }
}

function* lazyMap(iterable, mapper) {
    for (const item of iterable) {
        yield mapper(item);
    }
}

// Chain lazy operations
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const result = lazyMap(
    lazyFilter(numbers, n => n % 2 === 0),
    n => n * 2
);

// Only processes when consumed
console.log([...result]); // [4, 8, 12, 16, 20]

// C. COROUTINES & COOPERATIVE MULTITASKING
function* task(id, max) {
    for (let i = 1; i <= max; i++) {
        console.log(`Task ${id}: Step ${i}`);
        yield; // Yield control
    }
    console.log(`Task ${id}: Complete`);
}

function scheduler(...tasks) {
    const generators = tasks.map(gen => gen());
    let active = generators.length;
    
    while (active > 0) {
        for (const gen of generators) {
            const { done } = gen.next();
            if (done) active--;
        }
    }
}

// Run tasks cooperatively
scheduler(
    () => task('A', 3),
    () => task('B', 2),
    () => task('C', 4)
);

// D. GENERATOR PIPELINES
function* pipeline(input, ...stages) {
    let value = input;
    for (const stage of stages) {
        value = yield* stage(value);
    }
    return value;
}

function* stage1(data) {
    for (const item of data) {
        yield item * 2;
    }
}

function* stage2(data) {
    for (const item of data) {
        if (item > 5) yield item;
    }
}

const data = [1, 2, 3, 4, 5];
const processed = pipeline(data, stage1, stage2);
console.log([...processed]); // [6, 8, 10]

// ====== 4. ASYNC ITERATORS & GENERATORS ======

// A. ASYNC GENERATOR
async function* asyncNumberGenerator(limit) {
    for (let i = 1; i <= limit; i++) {
        // Simulate async operation
        await new Promise(resolve => setTimeout(resolve, 100));
        yield i;
    }
}

// Using for await...of
(async () => {
    for await (const num of asyncNumberGenerator(5)) {
        console.log('Async number:', num);
    }
})();

// B. ASYNC ITERABLE OBJECT
class AsyncDataSource {
    constructor(data, delay = 100) {
        this.data = data;
        this.delay = delay;
    }
    
    async *[Symbol.asyncIterator]() {
        for (const item of this.data) {
            await new Promise(resolve => setTimeout(resolve, this.delay));
            yield item;
        }
    }
}

// C. ASYNC GENERATOR WITH ERROR HANDLING
async function* robustAsyncGenerator() {
    try {
        yield await fetchData(1);
        yield await fetchData(2);
        yield await fetchData(3);
    } catch (error) {
        console.error('Generator error:', error);
        yield 'Error occurred';
    } finally {
        console.log('Generator cleanup');
    }
}

// ====== 5. PRACTICAL APPLICATIONS ======

// A. PAGINATION WITH GENERATORS
async function* paginate(endpoint, pageSize = 10) {
    let page = 1;
    let hasMore = true;
    
    while (hasMore) {
        const response = await fetch(`${endpoint}?page=${page}&size=${pageSize}`);
        const data = await response.json();
        
        if (data.items.length === 0) {
            hasMore = false;
        } else {
            yield* data.items;
            page++;
        }
    }
}

// Usage
(async () => {
    for await (const item of paginate('/api/items')) {
        console.log('Paginated item:', item);
    }
})();

// B. WEB SOCKET MESSAGE STREAM
function createWebSocketStream(url) {
    return {
        [Symbol.asyncIterator]() {
            const socket = new WebSocket(url);
            const messageQueue = [];
            let resolveCurrent = null;
            
            socket.onmessage = (event) => {
                if (resolveCurrent) {
                    resolveCurrent({ value: event.data, done: false });
                    resolveCurrent = null;
                } else {
                    messageQueue.push(event.data);
                }
            };
            
            socket.onclose = () => {
                if (resolveCurrent) {
                    resolveCurrent({ value: undefined, done: true });
                }
            };
            
            return {
                async next() {
                    if (messageQueue.length > 0) {
                        return { value: messageQueue.shift(), done: false };
                    }
                    
                    return new Promise((resolve) => {
                        resolveCurrent = resolve;
                    });
                },
                
                async return() {
                    socket.close();
                    return { value: undefined, done: true };
                }
            };
        }
    };
}

// C. CUSTOM DATA STRUCTURE WITH ITERATION
class Matrix {
    constructor(rows, cols, fill = 0) {
        this.rows = rows;
        this.cols = cols;
        this.data = Array(rows).fill().map(() => Array(cols).fill(fill));
    }
    
    // Row-major iteration
    *[Symbol.iterator]() {
        for (let i = 0; i < this.rows; i++) {
            for (let j = 0; j < this.cols; j++) {
                yield this.data[i][j];
            }
        }
    }
    
    // Column-major iteration
    *columns() {
        for (let j = 0; j < this.cols; j++) {
            for (let i = 0; i < this.rows; i++) {
                yield this.data[i][j];
            }
        }
    }
    
    // Diagonal iteration
    *diagonal() {
        const size = Math.min(this.rows, this.cols);
        for (let i = 0; i < size; i++) {
            yield this.data[i][i];
        }
    }
    
    // Spiral iteration
    *spiral() {
        let top = 0, bottom = this.rows - 1;
        let left = 0, right = this.cols - 1;
        
        while (top <= bottom && left <= right) {
            // Top row
            for (let i = left; i <= right; i++) {
                yield this.data[top][i];
            }
            top++;
            
            // Right column
            for (let i = top; i <= bottom; i++) {
                yield this.data[i][right];
            }
            right--;
            
            // Bottom row
            if (top <= bottom) {
                for (let i = right; i >= left; i--) {
                    yield this.data[bottom][i];
                }
                bottom--;
            }
            
            // Left column
            if (left <= right) {
                for (let i = bottom; i >= top; i--) {
                    yield this.data[i][left];
                }
                left++;
            }
        }
    }
}

// ====== 6. COMBINING ITERATORS WITH OTHER FEATURES ======

// A. ITERATORS WITH DESTRUCTURING
function* pairs() {
    yield ['name', 'Alice'];
    yield ['age', 30];
    yield ['city', 'New York'];
}

const person = Object.fromEntries(pairs());
console.log(person); // { name: 'Alice', age: 30, city: 'New York' }

// B. ITERATORS WITH SPREAD OPERATOR
function* alphabet() {
    yield* 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
}

const letters = [...alphabet()];
console.log(letters); // Array of A-Z

// C. ITERATOR HELPERS (ESNext Proposal)
class EnhancedIterator {
    constructor(iterable) {
        this.iterable = iterable;
    }
    
    *map(transform) {
        for (const item of this.iterable) {
            yield transform(item);
        }
    }
    
    *filter(predicate) {
        for (const item of this.iterable) {
            if (predicate(item)) {
                yield item;
            }
        }
    }
    
    *take(limit) {
        let count = 0;
        for (const item of this.iterable) {
            if (count++ >= limit) break;
            yield item;
        }
    }
    
    *drop(count) {
        let skipped = 0;
        for (const item of this.iterable) {
            if (skipped++ < count) continue;
            yield item;
        }
    }
}

// ====== 7. PERFORMANCE OPTIMIZATIONS ======

// A. LAZY EVALUATION FOR MEMORY EFFICIENCY
function* generateLargeDataset(size) {
    for (let i = 0; i < size; i++) {
        // Generate data on demand, not all at once
        yield { id: i, value: Math.random() };
    }
}

// Process 1 million items without storing all in memory
const largeData = generateLargeDataset(1000000);
let processedCount = 0;

for (const item of largeData) {
    if (item.value > 0.9) {
        processedCount++;
        // Process item without loading all into memory
    }
    
    if (processedCount >= 1000) break;
}

// B. GENERATOR MEMOIZATION
function memoizedGenerator(createGenerator) {
    let cache = null;
    
    return function* (...args) {
        if (cache === null) {
            cache = [...createGenerator(...args)];
        }
        yield* cache;
    };
}

// ====== 8. ERROR HANDLING IN ITERATORS ======

// Generator with error handling
function* safeGenerator() {
    try {
        yield 1;
        throw new Error('Something went wrong');
        yield 2; // Never reached
    } catch (error) {
        console.error('Caught in generator:', error.message);
        yield 'Recovered value';
    } finally {
        console.log('Generator cleanup');
        yield 'Cleanup complete';
    }
}

// Iterator with error propagation
class RobustIterator {
    constructor(iterable) {
        this.iterator = iterable[Symbol.iterator]();
    }
    
    next() {
        try {
            return this.iterator.next();
        } catch (error) {
            return {
                value: `Error: ${error.message}`,
                done: true
            };
        }
    }
    
    [Symbol.iterator]() {
        return this;
    }
}

// ====== 9. REAL-WORLD EXAMPLE: LOG PARSER ======

async function* logParser(fileStream) {
    let buffer = '';
    
    for await (const chunk of fileStream) {
        buffer += chunk;
        const lines = buffer.split('\n');
        
        // Keep last incomplete line in buffer
        buffer = lines.pop() || '';
        
        for (const line of lines) {
            if (line.trim()) {
                const parsed = parseLogLine(line);
                if (parsed) yield parsed;
            }
        }
    }
    
    // Process remaining buffer
    if (buffer.trim()) {
        const parsed = parseLogLine(buffer);
        if (parsed) yield parsed;
    }
}

// ====== 10. COMPARISON & BEST PRACTICES ======

/*
ITERATORS:
✅ Full control over iteration logic
✅ Can implement complex traversal algorithms
✅ Memory efficient for large datasets
❌ More verbose to implement
❌ Manual state management

GENERATORS:
✅ Clean, readable syntax
✅ Automatic iterator implementation
✅ Natural for infinite sequences
✅ Two-way communication with yield
❌ Slightly less control than manual iterators
❌ Can be overused for simple cases

BEST PRACTICES:
1. Use generators for most custom iteration
2. Implement [Symbol.iterator] for custom collections
3. Consider async generators for streaming data
4. Use yield* for delegation to other iterables
5. Handle errors properly in generators
6. Clean up resources in finally blocks
7. Document iteration behavior of custom objects
8. Consider performance for large datasets

WHEN TO USE:
- Iterators: Complex traversal, custom data structures
- Generators: Simple iteration, async sequences, state machines
- Async Iterators: Streaming data, pagination, real-time updates
*/

// Final comprehensive example
class DataPipeline {
    constructor(source) {
        this.source = source;
    }
    
    async *execute() {
        // Step 1: Fetch data
        const rawData = yield* this.fetchData();
        
        // Step 2: Transform
        const transformed = yield* this.transform(rawData);
        
        // Step 3: Filter
        const filtered = yield* this.filter(transformed);
        
        // Step 4: Batch process
        yield* this.batchProcess(filtered);
    }
    
    async *fetchData() {
        // Implementation
    }
    
    async *transform(data) {
        // Implementation
    }
    
    async *filter(data) {
        // Implementation
    }
    
    async *batchProcess(data) {
        // Implementation
    }
}
```

### **Key Takeaways:**

1. **Iterators** provide a standardized way to traverse collections with the Iterator protocol
2. **Generators** simplify iterator creation with `function*` syntax and `yield` keyword
3. **Async Iterators/Generators** enable handling of asynchronous sequences with `for await...of`
4. **Lazy Evaluation** allows processing large datasets without loading everything into memory
5. **State Machines** can be elegantly modeled using generators
6. **Real-world applications** include pagination, data streaming, log parsing, and reactive programming
7. **Performance benefits** come from on-demand value generation and memory efficiency
8. **Modern JavaScript** integrates iterators with destructuring, spread operator, and other language features

Iterators and generators are powerful tools that enable efficient data processing, custom iteration patterns, and elegant solutions to complex problems in JavaScript.