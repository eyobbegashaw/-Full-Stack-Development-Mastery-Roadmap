# Caching & Memoization in Data Fetching (React)

## 📘 **1. Memoization in Fetch**

Memoization stores fetched results to prevent duplicate network requests. It creates a cache key (usually the request URL/parameters) and returns cached data when available. This reduces API calls, improves performance, and minimizes loading states. Implementation can be simple (JavaScript Map) or sophisticated (React hooks). Memoization works best for relatively static data. A basic memoized fetch checks cache first, fetches if missing, stores result, then returns data. This pattern speeds up repeated accesses. Libraries like SWR and React Query build upon this principle. Always consider cache invalidation needs when implementing.


**Memoization** is a technique to store and reuse previously fetched data, avoiding unnecessary network requests.

### Why?
- Improves performance
- Reduces API calls
- Faster user experience
- Lower data usage

### How it works:
```javascript
const cache = new Map();

async function fetchWithMemo(url) {
  if (cache.has(url)) {
    return cache.get(url); // Return cached data
  }
  
  const response = await fetch(url);
  const data = await response.json();
  cache.set(url, data); // Store in cache
  return data;
}
```

---

## ⚛️ **2. React Cache (React 18+)**

React Cache refers to React's built-in and community caching solutions for data fetching. React 18+ integrates caching concepts with Suspense. Libraries like React Query and SWR provide comprehensive caching systems. They handle request deduplication, background updates, and cache synchronization across components. React Cache typically stores data by unique query keys, manages cache lifetimes, and provides hooks for data access. This allows multiple components to share cached data without prop drilling. Cache strategies include time-based expiration, dependency-based invalidation, and manual cache control. Proper cache configuration significantly improves application responsiveness.

React introduced built-in caching mechanisms for data fetching.

### Key Features:
- **Automatic caching** during render
- **Same request deduplication** - identical requests made simultaneously are fetched once
- **Integrated with Suspense** for loading states

### Example with `react-cache` (experimental):
```javascript
import { unstable_createResource } from 'react-cache';

const fetchUser = unstable_createResource(
  (userId) => fetch(`/api/users/${userId}`).then(res => res.json())
);

function UserProfile({ userId }) {
  const user = fetchUser.read(userId); // Cached per userId
  return <div>{user.name}</div>;
}
```

### React Query / TanStack Query (Popular Library):
```javascript
import { useQuery } from '@tanstack/react-query';

function UserComponent() {
  const { data, isLoading } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    staleTime: 5 * 60 * 1000, // Cache for 5 minutes
  });
}
```

---

## 🔄 **3. Revalidating Cached Data**

Revalidation updates stale cached data to ensure freshness while maintaining performance benefits. Strategies include: time-based intervals, event-based triggers (window focus, network reconnect), and manual invalidation. Stale-while-revalidate patterns serve cached data immediately while fetching updates in background. Optimistic updates modify cache immediately before server confirmation, rolling back on failure. Revalidation balances speed with accuracy—critical for collaborative or frequently changing data. Implementation involves cache timestamps, versioning, or ETag headers. Smart revalidation minimizes unnecessary requests while keeping data current. Most caching libraries provide built-in revalidation options.

**Revalidation** means updating cached data to ensure freshness.

### When to revalidate:
1. **Time-based** (stale-while-revalidate)
2. **On focus** (window refocus)
3. **On network reconnect**
4. **On-demand** (manual trigger)

### Revalidation Strategies:

#### A. **Time-based Revalidation**
```javascript
// Using SWR library example
import useSWR from 'swr';

function Profile() {
  const { data } = useSWR('/api/user', fetcher, {
    refreshInterval: 3000, // Revalidate every 3 seconds
    revalidateOnFocus: true, // Revalidate when window gains focus
  });
}
```

#### B. **Mutate and Revalidate** (Manual trigger)
```javascript
// After a POST/PUT request, revalidate related queries
const { mutate } = useSWR('/api/data', fetcher);

const updateData = async (newData) => {
  await fetch('/api/data', {
    method: 'POST',
    body: JSON.stringify(newData)
  });
  mutate(); // Trigger revalidation
};
```

#### C. **Optimistic Updates**
```javascript
mutate('/api/todos', async (todos) => {
  const newTodo = { id: Date.now(), text: 'New task' };
  
  // Update cache immediately
  const optimisticTodos = [...todos, newTodo];
  
  // Sync with server
  await fetch('/api/todos', { method: 'POST', body: JSON.stringify(newTodo) });
  
  return optimisticTodos;
}, {
  rollbackOnError: true, // Revert if request fails
});
```

---

## ❌ **4. Revalidation Errors**


Revalidation errors occur when updating cached data fails. Common causes: network issues, server errors, or data conflicts. Handling strategies include: retry logic with exponential backoff, fallback to stale data, user notifications, and graceful degradation. Error boundaries can catch and display cache failures. Retry mechanisms should consider error types—don't retry permanent 404s indefinitely. Maintain stale data display during retries rather than showing errors. Implement circuit breakers to prevent cascading failures. Log revalidation failures for monitoring. Good error handling ensures robust user experience despite intermittent network problems or API downtime.

Handling errors during revalidation is crucial for robust applications.

