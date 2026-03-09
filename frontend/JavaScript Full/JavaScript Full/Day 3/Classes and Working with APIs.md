# **XMLHttpRequest, Fetch API, Classes & Working with APIs**

## **Concept Overview (70%)**

### **Evolution of HTTP Requests in JavaScript**

#### **1. XMLHttpRequest (XHR) - The Original**
**XMLHttpRequest** is the legacy API for making HTTP requests in JavaScript, introduced by Microsoft in IE5 (1999) and later standardized. It was foundational for **AJAX** (Asynchronous JavaScript and XML) which enabled dynamic web applications without full page reloads.

**Key Characteristics:**
- Event-driven architecture with `onreadystatechange`
- Supports both synchronous and asynchronous requests
- Browser-specific inconsistencies in early implementations
- Verbose API with manual state management
- First to enable true single-page applications

#### **2. Fetch API - The Modern Standard**
**Fetch API** is the modern replacement for XHR, introduced with ES6 (2015). It provides a cleaner, promise-based interface for making HTTP requests, built on promises and streams.

**Key Advantages over XHR:**
- Promise-based (cleaner async handling)
- Simplified API with sensible defaults
- Built-in support for request/response streams
- Better integration with Service Workers
- More secure by default (CORS handling)

#### **3. Classes - Object-Oriented Structure**
ES6 **Classes** provide syntactic sugar over JavaScript's prototype-based inheritance, offering a cleaner syntax for creating objects and handling inheritance. In API context, classes are essential for:
- Creating reusable API client abstractions
- Organizing request/response handlers
- Implementing middleware/interceptor patterns
- Building SDKs and libraries

#### **4. Working with APIs - Complete Workflow**
**API Integration** involves more than just making requests. It's a comprehensive workflow including:
- **Authentication & Authorization** (API keys, OAuth, JWT)
- **Request Building** (headers, parameters, body formatting)
- **Response Handling** (parsing, error management, data transformation)
- **State Management** (caching, pagination, optimistic updates)
- **Performance Optimization** (debouncing, throttling, request deduplication)
- **Error Recovery** (retry logic, fallbacks, offline handling)

### **Detailed Comparison: XHR vs Fetch**

#### **XMLHttpRequest Architecture:**
```javascript
// Old-school event-driven model
xhr.onreadystatechange = function() {
    if (xhr.readyState === 4) {
        if (xhr.status === 200) {
            // Success
        } else {
            // Error
        }
    }
};
```

#### **Fetch API Architecture:**
```javascript
// Modern promise-based model
fetch(url)
    .then(response => {
        if (!response.ok) throw new Error('Network error');
        return response.json();
    })
    .then(data => console.log(data))
    .catch(error => console.error(error));
```

### **Modern API Patterns**

1. **API Client Abstraction**: Encapsulating API logic in reusable classes
2. **Interceptor/Middleware**: Global request/response transformation
3. **Request Cancellation**: Using AbortController
4. **Automatic Retry**: Exponential backoff strategies
5. **Response Caching**: Memory, localStorage, or service worker caches
6. **Type Safety**: Using TypeScript for API contracts
7. **Mocking & Testing**: Service virtualization for development/testing

## **Code Explanation (30%)**

