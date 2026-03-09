# **Next.js Advanced Routing - Complete Guide**

## **Static vs Dynamic**
 
**Static content** is pre-built at deployment and served identically to all users, offering maximum speed. 
**Dynamic content** is generated on-demand per request, allowing personalization. Next.js enables a hybrid approach: use static for unchanging pages (blogs, documentation) and dynamic for user-specific data (profiles, dashboards). You can mix both in one app, choosing per page. Static pages can be revalidated periodically to update content without rebuilding the entire site. This flexibility optimizes performance while maintaining fresh, relevant content where needed.
**Static** = Fixed, pre-made content  
**Dynamic** = Changes, personalized content  

**Static Pages:**
- Made once at build time
- Fast as lightning ⚡
- Great for: Blogs, docs, marketing pages
- Example: "About Us" page that never changes

**Dynamic Pages:**
- Made fresh for each request
- Personalized content 👤
- Great for: User dashboards, shopping carts
- Example: "Your Profile" page showing your data

```jsx
// Static (SSG) - built once
export default function AboutPage() {
  return <h1>About Our Company</h1>; // Same for everyone
}

// Dynamic (SSR) - fresh each time
export default async function ProfilePage() {
  const user = await getCurrentUser(); // Different per user
  return <h1>Welcome back, {user.name}!</h1>;
}
```

---

## **Caching**

**Caching** stores data at various levels to eliminate redundant work and accelerate delivery. Next.js implements multiple cache layers: data cache for fetched information, full route cache for rendered pages, and router cache for client-side navigation. Smart defaults ensure good performance out-of-the-box, while configuration options allow fine-tuning. Caching reduces server load, decreases latency, and improves user experience. Understanding cache invalidation strategies (time-based revalidation, on-demand revalidation) is key to balancing freshness with efficiency.
**Caching** = Storing data to reuse later = **No waiting!**  

**Think of it like:**  
Your brain remembering answers to save time

**Types of caching in Next.js:**
1. **Request Memoization** = Remember database calls
2. **Data Cache** = Remember full data results
3. **Full Route Cache** = Remember complete pages
4. **Router Cache** = Remember visited pages in browser

```jsx
// With caching - FAST ⚡
export default async function ProductPage() {
  // First time: ⏱️ Fetch from database
  // Next times: ⚡ Use cached version
  const product = await fetchProduct(id); // Auto-cached!
  return <div>{product.name}</div>;
}

// Control caching
export default async function Page() {
  const data = await fetch('https://api.com/data', {
    next: { revalidate: 3600 } // Refresh every hour
  });
}
```

---

## **Streaming**

**Streaming** progressively sends page content to users as it becomes ready, rather than waiting for everything. This improves perceived performance, especially for pages with slower data dependencies. React Suspense boundaries define where loading states appear. Next.js streamlines implementation with special files like `loading.js`. Users see immediate feedback (skeletons, spinners) while waiting for specific components. Streaming transforms user experience from "waiting for complete page" to "engaging with available content," reducing bounce rates and increasing satisfaction.

**Streaming** = Send page pieces as they're ready (not waiting for everything)

**Normal loading:**  
1. Wait for ALL data ⏳  
2. Show complete page ✅

**Streaming:**  
1. Show header NOW ⚡  
2. Show main content ⏳  
3. Show sidebar LATER ⏳

```jsx
// app/page.js
export default function Home() {
  return (
    <>
      <Header /> {/* Shows immediately */}
      
      {/* Shows loading spinner while waiting */}
      <Suspense fallback={<LoadingSpinner />}>
        <ProductList /> {/* Takes 2 seconds */}
      </Suspense>
      
      <Footer /> {/* Shows immediately */}
    </>
  );
}
```

---

## **Redirects**

**Redirects** automatically route users from one URL to another, essential for site maintenance, reorganization, and user experience. Next.js supports redirects at configuration level (in `next.config.js`) for site-wide rules and programmatically in components for conditional logic. Permanent redirects (301) signal search engines to update indexes; temporary redirects (302) handle short-term changes. Proper redirect management preserves SEO equity during site migrations and ensures users always reach relevant content, even when URLs change.

**Redirect** = Automatically send user to different page

**Use when:**
- Page moved permanently (301)
- Temporary maintenance (302)
- Authentication redirects
- Legacy URL support

```jsx
// In next.config.js
module.exports = {
  redirects: async () => [
    {
      source: '/old-about',
      destination: '/about',
      permanent: true, // 301 redirect
    },
    {
      source: '/login',
      destination: '/auth/login',
      permanent: false, // 302 redirect
    },
  ],
};

// In component/server action
import { redirect } from 'next/navigation';

function checkAuth() {
  if (!user) {
    redirect('/login'); // Send to login
  }
}
```

---

## **API Endpoints**

