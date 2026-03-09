# **Asynchronous JavaScript & Event Loop**

## **Core Concepts**

### **1. Asynchronous JavaScript**
- JavaScript is **single-threaded** but handles async operations
- **Non-blocking** - doesn't wait for slow operations to complete
- Uses **callback queue** and **event loop** to manage concurrency

### **2. Event Loop**
- **Runtime model** that executes code, collects/processes events
- **Phases**: Call Stack → Web APIs → Callback Queue → Event Loop → Call Stack
- Continuously checks if call stack is empty, then pushes callbacks from queue

### **3. Callbacks**
- Functions passed as arguments to be executed later
- **Problem**: Can lead to "Callback Hell" (nested, hard-to-read code)

### **4. Callback Hell**
- Deeply nested callbacks create "pyramid of doom"
- Hard to read, debug, and maintain
- Solution: **Promises** and **async/await**

### **5. Promises**
- Objects representing eventual completion/failure of async operation
- **States**: Pending → Fulfilled (resolved) / Rejected
- **Methods**: `.then()`, `.catch()`, `.finally()`
- Chaining eliminates nesting

### **6. async/await**
- Syntactic sugar over promises
- `async` functions always return promises
- `await` pauses execution until promise settles
- Makes async code look synchronous

### **7. setTimeout & setInterval**
- **setTimeout**: Execute code after delay (once)
- **setInterval**: Execute code repeatedly at intervals
- Both use **Web APIs** (browser environment)

## **Code Examples**

```javascript
// CALLBACKS
function fetchData(callback) {
    setTimeout(() => {
        callback('Data received');
    }, 1000);
}

// CALLBACK HELL (Pyramid of Doom)
fetchData((data1) => {
    process(data1, (result1) => {
        fetchMore(result1, (data2) => {
            processMore(data2, (result2) => {
                // ... nested deeper
            });
        });
    });
});

// PROMISES
const promise = new Promise((resolve, reject) => {
    setTimeout(() => {
        Math.random() > 0.5 
            ? resolve('Success!') 
            : reject('Error!');
    }, 1000);
});

promise
    .then(result => console.log(result))
    .catch(error => console.error(error))
    .finally(() => console.log('Done'));

// ASYNC/AWAIT
async function getData() {
    try {
        const result = await promise;
        console.log(result);
    } catch (error) {
        console.error(error);
    }
}

// TIMERS
setTimeout(() => console.log('After 2s'), 2000);
const intervalId = setInterval(() => console.log('Every 1s'), 1000);
clearInterval(intervalId); // Stop interval
```

## **Event Loop Visualization**
```
1. Synchronous code executes immediately (Call Stack)
2. Async operations (setTimeout, fetch) go to Web APIs
3. Web APIs complete → Callbacks go to Callback Queue (Microtask Queue for promises)
4. Event Loop checks: Is Call Stack empty?
5. If yes, moves callbacks from Queue to Stack
6. Repeat continuously
```

## **Key Points**
- **Microtask Queue** (Promises) has priority over **Callback Queue** (setTimeout)
- **Async code** never blocks the main thread
- **Promises/async-await** make async code more readable/maintainable
- **Event Loop** enables JavaScript's non-blocking nature despite being single-threaded