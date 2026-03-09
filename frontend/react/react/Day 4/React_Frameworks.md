# **🛠️ React Frameworks & Routing - Complete Guide**

## **🎯 Understanding the Framework Landscape**

### **Concept (70%)**
React **itself is a library**, not a framework. It needs to be paired with other tools to build complete applications. This has led to the creation of **meta-frameworks** that bundle React with routing, bundling, SSR, and other features.

**The React Framework Spectrum:**
```
        Library → Framework → Full-stack Framework
         (React)  (Next.js)      (Next.js + Backend)
```

**Why Use a Framework?**
1. **Batteries included** - Routing, bundling, SSR out of the box
2. **Best practices** - Framework guides you to good patterns
3. **Performance optimizations** - Built-in optimizations
4. **Developer experience** - Hot reloading, dev tools, error handling
5. **Production readiness** - Everything configured for deployment

---

## **1️⃣ React Router - The Client-Side Router**

### **Concept (70%)**
React Router is the **standard routing library** for React applications. It enables navigation between components while keeping the UI in sync with the URL.

**Core Philosophy:**
- **Declarative routing** - Routes are React components
- **Nested routing** - Parent/child route relationships
- **Dynamic routing** - URL parameters (`/users/:id`)
- **Client-side navigation** - No full page reloads

**When to Use React Router:**
- ✅ **Client-side apps** (Create React App, Vite)
- ✅ **SPAs** (Single Page Applications)
- ✅ **Simple routing needs**
- ❌ **SEO critical apps** (need SSR)
- ❌ **Complex data fetching** (need framework features)

**React Router v6 Changes:**
- `<Routes>` instead of `<Switch>`
- Relative links and routes
- `useNavigate` instead of `useHistory`
- `Outlet` for nested layouts

### **Code (30%)**
```jsx
// Basic React Router Setup
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
        <Link to="/users">Users</Link>
      </nav>
      
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/users" element={<Users />}>
          <Route index element={<UserList />} />
          <Route path=":userId" element={<UserProfile />} />
          <Route path=":userId/posts" element={<UserPosts />} />
        </Route>
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}

// Layout Pattern with Outlet
function DashboardLayout() {
  return (
    <div>
      <DashboardHeader />
      <Sidebar />
      {/* Child routes render here */}
      <Outlet />
      <DashboardFooter />
    </div>
  );
}

function App() {
  return (
    <Routes>
      <Route path="/dashboard" element={<DashboardLayout />}>
        <Route index element={<DashboardHome />} />
        <Route path="analytics" element={<Analytics />} />
        <Route path="settings" element={<Settings />} />
      </Route>
    </Routes>
  );
}

// Programmatic Navigation
import { useNavigate, useLocation } from 'react-router-dom';

function LoginPage() {
  const navigate = useNavigate();
  const location = useLocation();
  
  const handleLogin = async () => {
    await login();
    
    // Redirect to where they came from, or home
    const from = location.state?.from?.pathname || '/';
    navigate(from, { replace: true });
  };
  
  return (
    <div>
      <button onClick={handleLogin}>Login</button>
    </div>
  );
}

// Protected Routes Pattern
function RequireAuth({ children }) {
  const auth = useAuth();
  const location = useLocation();
  
  if (!auth.user) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  return children;
}

// Usage
<Route 
  path="/profile" 
  element={
    <RequireAuth>
      <Profile />
    </RequireAuth>
  } 
/>

// Data Loading with Loaders (v6.4+)
import { createBrowserRouter, RouterProvider } from 'react-router-dom';

const router = createBrowserRouter([
  {
    path: '/',
    element: <Root />,
    loader: () => fetch('/api/data'), // Runs before component
    children: [
      {
        path: 'user/:id',
        element: <User />,
        loader: async ({ params }) => {
          return fetch(`/api/users/${params.id}`);
        }
      }
    ]
  }
]);

function App() {
  return <RouterProvider router={router} />;
}

// Access loader data
function User() {
  const user = useLoaderData();
  return <div>{user.name}</div>;
}
```

---

## **2️⃣ Next.js - The Full-Stack Framework**

### **Concept (70%)**
Next.js is a **React framework for production** created by Vercel. It provides **server-side rendering, static site generation, API routes, and more** out of the box.

**Next.js Core Features:**
- 🎯 **File-based routing** - Pages based on file system
- 🌐 **Hybrid rendering** - SSR, SSG, ISR, CSR all in one
- ⚡ **Image optimization** - Automatic image optimization
- 🔄 **API routes** - Build API endpoints alongside pages
- 📦 **Zero config** - Works out of the box
- 🚀 **Fast refresh** - Instant feedback during development

**When to Choose Next.js:**
- ✅ **SEO-critical applications**
- ✅ **Marketing websites** (blogs, e-commerce)
- ✅ **Full-stack applications** (need backend API)
- ✅ **Performance-focused apps**
- ✅ **Large, complex applications**

