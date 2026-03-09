# **JavaScript: Equality Comparisons & Loops/Iterations**

## **Equality Comparisons (70% Concept / 30% Code)**

### **Three Main Equality Algorithms**

#### **1. SameValue (Most Strict)**
- Used by: `Object.is()`
- **Never does type coercion**
- Treats `NaN` as equal to `NaN`
- Treats `-0` as different from `+0`

```javascript
Object.is(NaN, NaN);      // true
Object.is(0, -0);         // false
Object.is("5", 5);        // false (no coercion)
```

#### **2. SameValueZero (Strict but Practical)**
- Used by: `Set`, `Map`, `Array.prototype.includes()`
- Like SameValue but `-0 === +0`
- **No type coercion**

```javascript
let set = new Set();
set.add(0);
set.has(-0);              // true
[NaN].includes(NaN);      // true
```

#### **3. isStrictlyEqual (===)**
- Used by: `===` and `!==` operators
- **No type coercion**
- `NaN !== NaN` (different from SameValue)
- `-0 === +0`

```javascript
5 === 5;                  // true
5 === "5";                // false
NaN === NaN;              // false (watch out!)
0 === -0;                 // true
```

#### **4. isLooselyEqual (==)**
- Used by: `==` and `!=` operators
- **Does type coercion** (can be surprising!)
- Complex rules for conversion

```javascript
5 == "5";                 // true (string → number)
"true" == true;          // false (both → number)
null == undefined;       // true (special case)
```

### **Equality Comparison Summary Table**

```javascript
// Quick Reference
Value               ==       ===      Object.is()
"" == false         true     false    false
0 == "0"            true     false    false
null == undefined   true     false    false
NaN == NaN          false    false    true
0 == -0             true     true     false
```

## **Loops & Iterations (50% Concept / 50% Code)**

### **Basic Loop Structures**

#### **`for` Loop (Classic Counter)**
```javascript
// Count-controlled iteration
for (let i = 0; i < 5; i++) {
    console.log(i); // 0, 1, 2, 3, 4
}
```

#### **`while` Loop (Condition-Controlled)**
```javascript
// Loop while condition is true
let i = 0;
while (i < 3) {
    console.log(i); // 0, 1, 2
    i++;
}
```

#### **`do...while` Loop (Execute at Least Once)**
```javascript
// Check condition AFTER first execution
let i = 0;
do {
    console.log(i); // 0 (always runs once)
    i++;
} while (i < 0);
```

### **Specialized Loops**

#### **`for...in` Loop (Object Properties)**
```javascript
// Iterates over ENUMERABLE property names
const obj = {a: 1, b: 2, c: 3};

for (let key in obj) {
    console.log(key, obj[key]); // "a 1", "b 2", "c 3"
}
```

#### **`for...of` Loop (Iterable Values)**
```javascript
// Iterates over VALUES of iterables
const arr = ["a", "b", "c"];

for (let value of arr) {
    console.log(value); // "a", "b", "c"
}
```

### **Loop Control Statements**

#### **`break` (Exit Loop Entirely)**
```javascript
for (let i = 0; i < 10; i++) {
    if (i === 5) {
        break; // Stops completely at i=5
    }
    console.log(i); // 0, 1, 2, 3, 4
}
```

#### **`continue` (Skip to Next Iteration)**
```javascript
for (let i = 0; i < 5; i++) {
    if (i === 2) {
        continue; // Skips i=2
    }
    console.log(i); // 0, 1, 3, 4
}
```

### **When to Use Which Loop**

```javascript
// Use for...in for OBJECT properties
let user = {name: "John", age: 30};
for (let key in user) {
    console.log(`${key}: ${user[key]}`);
}

// Use for...of for ARRAY values
let numbers = [1, 2, 3];
for (let num of numbers) {
    console.log(num * 2);
}

// Use classic for when you need the INDEX
for (let i = 0; i < numbers.length; i++) {
    console.log(`Index ${i}: ${numbers[i]}`);
}
```

### **Important Differences**

```javascript
// for...in vs for...of
let arr = ["a", "b", "c"];
arr.customProp = "test";

for (let key in arr) {
    console.log(key); // "0", "1", "2", "customProp"
}

for (let value of arr) {
    console.log(value); // "a", "b", "c" (no customProp)
}
```

## **Best Practices**

### **Equality Best Practices**
```javascript
// ALWAYS use === unless you specifically want coercion
if (value === null)          // Check for null
if (value === undefined)     // Check for undefined
if (Number.isNaN(value))     // Check for NaN (not value === NaN!)
if (Object.is(value, -0))    // Check for -0 specifically

// Avoid == except for these specific cases:
if (value == null)           // Checks BOTH null AND undefined
```

### **Loop Best Practices**
```javascript
// Cache array length for performance
for (let i = 0, len = arr.length; i < len; i++) {
    // Process arr[i]
}

// Use for...of for cleaner array iteration
for (const item of items) {
    // Process item
}

// Use for...in with hasOwnProperty for objects
for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
        // Process obj[key]
    }
}
```

## **Key Takeaways**

1. **Equality Operators**:
   - `===` for strict comparison (no coercion)
   - `==` for loose comparison (with coercion)
   - `Object.is()` for "SameValue" equality

2. **Loop Selection**:
   - `for` - Counting iterations
   - `while` - Unknown iterations, condition-based
   - `for...of` - Array values
   - `for...in` - Object keys

3. **Control Flow**:
   - `break` - Exit loop completely
   - `continue` - Skip to next iteration

Understanding these concepts ensures predictable comparisons and efficient iterations in your JavaScript code!