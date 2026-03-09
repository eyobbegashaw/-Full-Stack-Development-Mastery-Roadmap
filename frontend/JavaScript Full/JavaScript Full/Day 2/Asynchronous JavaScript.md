# **Asynchronous JavaScript: From Callbacks to Async/Await**

## **Concept Overview (70%)**

### **The Core Challenge: JavaScript's Single Thread**
JavaScript is **single-threaded** and **synchronous** by nature, meaning it can only execute one operation at a time, in sequence. Asynchronous programming allows JavaScript to perform **non-blocking operations**, handling tasks like API calls, file I/O, or timers without freezing the entire application.

### **Key Concepts in Detail:**

#### **1. Event Loop**
The **Event Loop** is JavaScript's concurrency model that enables asynchronous behavior. It's a **continuous monitoring system** that manages the execution of code, collecting and processing events, and executing queued sub-tasks.

**How it works:**
- **Call Stack**: Where synchronous code executes (LIFO - Last In, First Out)
- **Web APIs/C++ APIs**: Browser/node.js APIs that handle async operations
- **Callback Queue/Task Queue**: Holds callbacks from completed async operations
- **Microtask Queue**: For promises (higher priority than callback queue)
- **Event Loop**: Continuously checks if call stack is empty, then moves tasks from queues

**Visual Flow:**
1. Sync code executes in call stack
2. Async operations are passed to Web APIs
3. When async completes, callback moves to queue
4. Event loop pushes queued callbacks to call stack when empty
5. Microtasks (promises) run before regular callbacks

#### **2. Callbacks**
**Callbacks** are functions passed as arguments to other functions, to be executed later (often after an async operation completes). They're the **fundamental building block** of async JavaScript.

**Characteristics:**
- First-class functions in JavaScript enable callback pattern
- Essential for handling async operations
- Can lead to **Callback Hell** when nested deeply
- Error handling requires convention (Error-first callbacks)

#### **3. Callback Hell (Pyramid of Doom)**
**Callback Hell** occurs when multiple nested callbacks create deeply indented, hard-to-read code. This happens because each async operation requires its own callback, leading to **pyramid-shaped code** that's difficult to maintain and debug.

**Problems with Callback Hell:**
- Difficult to read and follow logic
- Hard to handle errors properly
- Challenging to manage control flow
- Results in tightly coupled code

#### **4. Promises**
**Promises** are objects representing the eventual completion (or failure) of an async operation. They provide a cleaner alternative to callbacks with **chainable methods**.

**Promise States:**
- **Pending**: Initial state, neither fulfilled nor rejected
- **Fulfilled**: Operation completed successfully
- **Rejected**: Operation failed

**Promise Methods:**
- `.then()`: Handles successful fulfillment
- `.catch()`: Handles rejections/errors
- `.finally()`: Executes regardless of outcome
- `.all()`: Waits for all promises to complete
- `.race()`: Returns first settled promise
- `.any()`: Returns first fulfilled promise
- `.allSettled()`: Waits for all to settle (fulfilled or rejected)

#### **5. Async/Await**
**Async/Await** is syntactic sugar built on promises that makes asynchronous code look and behave more like synchronous code. It uses **`async` functions** (return promises) and **`await` expressions** (pause execution until promise settles).

**Benefits:**
- Cleaner, more readable code
- Structured error handling with try/catch
- Easier debugging (linear code flow)
- Better stack traces

#### **6. Timers: setTimeout & setInterval**
- **`setTimeout(callback, delay)`**: Executes callback once after specified delay (ms)
- **`setInterval(callback, interval)`**: Repeatedly executes callback at specified intervals
- **Clear with**: `clearTimeout(timerId)` or `clearInterval(timerId)`
- **Important**: Timers are **minimum delays**, not guaranteed exact times (blocked by long tasks)

### **Evolution Timeline:**
1. **Callbacks** (ES3): Basic async handling
2. **Promises** (ES6): Better chaining and error handling
3. **Async/Await** (ES2017): Synchronous-looking async code
4. **Promise combinators** (ES2020): `.allSettled()`, `.any()`