**Next.js vs React Router:**
- **Next.js**: Full framework with SSR, API routes, optimizations
- **React Router**: Just routing for client-side apps

### **Code (30%)**
```jsx
// pages/index.js - File-based routing
export default function HomePage() {
  return <h1>Welcome to Next.js!</h1>;
}

// pages/about.js
export default function AboutPage() {
  return <h1>About Us</h1>;
}

// pages/blog/[slug].js - Dynamic route
export default function BlogPost({ post }) {
  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}

export async function getStaticProps({ params }) {
  const post = await fetchPost(params.slug);
  return { props: { post } };
}

export async function getStaticPaths() {
  const posts = await fetchAllPosts();
  const paths = posts.map(post => ({
    params: { slug: post.slug }
  }));
  
  return { paths, fallback: false };
}

// Data Fetching Patterns
// 1. Static Site Generation (SSG)
export async function getStaticProps() {
  const data = await fetchData();
  return { props: { data } };
}

// 2. Server-Side Rendering (SSR)
export async function getServerSideProps(context) {
  const { req, res, query } = context;
  const data = await fetchData(query);
  return { props: { data } };
}

// 3. Client-Side Fetching (CSR)
import { useEffect, useState } from 'react';

function ClientComponent() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetchData().then(setData);
  }, []);
  
  return <div>{data}</div>;
}

// 4. Incremental Static Regeneration (ISR)
export async function getStaticProps() {
  const data = await fetchData();
  return {
    props: { data },
    revalidate: 60, // Regenerate every 60 seconds
  };
}

// API Routes
// pages/api/users.js
export default function handler(req, res) {
  if (req.method === 'GET') {
    // Get users
    const users = await getUsers();
    res.status(200).json(users);
  } else if (req.method === 'POST') {
    // Create user
    const user = await createUser(req.body);
    res.status(201).json(user);
  }
}

// Using the API route
export async function getStaticProps() {
  const res = await fetch(`${process.env.NEXT_PUBLIC_API_URL}/api/users`);
  const users = await res.json();
  return { props: { users } };
}

// Layout Pattern (app router)
// app/layout.js
export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        <Header />
        <main>{children}</main>
        <Footer />
      </body>
    </html>
  );
}

// app/page.js
export default function HomePage() {
  return <h1>Home</h1>;
}

// Image Optimization
import Image from 'next/image';

function OptimizedImage() {
  return (
    <Image
      src="/profile.jpg"
      alt="Profile"
      width={500}
      height={300}
      priority // Load immediately
      sizes="(max-width: 768px) 100vw, 50vw"
    />
  );
}

// Dynamic Imports
import dynamic from 'next/dynamic';

const HeavyComponent = dynamic(
  () => import('../components/HeavyComponent'),
  { 
    loading: () => <p>Loading...</p>,
    ssr: false // Don't render on server
  }
);

function Page() {
  return <HeavyComponent />;
}

// Middleware (Next.js 12+)
// middleware.js
import { NextResponse } from 'next/server';

export function middleware(request) {
  const token = request.cookies.get('token');
  
  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
  
  return NextResponse.next();
}

export const config = {
  matcher: '/dashboard/:path*',
};
```

---

## **3️⃣ Astro - The Content-Focused Framework**

### **Concept (70%)**
Astro is a **content-focused web framework** that ships **zero JavaScript by default**. You can use React (or other frameworks) components inside Astro, but they're **opt-in for interactivity**.

**Astro's Unique Approach:**
- 🚀 **Zero JS by default** - Ships HTML + CSS only
- 🎯 **Island architecture** - Interactive components are isolated
- ⚡ **Blazing fast** - No hydration cost for static content
- 🔧 **Multi-framework** - Use React, Vue, Svelte in same project
- 📦 **Partial hydration** - Only hydrate what's needed

**When to Choose Astro:**
- ✅ **Content websites** (blogs, documentation, portfolios)
- ✅ **Marketing sites** where speed is critical
- ✅ **When you need minimal JavaScript**
- ✅ **Multi-framework projects**
- ❌ **Highly interactive apps** (like dashboards)

**Astro vs Next.js:**
- **Astro**: Content-focused, minimal JS, multi-framework
- **Next.js**: App-focused, full React, more features

