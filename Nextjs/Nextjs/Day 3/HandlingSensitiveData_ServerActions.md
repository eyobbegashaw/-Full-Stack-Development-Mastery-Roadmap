# Handling Sensitive Data, Server Actions & Data Fetching Patterns

## 🔐 **1. Handling Sensitive Data**

Never expose sensitive data (API keys, credentials) in client-side code. Use environment variables (server-side only), backend proxies, and HTTP-only cookies. Validate and sanitize all inputs. Encrypt data before transmission. Store secrets securely using services like Vault or AWS Secrets Manager. Never log sensitive information. Implement proper CORS policies and use HTTPS. Treat all client-side data as potentially visible to users. Centralize secret management for rotation and auditing.


### **The Problem**
Sensitive data includes:
- API keys/tokens
- User credentials (passwords)
- Payment information
- Personal Identifiable Information (PII)
- Private business logic

### **Client-Side Risks**
```javascript
// ❌ BAD - Exposes API key in client bundle
const API_KEY = 'sk_live_1234567890';
fetch(`https://api.service.com?key=${API_KEY}`);

// ❌ BAD - Token in localStorage (accessible via XSS)
localStorage.setItem('token', 'eyJhbGci...');
```

### **Secure Approaches**

#### **A. Environment Variables**
```javascript
// .env.local (NEVER committed to git)
REACT_APP_API_KEY=your_actual_key_here
NEXT_PUBLIC_API_KEY=public_key_only

// In code
const apiKey = process.env.REACT_APP_API_KEY; // Client-side
const secretKey = process.env.API_KEY; // Server-side only (Next.js)
```

#### **B. Backend Proxy Pattern**
```javascript
// ✅ GOOD - Client calls your backend, backend calls external API
// Client-side
fetch('/api/proxy/weather', {
  method: 'POST',
  body: JSON.stringify({ city: 'London' })
});

// Server-side (Node.js/Next.js API route)
export default async function handler(req, res) {
  const API_KEY = process.env.WEATHER_API_KEY; // Server-only
  const response = await fetch(
    `https://api.weatherapi.com/v1/current.json?key=${API_KEY}&q=${req.body.city}`
  );
  const data = await response.json();
  res.status(200).json(data);
}
```

#### **C. HTTP-only Cookies for Authentication**
```javascript
// Server sets secure, HTTP-only cookie
res.setHeader('Set-Cookie', [
  `token=${encryptedToken}; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=3600`
]);

// Client automatically includes cookie in requests
fetch('/api/protected-data'); // Cookie sent automatically
```

#### **D. Data Encryption**
```javascript
// Client-side encryption before sending sensitive data
import CryptoJS from 'crypto-js';

const encryptData = (data, secretKey) => {
  return CryptoJS.AES.encrypt(JSON.stringify(data), secretKey).toString();
};

// Send encrypted
fetch('/api/submit', {
  method: 'POST',
  body: JSON.stringify({
    encrypted: encryptData({ ssn: '123-45-6789' }, 'encryption-key')
  })
});
```

#### **E. Input Sanitization & Validation**
```javascript
// Sanitize user inputs
const sanitizeInput = (input) => {
  return input
    .replace(/[<>]/g, '') // Remove HTML tags
    .trim()
    .substring(0, 255); // Limit length
};

// Validate sensitive data patterns
const isValidSSN = (ssn) => {
  const ssnPattern = /^\d{3}-\d{2}-\d{4}$/;
  return ssnPattern.test(ssn) && !isFakeSSN(ssn);
};
```

---

## ⚡ **2. Server Actions (Next.js 13+)**

Next.js Server Actions enable secure server-side functions callable from client components. Marked with `'use server'`, they execute on the server, protecting sensitive logic. They support form handling, mutations, and revalidation without API routes. Integrate with `useFormState` for feedback and `useOptimistic` for instant UI updates. Server Actions simplify data mutations while maintaining security and type safety. They progressively enhance forms when JavaScript is disabled.


### **What are Server Actions?**
Functions that execute securely on the server, called from client components.

### **Benefits**
- No API routes needed
- Type-safe
- Built-in security
- Progressive enhancement
- Revalidation support

### **Basic Implementation**
```javascript
// app/actions/user-actions.js (Server Component)
'use server'; // Marks as Server Action

import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';

