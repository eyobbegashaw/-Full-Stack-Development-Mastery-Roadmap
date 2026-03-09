# **JavaScript: Operators & Control Flow**

## **Expressions & Operators (50% Concept / 50% Code)**

### **1. Arithmetic Operators**
```javascript
// Basic math
let sum = 10 + 5;      // 15 (addition)
let diff = 10 - 5;     // 5 (subtraction)
let product = 10 * 5;  // 50 (multiplication)
let quotient = 10 / 5; // 2 (division)
let remainder = 10 % 3; // 1 (modulus)
let power = 2 ** 3;    // 8 (exponentiation)

// Increment/Decrement
let count = 5;
count++;    // 6 (post-increment)
++count;    // 7 (pre-increment)
count--;    // 6 (post-decrement)
--count;    // 5 (pre-decrement)
```

### **2. Assignment Operators**
```javascript
let x = 10;    // Basic assignment

x += 5;        // x = x + 5 → 15
x -= 3;        // x = x - 3 → 12
x *= 2;        // x = x * 2 → 24
x /= 4;        // x = x / 4 → 6
x %= 4;        // x = x % 4 → 2
x **= 3;       // x = x ** 3 → 8
```

### **3. Comparison Operators**
```javascript
// Return boolean values
5 == 5;        // true (loose equality)
5 === 5;       // true (strict equality)
5 != "5";      // false (loose inequality)
5 !== "5";     // true (strict inequality)
10 > 5;        // true (greater than)
10 >= 10;      // true (greater than or equal)
5 < 10;        // true (less than)
5 <= 5;        // true (less than or equal)
```

### **4. Logical Operators**
```javascript
// AND (&&) - Both must be true
true && true;    // true
true && false;   // false

// OR (||) - At least one true
true || false;   // true
false || false;  // false

// NOT (!) - Inverts boolean
!true;           // false
!!"hello";       // true (double NOT converts to boolean)

// Short-circuit evaluation
let name = user && user.name;  // Safe property access
let value = input || "default"; // Fallback value
```

### **5. Bitwise Operators**
```javascript
// Work with binary representations
5 & 1;    // 1 (AND: 0101 & 0001 = 0001)
5 | 1;    // 5 (OR: 0101 | 0001 = 0101)
5 ^ 1;    // 4 (XOR: 0101 ^ 0001 = 0100)
~5;       // -6 (NOT: ~0101 = 1010)
5 << 1;   // 10 (Left shift: 0101 → 1010)
5 >> 1;   // 2 (Right shift: 0101 → 0010)
5 >>> 1;  // 2 (Zero-fill right shift)
```

### **6. Unary Operators**
```javascript
let x = 5;
+x;           // 5 (convert to number)
-"5";         // -5 (convert to negative number)
!"hello";     // false (convert to boolean)
typeof 42;    // "number"
void 0;       // undefined
delete obj.prop; // remove property
```

### **7. Conditional (Ternary) Operator**
```javascript
// Short if-else statement
let age = 18;
let status = age >= 18 ? "Adult" : "Minor";
// Same as:
if (age >= 18) {
    status = "Adult";
} else {
    status = "Minor";
}
```

### **8. Comma Operator**
```javascript
// Evaluates multiple expressions, returns last one
let a = (1, 2, 3); // a = 3

// Useful in for loops
for (let i = 0, j = 10; i < j; i++, j--) {
    console.log(i, j);
}
```

### **9. String Operators**
```javascript
// Concatenation
let greeting = "Hello" + " " + "World"; // "Hello World"
let name = "John";
let message = "Hello, " + name + "!";   // "Hello, John!"

// Compound assignment
let str = "Hello";
str += " World"; // "Hello World"
```

### **10. BigInt Operators**
```javascript
// Special operators for BigInt (use n suffix)
let big = 12345678901234567890n;
let result = big + 1n;    // Must use n with n
let product = big * 2n;   // Works with BigInt
```

## **Control Flow (50% Concept / 50% Code)**