### **Code (30%)**
```astro
<!-- pages/index.astro -->
---
// Frontmatter (runs at build time)
import Layout from '../layouts/Layout.astro';
import Card from '../components/Card.astro';
import Counter from '../components/Counter'; // React component
const title = "Welcome to Astro";
---

<Layout title={title}>
  <h1>{title}</h1>
  
  <!-- Astro component (no JavaScript) -->
  <Card title="Static Card">
    This card has zero JavaScript.
  </Card>
  
  <!-- React component with interactivity -->
  <Counter client:load />
  
  <!-- Vue component -->
  <VueButton client:idle />
  
  <!-- Svelte component -->
  <SvelteChart client:visible />
</Layout>

<!-- layouts/Layout.astro -->
---
import Header from '../components/Header.astro';
import Footer from '../components/Footer.astro';
const { title } = Astro.props;
---

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width" />
    <title>{title}</title>
  </head>
  <body>
    <Header />
    <main>
      <slot /> <!-- Content goes here -->
    </main>
    <Footer />
  </body>
</html>

<!-- components/Card.astro -->
---
const { title } = Astro.props;
---

<div class="card">
  <h2>{title}</h2>
  <slot />
</div>

<style>
  .card {
    border: 1px solid #ccc;
    border-radius: 8px;
    padding: 1rem;
    margin: 1rem 0;
  }
</style>

<!-- Using React components in Astro -->
// components/Counter.jsx
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>
        Increment
      </button>
    </div>
  );
}

<!-- pages/blog.astro -->
---
import { getCollection } from 'astro:content';
const posts = await getCollection('blog');
---

<h1>Blog Posts</h1>
<ul>
  {posts.map(post => (
    <li>
      <a href={`/blog/${post.slug}`}>{post.data.title}</a>
    </li>
  ))}
</ul>

<!-- Dynamic routes -->
// pages/blog/[slug].astro
---
import { getCollection } from 'astro:content';
export async function getStaticPaths() {
  const posts = await getCollection('blog');
  return posts.map(post => ({
    params: { slug: post.slug },
    props: { post },
  }));
}

const { post } = Astro.props;
---

<article>
  <h1>{post.data.title}</h1>
  <time>{post.data.publishedAt.toDateString()}</time>
  <div>{post.body}</div>
</article>

<!-- API endpoints -->
// pages/api/users.json.js
export async function get() {
  const users = await fetchUsers();
  
  return {
    body: JSON.stringify(users),
  };
}

<!-- Client directives (when to load JavaScript) -->
<!-- client:load - Load immediately (critical) -->
<ReactComponent client:load />

<!-- client:idle - Load when browser is idle -->
<ReactComponent client:idle />

<!-- client:visible - Load when component is visible -->
<ReactComponent client:visible />

<!-- client:media - Load based on media query -->
<ReactComponent client:media="(max-width: 768px)" />

<!-- Frameworks in same project -->
---
import ReactComponent from '../components/ReactComponent.jsx';
import VueComponent from '../components/VueComponent.vue';
import SvelteComponent from '../components/SvelteComponent.svelte';
---

<div>
  <ReactComponent client:load />
  <VueComponent client:idle />
  <SvelteComponent client:visible />
</div>
```

---

## **📊 Framework Comparison**

| **Feature** | **React Router** | **Next.js** | **Astro** |
|------------|-----------------|-------------|-----------|
| **Type** | Routing library | Full framework | Content framework |
| **Rendering** | Client-side only | Hybrid (SSR/SSG/CSR) | Static by default |
| **JavaScript** | Full React app | Full React app | Zero JS by default |
| **Performance** | Good | Excellent | Outstanding |
| **SEO** | Poor (CSR only) | Excellent | Excellent |
| **Learning Curve** | Low | Medium | Medium |
| **Best For** | SPAs, dashboards | Production apps, SEO sites | Content sites, blogs |
| **Multi-framework** | ❌ No | ❌ No | ✅ Yes |

---

## **🎯 Choosing the Right Framework**

### **Decision Tree:**
```
What are you building?
├── SPA/Dashboard → React Router + Vite/CRA
│
├── Production App → 
│   ├── Need SEO? → Yes → Next.js
│   │               └── No → React Router
│   │
│   └── Need API routes? → Yes → Next.js
│                         └── No → React Router
│
└── Content Site →
    ├── Need interactivity? → Minimal → Astro
    │                        └── Lots → Next.js
    │
    └── Multi-team with diff frameworks? → Yes → Astro
                                          └── No → Next.js
```

### **Modern Recommendations:**

**1. Dashboard/Internal Tool:**
```jsx
// Vite + React Router + TanStack Query
// Why: Fast dev, client-side only, no SEO needed
```

**2. E-commerce/Marketing Site:**
```jsx
// Next.js
// Why: SEO critical, need SSR, image optimization
```

**3. Blog/Portfolio:**
```jsx
// Astro
// Why: Content-focused, minimal JS, fastest loading
```

**4. Enterprise Application:**
```jsx
// Next.js
// Why: Full-stack capabilities, scalability, TypeScript
```

---

## **🚀 Advanced Patterns**

