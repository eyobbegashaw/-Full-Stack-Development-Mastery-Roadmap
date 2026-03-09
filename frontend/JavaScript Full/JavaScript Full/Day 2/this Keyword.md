# **`this` Keyword, Function Borrowing & Explicit Binding**

## **Concept Overview (70%)**

### **Understanding the `this` Keyword**

The `this` keyword in JavaScript refers to the **execution context** of a function - what object the function is being called on. Its value is determined by **how a function is called**, not where it's defined.

#### **Five Contexts for `this`:**

1. **In a Method**: When a function is called as a method of an object, `this` refers to that object
2. **In a Function**: When a function is called standalone (not as a method), `this` refers to:
   - **Non-strict mode**: Global object (window in browsers)
   - **Strict mode**: `undefined`
3. **Using it Alone**: In global scope, `this` refers to the global object
4. **In Event Handlers**: When used in DOM event handlers, `this` refers to the element that received the event
5. **In Arrow Functions**: Arrow functions don't have their own `this` - they inherit `this` from their parent scope (lexical scoping)

### **Function Borrowing**

Function borrowing allows you to **use a method from one object on another object**. This is possible because in JavaScript, functions are "first-class citizens" and can be passed around as values.

**Key Concept**: When you borrow a method, you're temporarily giving another object access to use that method, with `this` pointing to the borrowing object.

### **Explicit Binding**

Explicit binding gives you **control over what `this` refers to** when calling a function. JavaScript provides three methods for explicit binding:

1. **call()**: Immediately invokes the function with a specified `this` value and individual arguments
2. **apply()**: Immediately invokes the function with a specified `this` value and an array of arguments
3. **bind()**: Returns a new function with a bound `this` value (and optionally, pre-set arguments), doesn't immediately invoke

#### **Comparison:**
- **call()**: `func.call(thisArg, arg1, arg2, ...)`
- **apply()**: `func.apply(thisArg, [arg1, arg2, ...])`
- **bind()**: `const boundFunc = func.bind(thisArg, arg1, arg2)`

## **Code Explanation (30%)**

```javascript
// ====== THIS KEYWORD IN DIFFERENT CONTEXTS ======

// 1. IN A METHOD
const person = {
    name: 'Alice',
    greet: function() {
        console.log(`Hello, I'm ${this.name}`);
        // 'this' refers to the person object
    }
};
person.greet(); // "Hello, I'm Alice"

// 2. IN A FUNCTION
function showThis() {
    console.log(this);
}

showThis(); // Window (non-strict) or undefined (strict)

// 3. USING IT ALONE (GLOBAL SCOPE)
console.log(this); // Window (in browser global scope)

// 4. IN EVENT HANDLERS
const button = document.createElement('button');
button.textContent = 'Click me';
button.addEventListener('click', function() {
    console.log(this); // 'this' refers to the button element
    console.log(this.textContent); // "Click me"
});

// 5. IN ARROW FUNCTIONS
const obj = {
    name: 'Bob',
    regularFunc: function() {
        console.log(this.name); // 'Bob'
        
        const innerArrow = () => {
            console.log(this.name); // 'Bob' (inherits from parent scope)
        };
        innerArrow();
    },
    arrowFunc: () => {
        console.log(this); // Window/undefined (inherits from global)
        // Not the object - arrow functions don't bind their own 'this'
    }
};

// ====== FUNCTION BORROWING ======

const car = {
    brand: 'Toyota',
    start: function(speed) {
        console.log(`${this.brand} starting at ${speed}km/h`);
    }
};

const bike = {
    brand: 'Yamaha'
};

// Borrowing the start method from car to use on bike
car.start.call(bike, 60); // "Yamaha starting at 60km/h"
// 'this' inside start now refers to bike object

// Practical example: Borrowing array methods
const arrayLike = {
    0: 'first',
    1: 'second',
    length: 2
};

// Borrowing Array.prototype.push
Array.prototype.push.call(arrayLike, 'third');
console.log(arrayLike); // {0: 'first', 1: 'second', 2: 'third', length: 3}

// Borrowing Array.prototype.slice to convert to real array
const realArray = Array.prototype.slice.call(arrayLike);
console.log(realArray); // ['first', 'second', 'third']

// ====== EXPLICIT BINDING ======

// 1. CALL() - Immediate execution with individual arguments
function introduce(greeting, punctuation) {
    console.log(`${greeting}, I'm ${this.name}${punctuation}`);
}

const user1 = { name: 'John' };
const user2 = { name: 'Sarah' };

introduce.call(user1, 'Hi', '!'); // "Hi, I'm John!"
introduce.call(user2, 'Hello', '.'); // "Hello, I'm Sarah."

// 2. APPLY() - Immediate execution with array of arguments
function calculate(a, b, c) {
    console.log(`${this.name}: ${a} + ${b} + ${c} = ${a + b + c}`);
}

