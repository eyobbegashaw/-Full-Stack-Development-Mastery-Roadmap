# **Next.js Routing - Notes & Concepts**

## **Routing**
Routing determines how users navigate between different pages in your web application. In Next.js, routing is **file-system based** - the structure of files in your project determines the routes in your app. This means you don't need to configure routes manually; Next.js automatically maps files to routes. It supports both **server components** (default) and **client components**, with built-in optimizations like automatic code splitting and prefetching.

```jsx
// Folder structure = Route structure
app/
├── page.js           → /
├── about/
│   └── page.js      → /about
└── blog/
    ├── page.js      → /blog
    └── [slug]/
        └── page.js  → /blog/:slug
```

## **Types of Routers**
Next.js has evolved through two main routing systems:

1. **Pages Router** (Legacy): The original routing system using `pages/` directory
2. **App Router** (Current): The newer, more powerful system using `app/` directory

The choice affects your entire project's architecture, data fetching patterns, and available features. While both work, the App Router represents Next.js's future direction with React Server Components and improved performance patterns.

```bash
# Project structure comparison
my-app/
├── pages/     # Old way (still works)
└── app/       # New way (recommended)
```

## **Pages Router (Legacy)**
The original Next.js router using file-based routing in a `pages/` directory. Each `.js`/`.tsx` file becomes a route. It uses **client-side routing** by default and requires manual configuration for server-side rendering per page. Simple but less powerful than the App Router.

```jsx
// pages/about.js - Simple page component
export default function AboutPage() {
  return <h1>About Us</h1>;
}

// Dynamic route: pages/blog/[slug].js
export default function BlogPost({ params }) {
  return <h1>Blog: {params.slug}</h1>;
}
```

## **App Router (Current)**
The modern routing system (Next.js 13+) using the `app/` directory. It introduces **React Server Components** by default, nested layouts, improved data fetching, and better performance. Routes are defined by folders containing `page.js` files, and it supports parallel routes, intercepting routes, and loading states.

```jsx
// app/dashboard/page.js - Server Component by default
export default async function DashboardPage() {
  const data = await fetchData(); // Runs on server
  return <Dashboard data={data} />;
}

// app/dashboard/settings/page.js → /dashboard/settings
// app/blog/[slug]/page.js → /blog/:slug (dynamic)
```

## **Why Use App Router?**
The App Router offers significant advantages over the Pages Router:

1. **Server Components by Default**: Better performance, smaller client bundles
2. **Nested Layouts**: Share UI between routes without remounting
3. **Improved Data Fetching**: Simpler async/await in components
4. **Enhanced Performance**: Streaming, React Suspense integration
5. **Better DX**: Loading states, error boundaries, parallel routes

```jsx
// App Router features in action
app/
├── layout.js           # Root layout (shared)
├── loading.js          # Automatic loading UI
├── error.js           # Error boundary
├── page.js            # Home page
└── dashboard/
    ├── layout.js      # Nested layout (persists)
    ├── page.js        # Dashboard page
    └── analytics/
        └── page.js    # /dashboard/analytics
```

## **Routing Basics**
Next.js routing revolves around special file names that create specific behaviors:

- `page.js` → The UI for a route
- `layout.js` → Shared UI for segments
- `loading.js` → Loading UI for Suspense
- `error.js` → Error boundary for segments
- `route.js` → API endpoint (App Router only)

Dynamic routes use square brackets (`[param]`), catch-all segments use ellipsis (`[...slug]`), and optional catch-all segments use double brackets (`[[...slug]]`). Navigation uses the `<Link>` component for client-side transitions.

```jsx
// Dynamic routes examples
app/shop/[category]/[product]/page.js    → /shop/electronics/phone
app/docs/[...slug]/page.js               → /docs/a/b/c (catches all segments)
app/search/[[...query]]/page.js          → /search OR /search/term (optional)

// Navigation between pages
import Link from 'next/link';

export default function Nav() {
  return (
    <nav>
      <Link href="/">Home</Link>
      <Link href="/about">About</Link>
      <Link href="/blog/nextjs-tips">Blog Post</Link>
    </nav>
  );
}
```

# **Next.js Routing Concepts - Simple Explanations**

## **Routing Terminology**
These are the key terms you'll encounter in Next.js routing:

- **Route**: A path/URL in your app (like `/about`, `/products/laptops`)
- **Segment**: Each part of a route separated by `/` (`/dashboard/analytics` has 2 segments)
- **Layout**: Shared UI that wraps pages (like header/footer)
- **Page**: The main UI component for a specific route
- **Dynamic Segment**: A route that changes, like `[userId]` or `[slug]`
- **Client Component**: Runs in the browser (use `'use client'` at top)
- **Server Component**: Runs on the server (default in App Router)
- **Nesting**: Layouts inside layouts, like Russian dolls

