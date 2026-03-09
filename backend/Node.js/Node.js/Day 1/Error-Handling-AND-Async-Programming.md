# **Error Handling in Node.js - Detailed Notes & Code**

## **Types of Errors**

### **System Errors**
System errors originate from the operating system or Node.js runtime environment, such as file system issues, network failures, or memory constraints. These errors have properties like `code`, `errno`, `syscall`, and `path`. They typically occur during I/O operations and inherit from the `Error` class. Common examples include `ENOENT` (file not found) or `ECONNREFUSED` (connection refused). Handling system errors is crucial for building robust applications that gracefully recover from external failures.

```javascript
const fs = require('fs');
fs.readFile('nonexistent.txt', (err, data) => {
  if (err && err.code === 'ENOENT') {
    console.log('File not found error:', err.message);
  }
});
```

### **User Specified Errors**
User-specified errors are custom errors created by developers to indicate application-specific failures. Use `new Error()` with custom messages or create custom error classes by extending the `Error` class. These errors help communicate specific failure conditions, provide meaningful messages, and enable targeted error handling. They're essential for API development, validation failures, and business logic errors.

```javascript
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
  }
}

function validateUser(user) {
  if (!user.email) {
    throw new ValidationError('Email is required', 'email');
  }
}
```

### **Assertion Errors**
Assertion errors occur when an assumption in code fails, typically used with Node.js's built-in `assert` module or testing frameworks. They validate conditions that should always be true during development. Assertions help catch programming errors early but are often disabled in production. The `assert` module throws `AssertionError` when conditions fail.

```javascript
const assert = require('assert');

function divide(a, b) {
  assert(b !== 0, 'Division by zero is not allowed');
  return a / b;
}

divide(10, 0); // Throws AssertionError
```

### **JavaScript Errors**
JavaScript errors are standard language errors like `SyntaxError`, `TypeError`, `ReferenceError`, and `RangeError`. These occur due to incorrect code, such as accessing undefined variables, calling non-functions, or syntax mistakes. They're part of the ECMAScript specification and help identify programming mistakes during development and execution.

```javascript
// TypeError example
const obj = null;
console.log(obj.property); // TypeError: Cannot read property 'property' of null

// ReferenceError example
console.log(undefinedVariable); // ReferenceError: undefinedVariable is not defined
```

## **Error Handling Concepts**

### **Uncaught Exceptions**
Uncaught exceptions are errors that bubble up to the top of the call stack without being caught by any `try...catch` block. In Node.js, they can crash the process unless handled globally using `process.on('uncaughtException')`. However, this should only be used for cleanup before exiting, as the application may be in an unstable state. Always prefer local error handling.

```javascript
// Global uncaught exception handler (use cautiously)
process.on('uncaughtException', (err) => {
  console.error('Uncaught Exception:', err.message);
  // Perform cleanup
  process.exit(1); // Exit after cleanup
});

// This error will trigger the handler
throw new Error('This is uncaught!');
```

### **Handling Async Errors**
Async error handling differs from synchronous code due to Node.js's non-blocking nature. For callbacks, errors are the first parameter. For Promises, use `.catch()` or `async/await` with `try...catch`. Unhandled promise rejections can be caught with `process.on('unhandledRejection')`. Proper async error handling prevents silent failures and ensures application stability.

```javascript
// Async/await with try-catch
async function fetchData() {
  try {
    const data = await someAsyncOperation();
    return data;
  } catch (err) {
    console.error('Async error:', err);
    throw err; // Re-throw or handle
  }
}

// Promise .catch()
somePromise()
  .then(data => console.log(data))
  .catch(err => console.error('Promise error:', err));
```

### **Callstack / Stack Trace**
The call stack (stack trace) shows the sequence of function calls leading to an error. It includes file paths, line numbers, and function names, helping debug by tracing error origins. Stack traces are automatically included in `Error` objects via `error.stack`. In production, consider sanitizing stack traces to avoid exposing sensitive information.

```javascript
function a() { b(); }
function b() { c(); }
function c() { throw new Error('Stack trace demo'); }

try {
  a();
} catch (err) {
  console.log('Stack trace:', err.stack);
  // Shows: Error: Stack trace demo
  //     at c (file.js:4:15)
  //     at b (file.js:3:15)
  //     at a (file.js:2:15)
}
```

### **Using Debugger**
Node.js includes a built-in debugger accessible via `node inspect` or using the `--inspect` flag with Chrome DevTools. Set breakpoints with `debugger` statement, step through code, inspect variables, and analyze the call stack. IDE integrations (VSCode, WebStorm) provide visual debugging interfaces. Debugging is essential for understanding error flow and complex issues.

```javascript
function problematicFunction() {
  const x = 10;
  const y = 0;
  debugger; // Execution pauses here in debugger
  return x / y; // Potential error
}

// Run with: node inspect app.js
// or: node --inspect app.js
```

## **Error Handling Patterns**

### **Centralized Error Handling**
Create a centralized error handler for consistency across your application. This pattern ensures uniform error responses, logging, and formatting, especially useful in web applications and APIs.