### **1. Hybrid Rendering Strategy (Next.js)**
```jsx
// Mix SSG, SSR, and CSR based on page needs
export default function ProductPage({ product }) {
  // Product data from SSG
  const [reviews, setReviews] = useState([]); // Client-side fetch
  
  useEffect(() => {
    // Fetch reviews client-side (frequently updated)
    fetchReviews(product.id).then(setReviews);
  }, [product.id]);
  
  return (
    <div>
      {/* Static product info */}
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      
      {/* Dynamic reviews */}
      <ReviewsList reviews={reviews} />
    </div>
  );
}

export async function getStaticProps({ params }) {
  // Product data at build time
  const product = await fetchProduct(params.id);
  return { props: { product }, revalidate: 3600 }; // ISR every hour
}

export async function getStaticPaths() {
  // Pre-render top products
  const products = await fetchPopularProducts();
  const paths = products.map(product => ({
    params: { id: product.id }
  }));
  
  return { paths, fallback: 'blocking' }; // Generate others on-demand
}
```

### **2. Micro-frontends with Module Federation**
```jsx
// Next.js with Module Federation (advanced)
// host/next.config.js
const { NextFederationPlugin } = require('@module-federation/nextjs-mf');

module.exports = {
  webpack(config, options) {
    config.plugins.push(
      new NextFederationPlugin({
        name: 'host',
        remotes: {
          shop: `shop@http://localhost:3001/_next/static/chunks/remoteEntry.js`,
          blog: `blog@http://localhost:3002/_next/static/chunks/remoteEntry.js`,
        },
        shared: {
          react: { singleton: true },
          'react-dom': { singleton: true },
        },
      })
    );
    
    return config;
  },
};

// Using remote components
import dynamic from 'next/dynamic';

const RemoteShop = dynamic(
  () => import('shop/ProductList'),
  { ssr: false }
);

function HomePage() {
  return (
    <div>
      <h1>Main App</h1>
      <RemoteShop />
    </div>
  );
}
```

### **3. Progressive Enhancement (Astro)**
```astro
<!-- pages/search.astro -->
---
import SearchBox from '../components/SearchBox.jsx';
---

<form action="/search" method="GET">
  <input type="text" name="q" placeholder="Search..." />
  <button type="submit">Search</button>
</form>

<!-- Enhanced search with React (if JS loads) -->
<SearchBox client:load />
```

```jsx
// components/SearchBox.jsx
import { useState } from 'react';

export default function SearchBox() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  const handleSearch = async (e) => {
    e.preventDefault();
    const res = await fetch(`/api/search?q=${query}`);
    const data = await res.json();
    setResults(data);
  };
  
  return (
    <div>
      <form onSubmit={handleSearch}>
        <input
          value={query}
          onChange={(e) => setQuery(e.target.value)}
          placeholder="Instant search..."
        />
      </form>
      <SearchResults results={results} />
    </div>
  );
}
```

---

## **📋 Best Practices**

### **React Router:**
1. ✅ **Use data loaders** (v6.4+) for data fetching
2. ✅ **Implement error boundaries** for route errors
3. ✅ **Use relative navigation** (`../` instead of full paths)
4. ✅ **Lazy load routes** with `React.lazy()`
5. ❌ **Don't put everything in one route component**

### **Next.js:**
1. ✅ **Use Image component** for automatic optimization
2. ✅ **Choose right data fetching method** (SSG vs SSR vs CSR)
3. ✅ **Implement ISR** for dynamic content that doesn't change often
4. ✅ **Use middleware** for authentication/redirects
5. ✅ **Enable compression** and CDN caching
6. ❌ **Don't use `getInitialProps`** (legacy API)

### **Astro:**
1. ✅ **Start with zero JS**, add interactivity only when needed
2. ✅ **Use appropriate client directives** (`load`, `idle`, `visible`)
3. ✅ **Leverage content collections** for type-safe content
4. ✅ **Use View Transitions API** for smooth page transitions
5. ❌ **Don't over-hydrate** - only make interactive what needs to be

---

## **🎓 Key Takeaways:**

1. **React Router**: For client-side SPAs, dashboards, tools
2. **Next.js**: For production apps, SEO sites, full-stack apps
3. **Astro**: For content sites, blogs, marketing pages
4. **Choose based on**: SEO needs, interactivity level, team skills

**Modern Framework Stack 2024:**
```javascript
// For web apps: Next.js
// Why: Production-ready, SSR, API routes, Vercel hosting

// For content sites: Astro  
// Why: Fastest performance, zero JS by default

// For SPAs: React Router + Vite
// Why: Fast dev experience, client-side only
```

**Golden Rule**: 
> **Start with the simplest solution that meets your needs.** Don't use Next.js for a simple portfolio, and don't use plain React Router for an e-commerce site.

**Remember**: The framework should **solve your problems**, not create new ones. Choose based on your **actual requirements**, not just what's trending! 🚀