```jsx
// Route: /dashboard/analytics
app/
└── dashboard/          ← Segment 1
    ├── layout.js      ← Layout for /dashboard/*
    └── analytics/     ← Segment 2
        └── page.js    ← Page for /dashboard/analytics
```

## **Rendering Pages**
How pages get created and shown to users:

- **Static Generation (SSG)**: Page is made once at build time, then reused (fastest)
- **Server-Side Rendering (SSR)**: Page is made fresh on the server for each request
- **Client-Side Rendering (CSR)**: Page is made in the browser using JavaScript

In Next.js App Router, pages are **Server Components by default**, meaning they render on the server first. This is great for SEO and speed.

```jsx
// Server Component (default in App Router) - renders on server
export default async function ProductPage({ params }) {
  const product = await getProduct(params.id); // Server fetch
  return <h1>{product.name}</h1>; // HTML sent to browser
}

// Client Component - renders in browser
'use client';
export default function InteractivePage() {
  const [count, setCount] = useState(0); // Browser-only
  return <button onClick={() => setCount(count + 1)}>Count: {count}</button>;
}
```

## **Layouts and Templates**
Layouts wrap your pages to provide shared UI:

**Layout** = Shared wrapper that **persists** between page navigations (keeps state)
- Header stays visible when clicking links
- Sidebar remains open
- State isn't lost when changing pages

**Template** = Similar to layout but **re-creates** on each navigation
- Useful for animations between page transitions
- State resets when navigating

```jsx
// app/layout.js - ROOT LAYOUT (wraps EVERYTHING)
export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <Header />  ← Stays on all pages
        {children}  ← Page content changes
        <Footer />  ← Stays on all pages
      </body>
    </html>
  );
}

// app/dashboard/layout.js - NESTED LAYOUT
export default function DashboardLayout({ children }) {
  return (
    <div className="dashboard">
      <Sidebar />   ← Only shows on /dashboard/* pages
      <main>{children}</main>
    </div>
  );
}
```

## **Loading and Streaming**
How Next.js handles "loading states" while content is being fetched:

- **loading.js**: Special file that automatically shows a loading spinner/UI
- **Streaming**: Sending parts of the page as they become ready (not waiting for everything)
- **Suspense**: React feature that lets you "pause" rendering while waiting for data

```jsx
// app/dashboard/loading.js - AUTOMATIC LOADING UI
export default function Loading() {
  return <div className="spinner">Loading dashboard...</div>;
}

// In your page - Streaming with Suspense
export default function DashboardPage() {
  return (
    <div>
      <h1>Dashboard</h1>
      
      {/* This shows immediately */}
      <QuickStats />
      
      {/* This shows loading fallback while waiting */}
      <Suspense fallback={<div>Loading charts...</div>}>
        <SlowCharts />  ← Takes time to load
      </Suspense>
    </div>
  );
}
```

## **Error States**
How to handle when things go wrong:

- **error.js**: Special file that catches errors in its segment
- **Error Boundary**: A React component that catches JavaScript errors
- **Global Error**: Catches errors in root layout
- **Not Found**: For 404 pages with `not-found.js`

```jsx
// app/dashboard/error.js - ERROR BOUNDARY
'use client'; // Error components must be Client Components
export default function Error({ error, reset }) {
  return (
    <div className="error">
      <h2>Something went wrong!</h2>
      <p>{error.message}</p>
      <button onClick={reset}>Try Again</button>
    </div>
  );
}

// app/not-found.js - 404 PAGE
export default function NotFound() {
  return (
    <div>
      <h2>Page Not Found</h2>
      <p>Could not find the page you're looking for</p>
    </div>
  );
}

// Using in page
export default async function ProductPage({ params }) {
  const product = await getProduct(params.id);
  
  if (!product) {
    return notFound(); // Shows not-found.js
  }
  
  return <div>{product.name}</div>;
}
```

## **Quick Summary**
1. **Routing Terms**: Routes = URLs, Segments = parts of URL, Layouts = shared UI
2. **Rendering**: Server (fast, SEO-friendly) vs Client (interactive)
3. **Layouts**: Persistent wrappers that keep state between pages
4. **Loading**: Automatic spinners while data loads
5. **Error States**: Graceful handling when things break

This system makes Next.js apps fast by default - pages load quickly, shared parts don't re-render unnecessarily, and errors don't crash the whole app.