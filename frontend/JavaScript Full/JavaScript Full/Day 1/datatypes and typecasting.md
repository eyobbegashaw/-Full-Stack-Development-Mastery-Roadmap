# **JavaScript Data Types & Type System**

## **Data Types in JavaScript**
JavaScript has 7 **Primitive Types** and 1 **Object Type**

### **Primitive Types** (Immutable, Stored by Value)
```javascript
// 1. string - Text data
let name = "John";
let greeting = 'Hello';

// 2. number - Floating point numbers
let age = 25;
let price = 99.99;

// 3. bigint - Very large integers
let bigNumber = 9007199254740991n;

// 4. boolean - True/False
let isActive = true;
let isLoggedIn = false;

// 5. undefined - Declared but not assigned
let x;
console.log(x); // undefined

// 6. null - Intentional empty value
let empty = null;

// 7. symbol - Unique identifier
let id = Symbol("id");
```

### **Object Type** (Mutable, Stored by Reference)
```javascript
// Objects, Arrays, Functions, Dates, etc.
let person = { name: "John", age: 30 };
let colors = ["red", "green", "blue"];
let today = new Date();
```

## **`typeof` Operator**
Check what type a value is:
```javascript
typeof "hello"     // "string"
typeof 42          // "number"
typeof 42n         // "bigint"
typeof true        // "boolean"
typeof undefined   // "undefined"
typeof null        // "object" (famous bug!)
typeof Symbol()    // "symbol"
typeof {}          // "object"
typeof []          // "object"
typeof function(){} // "function"
```

**Note:** `typeof null` returns `"object"` - a historical bug that can't be fixed!

## **Type Conversion vs Coercion**

### **Explicit Type Casting** (You control it)
```javascript
// String to Number
Number("123")      // 123
parseInt("123")    // 123
parseFloat("12.3") // 12.3
+"123"             // 123 (unary plus)

// Number to String
String(123)        // "123"
(123).toString()   // "123"
123 + ""           // "123"

// Boolean conversion
Boolean(1)         // true
Boolean(0)         // false
!!"hello"          // true (double NOT)
```

### **Implicit Type Casting** (JavaScript does it automatically)
```javascript
// String concatenation
"5" + 3            // "53" (number → string)

// Mathematical operations
"10" - 5           // 5 (string → number)
"10" * "2"         // 20
"10" / "2"         // 5

// Loose equality (==) does type coercion
5 == "5"           // true
0 == false         // true
null == undefined  // true

// Strict equality (===) does NOT
5 === "5"          // false
```

## **Built-in Objects**
Constructor functions for primitives:
```javascript
// String object wrapper
let str = new String("hello");

// Number object wrapper  
let num = new Number(123);

// Boolean object wrapper
let bool = new Boolean(true);
```

**Note:** Usually avoid these - use primitives instead!

## **Object Prototype & Prototypal Inheritance**
Every object has a hidden `[[Prototype]]` property:

```javascript
// Parent object
let animal = {
  eats: true,
  walk() {
    console.log("Animal walks");
  }
};

// Child object inherits from animal
let rabbit = {
  jumps: true,
  __proto__: animal  // Set prototype
};

rabbit.walk(); // "Animal walks" (inherited)
console.log(rabbit.eats); // true (inherited)
```

### **Modern way using Object.create()**
```javascript
let animal = {
  eats: true
};

let rabbit = Object.create(animal);
rabbit.jumps = true;

console.log(rabbit.eats); // true (from prototype)
```

## **Key Concepts Summary**

### **Primitive vs Object**
```javascript
// Primitive - copied by value
let a = 5;
let b = a; // Copy value
b = 10;
console.log(a); // 5 (unchanged)

// Object - copied by reference
let obj1 = { x: 5 };
let obj2 = obj1; // Copy reference
obj2.x = 10;
console.log(obj1.x); // 10 (changed!)
```

### **Type Coercion Rules**
1. `+` operator favors strings
2. Other math operators favor numbers
3. `==` tries to convert types
4. `===` checks type AND value

### **Best Practices**
```javascript
// 1. Use === instead of ==
if (age === "25")   // false (different types)
if (age == "25")    // true (type coercion)

// 2. Explicit conversion is clearer
let total = Number(price) + Number(tax);

// 3. Check for null/undefined
if (value === null || value === undefined)
// OR
if (value == null)  // catches both null AND undefined

// 4. Use primitive types, not object wrappers
let good = "hello";   // ✓
let bad = new String("hello"); // ✗
```

## **Quick Reference Table**

| Type | Example | `typeof` | Mutable |
|------|---------|----------|---------|
| string | `"hello"` | `"string"` | No |
| number | `42`, `3.14` | `"number"` | No |
| bigint | `123n` | `"bigint"` | No |
| boolean | `true` | `"boolean"` | No |
| undefined | `undefined` | `"undefined"` | No |
| null | `null` | `"object"` | No |
| symbol | `Symbol()` | `"symbol"` | No |
| object | `{}`, `[]` | `"object"` | Yes |

## **Remember:**
- **7 Primitive Types** + **Objects**
- **Explicit Casting**: You control it with `String()`, `Number()`, etc.
- **Implicit Coercion**: JavaScript does it automatically (can be tricky!)
- **Prototype**: Hidden link objects use to inherit properties
- **Always use `===`** unless you specifically need type coercion
- `null` is an **intentional empty value**, `undefined` means **not assigned yet**

Understanding types is crucial for debugging and writing predictable JavaScript code!