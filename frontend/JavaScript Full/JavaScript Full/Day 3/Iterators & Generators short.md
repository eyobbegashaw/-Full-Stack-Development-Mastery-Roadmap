# **Iterators & Generators - Complete Guide**

## **Concept Overview (70%)**

### **Iterators**

**Iterators** are objects that implement the **Iterator Protocol** - a standardized way to produce sequences of values. They provide a consistent interface for looping over data structures.

#### **Core Concepts:**
- **Iterator Protocol**: Any object with a `next()` method that returns `{ value, done }`
- **Iterable Protocol**: Objects that have `[Symbol.iterator]()` method returning an iterator
- **Built-in Iterables**: Arrays, Strings, Maps, Sets, NodeLists, arguments object
- **Consumption**: Can be consumed by `for...of` loops, spread operator, destructuring

#### **Iterator Flow:**
1. Get iterator via `[Symbol.iterator]()` method
2. Repeatedly call `.next()` until `done: true`
3. `value` contains the current item, `done` indicates completion

### **Generators**

**Generators** are special functions that can pause and resume execution. They simplify iterator creation by providing syntactic sugar for the iterator protocol.

#### **Key Features:**
- Created with `function*` syntax
- Use `yield` to pause execution and return a value
- `yield*` delegates to another generator/iterable
- Maintain state between calls
- Can be both **iterators** and **iterables**

#### **Generator Methods:**
- `.next(value)` - Resume execution, optionally pass value back
- `.return(value)` - Terminate generator
- `.throw(error)` - Throw error into generator

## **Code Explanation (30%)**