export async function createUser(formData) {
  const rawFormData = {
    name: formData.get('name'),
    email: formData.get('email'),
  };
  
  // Validate input
  if (!isValidEmail(rawFormData.email)) {
    return { error: 'Invalid email' };
  }
  
  // Server-side operations (safe for sensitive data)
  const userId = await db.user.create({
    data: {
      name: rawFormData.name,
      email: rawFormData.email,
      password: await hashPassword(formData.get('password')), // Safe!
    },
  });
  
  // Revalidate cache and redirect
  revalidatePath('/users');
  redirect(`/users/${userId}`);
}
```

### **Client Component Usage**
```javascript
// app/components/user-form.js (Client Component)
'use client';

import { createUser } from '@/app/actions/user-actions';
import { useFormState } from 'react-dom';

export function UserForm() {
  const [state, formAction] = useFormState(createUser, null);
  
  return (
    <form action={formAction}>
      <input name="name" placeholder="Name" />
      <input name="email" type="email" placeholder="Email" />
      <input name="password" type="password" placeholder="Password" />
      <button type="submit">Create User</button>
      {state?.error && <p className="error">{state.error}</p>}
    </form>
  );
}
```

### **Advanced Patterns**

#### **A. Optimistic Updates with Server Actions**
```javascript
// app/actions/todo-actions.js
'use server';

let todos = []; // In real app, use database

export async function addTodoOptimistic(prevState, formData) {
  const text = formData.get('text');
  
  // Immediate optimistic update
  const newTodo = { id: Date.now(), text, completed: false };
  
  try {
    // Simulate server delay
    await new Promise(resolve => setTimeout(resolve, 1000));
    
    // Actual server operation
    todos.push({ id: Date.now(), text, completed: false });
    
    return { success: true, todo: newTodo };
  } catch (error) {
    return { success: false, error: 'Failed to add todo' };
  }
}

// Client usage with useOptimistic
'use client';
import { useOptimistic } from 'react';
import { addTodoOptimistic } from './actions';

function TodoList({ todos }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo) => [...state, newTodo]
  );
  
  const formAction = async (formData) => {
    const newTodo = { id: Date.now(), text: formData.get('text') };
    addOptimisticTodo(newTodo);
    await addTodoOptimistic(null, formData);
  };
  
  return (
    <form action={formAction}>
      {/* Render optimisticTodos immediately */}
    </form>
  );
}
```

#### **B. File Uploads with Server Actions**
```javascript
// app/actions/upload-action.js
'use server';

