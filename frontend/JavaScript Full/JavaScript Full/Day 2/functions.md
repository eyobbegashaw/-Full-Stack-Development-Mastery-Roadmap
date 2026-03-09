# **JavaScript Functions: Complete Guide**

## **1. Function Basics (20%)**
Functions are reusable blocks of code that perform specific tasks.

```javascript
// Function Declaration (hoisted)
function greet(name) {
    return `Hello, ${name}!`;
}

// Function Expression
const greet = function(name) {
    return `Hello, ${name}!`;
};

// Arrow Function (ES6+)
const greet = (name) => `Hello, ${name}!`;
```

## **2. Function Parameters (15%)**

### **Default Parameters**
```javascript
function multiply(a, b = 1) {
    return a * b;
}
multiply(5);    // 5 (b defaults to 1)
multiply(5, 2); // 10
```

### **Rest Parameters (...args)**
```javascript
function sum(...numbers) {
    return numbers.reduce((total, num) => total + num, 0);
}
sum(1, 2, 3, 4); // 10
```

### **arguments Object (Legacy)**
```javascript
function legacySum() {
    let total = 0;
    for (let i = 0; i < arguments.length; i++) {
        total += arguments[i];
    }
    return total;
}
// Note: arguments is array-like, not a real array
```

## **3. Arrow Functions (15%)**
Shorter syntax, no own `this`, `arguments`, `super`, or `new.target`.

```javascript
// Regular function
const add = function(a, b) {
    return a + b;
};

// Arrow function equivalent
const add = (a, b) => a + b;

// Single parameter - no parentheses needed
const square = x => x * x;

// Multiple statements need braces
const process = (x, y) => {
    const sum = x + y;
    return sum * 2;
};

// Returning object literal needs parentheses
const createUser = (name, age) => ({ name, age });
```

## **4. Scope & Lexical Scoping (15%)**

### **Function Scope**
```javascript
function outer() {
    const outerVar = "I'm outside!";
    
    function inner() {
        const innerVar = "I'm inside!";
        console.log(outerVar); // Can access outerVar
    }
    
    console.log(innerVar); // ERROR: innerVar not accessible
}
```

### **Lexical Scoping**
Functions remember where they were created, not where they're called.

```javascript
function createCounter() {
    let count = 0; // Lexically scoped to createCounter
    
    return function() {
        count++; // Remembers count from parent scope
        return count;
    };
}

const counter = createCounter();
counter(); // 1
counter(); // 2
```

## **5. Closures (10%)**
Functions that remember their lexical scope even when executed outside it.

```javascript
function createGreeting(greeting) {
    return function(name) {
        return `${greeting}, ${name}!`;
    };
}

const sayHello = createGreeting("Hello");
const sayHi = createGreeting("Hi");

sayHello("John"); // "Hello, John!" (remembers "Hello")
sayHi("Jane");    // "Hi, Jane!" (remembers "Hi")
```

**Practical Closure Example:**
```javascript
function createPrivateCounter() {
    let count = 0; // Private variable
    
    return {
        increment: () => ++count,
        decrement: () => --count,
        getCount: () => count
    };
}

const counter = createPrivateCounter();
counter.increment(); // 1
counter.getCount();  // 1
// count is NOT accessible directly
```

## **6. IIFEs (Immediately Invoked Function Expressions) (5%)**
Functions that run immediately after definition.

```javascript
// Basic IIFE
(function() {
    console.log("Runs immediately!");
})();

// With parameters
(function(name) {
    console.log(`Hello, ${name}!`);
})("John");

// Arrow function IIFE
(() => {
    console.log("Arrow IIFE");
})();
```

**Modern Use Case (Module Pattern):**
```javascript
const calculator = (function() {
    let memory = 0;
    
    return {
        add: (a, b) => a + b,
        subtract: (a, b) => a - b,
        store: (value) => memory = value,
        recall: () => memory
    };
})();
```

## **7. Recursion (10%)**
Functions that call themselves.

```javascript
// Factorial: 5! = 5 × 4 × 3 × 2 × 1 = 120
function factorial(n) {
    if (n <= 1) return 1; // Base case
    return n * factorial(n - 1); // Recursive case
}

// Fibonacci sequence
function fibonacci(n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}
```