```javascript
// ====== ITERATORS ======

// 1. Custom Iterator
const customIterator = {
    data: [1, 2, 3, 4, 5],
    index: 0,
    
    next() {
        return this.index < this.data.length 
            ? { value: this.data[this.index++], done: false }
            : { value: undefined, done: true };
    },
    
    [Symbol.iterator]() {
        // Reset index for reusability
        this.index = 0;
        return this;
    }
};

// Using the iterator
const iterator = customIterator[Symbol.iterator]();
console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next()); // { value: 2, done: false }

// for...of loop works because we implemented [Symbol.iterator]
for (const num of customIterator) {
    console.log(num); // 1, 2, 3, 4, 5
}

// 2. Built-in Iterators
const arr = ['a', 'b', 'c'];
const arrIterator = arr[Symbol.iterator]();

console.log(arrIterator.next()); // { value: 'a', done: false }

// 3. Spread operator uses iterators
console.log([...customIterator]); // [1, 2, 3, 4, 5]

// 4. Infinite Iterator
const infiniteCounter = {
    current: 0,
    next() {
        return { value: this.current++, done: false };
    },
    [Symbol.iterator]() { return this; }
};

// Can't use for...of (infinite!), but can manually call .next()
console.log(infiniteCounter.next().value); // 0
console.log(infiniteCounter.next().value); // 1

// ====== GENERATORS ======

// 1. Basic Generator
function* numberGenerator() {
    yield 1;
    yield 2;
    yield 3;
    return 4; // return ends generator (value appears in last next() call)
}

const gen = numberGenerator();
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 3, done: false }
console.log(gen.next()); // { value: 4, done: true }

// 2. Generator as Iterable
for (const num of numberGenerator()) {
    console.log(num); // 1, 2, 3 (doesn't include 4 from return!)
}

// 3. Passing values into generator
function* twoWayGenerator() {
    const name = yield "What's your name?";
    const age = yield `Hello ${name}, how old are you?`;
    yield `You are ${age} years old.`;
}

const chat = twoWayGenerator();
console.log(chat.next().value);           // "What's your name?"
console.log(chat.next("Alice").value);    // "Hello Alice, how old are you?"
console.log(chat.next(30).value);         // "You are 30 years old."

// 4. yield* for delegation
function* innerGenerator() {
    yield 2;
    yield 3;
}

function* outerGenerator() {
    yield 1;
    yield* innerGenerator(); // delegate to another generator
    yield 4;
}

console.log([...outerGenerator()]); // [1, 2, 3, 4]

// 5. Generator with try/catch
function* errorHandlingGenerator() {
    try {
        yield 'Start';
        yield 'Middle';
        throw new Error('Generator error!');
    } catch (err) {
        yield `Caught: ${err.message}`;
    }
    yield 'End';
}

const errorGen = errorHandlingGenerator();
console.log(errorGen.next().value); // 'Start'
console.log(errorGen.next().value); // 'Middle'
console.log(errorGen.next().value); // 'Caught: Generator error!'
console.log(errorGen.next().value); // 'End'

// ====== PRACTICAL EXAMPLES ======

// 1. Lazy Evaluation (Process large datasets efficiently)
function* fibonacci(limit) {
    let a = 0, b = 1;
    for (let i = 0; i < limit; i++) {
        yield a;
        [a, b] = [b, a + b];
    }
}

// Only computes as needed
for (const num of fibonacci(10)) {
    if (num > 50) break;
    console.log(num); // 0, 1, 1, 2, 3, 5, 8, 13, 21, 34
}

// 2. Async Generator (ES2018)
async function* asyncGenerator() {
    const urls = ['/api/data1', '/api/data2', '/api/data3'];
    
    for (const url of urls) {
        const response = await fetch(url);
        const data = await response.json();
        yield data;
    }
}

// Consume async generator
(async () => {
    for await (const data of asyncGenerator()) {
        console.log('Received:', data);
    }
})();

// 3. State Machine
function* trafficLight() {
    while (true) {
        yield 'RED';
        yield 'GREEN';
        yield 'YELLOW';
    }
}

const light = trafficLight();
setInterval(() => {
    console.log('Light is:', light.next().value);
}, 2000);

// 4. Custom Range Generator
function* range(start, end, step = 1) {
    for (let i = start; i <= end; i += step) {
        yield i;
    }
}

console.log([...range(1, 5)]); // [1, 2, 3, 4, 5]
console.log([...range(0, 10, 2)]); // [0, 2, 4, 6, 8, 10]

// 5. Pipeline Processing
function* filter(iterable, predicate) {
    for (const item of iterable) {
        if (predicate(item)) {
            yield item;
        }
    }
}

function* map(iterable, transform) {
    for (const item of iterable) {
        yield transform(item);
    }
}

const numbers = [1, 2, 3, 4, 5];
const result = map(
    filter(numbers, n => n % 2 === 0),
    n => n * 2
);

console.log([...result]); // [4, 8]

// ====== ADVANCED PATTERNS ======

// 1. Generator with return() and throw()
function* advancedGenerator() {
    try {
        yield 'Step 1';
        yield 'Step 2';
        yield 'Step 3';
    } finally {
        console.log('Cleaning up...');
    }
}

const adv = advancedGenerator();
console.log(adv.next().value); // 'Step 1'
console.log(adv.return('Early exit').value); // 'Early exit'
// Output: 'Cleaning up...'

// 2. Coroutine Pattern
function coroutine(generatorFunction) {
    return function(...args) {
        const generator = generatorFunction(...args);
        generator.next(); // Initialize
        return generator;
    };
}

const processData = coroutine(function* () {
    let data;
    while (true) {
        data = yield; // Receive data
        console.log('Processing:', data * 2);
    }
});

const processor = processData();
processor.next(5); // Initialize
processor.next(10); // Processing: 20
processor.next(20); // Processing: 40

// 3. Observable-like Pattern
function createObservable(subscriber) {
    return {
        subscribe(observer) {
            const generator = subscriber(observer);
            generator.next(); // Start
            return {
                unsubscribe() {
                    generator.return();
                }
            };
        }
    };
}

const observable = createObservable(function*(observer) {
    let count = 0;
    const interval = setInterval(() => {
        observer.next(count++);
    }, 1000);
    
    try {
        yield; // Pause
    } finally {
        clearInterval(interval);
    }
});

const subscription = observable.subscribe({
    next(value) { console.log('Value:', value); }
});

setTimeout(() => subscription.unsubscribe(), 5000);

// ====== KEY DIFFERENCES ======

/*
ITERATORS:
- Manual implementation
- Need to manage state manually
- More control
- Verbose for complex sequences

GENERATORS:
- Syntactic sugar for iterators
- Automatic state management
- Can pause/resume
- Can receive values
- Cleaner syntax
- Built-in error handling
*/

// ====== WHEN TO USE ======

/*
USE ITERATORS WHEN:
- You need maximum control
- Implementing simple sequences
- Working with legacy code
- Performance is critical

USE GENERATORS WHEN:
- Creating complex sequences
- Implementing state machines
- Handling async operations (async generators)
- Creating data pipelines
- Need two-way communication
*/

// ====== REAL-WORLD EXAMPLE: PAGINATION ======

async function* paginatedAPI(endpoint, pageSize = 10) {
    let page = 1;
    let hasMore = true;
    
    while (hasMore) {
        const response = await fetch(`${endpoint}?page=${page}&limit=${pageSize}`);
        const data = await response.json();
        
        if (data.items.length === 0) {
            hasMore = false;
        } else {
            for (const item of data.items) {
                yield item;
            }
            page++;
        }
    }
}

// Usage
(async () => {
    for await (const item of paginatedAPI('/api/products')) {
        console.log('Product:', item);
        // Process items as they arrive, memory efficient
    }
})();

// ====== SUMMARY ======
/*
Iterators and Generators are powerful tools for:
1. Creating custom sequences
2. Processing data lazily
3. Managing stateful operations
4. Implementing complex control flows
5. Handling async operations elegantly
6. Creating reusable data processing pipelines

Key Benefits:
- Memory efficiency (process on-demand)
- Clean separation of concerns
- Reusable patterns
- Better control flow
- Simplified async code with async generators
*/
```

### **Key Takeaways:**

1. **Iterators** provide a standard way to traverse data structures
2. **Generators** simplify iterator creation with `function*` and `yield`
3. **for...of** and **spread operator** automatically use iterators
4. **Generators can pause/resume**, making them ideal for async operations
5. **yield*** delegates to other generators/iterables
6. **Async generators** combine generators with async/await for streaming data
7. **Practical uses**: lazy loading, pagination, state machines, data pipelines

This combination enables powerful patterns for efficient data processing and complex control flows in JavaScript.