export async function uploadFile(formData) {
  const file = formData.get('file');
  
  if (!file) {
    return { error: 'No file provided' };
  }
  
  // Validate file type and size
  const allowedTypes = ['image/jpeg', 'image/png', 'application/pdf'];
  const maxSize = 5 * 1024 * 1024; // 5MB
  
  if (!allowedTypes.includes(file.type)) {
    return { error: 'Invalid file type' };
  }
  
  if (file.size > maxSize) {
    return { error: 'File too large' };
  }
  
  // Process file on server (never trust client data)
  const bytes = await file.arrayBuffer();
  const buffer = Buffer.from(bytes);
  
  // Save securely
  const fileName = `/uploads/${Date.now()}-${file.name}`;
  await fs.writeFile(`./public${fileName}`, buffer);
  
  return { success: true, url: fileName };
}
```

---

## 🔄 **3. Data Fetching Patterns**


Common patterns include: Fetch-on-Render (useEffect), Fetch-Then-Render (server components), and Render-as-You-Fetch (Suspense). Choose based on needs: Fetch-on-Render for dynamic client data, Fetch-Then-Render for server-rendered pages, and Suspense for complex, nested loading states. Consider waterfall vs parallel approaches. Libraries like React Query abstract these patterns. Implement error boundaries and loading states for each approach.

### **Pattern 1: Render-as-You-Fetch**
```javascript
// Start fetching early, render when ready
function UserProfile({ userId }) {
  const userPromise = fetchUser(userId); // Start fetching
  const postsPromise = fetchUserPosts(userId); // Start fetching
  
  return (
    <Suspense fallback={<ProfileSkeleton />}>
      <UserDetails userPromise={userPromise} />
      <Suspense fallback={<PostsSkeleton />}>
        <UserPosts postsPromise={postsPromise} />
      </Suspense>
    </Suspense>
  );
}
```

### **Pattern 2: Fetch-on-Render**
```javascript
// Traditional approach - fetch in useEffect
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetchUser(userId).then(data => {
      setUser(data);
      setLoading(false);
    });
  }, [userId]);
  
  if (loading) return <Loader />;
  return <div>{user.name}</div>;
}
```

### **Pattern 3: Fetch-Then-Render**
```javascript
// Server Components pattern
async function UserProfile({ userId }) {
  const user = await fetchUser(userId); // Server-side fetch
  const posts = await fetchUserPosts(userId);
  
  return (
    <>
      <h1>{user.name}</h1>
      <PostsList posts={posts} />
    </>
  );
}
```

---

## ⚖️ **4. Parallel vs Sequential Fetching**

Parallel fetching executes multiple requests simultaneously using `Promise.all()`, reducing total time to the slowest request. Sequential fetches run one after another, simpler but slower. Use parallel for independent data, sequential when requests depend on previous results. Balance with controlled concurrency to avoid overwhelming servers. Handle race conditions in parallel flows. Measure performance to choose the optimal strategy for your use case.


### **Sequential Fetching**
```javascript
// One after another - slower but simpler
async function getUserData(userId) {
  const user = await fetchUser(userId); // Wait for completion
  const posts = await fetchUserPosts(userId); // Then fetch posts
  const friends = await fetchUserFriends(userId); // Then fetch friends
  
  return { user, posts, friends }; // Total time = sum of all fetches
}
```

### **Parallel Fetching**
```javascript
// All at once - faster but more complex
async function getUserDataParallel(userId) {
  // Start all fetches simultaneously
  const [user, posts, friends] = await Promise.all([
    fetchUser(userId),
    fetchUserPosts(userId),
    fetchUserFriends(userId)
  ]);
  
  return { user, posts, friends }; // Total time = longest fetch
}
```

### **Advanced: Controlled Parallelism**
```javascript
// Limit concurrent requests
async function fetchWithConcurrency(urls, maxConcurrent = 3) {
  const results = [];
  
  for (let i = 0; i < urls.length; i += maxConcurrent) {
    const batch = urls.slice(i, i + maxConcurrent);
    const batchResults = await Promise.all(
      batch.map(url => fetch(url).then(res => res.json()))
    );
    results.push(...batchResults);
  }
  
  return results;
}

// Race conditions handling
async function fetchWithRaceConditionProtection(userId) {
  let user, posts;
  
  // Fetch in parallel but handle dependencies
  await Promise.all([
    fetchUser(userId).then(result => { user = result; }),
    fetchUserPosts(userId).then(result => { posts = result; })
  ]);
  
  // Ensure user exists before processing posts
  if (!user) throw new Error('User not found');
  
  return { user, posts };
}
```

---

## 🚀 **5. Preloading Data**

Preloading initiates data fetch before it's needed, improving perceived performance. Techniques include: link hover/touch detection, route-based preloading, and predictive loading based on user patterns. Use the `preload` API or prefetch attributes. Implement priority queues for critical resources. Combine with Intersection Observer for viewport-based loading. Monitor preload effectiveness to avoid bandwidth waste. Works with images, scripts, and API data.

### **Why Preload?**
- Reduce perceived loading time
- Prepare data before user needs it
- Improve user experience

### **Preloading Techniques**

#### **A. Link Hover Preloading**
```javascript
import { preload } from 'react-dom';

function Navigation() {
  const handleMouseEnter = () => {
    // Preload data when user hovers over link
    preload('/api/user-data', { as: 'fetch' });
    preload('/api/user-posts', { as: 'fetch' });
  };
  
  return (
    <a 
      href="/profile" 
      onMouseEnter={handleMouseEnter}
      onTouchStart={handleMouseEnter} // Mobile touch
    >
      Profile
    </a>
  );
}
```

#### **B. Route-Based Preloading (Next.js)**
```javascript
// app/profile/page.js
import { preload } from 'react-dom';

async function ProfilePage() {
  // Preload data at component level
  preload('/api/user-stats', { as: 'fetch' });
  
  const user = await fetchUser();
  
  return (
    <>
      <UserProfile user={user} />
      {/* Stats will load faster */}
    </>
  );
}
```

#### **C. Priority-Based Preloading**
```javascript
class PreloadManager {
  constructor() {
    this.queue = [];
    this.inProgress = new Set();
  }
  
  async preload(resource, priority = 'low') {
    // High priority goes to front of queue
    if (priority === 'high') {
      this.queue.unshift({ resource, priority });
    } else {
      this.queue.push({ resource, priority });
    }
    
    this.processQueue();
  }
  