**API Endpoints** enable full-stack development within Next.js by handling backend logic directly. Located in `app/api/` or `pages/api/`, these routes process HTTP requests, interact with databases, and return responses. This architecture eliminates separate backend services for many use cases. Endpoints support all HTTP methods and integrate seamlessly with frontend components. They benefit from Next.js optimizations like serverless deployment and automatic scaling, providing a cohesive development experience from database to UI.

**API Endpoints** = Backend code in your Next.js app

**Location:** `app/api/route.js` files  
**Purpose:** Handle form submissions, database updates, authentication

```jsx
// app/api/users/route.js
export async function GET(request) {
  // Handle GET request (read data)
  return Response.json({ users: ['Alice', 'Bob'] });
}

export async function POST(request) {
  // Handle POST request (create data)
  const data = await request.json();
  return Response.json({ message: 'User created!' });
}

// From frontend component
async function fetchUsers() {
  const response = await fetch('/api/users');
  const data = await response.json();
  console.log(data.users);
}
```

---

## **Parallel Routes**

**Parallel Routes** render multiple independent pages simultaneously within the same layout, like dashboard panels or split views. Each route segment loads in isolation with its own error and loading states. This enables complex UIs where different sections update independently without affecting others. Implementation uses named "slots" (`@folder`) in the layout to position each parallel route. This pattern maximizes performance for modular interfaces while maintaining clean separation of concerns between distinct application sections.


**Parallel Routes** = Show multiple pages at same time on same URL

**Think of it like:** Dashboard with multiple panels that load independently

```jsx
// app/dashboard/@analytics/page.js → Analytics panel
// app/dashboard/@revenue/page.js → Revenue panel
// app/dashboard/@users/page.js → Users panel

// app/dashboard/layout.js
export default function DashboardLayout({
  children,
  analytics,
  revenue,
  users,
}) {
  return (
    <div className="dashboard">
      <div className="main">{children}</div>
      <div className="sidebar">
        {analytics} {/* Loads independently */}
        {revenue}   {/* Loads independently */}
        {users}     {/* Loads independently */}
      </div>
    </div>
  );
}
```

**Access at:** `/dashboard` shows ALL panels together!

---

## **Intercepting Routes**

**Intercepting Routes** capture navigation requests and display targeted content in a layered context (like modals) without leaving the current page. Users experience seamless transitions—clicking a photo opens it in an overlay rather than navigating away. The original page remains visible in the background, preserving context. Implemented with special folder naming conventions (`(.)`, `(..)`), this pattern enhances user experience for previews, quick edits, and detailed views, making applications feel more responsive and fluid.


**Intercepting** = Show content in a modal/overlay without leaving current page

**Before:** Click link → New page loads (leave current)  
**After:** Click link → Modal opens (stay on current)

```jsx
// Structure:
app/
├── @modal/           ← Intercepting slot
│   └── (.)photo/     ← (. ) means intercept
│       └── [id]/
│           └── page.js
└── photo/
    └── [id]/
        └── page.js

// Click /photo/123 normally → Full page
// Click /photo/123 from gallery → Modal overlay!
```

**Perfect for:** Photo galleries, quick edits, previews

---

## **Routing Patterns**

**Routing Patterns** are organizational strategies for complex applications. Route groups (`(folder)`) logically organize files without affecting URL structure. Dynamic segments (`[param]`) handle variable data. Catch-all segments (`[...slug]`) accommodate flexible paths. Optional catch-all segments (`[[...slug]]`) support optional parameters. These patterns, combined with nested layouts and middleware, enable scalable, maintainable application structures. They allow developers to match URL design to information architecture while keeping code organization clean and predictable.

Common ways to organize complex applications:

### 1. **Grouped Routes** - Organize without changing URL
```jsx
app/
├── (auth)/           ← Group (no URL change)
│   ├── login/
│   └── register/
└── (shop)/
    ├── products/
    └── cart/
// URLs: /login, /products (no /auth/login)
```

### 2. **Dynamic Routes** - Handle changing data
```jsx
app/blog/[slug]/page.js       → /blog/my-post
app/users/[id]/page.js        → /users/123
app/category/[...slug]/page.js → /category/electronics/phones
```

### 3. **Route Handlers** - API + pages combined
```jsx
app/products/[id]/
├── page.js      // View product
├── edit/
│   └── page.js  // Edit product
└── route.js     // API for this product
```

---

## **Key Takeaways**

1. **Static** = Fixed (fast) | **Dynamic** = Changing (fresh)
2. **Caching** = Remember to save time
3. **Streaming** = Send pieces as ready
4. **Redirects** = Automatic page forwarding
5. **API Endpoints** = Backend in your frontend
6. **Parallel Routes** = Multiple panels on one page
7. **Intercepting Routes** = Modal without leaving
8. **Routing Patterns** = Smart organization methods

**Real-world analogy:**  
Think of a **shopping mall**:
- **Static** = Mall directory (fixed)
- **Dynamic** = Your cart (changes)
- **Caching** = Remembering store locations
- **Streaming** = Food court serving as ready
- **Parallel Routes** = Mall with multiple floors
- **Intercepting** = Quick kiosk without leaving mall