## **8. Function Stack (5%)**
JavaScript uses a call stack to manage function execution.

```javascript
function first() {
    console.log("First");
    second();
    console.log("Back to first");
}

function second() {
    console.log("Second");
    third();
    console.log("Back to second");
}

function third() {
    console.log("Third");
}

first();
// Output: First, Second, Third, Back to second, Back to first
```

## **9. Built-in Functions (5%)**
JavaScript provides many useful built-in functions.

```javascript
// Math functions
Math.max(1, 2, 3);    // 3
Math.random();        // Random number 0-1
Math.floor(4.7);      // 4

// String functions
"hello".toUpperCase(); // "HELLO"
"Hello World".split(" "); // ["Hello", "World"]

// Array functions
[1, 2, 3].map(x => x * 2); // [2, 4, 6]
[1, 2, 3].filter(x => x > 1); // [2, 3]
```

## **Best Practices**

1. **Use arrow functions** for callbacks and methods that don't need `this`
2. **Prefer default parameters** over conditional checks inside functions
3. **Use rest parameters** instead of `arguments` object
4. **Keep functions small and focused** (single responsibility)
5. **Use descriptive names** that indicate what the function does

```javascript
// Good example
function calculateTotalPrice(items, taxRate = 0.08) {
    const subtotal = items.reduce((sum, item) => sum + item.price, 0);
    return subtotal * (1 + taxRate);
}

// Use closure for data privacy
function createBankAccount(initialBalance) {
    let balance = initialBalance;
    
    return {
        deposit: (amount) => balance += amount,
        withdraw: (amount) => {
            if (amount <= balance) {
                balance -= amount;
                return amount;
            }
            return 0;
        },
        getBalance: () => balance
    };
}
```

## **Key Takeaways**
- **Functions are first-class citizens** (can be assigned, passed, returned)
- **Arrow functions** provide concise syntax but behave differently with `this`
- **Closures** enable data privacy and function factories
- **Recursion** solves problems by breaking them into smaller instances
- **Scope chain** determines variable accessibility
# **JavaScript Functions: Core Concepts & Notes**

## **Fundamental Concepts**

### **What Are Functions?**
Functions are reusable blocks of code that perform specific tasks. They are first-class citizens in JavaScript, meaning they can be:
- Assigned to variables
- Passed as arguments to other functions
- Returned from other functions
- Stored in data structures

### **Function Declaration vs Expression**
- **Function Declarations** are hoisted (available before declaration)
- **Function Expressions** are not hoisted, evaluated at runtime
- **Arrow Functions** provide concise syntax and lexical `this` binding

## **Parameters & Arguments**

### **Parameter Types**
1. **Regular Parameters**: Standard inputs to functions
2. **Default Parameters**: Provide fallback values when arguments are missing
3. **Rest Parameters (`...args`)**: Collect multiple arguments into an array
4. **Destructuring Parameters**: Extract values from objects/arrays directly in parameters

### **The `arguments` Object**
- Available in all non-arrow functions
- Array-like object containing all passed arguments
- Not available in arrow functions (use rest parameters instead)
- Legacy feature, rest parameters are preferred in modern code

## **Scope & Execution Context**

### **Lexical Scoping**
- Variables are accessible based on where they are declared in the source code
- Inner functions can access variables from outer functions
- Outer functions cannot access variables from inner functions
- Creates a scope chain that JavaScript traverses to find variables

### **Execution Context**
Every function invocation creates a new execution context containing:
- Variable environment (local variables, parameters)
- Reference to outer environment (scope chain)
- `this` binding (except arrow functions)

### **Function Stack (Call Stack)**
- LIFO (Last In, First Out) data structure
- Tracks function execution order
- Each function call adds a new frame to the stack
- When function completes, its frame is removed
- Stack overflow occurs with deep recursion

## **Closures - The Powerhouse Concept**

### **What Are Closures?**
A closure is a function that remembers and can access variables from its lexical scope even when that function is executed outside its original scope.

### **Closure Mechanics**
1. **Creation**: When a function is defined inside another function
2. **Memory**: Inner function maintains reference to outer function's variables
3. **Access**: Even after outer function returns, inner function can still access those variables

