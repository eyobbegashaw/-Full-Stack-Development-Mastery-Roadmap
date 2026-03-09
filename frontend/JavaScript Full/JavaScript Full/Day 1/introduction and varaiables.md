# **JavaScript: From Zero to Hero**

## **What is JavaScript?**
A programming language that makes web pages interactive. Originally created in 1995 to add simple animations, now powers complex web apps.

```javascript
// Your first JavaScript code:
console.log("Hello, JavaScript!");
```

## **How to Run JavaScript**
1. In browser: Press F12 → Console tab
2. In HTML: `<script> your code here </script>`
3. Using Node.js (for server-side)

## **History & Versions**
- 1995: Born as "Mocha" → "LiveScript" → "JavaScript"
- 2015: ES6 (Modern JavaScript) added major features
- Yearly updates since then (ES2016, ES2017, etc.)

## **Variable Declarations**
Three ways to create variables:

## **Variables – Storing Data**

### **Variable Declarations:**
- **`var`** – Old keyword, function-scoped, can be re-declared.
- **`let`** – Modern, block-scoped, can be updated but not re-declared in same scope.
- **`const`** – Modern, block-scoped, cannot be updated or re-declared.

---

## **Variable Naming Rules**
- Start with letter, `$`, or `_`
- Case-sensitive
- No reserved keywords (e.g., `let`, `if`)

---

## **Variable Scopes**
- **Global** – Accessible everywhere (declared outside functions/blocks).
- **Function** – Accessible only within the function.
- **Block** – Accessible only within `{}` (for `let` and `const`).


**From Basics to Advanced: Start with `const` and `let`, understand scope, then master hoisting & closures for advanced patterns.**
### **`var`** (The Old Way)
```javascript
var name = "John"; // Function-scoped
name = "Jane"; // Can be re-declared and updated
```

### **`let`** (The Modern Way)
```javascript
let age = 25; // Block-scoped
age = 26; // Can be updated but NOT re-declared
```

### **`const`** (For Constants)
```javascript
const pi = 3.14; // Block-scoped
// pi = 3.15; // ERROR: Cannot be updated or re-declared
```

## **Variable Naming Rules**
```javascript
let userName;     // ✓ Camel case (recommended)
let user_name;    // ✓ Snake case
let 1user;        // ✗ Cannot start with number
let user-name;    // ✗ No hyphens
let let;          // ✗ Cannot use reserved words
```

## **Variable Scopes**
Where your variable is accessible:

### **Global Scope** (Accessible everywhere)
```javascript
let globalVar = "I'm everywhere!";
```

### **Function Scope** (Inside function only)
```javascript
function myFunction() {
  var functionVar = "Inside function only";
  // Accessible only here
}
```

### **Block Scope** (Inside curly braces {})
```javascript
if (true) {
  let blockVar = "Inside block only";
  const anotherVar = "Me too!";
  // var here would leak outside!
}
```

## **Hoisting** (Weird JavaScript Behavior)
JavaScript "lifts" declarations to the top:

```javascript
console.log(x); // undefined (not error!)
var x = 5;

console.log(y); // ERROR!
let y = 10;
```

**Why?** `var` gets hoisted, `let`/`const` don't.

## **Quick Comparison Table**

| Feature        | `var`       | `let`       | `const`     |
|----------------|-------------|-------------|-------------|
| Scope          | Function    | Block       | Block       |
| Can be updated | Yes         | Yes         | No          |
| Can be re-declared | Yes   | No          | No          |
| Hoisted        | Yes         | No          | No          |

## **Best Practices Today**
1. **Use `const` by default**
2. **Use `let` when you need to change values**
3. **Avoid `var`** (legacy code only)
4. **Always declare variables** (prevents global leaks)

```javascript
// Good modern practice:
const userId = 12345; // Won't change
let isLoggedIn = false; // Might change
isLoggedIn = true; // Allowed
```

## **Remember:**
- **`var`** = Old, function-scoped, avoid in new code
- **`let`** = Modern, block-scoped, for changing values
- **`const`** = Modern, block-scoped, for constants
- Scope determines **where** you can use a variable
- Always declare variables before using them

JavaScript evolves, but these fundamentals remain key to writing clean, bug-free code!