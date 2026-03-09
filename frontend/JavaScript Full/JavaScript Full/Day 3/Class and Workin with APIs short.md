# **XMLHttpRequest, Fetch, Classes & Working with APIs**

## **Concept Overview (70%)**

### **1. XMLHttpRequest (XHR)**
- **Legacy API** for making HTTP requests (pre-ES6)
- **Event-driven** with `onload`, `onerror`, `onprogress` handlers
- **Complex syntax** but full control over request lifecycle
- **Supports older browsers** (IE7+)
- **No promise support** natively (must wrap in promises manually)

### **2. Fetch API**
- **Modern replacement** for XHR (ES6+)
- **Promise-based** - cleaner syntax with `.then()`/`.catch()`
- **Simpler API** but less granular control than XHR
- **Stream-based** response handling
- **No automatic cookie sending** (unlike XHR)
- **Requires polyfill** for older browsers

### **3. Classes in JavaScript**
- **ES6 syntactic sugar** over prototype-based inheritance
- **Constructor method** for initialization
- **Methods** defined inside class (no `function` keyword)
- **Static methods** called on class itself, not instances
- **Inheritance** via `extends` keyword
- **Private fields** (`#field`) for encapsulation (ES2022)

### **4. Working with APIs**
- **RESTful APIs**: Standard HTTP methods (GET, POST, PUT, DELETE)
- **API Communication**: Send/receive data (JSON most common)
- **Authentication**: API keys, tokens, OAuth
- **Error Handling**: HTTP status codes, network errors
- **Best Practices**: Rate limiting, caching, error handling

## **Code Explanation (30%)**

```javascript
// ====== XMLHttpRequest (Legacy) ======
const xhr = new XMLHttpRequest();
xhr.open('GET', 'https://api.example.com/data');
xhr.responseType = 'json';

xhr.onload = function() {
    if (xhr.status === 200) {
        console.log('XHR Success:', xhr.response);
    } else {
        console.error('XHR Error:', xhr.status);
    }
};

xhr.onerror = function() {
    console.error('XHR Network Error');
};

xhr.send();

// ====== Fetch API (Modern) ======
// Basic GET request
fetch('https://api.example.com/data')
    .then(response => {
        if (!response.ok) throw new Error('Network error');
        return response.json();
    })
    .then(data => console.log('Fetch Success:', data))
    .catch(error => console.error('Fetch Error:', error));

// POST request with headers
fetch('https://api.example.com/data', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer token123'
    },
    body: JSON.stringify({ name: 'John', age: 30 })
});

// ====== Classes for API Handling ======
class APIClient {
    constructor(baseURL, apiKey) {
        this.baseURL = baseURL;
        this.apiKey = apiKey;
    }
    
    async get(endpoint) {
        const response = await fetch(`${this.baseURL}/${endpoint}`, {
            headers: { 'Authorization': this.apiKey }
        });
        return this.handleResponse(response);
    }
    
    async post(endpoint, data) {
        const response = await fetch(`${this.baseURL}/${endpoint}`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': this.apiKey
            },
            body: JSON.stringify(data)
        });
        return this.handleResponse(response);
    }
    
    async handleResponse(response) {
        if (!response.ok) {
            throw new Error(`API Error: ${response.status}`);
        }
        return await response.json();
    }
}

// Usage
const api = new APIClient('https://api.example.com', 'key123');
api.get('users')
    .then(data => console.log(data))
    .catch(error => console.error(error));

// ====== Working with REST APIs ======
// GET - Retrieve data
fetch('https://jsonplaceholder.typicode.com/posts/1')
    .then(res => res.json())
    .then(data => console.log('GET:', data));

// POST - Create data
fetch('https://jsonplaceholder.typicode.com/posts', {
    method: 'POST',
    body: JSON.stringify({ title: 'New Post', body: 'Content' })
});

// PUT - Update data
fetch('https://jsonplaceholder.typicode.com/posts/1', {
    method: 'PUT',
    body: JSON.stringify({ title: 'Updated Title' })
});

// DELETE - Remove data
fetch('https://jsonplaceholder.typicode.com/posts/1', {
    method: 'DELETE'
});
```

### **Key Points:**
- **XHR**: Legacy, event-based, complex but powerful
- **Fetch**: Modern, promise-based, simpler but less control  
- **Classes**: Organize API logic, promote reusability
- **APIs**: Use HTTP methods, handle responses/errors properly