```javascript
// Error middleware for Express
app.use((err, req, res, next) => {
  console.error(err.stack);
  
  const statusCode = err.statusCode || 500;
  const message = err.message || 'Internal Server Error';
  
  res.status(statusCode).json({
    error: {
      message,
      status: statusCode
    }
  });
});
```

### **Error Wrapping**
Wrap lower-level errors with higher-level context to provide more meaningful error messages while preserving the original error chain.

```javascript
class DatabaseError extends Error {
  constructor(message, originalError) {
    super(message);
    this.name = 'DatabaseError';
    this.originalError = originalError;
  }
}

async function queryDatabase() {
  try {
    // Database operation
  } catch (err) {
    throw new DatabaseError('Failed to query database', err);
  }
}
```

### **Graceful Shutdown**
Implement graceful shutdown procedures to handle errors without data loss or corruption, especially in servers and long-running processes.

```javascript
process.on('SIGTERM', () => {
  console.log('SIGTERM received. Starting graceful shutdown...');
  server.close(() => {
    console.log('Server closed');
    process.exit(0);
  });
});
```

## **Best Practices Summary**

1. **Always handle errors locally** when possible
2. **Use specific error types** for different failure modes
3. **Log errors with context** (timestamp, user ID, request ID)
4. **Don't expose sensitive information** in error messages
5. **Use structured error responses** in APIs
6. **Implement retry logic** for transient failures
7. **Monitor uncaught exceptions** and promise rejections
8. **Test error scenarios** as part of your test suite

```javascript
// Comprehensive error handling example
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}

process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection at:', promise, 'reason:', reason);
});

process.on('uncaughtException', (err) => {
  console.error('Uncaught Exception:', err);
  if (!err.isOperational) {
    process.exit(1);
  }
});
```

Effective error handling makes applications more robust, maintainable, and user-friendly by providing clear failure messages and recovery paths.

# **Async Programming in Node.js - Detailed Notes & Code**

## **Async Programming Concepts**

### **Async Programming**
Asynchronous programming allows Node.js to perform non-blocking I/O operations, enabling high concurrency without threads. Instead of waiting for operations (file I/O, network requests, DB queries) to complete, Node.js continues executing other code. When async operations finish, their callbacks execute. This event-driven model is fundamental to Node.js's performance, allowing single-threaded servers to handle thousands of concurrent connections efficiently.

```javascript
console.log('Start');
setTimeout(() => console.log('Async complete'), 1000);
console.log('End');
// Output: Start, End, Async complete
```

### **Writing Async Code**
Writing async code involves managing operations that complete at unpredictable times. Patterns include callbacks, Promises, and async/await. Key principles: avoid callback hell, handle errors properly, understand execution order, and use async control flow libraries when needed. Always consider error propagation and resource cleanup in async operations.

```javascript
// Async function example
async function fetchData() {
  try {
    const data = await apiCall();
    const processed = await processData(data);
    return processed;
  } catch (error) {
    console.error('Failed:', error);
  }
}
```

## **Core Async Mechanisms**

### **Callbacks**
Callbacks are functions passed as arguments to be executed after an async operation completes. In Node.js, callback conventions use error-first pattern: `(error, result) => {}`. This pattern ensures errors are always handled. Callbacks were the original async pattern in Node.js but can lead to "callback hell" (deep nesting) when managing multiple async operations.

```javascript
const fs = require('fs');
fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) {
    console.error('Error reading:', err);
    return;
  }
  console.log('File content:', data);
});
```

### **Promises**
Promises represent eventual completion (or failure) of an async operation and its resulting value. States: pending, fulfilled, rejected. Methods: `.then()` for success, `.catch()` for errors, `.finally()` for cleanup. Promises chain nicely, avoiding callback nesting. They're the foundation for async/await and modern async patterns in JavaScript.

```javascript
function fetchUser(id) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (id > 0) resolve({ id, name: 'John' });
      else reject(new Error('Invalid ID'));
    }, 1000);
  });
}

fetchUser(1)
  .then(user => console.log('User:', user))
  .catch(err => console.error('Error:', err));
```

### **async/await**
async/await is syntactic sugar over Promises, making async code look synchronous. `async` functions always return a Promise. `await` pauses execution until the Promise resolves. Use `try/catch` for error handling. This pattern improves readability and error handling compared to Promise chains or callbacks.

```javascript
async function getUserData(userId) {
  try {
    const user = await fetchUser(userId);
    const posts = await fetchPosts(user.id);
    return { user, posts };
  } catch (error) {
    console.error('Failed to fetch data:', error);
    throw error;
  }
}
```

## **Timing Functions**

### **setTimeout**
`setTimeout()` schedules a function to run after a specified delay (in milliseconds). Returns a `Timeout` object for cancellation with `clearTimeout()`. The delay is minimum, not guaranteed - Node.js may execute later if the event loop is busy. Used for delaying execution, debouncing, or timeouts.