```javascript
// ====== 1. XMLHTTPREQUEST (LEGACY) ======

// Basic GET request
const xhrGet = new XMLHttpRequest();
xhrGet.open('GET', 'https://api.example.com/data', true); // async=true
xhrGet.setRequestHeader('Content-Type', 'application/json');
xhrGet.setRequestHeader('Authorization', 'Bearer token123');

xhrGet.onreadystatechange = function() {
    if (xhrGet.readyState === 4) { // DONE
        if (xhrGet.status >= 200 && xhrGet.status < 300) {
            try {
                const data = JSON.parse(xhrGet.responseText);
                console.log('XHR Success:', data);
            } catch (e) {
                console.error('XHR Parse Error:', e);
            }
        } else {
            console.error('XHR Error:', xhrGet.status, xhrGet.statusText);
        }
    }
};

xhrGet.onprogress = function(event) {
    if (event.lengthComputable) {
        const percentComplete = (event.loaded / event.total) * 100;
        console.log(`XHR Progress: ${percentComplete.toFixed(2)}%`);
    }
};

xhrGet.ontimeout = function() {
    console.error('XHR Timeout');
};

xhrGet.timeout = 10000; // 10 seconds
xhrGet.send();

// POST request with form data
const xhrPost = new XMLHttpRequest();
xhrPost.open('POST', 'https://api.example.com/users', true);

const formData = new FormData();
formData.append('username', 'john_doe');
formData.append('email', 'john@example.com');

xhrPost.onload = function() {
    if (xhrPost.status === 201) {
        console.log('User created:', xhrPost.response);
    }
};

xhrPost.send(formData);

// Synchronous request (BLOCKS UI - avoid!)
const xhrSync = new XMLHttpRequest();
xhrSync.open('GET', 'https://api.example.com/data', false); // false = sync
xhrSync.send();
console.log('This runs AFTER request completes');

// ====== 2. FETCH API (MODERN) ======

// Basic GET request
fetch('https://api.example.com/data')
    .then(response => {
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        return response.json(); // Parse JSON
    })
    .then(data => console.log('Fetch Success:', data))
    .catch(error => console.error('Fetch Error:', error))
    .finally(() => console.log('Fetch completed'));

// POST request with JSON
const postData = {
    title: 'New Post',
    body: 'Content here',
    userId: 1
};

fetch('https://jsonplaceholder.typicode.com/posts', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer token123'
    },
    body: JSON.stringify(postData)
})
    .then(response => response.json())
    .then(data => console.log('Created post:', data));

// Fetch with timeout using AbortController
async function fetchWithTimeout(url, timeout = 5000) {
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), timeout);
    
    try {
        const response = await fetch(url, {
            signal: controller.signal
        });
        clearTimeout(timeoutId);
        
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        return await response.json();
    } catch (error) {
        clearTimeout(timeoutId);
        if (error.name === 'AbortError') {
            throw new Error(`Request timed out after ${timeout}ms`);
        }
        throw error;
    }
}

// Fetch with progress (for large downloads)
async function fetchWithProgress(url, onProgress) {
    const response = await fetch(url);
    const reader = response.body.getReader();
    const contentLength = +response.headers.get('Content-Length');
    let receivedLength = 0;
    let chunks = [];
    
    while (true) {
        const { done, value } = await reader.read();
        
        if (done) break;
        
        chunks.push(value);
        receivedLength += value.length;
        
        if (contentLength) {
            const percent = (receivedLength / contentLength) * 100;
            onProgress(percent);
        }
    }
    
    // Combine chunks
    const allChunks = new Uint8Array(receivedLength);
    let position = 0;
    for (const chunk of chunks) {
        allChunks.set(chunk, position);
        position += chunk.length;
    }
    
    return new TextDecoder().decode(allChunks);
}

// ====== 3. CLASSES FOR API ABSTRACTION ======

// Base API Client Class
class ApiClient {
    constructor(baseURL, defaultOptions = {}) {
        this.baseURL = baseURL;
        this.defaultOptions = {
            headers: {
                'Content-Type': 'application/json',
                ...defaultOptions.headers
            },
            ...defaultOptions
        };
        this.interceptors = {
            request: [],
            response: []
        };
    }
    
    // Request interceptor
    addRequestInterceptor(interceptor) {
        this.interceptors.request.push(interceptor);
    }
    
    // Response interceptor
    addResponseInterceptor(interceptor) {
        this.interceptors.response.push(interceptor);
    }
    
    // Apply interceptors
    async applyInterceptors(type, value) {
        let result = value;
        for (const interceptor of this.interceptors[type]) {
            result = await interceptor(result);
        }
        return result;
    }
    
    // Core request method
    async request(endpoint, options = {}) {
        const url = `${this.baseURL}${endpoint}`;
        
        // Merge options
        const requestOptions = {
            ...this.defaultOptions,
            ...options,
            headers: {
                ...this.defaultOptions.headers,
                ...options.headers
            }
        };
        
        // Apply request interceptors
        let interceptedOptions = await this.applyInterceptors('request', requestOptions);
        
        try {
            const response = await fetch(url, interceptedOptions);
            
            // Apply response interceptors
            const interceptedResponse = await this.applyInterceptors('response', response);
            
            if (!interceptedResponse.ok) {
                throw new HttpError(
                    interceptedResponse.status,
                    interceptedResponse.statusText,
                    interceptedResponse
                );
            }
            
            // Auto-parse based on content type
            const contentType = interceptedResponse.headers.get('content-type');
            if (contentType?.includes('application/json')) {
                return await interceptedResponse.json();
            } else if (contentType?.includes('text/')) {
                return await interceptedResponse.text();
            } else {
                return interceptedResponse;
            }
        } catch (error) {
            if (error instanceof HttpError) throw error;
            throw new NetworkError(error.message, url);
        }
    }
    
    // Convenience methods
    get(endpoint, options = {}) {
        return this.request(endpoint, { ...options, method: 'GET' });
    }
    
    post(endpoint, data, options = {}) {
        return this.request(endpoint, {
            ...options,
            method: 'POST',
            body: JSON.stringify(data)
        });
    }
    
    put(endpoint, data, options = {}) {
        return this.request(endpoint, {
            ...options,
            method: 'PUT',
            body: JSON.stringify(data)
        });
    }
    
    delete(endpoint, options = {}) {
        return this.request(endpoint, { ...options, method: 'DELETE' });
    }
    
    patch(endpoint, data, options = {}) {
        return this.request(endpoint, {
            ...options,
            method: 'PATCH',
            body: JSON.stringify(data)
        });
    }
}

// Custom Error Classes
class HttpError extends Error {
    constructor(status, statusText, response) {
        super(`HTTP ${status}: ${statusText}`);
        this.name = 'HttpError';
        this.status = status;
        this.statusText = statusText;
        this.response = response;
    }
}

class NetworkError extends Error {
    constructor(message, url) {
        super(`Network error: ${message}`);
        this.name = 'NetworkError';
        this.url = url;
    }
}

// ====== 4. WORKING WITH APIS - COMPLETE EXAMPLE ======

// Specialized API Client
class GitHubAPI extends ApiClient {
    constructor(token = null) {
        super('https://api.github.com', {
            headers: token ? {
                'Authorization': `token ${token}`,
                'Accept': 'application/vnd.github.v3+json'
            } : {}
        });
        
        // Add auth interceptor
        this.addRequestInterceptor((options) => {
            if (this.token && !options.headers.Authorization) {
                options.headers.Authorization = `token ${this.token}`;
            }
            return options;
        });
        
        // Add rate limit monitoring
        this.addResponseInterceptor(async (response) => {
            const remaining = response.headers.get('X-RateLimit-Remaining');
            const reset = response.headers.get('X-RateLimit-Reset');
            
            if (remaining && parseInt(remaining) < 10) {
                console.warn(`GitHub API rate limit low: ${remaining} remaining`);
                console.log(`Resets at: ${new Date(reset * 1000)}`);
            }
            
            return response;
        });
    }
    
    // User operations
    async getUser(username) {
        return this.get(`/users/${username}`);
    }
    
    async getRepos(username, params = {}) {
        return this.get(`/users/${username}/repos`, { params });
    }
    
    async createRepo(name, options = {}) {
        return this.post('/user/repos', {
            name,
            private: options.private || false,
            description: options.description || ''
        });
    }
    
    // Search operations
    async searchRepos(query, options = {}) {
        const params = new URLSearchParams({
            q: query,
            sort: options.sort || 'stars',
            order: options.order || 'desc',
            per_page: options.per_page || 30,
            page: options.page || 1
        });
        
        return this.get(`/search/repositories?${params}`);
    }
    
    // Pagination helper
    async *paginate(endpoint, options = {}) {
        let page = 1;
        let hasMore = true;
        
        while (hasMore) {
            const params = new URLSearchParams({
                per_page: 30,
                page,
                ...options
            });
            
            const items = await this.get(`${endpoint}?${params}`);
            
            if (items.length === 0) {
                hasMore = false;
            } else {
                yield items;
                page++;
                
                // Respect rate limiting
                await new Promise(resolve => setTimeout(resolve, 100));
            }
        }
    }
}

// ====== 5. ADVANCED API PATTERNS ======

// Request deduplication/caching
class RequestCache {
    constructor() {
        this.cache = new Map();
        this.pending = new Map();
    }
    
    async get(key, fetcher) {
        // Check cache first
        if (this.cache.has(key)) {
            return this.cache.get(key);
        }
        
        // Check if request is already in flight
        if (this.pending.has(key)) {
            return this.pending.get(key);
        }
        
        // Create new request
        const promise = fetcher().then(result => {
            this.cache.set(key, result);
            this.pending.delete(key);
            return result;
        }).catch(error => {
            this.pending.delete(key);
            throw error;
        });
        
        this.pending.set(key, promise);
        return promise;
    }
    
    invalidate(key) {
        this.cache.delete(key);
    }
    
    clear() {
        this.cache.clear();
        this.pending.clear();
    }
}

// Retry with exponential backoff
async function fetchWithRetry(url, options = {}, maxRetries = 3) {
    let lastError;
    
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
        try {
            const controller = new AbortController();
            const timeoutId = setTimeout(() => controller.abort(), options.timeout || 10000);
            
            const response = await fetch(url, {
                ...options,
                signal: controller.signal
            });
            
            clearTimeout(timeoutId);
            
            if (!response.ok) {
                throw new HttpError(response.status, response.statusText, response);
            }
            
            return response;
        } catch (error) {
            lastError = error;
            
            if (attempt === maxRetries) break;
            if (error.name === 'AbortError') {
                console.log(`Attempt ${attempt}: Timeout`);
            } else {
                console.log(`Attempt ${attempt} failed:`, error.message);
            }
            
            // Exponential backoff
            const delay = Math.pow(2, attempt) * 1000;
            await new Promise(resolve => setTimeout(resolve, delay));
        }
    }
    
    throw lastError;
}

// Batch requests
class RequestBatcher {
    constructor(batchSize = 5, batchInterval = 100) {
        this.batchSize = batchSize;
        this.batchInterval = batchInterval;
        this.queue = [];
        this.timer = null;
    }
    
    async add(request) {
        return new Promise((resolve, reject) => {
            this.queue.push({ request, resolve, reject });
            
            if (this.queue.length >= this.batchSize) {
                this.processBatch();
            } else if (!this.timer) {
                this.timer = setTimeout(() => this.processBatch(), this.batchInterval);
            }
        });
    }
    
    async processBatch() {
        if (this.timer) {
            clearTimeout(this.timer);
            this.timer = null;
        }
        
        if (this.queue.length === 0) return;
        
        const batch = this.queue.splice(0, this.batchSize);
        
        try {
            const results = await Promise.all(
                batch.map(({ request }) => request())
            );
            
            batch.forEach(({ resolve }, index) => {
                resolve(results[index]);
            });
        } catch (error) {
            batch.forEach(({ reject }) => {
                reject(error);
            });
        }
    }
}

// ====== 6. REAL-WORLD API INTEGRATION EXAMPLE ======

class WeatherAPI extends ApiClient {
    constructor(apiKey) {
        super('https://api.openweathermap.org/data/2.5', {
            headers: {
                'Accept': 'application/json'
            }
        });
        
        this.apiKey = apiKey;
        this.cache = new RequestCache();
        this.batcher = new RequestBatcher();
        
        // Add API key to all requests
        this.addRequestInterceptor((options) => {
            const url = new URL(options.url || '');
            url.searchParams.set('appid', this.apiKey);
            url.searchParams.set('units', 'metric');
            return { ...options, url: url.toString() };
        });
        
        // Cache successful responses
        this.addResponseInterceptor(async (response) => {
            const url = response.url;
            const clonedResponse = response.clone();
            
            if (response.ok) {
                const data = await response.json();
                this.cache.cache.set(url, data);
                return clonedResponse;
            }
            
            return response;
        });
    }
    
    async getWeather(city) {
        const cacheKey = `weather:${city}`;
        
        return this.cache.get(cacheKey, () => 
            this.batcher.add(() => 
                this.get(`/weather?q=${encodeURIComponent(city)}`)
            )
        );
    }
    
    async getForecast(city) {
        return this.get(`/forecast?q=${encodeURIComponent(city)}`);
    }
    
    async getWeatherByCoords(lat, lon) {
        return this.get(`/weather?lat=${lat}&lon=${lon}`);
    }
}

// ====== 7. TESTING & MOCKING ======

// Mock API for testing
class MockApiClient extends ApiClient {
    constructor(mockData = {}) {
        super('', { mode: 'no-cors' });
        this.mockData = mockData;
        this.requestLog = [];
    }
    
    async request(endpoint, options = {}) {
        this.requestLog.push({ endpoint, options, timestamp: Date.now() });
        
        // Simulate network delay
        await new Promise(resolve => setTimeout(resolve, 100));
        
        const mockResponse = this.mockData[endpoint] || this.mockData[options.method];
        
        if (mockResponse) {
            if (mockResponse.error) {
                throw new HttpError(
                    mockResponse.status || 500,
                    mockResponse.error,
                    { ok: false }
                );
            }
            return mockResponse.data;
        }
        
        throw new Error(`No mock data for ${endpoint}`);
    }
    
    clearLog() {
        this.requestLog = [];
    }
}

// ====== 8. COMPREHENSIVE USAGE EXAMPLE ======

async function demonstrateCompleteWorkflow() {
    // Create API client
    const api = new GitHubAPI(process.env.GITHUB_TOKEN);
    
    try {
        // 1. Get user data
        const user = await api.getUser('octocat');
        console.log('User:', user.login, user.name);
        
        // 2. Get repositories with pagination
        console.log('Fetching repositories...');
        for await (const repos of api.paginate('/users/octocat/repos')) {
            console.log(`Fetched ${repos.length} repos`);
            
            // Process each repo
            repos.forEach(repo => {
                console.log(`- ${repo.full_name}: ${repo.stargazers_count} stars`);
            });
        }
        
        // 3. Search repositories
        const searchResults = await api.searchRepos('javascript framework');
        console.log(`Found ${searchResults.total_count} repos`);
        
        // 4. Error handling example
        try {
            await api.getUser('nonexistent-user-123456');
        } catch (error) {
            if (error instanceof HttpError && error.status === 404) {
                console.log('User not found (expected)');
            } else {
                throw error;
            }
        }
        
        // 5. Concurrent requests
        const [userRepos, userFollowers, userFollowing] = await Promise.all([
            api.getRepos('octocat'),
            api.get('/users/octocat/followers'),
            api.get('/users/octocat/following')
        ]);
        
        console.log(`Octocat has ${userRepos.length} repos, ${userFollowers.length} followers`);
        
    } catch (error) {
        console.error('API Error:', error);
        
        if (error instanceof HttpError) {
            switch (error.status) {
                case 401:
                    console.error('Authentication failed');
                    break;
                case 403:
                    console.error('Rate limited');
                    break;
                case 404:
                    console.error('Resource not found');
                    break;
                case 429:
                    console.error('Too many requests');
                    break;
                case 500:
                    console.error('Server error');
                    break;
            }
        } else if (error instanceof NetworkError) {
            console.error('Network issue - check connection');
        }
    }
}

// ====== 9. COMPARISON SUMMARY ======

/*
XMLHTTPREQUEST (Legacy):
✅ Full browser support
✅ Progress events
✅ Synchronous mode (but avoid!)
❌ Verbose API
❌ Callback hell
❌ No promise support

FETCH API (Modern):
✅ Promise-based
✅ Clean, simple API
✅ Stream support
✅ Service Worker integration
❌ No progress events (without streams)
❌ No timeout (need AbortController)
❌ No synchronous mode

CLASSES:
✅ Organize API logic
✅ Reusable abstractions
✅ Encapsulation
✅ Inheritance for specialization

BEST PRACTICES:
1. Use Fetch for new projects
2. Wrap Fetch in classes for organization
3. Implement interceptors for cross-cutting concerns
4. Add proper error handling
5. Implement caching strategies
6. Handle authentication properly
7. Monitor rate limits
8. Test with mock APIs
*/
```

### **Key Takeaways:**

1. **XMLHttpRequest** is legacy but still functional; understand it for maintaining old code
2. **Fetch API** is the modern standard with promise-based architecture
3. **Classes** provide structure for building robust API clients and SDKs
4. **API Integration** requires more than just making requests - includes error handling, caching, pagination, and optimization
5. **Modern patterns** include interceptors, request caching, retry logic, and batching
6. **Type safety** with TypeScript can prevent many runtime errors
7. **Testing** with mock APIs is crucial for reliable applications

This comprehensive approach to working with APIs demonstrates professional JavaScript development practices, from basic HTTP requests to enterprise-grade API client implementations.