## **Code Explanation (30%)**

```javascript
// ====== 1. EVENT LOOP DEMONSTRATION ======
console.log('1: Start (Sync)');

setTimeout(() => {
    console.log('2: Timeout (Macrotask)');
}, 0);

Promise.resolve()
    .then(() => {
        console.log('3: Promise (Microtask)');
    });

console.log('4: End (Sync)');

// Output order:
// 1: Start (Sync)
// 4: End (Sync)
// 3: Promise (Microtask)  ← Microtasks run before macrotasks
// 2: Timeout (Macrotask)

// ====== 2. CALLBACKS ======

// Basic callback pattern
function fetchData(callback) {
    // Simulate async operation
    setTimeout(() => {
        const data = { id: 1, name: 'John' };
        // Error-first callback convention
        callback(null, data);
    }, 1000);
}

// Using the callback
fetchData((error, data) => {
    if (error) {
        console.error('Error:', error);
        return;
    }
    console.log('Data received:', data);
});

// ====== 3. CALLBACK HELL (PYRAMID OF DOOM) ======

// Nested callbacks become hard to read
function getUserData(userId, callback) {
    setTimeout(() => {
        console.log(`1. Fetching user ${userId}...`);
        callback(null, { id: userId, name: 'Alice' });
    }, 500);
}

function getPosts(user, callback) {
    setTimeout(() => {
        console.log(`2. Fetching posts for ${user.name}...`);
        callback(null, ['Post 1', 'Post 2']);
    }, 500);
}

function getComments(posts, callback) {
    setTimeout(() => {
        console.log(`3. Fetching comments for ${posts.length} posts...`);
        callback(null, ['Comment 1', 'Comment 2']);
    }, 500);
}

// CALLBACK HELL - Difficult to read and maintain
getUserData(1, (err1, user) => {
    if (err1) {
        console.error('Error fetching user:', err1);
        return;
    }
    
    getPosts(user, (err2, posts) => {
        if (err2) {
            console.error('Error fetching posts:', err2);
            return;
        }
        
        getComments(posts, (err3, comments) => {
            if (err3) {
                console.error('Error fetching comments:', err3);
                return;
            }
            
            console.log('Final result:', { user, posts, comments });
            // Imagine more nesting here...
        });
    });
});

// ====== 4. PROMISES (SOLUTION TO CALLBACK HELL) ======

// Creating promises
function getUserDataPromise(userId) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            console.log(`1. [Promise] Fetching user ${userId}...`);
            
            // Simulate random failure
            if (Math.random() < 0.1) {
                reject(new Error('Failed to fetch user'));
            } else {
                resolve({ id: userId, name: 'Alice' });
            }
        }, 500);
    });
}

function getPostsPromise(user) {
    return new Promise((resolve) => {
        setTimeout(() => {
            console.log(`2. [Promise] Fetching posts for ${user.name}...`);
            resolve(['Post 1', 'Post 2']);
        }, 500);
    });
}

function getCommentsPromise(posts) {
    return new Promise((resolve) => {
        setTimeout(() => {
            console.log(`3. [Promise] Fetching comments for ${posts.length} posts...`);
            resolve(['Comment 1', 'Comment 2']);
        }, 500);
    });
}

// PROMISE CHAINING - Flatter, more readable
getUserDataPromise(1)
    .then(user => {
        console.log('User received:', user);
        return getPostsPromise(user);
    })
    .then(posts => {
        console.log('Posts received:', posts);
        return getCommentsPromise(posts);
    })
    .then(comments => {
        console.log('Comments received:', comments);
        console.log('All data loaded successfully!');
        return { user: comments, posts: comments };
    })
    .catch(error => {
        console.error('Error in promise chain:', error);
        // Handle all errors from any step in the chain
    })
    .finally(() => {
        console.log('Promise chain completed (success or failure)');
    });

// PROMISE COMBINATORS (Advanced Patterns)
const promise1 = Promise.resolve('First');
const promise2 = new Promise(resolve => 
    setTimeout(() => resolve('Second'), 1000));
const promise3 = Promise.reject(new Error('Third failed'));

// Promise.all - ALL must succeed
Promise.all([promise1, promise2])
    .then(results => console.log('All succeeded:', results))
    .catch(error => console.error('All failed:', error));

// Promise.race - First to settle (success or failure)
Promise.race([promise1, promise2, promise3])
    .then(result => console.log('Race winner:', result))
    .catch(error => console.error('Race error:', error));

// Promise.allSettled - Wait for all, get status of each
Promise.allSettled([promise1, promise2, promise3])
    .then(results => {
        results.forEach((result, index) => {
            if (result.status === 'fulfilled') {
                console.log(`Promise ${index}:`, result.value);
            } else {
                console.log(`Promise ${index} failed:`, result.reason);
            }
        });
    });

// Promise.any - First to SUCCEED
Promise.any([promise3, promise1, promise2])
    .then(firstSuccess => console.log('First success:', firstSuccess))
    .catch(error => console.error('All promises rejected:', error));

// ====== 5. ASYNC/AWAIT (EVEN CLEANER) ======

// Async function always returns a promise
async function loadAllData(userId) {
    try {
        console.log('Starting async/await loading...');
        
        // Sequential execution (each waits for previous)
        const user = await getUserDataPromise(userId);
        console.log('User loaded:', user);
        
        const posts = await getPostsPromise(user);
        console.log('Posts loaded:', posts);
        
        const comments = await getCommentsPromise(posts);
        console.log('Comments loaded:', comments);
        
        return { user, posts, comments };
        
    } catch (error) {
        console.error('Error in async function:', error);
        throw error; // Re-throw for caller to handle
    } finally {
        console.log('Async function completed');
    }
}

// Using async/await
loadAllData(1)
    .then(data => console.log('Final data:', data))
    .catch(error => console.error('Failed to load:', error));

// Parallel execution with Promise.all + async/await
async function loadDataParallel(userId) {
    try {
        const userPromise = getUserDataPromise(userId);
        const postsPromise = getPostsPromise({ name: 'Default' });
        const commentsPromise = getCommentsPromise([]);
        
        // Execute all in parallel
        const [user, posts, comments] = await Promise.all([
            userPromise,
            postsPromise,
            commentsPromise
        ]);
        
        return { user, posts, comments };
    } catch (error) {
        console.error('Parallel loading failed:', error);
    }
}

// IIFE with async/await
(async () => {
    console.log('IIFE with async/await');
    const data = await loadDataParallel(2);
    console.log('Parallel data:', data);
})();

// ====== 6. TIMERS: setTimeout & setInterval ======

// setTimeout - execute once
console.log('Timer demo starting...');

const timeoutId = setTimeout(() => {
    console.log('This runs after 2 seconds');
}, 2000);

// Can cancel before it executes
// clearTimeout(timeoutId);

// setInterval - execute repeatedly
let counter = 0;
const intervalId = setInterval(() => {
    counter++;
    console.log(`Interval tick #${counter}`);
    
    if (counter >= 5) {
        clearInterval(intervalId);
        console.log('Interval stopped');
    }
}, 1000);