### Common Error Scenarios:
1. **Network failures**
2. **API response errors**
3. **Stale data conflicts**
4. **Rate limiting**

### Error Handling Strategies:

#### A. **Retry Logic**
```javascript
const { data, error } = useSWR('/api/data', fetcher, {
  onErrorRetry: (error, key, config, revalidate, { retryCount }) => {
    // Don't retry on 404
    if (error.status === 404) return;
    
    // Only retry up to 3 times
    if (retryCount >= 3) return;
    
    // Retry after 1.5 seconds
    setTimeout(() => revalidate({ retryCount: retryCount + 1 }), 1500);
  }
});
```

#### B. **Error Boundaries for Cache Errors**
```javascript
class CacheErrorBoundary extends React.Component {
  state = { hasError: false };
  
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
  
  render() {
    if (this.state.hasError) {
      return <FallbackUI />;
    }
    return this.props.children;
  }
}

// Usage
<CacheErrorBoundary>
  <ComponentWithDataFetching />
</CacheErrorBoundary>
```

#### C. **Fallback Data & Loading States**
```javascript
function DataComponent() {
  const { data, error, isLoading } = useQuery({
    queryKey: ['data'],
    queryFn: fetchData,
    retry: 2,
    initialData: () => {
      // Return cached data from localStorage as fallback
      const cached = localStorage.getItem('data-cache');
      return cached ? JSON.parse(cached) : undefined;
    }
  });

  if (isLoading) return <LoadingSkeleton />;
  if (error) return <ErrorDisplay error={error} />;
  return <DisplayData data={data} />;
}
```

#### D. **Exponential Backoff for Revalidation**
```javascript
async function revalidateWithBackoff(key, fetcher, options = {}) {
  const maxRetries = options.maxRetries || 3;
  const baseDelay = options.baseDelay || 1000;
  
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fetcher(key);
    } catch (error) {
      if (attempt === maxRetries) throw error;
      
      const delay = baseDelay * Math.pow(2, attempt); // Exponential backoff
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}
```

---

## 🏆 **Advanced Patterns**

### 1. **Cache Invalidation with Tags**
```javascript
// Group related queries with tags
const queryClient = useQueryClient();

// Invalidate all queries with 'users' tag
queryClient.invalidateQueries({ queryKey: ['users'] });

// More specific invalidation
queryClient.invalidateQueries({
  queryKey: ['user', userId],
  exact: true,
});
```

### 2. **Persistent Cache (LocalStorage)**
```javascript
const localStorageCache = {
  get: (key) => {
    const item = localStorage.getItem(`cache-${key}`);
    return item ? JSON.parse(item) : null;
  },
  set: (key, value) => {
    localStorage.setItem(`cache-${key}`, JSON.stringify(value));
  },
  delete: (key) => {
    localStorage.removeItem(`cache-${key}`);
  }
};
```

### 3. **Custom Cache Implementation**
```javascript
class CustomCache {
  constructor(maxSize = 100) {
    this.cache = new Map();
    this.maxSize = maxSize;
  }

  get(key) {
    if (!this.cache.has(key)) return null;
    
    const item = this.cache.get(key);
    // Move to end (most recently used)
    this.cache.delete(key);
    this.cache.set(key, item);
    
    return item;
  }

  set(key, value) {
    // Remove oldest if at capacity
    if (this.cache.size >= this.maxSize) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
    
    this.cache.set(key, {
      value,
      timestamp: Date.now(),
    });
  }

  clearStale(maxAge = 5 * 60 * 1000) { // 5 minutes
    const now = Date.now();
    for (const [key, item] of this.cache.entries()) {
      if (now - item.timestamp > maxAge) {
        this.cache.delete(key);
      }
    }
  }
}
```

---

## 📊 **Summary Table**

| Concept | Purpose | Key Benefit |
|---------|---------|-------------|
| **Memoization** | Store fetch results | Avoid duplicate requests |
| **React Cache** | Built-in React caching | Seamless integration |
| **Revalidation** | Update stale data | Data freshness |
| **Error Handling** | Graceful failures | Better UX during errors |

---

## 🔧 **Best Practices**

1. **Set appropriate stale times** based on data volatility
2. **Implement retry logic** for transient failures
3. **Use optimistic updates** for better UX
4. **Clear cache periodically** to prevent memory issues
5. **Monitor cache hit rates** for optimization
6. **Provide loading states** during revalidation
7. **Implement offline support** with persistent cache

---

## 📱 **Quick Reference**

```javascript
// Complete example with all concepts
const { data, error, isLoading, mutate } = useSWR('/api/data', fetcher, {
  // Caching
  dedupingInterval: 2000,
  
  // Revalidation
  refreshInterval: 30000,
  revalidateOnFocus: true,
  revalidateOnReconnect: true,
  
  // Error handling
  shouldRetryOnError: true,
  errorRetryInterval: 5000,
  errorRetryCount: 3,
  
  // Fallbacks
  fallbackData: initialData,
  onError: (err) => console.error('Fetch failed:', err),
  
  // Optimization
  revalidateIfStale: true,
  keepPreviousData: true,
});
```

This comprehensive approach to caching and memoization ensures your React applications are **fast, reliable, and efficient** while maintaining data consistency across network conditions.