const calculator = { name: 'Calc' };
const numbers = [10, 20, 30];

calculate.apply(calculator, numbers); // "Calc: 10 + 20 + 30 = 60"

// Common use case: Finding max/min in array
const scores = [95, 88, 72, 64, 91];
const maxScore = Math.max.apply(null, scores); // 95
// 'null' because Math.max doesn't use 'this'

// 3. BIND() - Creates new function with bound context
const user3 = { name: 'Mike' };
const user4 = { name: 'Lisa' };

// Create bound functions
const mikeIntro = introduce.bind(user3, 'Hey');
const lisaIntro = introduce.bind(user4);

// Execute later
mikeIntro('!'); // "Hey, I'm Mike!"
lisaIntro('Hi', '!'); // "Hi, I'm Lisa!"

// Partial application (pre-setting arguments)
function multiply(a, b) {
    return a * b;
}

const double = multiply.bind(null, 2); // 'null' for this, 2 for first arg
console.log(double(5)); // 10 (2 * 5)
console.log(double(10)); // 20 (2 * 10)

const triple = multiply.bind(null, 3);
console.log(triple(7)); // 21 (3 * 7)

// ====== PRACTICAL EXAMPLES ======

// Event handlers with bind
class ButtonComponent {
    constructor(label) {
        this.label = label;
        this.clickCount = 0;
        
        this.button = document.createElement('button');
        this.button.textContent = label;
        
        // Without bind: 'this' would refer to the button element
        // With bind: 'this' refers to ButtonComponent instance
        this.button.addEventListener('click', this.handleClick.bind(this));
        
        // Alternative: Arrow function (inherits 'this' from constructor)
        // this.button.addEventListener('click', () => this.handleClick());
    }
    
    handleClick() {
        this.clickCount++;
        console.log(`${this.label} clicked ${this.clickCount} times`);
    }
}

// Constructor stealing with call/apply
function Vehicle(type) {
    this.type = type;
    this.isRunning = false;
}

function Car(brand, model) {
    // Borrow Vehicle constructor with 'this' set to new Car instance
    Vehicle.call(this, 'car');
    this.brand = brand;
    this.model = model;
}

function Motorcycle(brand, engine) {
    Vehicle.apply(this, ['motorcycle']);
    this.brand = brand;
    this.engine = engine;
}

const myCar = new Car('Toyota', 'Camry');
console.log(myCar.type); // 'car'
console.log(myCar.brand); // 'Toyota'

// Debouncing with bind
function debounce(func, delay) {
    let timeout;
    return function(...args) {
        clearTimeout(timeout);
        // Bind 'this' context from the returned function
        timeout = setTimeout(() => func.apply(this, args), delay);
    };
}

const searchInput = document.createElement('input');
const handleSearch = debounce(function(query) {
    // 'this' properly refers to the input element
    console.log(`Searching for: ${query} in ${this.id}`);
}, 300);

searchInput.addEventListener('input', function(e) {
    handleSearch.call(this, e.target.value);
});

// ====== SUMMARY: WHEN TO USE EACH ======

/*
USE CALL() WHEN:
- You need to immediately execute the function
- You know all arguments and they're individual values
- You want to borrow a method for one-time use

USE APPLY() WHEN:
- You need to immediately execute the function  
- Your arguments are in an array/array-like object
- You're working with variable number of arguments

USE BIND() WHEN:
- You need a function that can be called later
- You want to create a function with preset arguments (partial application)
- You're setting up event handlers or callbacks
- You want to create a specialized version of a function
*/

// Real-world pattern: Method chaining with bound methods
const api = {
    baseURL: 'https://api.example.com',
    
    get(endpoint) {
        console.log(`GET ${this.baseURL}/${endpoint}`);
        return this; // Return 'this' for chaining
    },
    
    post(endpoint, data) {
        console.log(`POST ${this.baseURL}/${endpoint}`, data);
        return this;
    }
};

// Create bound versions for specific endpoints
const usersAPI = {
    getAll: api.get.bind(api, 'users'),
    create: api.post.bind(api, 'users')
};

usersAPI.getAll(); // "GET https://api.example.com/users"
usersAPI.create({ name: 'John' }); // "POST https://api.example.com/users" {name: 'John'}

// Arrow functions don't work with bind/call/apply for 'this' binding
const arrowFunc = () => console.log(this);
const obj2 = { value: 42 };

arrowFunc.call(obj2); // Still logs global/window, not obj2
// Arrow functions ignore explicit binding for 'this'
```

### **Key Takeaways:**

1. **`this` is dynamic** - determined by how a function is called
2. **Arrow functions** have lexical `this` (inherit from surrounding scope)
3. **call()/apply()** immediately invoke with new `this` context
4. **bind()** creates a new function with permanent `this` binding
5. **Function borrowing** enables code reuse across different objects
6. **Explicit binding** gives you control over execution context