// setTimeout with recursion = more control than setInterval
function recursiveTimeout(callback, delay, maxCalls) {
    let calls = 0;
    
    function execute() {
        callback();
        calls++;
        
        if (calls < maxCalls) {
            setTimeout(execute, delay);
        }
    }
    
    setTimeout(execute, delay);
}

recursiveTimeout(() => {
    console.log('Recursive timeout tick');
}, 500, 3);

// ====== 7. REAL-WORLD EXAMPLE: API CALL WITH RETRY LOGIC ======

async function fetchWithRetry(url, maxRetries = 3) {
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
        try {
            console.log(`Attempt ${attempt}/${maxRetries}`);
            
            const response = await fetch(url);
            
            if (!response.ok) {
                throw new Error(`HTTP ${response.status}`);
            }
            
            const data = await response.json();
            return data;
            
        } catch (error) {
            console.error(`Attempt ${attempt} failed:`, error.message);
            
            if (attempt === maxRetries) {
                throw new Error(`Failed after ${maxRetries} attempts: ${error.message}`);
            }
            
            // Exponential backoff
            const delay = Math.pow(2, attempt) * 1000;
            console.log(`Waiting ${delay}ms before retry...`);
            await new Promise(resolve => setTimeout(resolve, delay));
        }
    }
}