```javascript
console.log('Before timeout');
const timerId = setTimeout(() => {
  console.log('Executed after 2 seconds');
}, 2000);
// clearTimeout(timerId); // Cancel if needed
```

### **setInterval**
`setInterval()` repeatedly executes a function at specified intervals. Returns an `Interval` object for stopping with `clearInterval()`. Be cautious of interval drift and overlapping executions. Useful for periodic tasks like polling, animations, or cleanup.

```javascript
let counter = 0;
const intervalId = setInterval(() => {
  counter++;
  console.log(`Tick ${counter}`);
  if (counter >= 5) clearInterval(intervalId);
}, 1000);
```

### **setImmediate**
`setImmediate()` executes a callback after the current poll phase completes, during the check phase. It's similar to `setTimeout(callback, 0)` but more efficient. Useful for deferring execution to avoid blocking the event loop, especially in I/O-heavy operations.

```javascript
console.log('Start');
setImmediate(() => {
  console.log('setImmediate callback');
});
console.log('End');
// Output: Start, End, setImmediate callback
```

### **process.nextTick**
`process.nextTick()` executes a callback at the end of the current operation, before the next event loop tick. Higher priority than `setImmediate()`. Can lead to I/O starvation if misused (recursive `nextTick()` calls). Used for API design to ensure callbacks are async, or for breaking up large operations.

```javascript
console.log('Start');
process.nextTick(() => {
  console.log('nextTick callback');
});
console.log('End');
// Output: Start, End, nextTick callback
```

## **Event System**

### **Event Emitter**
EventEmitter is Node.js's implementation of the observer pattern for handling events. Objects emit named events that cause listeners (callbacks) to be called. Core module for many Node.js APIs (streams, HTTP server). Methods: `on()`/`addListener()`, `emit()`, `once()`, `removeListener()`. Essential for decoupled, event-driven architectures.

```javascript
const EventEmitter = require('events');
class MyEmitter extends EventEmitter {}

const emitter = new MyEmitter();
emitter.on('data', (chunk) => {
  console.log('Received:', chunk);
});
emitter.emit('data', 'Hello World');
```

### **Event Loop**
The event loop is Node.js's runtime model that handles async operations. Phases: timers (`setTimeout`, `setInterval`), pending callbacks, poll (I/O), check (`setImmediate`), close callbacks. The loop processes one phase at a time, executing callbacks from that phase's queue. Understanding the loop helps optimize performance and avoid blocking.

```javascript
// Demonstrating event loop phases
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
process.nextTick(() => console.log('nextTick'));
// Output: nextTick, timeout, immediate (or timeout/immediate order may vary)
```

## **Async Patterns & Best Practices**

### **Error Handling Patterns**
```javascript
// Promise error handling
asyncOperation()
  .then(result => processResult(result))
  .catch(error => handleError(error))
  .finally(() => cleanup());

// Async/await with try-catch
async function process() {
  try {
    const a = await step1();
    const b = await step2(a);
    return await step3(b);
  } catch (error) {
    console.error('Process failed:', error);
    throw error;
  }
}
```

### **Controlling Execution Order**
```javascript
// Sequential execution
async function sequential() {
  const result1 = await task1();
  const result2 = await task2(result1);
  return result2;
}

// Parallel execution
async function parallel() {
  const [res1, res2] = await Promise.all([task1(), task2()]);
  return { res1, res2 };
}

// Race conditions
async function race() {
  const winner = await Promise.race([fastTask(), slowTask()]);
  return winner;
}
```

### **Avoiding Common Pitfalls**
```javascript
// DON'T: Forget error handling in promises
somePromise().then(data => console.log(data)); // Unhandled rejection!

// DO: Always handle errors
somePromise()
  .then(data => console.log(data))
  .catch(err => console.error('Error:', err));

// DON'T: Block the event loop
function blockLoop() {
  // CPU-intensive sync operation
  for (let i = 0; i < 1e9; i++) {}
}

// DO: Use worker threads or break up work
async function nonBlocking() {
  // Use setImmediate or nextTick to yield
}
```

### **Event Loop Priority Example**
```javascript
// Understanding execution order
console.log('1: Start');

process.nextTick(() => console.log('2: nextTick'));

Promise.resolve().then(() => console.log('3: Promise'));

setTimeout(() => console.log('4: setTimeout 0'), 0);

setImmediate(() => console.log('5: setImmediate'));

console.log('6: End');

/* Typical output:
1: Start
6: End
2: nextTick
3: Promise
4: setTimeout 0
5: setImmediate
*/
```

## **Key Takeaways**

1. **Callbacks** are fundamental but can lead to nesting hell
2. **Promises** provide cleaner chaining and error handling
3. **async/await** makes async code more readable
4. **Timing functions** have different priorities: `nextTick` > Promises > `setTimeout(0)` ≈ `setImmediate`
5. **EventEmitter** enables publish-subscribe patterns
6. **Event loop** understanding prevents performance issues
7. **Always handle errors** in async code
8. **Avoid blocking** the event loop with sync operations

Mastering async programming is essential for building efficient, scalable Node.js applications that leverage its non-blocking I/O model effectively.