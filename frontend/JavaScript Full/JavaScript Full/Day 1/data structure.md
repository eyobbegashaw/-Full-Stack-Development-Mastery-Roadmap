# **JavaScript Data Structures & Collections**

## **Indexed Collections**
These maintain order based on numeric indices.

### **Arrays**
The most fundamental collection type in JavaScript. Arrays are ordered lists that can hold mixed data types and dynamically resize.

**Key Characteristics:**
- Zero-based indexing
- Dynamic size (grow/shrink automatically)
- Can contain any data type
- Maintains insertion order
- Hundreds of built-in methods

```javascript
let colors = ["red", "green", "blue"];
```

### **Typed Arrays**
Special array-like objects for working with raw binary data. They provide a mechanism for accessing raw memory without the overhead of regular JavaScript objects.

**When to Use:**
- WebGL graphics
- Audio/video processing
- Network protocols
- Large datasets where performance matters

```javascript
let buffer = new Int16Array(10); // Fixed size
```

## **Keyed Collections**
These use keys (not just indices) to organize data.

### **Map vs Object**
While plain objects can store key-value pairs, Maps are superior for true key-value collections:

**Map Advantages:**
- Keys can be ANY type (objects, functions, etc.)
- Maintains insertion order
- Built-in size property
- Better performance for frequent additions/removals
- No prototype chain interference

```javascript
let map = new Map();
map.set(document.body, "page element");
```

### **WeakMap**
Specialized Map where keys are "weakly" referenced. This means if nothing else references the key object, it can be garbage collected even if it's in the WeakMap.

**Key Use Cases:**
- Storing private data
- Caching without memory leaks
- DOM element metadata

```javascript
let weakMap = new WeakMap();
weakMap.set(element, { clicks: 0 });
```

### **Set**
Collection of unique values. Automatically prevents duplicates.

**Common Uses:**
- Removing duplicates from arrays
- Membership testing
- Mathematical set operations

```javascript
let uniqueTags = new Set(["js", "html", "js", "css"]);
// Result: {"js", "html", "css"}
```

### **WeakSet**
Like WeakMap but for values instead of key-value pairs. Holds objects weakly.

**Typical Use:**
- Tracking objects without preventing garbage collection
- Marking objects as "seen" or "processed"

```javascript
let processed = new WeakSet();
processed.add(userObject);
```

## **Structured Data**

### **JSON (JavaScript Object Notation)**
Lightweight data interchange format that's language-independent but based on JavaScript object syntax.

**Key Points:**
- String representation of JavaScript objects
- Uses double quotes for all strings
- No functions, undefined, or comments
- Universal standard for APIs

```javascript
let jsonString = '{"name":"John","age":30}';
let obj = JSON.parse(jsonString);
let str = JSON.stringify(obj);
```

## **Comparison & When to Use**

### **Array vs Set**
- **Array**: When order matters, duplicates needed, or you need indexed access
- **Set**: When you need uniqueness, fast lookup, or set operations

### **Object vs Map**
- **Object**: When keys are strings/symbols and you need JSON compatibility
- **Map**: When keys are complex objects, need insertion order, or frequent additions/removals

### **Regular vs Weak Collections**
- **Regular**: When you need to prevent garbage collection of keys/values
- **Weak**: When you want automatic cleanup, avoiding memory leaks

## **Best Practices**

1. **Default to Arrays** for ordered lists
2. **Use Sets** for uniqueness requirements
3. **Choose Maps over Objects** for true key-value collections
4. **Use Weak collections** when storing references to DOM elements or large objects
5. **JSON.stringify()** for data serialization before sending to APIs

## **Performance Considerations**
- **Maps** are faster for frequent additions/removals
- **Sets** provide O(1) lookup for checking membership
- **Typed Arrays** offer significant performance benefits for numerical computations
- **Weak collections** help prevent memory leaks in long-running applications

## **Real-World Applications**

1. **Array**: Shopping cart items, to-do lists, paginated data
2. **Set**: User tags, visited pages, unique IP addresses
3. **Map**: Caching system, user sessions, DOM element metadata
4. **WeakMap**: Private object properties, DOM element cleanup
5. **JSON**: API responses, configuration files, data storage

Understanding these structures helps you choose the right tool for each programming task, leading to more efficient and maintainable code. Each has specific strengths that make it ideal for particular scenarios in modern JavaScript development.