  async processQueue() {
    if (this.inProgress.size >= 3) return; // Limit concurrent
    
    const next = this.queue.shift();
    if (!next) return;
    
    this.inProgress.add(next.resource);
    try {
      await fetch(next.resource);
    } finally {
      this.inProgress.delete(next.resource);
      this.processQueue();
    }
  }
}
```

#### **D. Predictive Preloading**
```javascript
// Predict user's next action based on behavior
class PredictivePreloader {
  constructor() {
    this.userPatterns = new Map();
    this.currentPath = window.location.pathname;
  }
  
  trackNavigation(from, to) {
    if (!this.userPatterns.has(from)) {
      this.userPatterns.set(from, new Map());
    }
    
    const fromPatterns = this.userPatterns.get(from);
    fromPatterns.set(to, (fromPatterns.get(to) || 0) + 1);
  }
  
  predictNextPage() {
    const patterns = this.userPatterns.get(this.currentPath);
    if (!patterns) return null;
    
    // Return most frequent next page
    return Array.from(patterns.entries())
      .sort((a, b) => b[1] - a[1])[0]?.[0];
  }
  
  preloadPredicted() {
    const nextPage = this.predictNextPage();
    if (nextPage) {
      preload(nextPage, { as: 'fetch' });
    }
  }
}
```

### **E. Intersection Observer Preloading**
```javascript
// Preload when element comes into viewport
function LazyPreloadComponent({ resource }) {
  const ref = useRef();
  
  useEffect(() => {
    const observer = new IntersectionObserver((entries) => {
      if (entries[0].isIntersecting) {
        // Element is visible, preload data
        preload(resource, { as: 'fetch' });
        observer.disconnect();
      }
    }, { rootMargin: '200px' }); // Start 200px before visible
    
    observer.observe(ref.current);
    
    return () => observer.disconnect();
  }, [resource]);
  
  return <div ref={ref}>Content will load soon</div>;
}
```

---

## 🏗️ **Complete Example: Secure Dashboard**

```javascript
// app/dashboard/page.js (Server Component)
import { preload } from 'react-dom';
import DashboardClient from './dashboard-client';

async function DashboardPage() {
  // Preload sensitive data securely on server
  preload('/api/sensitive-data', { 
    as: 'fetch',
    // Server-only headers
    headers: { 'Authorization': `Bearer ${process.env.API_TOKEN}` }
  });
  
  // Fetch user data (parallel)
  const [userData, notifications] = await Promise.all([
    fetchUserData(),
    fetchNotifications()
  ]);
  
  return (
    <DashboardClient 
      initialUserData={userData}
      initialNotifications={notifications}
    />
  );
}

// app/dashboard/dashboard-client.js (Client Component)
'use client';

import { use } from 'react';

export default function DashboardClient({ 
  initialUserData, 
  initialNotifications 
}) {
  // Use preloaded data
  const sensitiveData = use(
    fetch('/api/sensitive-data').then(res => res.json())
  );
  
  // Server Action for secure operations
  async function updateProfile(formData) {
    'use server';
    // Secure server-side processing
    const sanitizedData = sanitizeInput(formData);
    await db.user.update(sanitizedData);
    revalidatePath('/dashboard');
  }
  
  return (
    <div>
      <SecureForm action={updateProfile} />
      <SensitiveDataDisplay data={sensitiveData} />
    </div>
  );
}
```

---

## 📊 **Comparison Table**

| Pattern | Use Case | Pros | Cons |
|---------|----------|------|------|
| **Sequential** | Dependent data | Simple, predictable | Slow |
| **Parallel** | Independent data | Fast, efficient | Complex error handling |
| **Preloading** | Predictable navigation | Better UX | Wasted bandwidth |
| **Server Actions** | Sensitive operations | Secure, type-safe | Next.js specific |

---

## 🔧 **Best Practices Summary**

1. **Always handle sensitive data on the server**
2. **Use Server Actions for mutations in Next.js**
3. **Fetch in parallel when possible**
4. **Preload data for critical paths**
5. **Implement proper error boundaries**
6. **Sanitize and validate ALL inputs**
7. **Use HTTP-only cookies for auth tokens**
8. **Limit concurrent requests to avoid rate limiting**
9. **Implement retry logic with exponential backoff**
10. **Monitor and optimize data fetching performance**

This comprehensive approach ensures your application is **secure, performant, and scalable** while providing an excellent user experience.