### **Practical Applications of Closures**
1. **Data Privacy/Encapsulation**: Create private variables
2. **Function Factories**: Generate specialized functions
3. **Partial Application & Currying**: Pre-fill some arguments
4. **Event Handlers & Callbacks**: Maintain state between events
5. **Memoization**: Cache expensive function results

## **Special Function Patterns**

### **IIFE (Immediately Invoked Function Expressions)**
- Functions that execute immediately after definition
- Creates a private scope without polluting global namespace
- Historically used for module patterns before ES6 modules
- Still useful for one-time execution and private data

### **Recursion**
- Functions that call themselves
- Must have a base case to prevent infinite loops
- Each recursive call creates new execution context
- Can be memory-intensive but elegant for certain problems
- Tail recursion optimization available in some engines

## **Arrow Functions Nuances**

### **Key Differences from Regular Functions**
1. **No `arguments` object**: Use rest parameters instead
2. **Lexical `this`**: Inherits `this` from surrounding scope
3. **Cannot be constructors**: No `new` keyword
4. **No `prototype` property**: Not for object creation
5. **Implicit return**: Single expressions return automatically

### **When to Use Arrow Functions**
- **Use**: Callbacks, array methods, lexical `this` binding
- **Avoid**: Object methods, constructors, prototype methods, event handlers needing dynamic `this`

## **Built-in Functions & Methods**

### **Global Functions**
- Type checking: `isNaN()`, `isFinite()`
- Parsing: `parseInt()`, `parseFloat()`
- Encoding: `encodeURI()`, `decodeURI()`
- Evaluation: `eval()` (avoid in production)

### **Function Object Properties**
All functions have:
- `name`: Function identifier
- `length`: Number of expected parameters
- `prototype`: For constructor functions
- Callable methods: `call()`, `apply()`, `bind()`

## **Memory Management Considerations**

### **Closures and Memory**
- Closures maintain references to outer variables
- Can prevent garbage collection of those variables
- Be mindful of memory leaks in long-lived closures
- Circular references can cause memory issues

### **Optimization Strategies**
1. Release unnecessary references in closures
2. Use weak references (`WeakMap`, `WeakSet`) when appropriate
3. Avoid creating closures in performance-critical loops
4. Consider function inlining for small, frequently called functions

## **Best Practices & Patterns**

### **Function Design Principles**
1. **Single Responsibility**: Each function should do one thing well
2. **Pure Functions**: Same input → same output, no side effects
3. **Descriptive Names**: Verb-noun combinations that reveal intent
4. **Parameter Limits**: 3 or fewer parameters ideally, use objects for more
5. **Avoid Side Effects**: Minimize external state mutations

### **Error Handling in Functions**
- Use `try...catch` for predictable errors
- Throw meaningful error objects
- Validate parameters at function entry
- Document expected behavior and edge cases

## **Modern JavaScript Features**

### **ES6+ Enhancements**
1. Default parameters for cleaner function calls
2. Rest parameters for variable argument handling
3. Arrow functions for concise syntax
4. Parameter destructuring for cleaner API design
5. Optional chaining and nullish coalescing in function bodies

### **Functional Programming Patterns**
- Higher-order functions (functions that take/return functions)
- Function composition (combining simple functions)
- Currying (transforming multi-argument functions)
- Immutability patterns through pure functions

## **Debugging & Testing Considerations**

### **Common Pitfalls**
1. Scope confusion with `var` vs `let/const`
2. `this` binding issues in callbacks
3. Memory leaks from unintended closures
4. Recursion depth limits
5. Hoisting misunderstandings

### **Testing Strategies**
1. Unit test pure functions independently
2. Mock dependencies for functions with side effects
3. Test edge cases and boundary conditions
4. Use function stubs and spies for complex interactions

## **Performance Implications**

### **Function Creation Costs**
- Function declarations are parsed at compile time
- Function expressions are evaluated at runtime
- Arrow functions have similar performance to function expressions
- Nested functions have creation overhead in loops

### **Optimization Techniques**
1. Cache frequently used function references
2. Avoid creating functions inside hot loops
3. Use function memoization for expensive calculations
4. Consider inline functions for performance-critical code

This comprehensive understanding of functions is crucial for writing clean, efficient, and maintainable JavaScript code. Functions are not just code organization tools but powerful constructs that enable advanced programming patterns and paradigms.
Mastering functions is essential for writing clean, modular, and maintainable JavaScript code!