// ====== 8. ADVANCED PATTERN: CONCURRENCY LIMITING ======

class TaskQueue {
    constructor(concurrency) {
        this.concurrency = concurrency;
        this.running = 0;
        this.queue = [];
    }
    
    addTask(task) {
        return new Promise((resolve, reject) => {
            this.queue.push({ task, resolve, reject });
            this.next();
        });
    }
    
    next() {
        while (this.running < this.concurrency && this.queue.length) {
            const { task, resolve, reject } = this.queue.shift();
            this.running++;
            
            task()
                .then(resolve)
                .catch(reject)
                .finally(() => {
                    this.running--;
                    this.next();
                });
        }
    }
}

// Example usage
const queue = new TaskQueue(2); // Max 2 concurrent tasks

async function simulateTask(id, duration) {
    return new Promise(resolve => {
        setTimeout(() => {
            console.log(`Task ${id} completed in ${duration}ms`);
            resolve(`Result ${id}`);
        }, duration);
    });
}

// Add 5 tasks to queue with concurrency limit of 2
for (let i = 1; i <= 5; i++) {
    queue.addTask(() => simulateTask(i, Math.random() * 2000));
}

// ====== 9. ERROR HANDLING COMPARISON ======

// Callback error handling (error-first pattern)
function callbackStyle(callback) {
    setTimeout(() => {
        const success = Math.random() > 0.5;
        if (success) {
            callback(null, 'Success');
        } else {
            callback(new Error('Failed'), null);
        }
    }, 100);
}

// Promise error handling
function promiseStyle() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            Math.random() > 0.5 ? resolve('Success') : reject(new Error('Failed'));
        }, 100);
    });
}

// Async/await error handling
async function asyncStyle() {
    try {
        const result = await promiseStyle();
        console.log('Async success:', result);
        return result;
    } catch (error) {
        console.error('Async error:', error);
        throw error;
    }
}

// ====== 10. RECAP & BEST PRACTICES ======

/*
EVOLUTION RECAP:
1. Callbacks → Basic async, leads to callback hell
2. Promises → Chainable, better error handling
3. Async/Await → Cleanest syntax, synchronous-looking

BEST PRACTICES:
1. Use async/await for most new code
2. Handle errors with try/catch in async functions
3. Use Promise.all for parallel independent operations
4. Be careful with mixing async/await and .then()
5. Understand event loop for performance optimization
6. Use microtasks (promises) for high-priority async work
7. Avoid blocking the event loop with long synchronous tasks
8. Consider using AbortController for cancelling async operations
*/

// Modern fetch with AbortController
async function fetchWithTimeout(url, timeout = 5000) {
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), timeout);
    
    try {
        const response = await fetch(url, { signal: controller.signal });
        clearTimeout(timeoutId);
        return await response.json();
    } catch (error) {
        clearTimeout(timeoutId);
        if (error.name === 'AbortError') {
            throw new Error(`Request timeout after ${timeout}ms`);
        }
        throw error;
    }
}
```

### **Key Takeaways:**

1. **Event Loop** manages async execution with call stack, queues, and Web APIs
2. **Callbacks** are foundational but lead to callback hell when nested
3. **Promises** provide cleaner chaining and error handling
4. **Async/Await** makes async code look synchronous and is easiest to read
5. **Timers** (`setTimeout`, `setInterval`) schedule future execution
6. **Error handling** improves with each evolution (callbacks → promises → async/await)
7. **Concurrency control** is possible with patterns like task queues
8. **Modern patterns** include retry logic, timeouts, and cancellation

This progression from callbacks to async/await represents JavaScript's evolution toward more maintainable, readable asynchronous code while maintaining backward compatibility.