### **1. Conditional Statements (if...else)**
```javascript
// Basic if
if (score >= 60) {
    console.log("Pass");
}

// if...else
if (score >= 60) {
    console.log("Pass");
} else {
    console.log("Fail");
}

// if...else if...else
if (score >= 90) {
    grade = "A";
} else if (score >= 80) {
    grade = "B";
} else if (score >= 70) {
    grade = "C";
} else {
    grade = "F";
}
```

### **2. Switch Statement**
```javascript
// Alternative to multiple if...else
let day = 3;
let dayName;

switch (day) {
    case 1:
        dayName = "Monday";
        break;
    case 2:
        dayName = "Tuesday";
        break;
    case 3:
        dayName = "Wednesday";
        break;
    default:
        dayName = "Unknown";
        break;
}
```

### **3. Exceptional Handling (try/catch/finally)**
```javascript
// Basic error handling
try {
    // Code that might throw an error
    let result = riskyOperation();
    console.log(result);
} catch (error) {
    // Handle the error
    console.error("Something went wrong:", error.message);
} finally {
    // Always executes (cleanup code)
    console.log("Operation complete");
}
```

### **4. throw Statement**
```javascript
// Throw custom errors
function divide(a, b) {
    if (b === 0) {
        throw new Error("Cannot divide by zero");
    }
    return a / b;
}

// Throwing different error types
throw new Error("Generic error");
throw new TypeError("Invalid type");
throw new RangeError("Value out of range");
throw new SyntaxError("Invalid syntax");
```

### **5. Error Objects**
```javascript
// Built-in error types
new Error("Message");
new SyntaxError("Invalid syntax");
new ReferenceError("Undefined variable");
new TypeError("Wrong type");
new RangeError("Out of range");
new URIError("Bad URI");
new EvalError("Eval error");

// Custom error
class ValidationError extends Error {
    constructor(message) {
        super(message);
        this.name = "ValidationError";
    }
}
```

## **Operator Precedence (Concept)**
Operators are evaluated in this order (highest to lowest):
1. **Grouping**: `()`
2. **Member Access**: `.`, `[]`, `new` (with args)
3. **Function Call**: `()`, `new` (no args)
4. **Postfix**: `++`, `--`
5. **Prefix**: `!`, `~`, `+`, `-`, `++`, `--`, `typeof`, `void`, `delete`
6. **Exponential**: `**`
7. **Multiplicative**: `*`, `/`, `%`
8. **Additive**: `+`, `-`
9. **Bitwise Shift**: `<<`, `>>`, `>>>`
10. **Relational**: `<`, `<=`, `>`, `>=`, `in`, `instanceof`
11. **Equality**: `==`, `!=`, `===`, `!==`
12. **Bitwise AND**: `&`
13. **Bitwise XOR**: `^`
14. **Bitwise OR**: `|`
15. **Logical AND**: `&&`
16. **Logical OR**: `||`
17. **Conditional**: `?:`
18. **Assignment**: `=`, `+=`, `-=`, etc.
19. **Comma**: `,`

## **Best Practices**

```javascript
// 1. Use strict equality (===) by default
if (value === null) { /* ... */ }

// 2. Use descriptive variable names
let isUserLoggedIn = true; // ✓
let flag = true;           // ✗

// 3. Handle errors gracefully
try {
    processUserInput(input);
} catch (error) {
    logError(error);
    showUserFriendlyMessage();
}

// 4. Use ternary for simple conditions
let message = isValid ? "Valid" : "Invalid";

// 5. Avoid deep nesting
// Instead of:
if (condition1) {
    if (condition2) {
        if (condition3) {
            // do something
        }
    }
}

// Use:
if (!condition1) return;
if (!condition2) return;
if (!condition3) return;
// do something
```

## **Key Takeaways**
1. **Operators** transform and combine values
2. **Control flow** determines execution path
3. **Error handling** prevents crashes
4. **Operator precedence** matters in complex expressions
5. Choose the right operator/statement for each task

Understanding these fundamentals is crucial for writing clean, efficient, and maintainable JavaScript code!