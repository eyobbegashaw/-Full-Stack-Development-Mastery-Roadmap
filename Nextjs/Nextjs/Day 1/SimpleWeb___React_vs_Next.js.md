I'll create a simple Next.js website with multiple pages, navigation, and different rendering strategies. This uses the modern App Router.

## **Project Structure:**
```
my-simple-site/
├── app/
│   ├── globals.css
│   ├── layout.js
│   ├── page.js          (Home page)
│   ├── about/
│   │   └── page.js      (Static page)
│   ├── products/
│   │   └── page.js      (Dynamic page)
│   └── dashboard/
│       └── page.js      (Client-side page)
├── components/
│   ├── Navbar.js
│   └── Footer.js
└── package.json
```

## **Step 1: Create the Project**
```bash
npx create-next-app@latest simple-site
cd simple-site
npm run dev
```

## **Step 2: Main Layout (`app/layout.js`)**
```jsx
import './globals.css';
import Navbar from '@/components/Navbar';
import Footer from '@/components/Footer';

export const metadata = {
  title: 'Simple Next.js Site',
  description: 'A demo website built with Next.js',
};

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        <Navbar />
        <main>{children}</main>
        <Footer />
      </body>
    </html>
  );
}
```

## **Step 3: Components**

### **Navbar Component (`components/Navbar.js`)**
```jsx
'use client';
import Link from 'next/link';
import { usePathname } from 'next/navigation';

export default function Navbar() {
  const pathname = usePathname();
  
  return (
    <nav>
      <div className="logo">🚀 NextSite</div>
      <div className="links">
        <Link href="/" className={pathname === '/' ? 'active' : ''}>
          Home
        </Link>
        <Link href="/about" className={pathname === '/about' ? 'active' : ''}>
          About
        </Link>
        <Link href="/products" className={pathname === '/products' ? 'active' : ''}>
          Products
        </Link>
        <Link href="/dashboard" className={pathname === '/dashboard' ? 'active' : ''}>
          Dashboard
        </Link>
      </div>
    </nav>
  );
}
```

### **Footer Component (`components/Footer.js`)**
```jsx
export default function Footer() {
  return (
    <footer>
      <p>© 2024 Simple Next.js Site. Built with ❤️ using Next.js 14.</p>
      <p>Rendering Strategies: SSG | SSR | CSR</p>
    </footer>
  );
}
```

## **Step 4: Pages**

### **Home Page (`app/page.js`) - Static by default**
```jsx
import Link from 'next/link';

export default function Home() {
  return (
    <div className="home">
      <h1>Welcome to Our Simple Next.js Site! 🎉</h1>
      <p>This demonstrates different rendering strategies in Next.js.</p>
      
      <div className="cards">
        <div className="card">
          <h3>⚡ Static Site Generation (SSG)</h3>
          <p>Pre-rendered at build time. Fastest option!</p>
          <Link href="/about">See About Page →</Link>
        </div>
        
        <div className="card">
          <h3>🔄 Server-Side Rendering (SSR)</h3>
          <p>Rendered on each request. Always fresh data!</p>
          <Link href="/products">See Products →</Link>
        </div>
        
        <div className="card">
          <h3>💻 Client-Side Rendering (CSR)</h3>
          <p>Rendered in browser. Interactive experiences!</p>
          <Link href="/dashboard">Try Dashboard →</Link>
        </div>
      </div>
    </div>
  );
}
```

### **About Page (`app/about/page.js`) - Static (SSG)**
```jsx
export default function About() {
  return (
    <div className="about">
      <h1>About Us 📖</h1>
      <p>This page is <strong>statically generated</strong> (SSG).</p>
      <p>It was created at build time and served as a static file.</p>
      
      <h2>Our Mission</h2>
      <p>To demonstrate Next.js rendering strategies in a simple way!</p>
      
      <h2>Features</h2>
      <ul>
        <li>Multiple rendering methods</li>
        <li>Fast page loads</li>
        <li>SEO friendly</li>
        <li>Modern React patterns</li>
      </ul>
    </div>
  );
}
```

### **Products Page (`app/products/page.js`) - Server-Side Rendered (SSR)**
```jsx
async function getProducts() {
  // Simulate API call - happens on server
  await new Promise(resolve => setTimeout(resolve, 500));
  
  return [
    { id: 1, name: 'Next.js Pro', price: '$99' },
    { id: 2, name: 'React Toolkit', price: '$49' },
    { id: 3, name: 'Web Dev Suite', price: '$199' },
  ];
}

export default async function Products() {
  const products = await getProducts(); // Server-side data fetch
  
  const currentTime = new Date().toLocaleTimeString();
  
  return (
    <div className="products">
      <h1>Our Products 🛒</h1>
      <p>This page is <strong>server-side rendered</strong> (SSR).</p>
      <p>Generated on server at: {currentTime}</p>
      
      <div className="product-list">
        {products.map(product => (
          <div key={product.id} className="product-card">
            <h3>{product.name}</h3>
            <p className="price">{product.price}</p>
            <button>Add to Cart</button>
          </div>
        ))}
      </div>
    </div>
  );
}
```

### **Dashboard Page (`app/dashboard/page.js`) - Client-Side Rendered (CSR)**
```jsx
'use client';
import { useState, useEffect } from 'react';

export default function Dashboard() {
  const [count, setCount] = useState(0);
  const [time, setTime] = useState('Loading...');
  const [data, setData] = useState(null);
  
  useEffect(() => {
    // Client-side only code
    setTime(new Date().toLocaleTimeString());
    
    // Simulate client-side API call
    setTimeout(() => {
      setData({ user: 'Demo User', plan: 'Premium' });
    }, 1000);
  }, []);
  
  return (
    <div className="dashboard">
      <h1>User Dashboard 📊</h1>
      <p>This page is <strong>client-side rendered</strong> (CSR).</p>
      <p>Interactive components run in your browser.</p>
      
      <div className="stats">
        <div className="stat-card">
          <h3>Counter</h3>
          <p>{count}</p>
          <button onClick={() => setCount(count + 1)}>+1</button>
          <button onClick={() => setCount(0)}>Reset</button>
        </div>
        
        <div className="stat-card">
          <h3>Current Time (Client)</h3>
          <p>{time}</p>
        </div>
        
        <div className="stat-card">
          <h3>User Data</h3>
          {data ? (
            <div>
              <p>👤 {data.user}</p>
              <p>📋 {data.plan}</p>
            </div>
          ) : (
            <p>Loading data...</p>
          )}
        </div>
      </div>
    </div>
  );
}
```

## **Step 5: Global Styles (`app/globals.css`)**
```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  background: #f5f5f5;
}

nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  background: white;
  border-radius: 10px;
  margin-bottom: 30px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.links a {
  margin-left: 20px;
  text-decoration: none;
  color: #333;
  padding: 8px 16px;
  border-radius: 5px;
}

.links a:hover {
  background: #f0f0f0;
}

.links a.active {
  background: #0070f3;
  color: white;
}

.home, .about, .products, .dashboard {
  background: white;
  padding: 40px;
  border-radius: 10px;
  margin-bottom: 30px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

h1 {
  margin-bottom: 20px;
  color: #333;
}

.cards, .product-list, .stats {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 20px;
  margin-top: 30px;
}

.card, .product-card, .stat-card {
  padding: 20px;
  border-radius: 10px;
  background: #f8f9fa;
  border: 1px solid #e9ecef;
}

.card h3, .product-card h3 {
  margin-bottom: 10px;
}

.price {
  font-size: 1.5em;
  font-weight: bold;
  color: #0070f3;
  margin: 10px 0;
}

button {
  background: #0070f3;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 5px;
  cursor: pointer;
  margin: 5px;
}

button:hover {
  background: #0051cc;
}

footer {
  text-align: center;
  padding: 20px;
  color: #666;
  margin-top: 50px;
}
```

## **Step 6: Run the Website**
```bash
npm run dev
```
Then visit `http://localhost:3000`

## **Features Demonstrated:**
1. **Static Generation (SSG)**: Home and About pages
2. **Server-Side Rendering (SSR)**: Products page with dynamic data
3. **Client-Side Rendering (CSR)**: Dashboard with interactive components
4. **Navigation**: Client-side transitions between pages
5. **Layout**: Shared header/footer across all pages
6. **Responsive Design**: Grid layouts that adapt to screen size

This simple website shows how Next.js handles different rendering strategies in a single application!

 # **React and Next.js:**

**React** = Just the UI library. You build components, but you need to add everything else yourself (routing, SSR, bundling, etc.).

**Next.js** = A **complete framework built on top of React**. It gives you React PLUS:
- Built-in routing
- Server-side rendering (SSR)
- Static generation (SSG)
- API routes
- Optimized images
- And much more

**Simple analogy:**
- **React** = Just the **engine** of a car
- **Next.js** = The **complete car** with engine, wheels, seats, and AC included

 **Yes, the UI code is exactly the same.**

```jsx
// This React component works in BOTH React AND Next.js:
function Button({ text }) {
  return <button onClick={() => alert('Clicked!')}>{text}</button>;
}
```

**Same syntax:**
- `useState`, `useEffect` hooks ✅
- JSX ✅  
- Components ✅
- Props ✅

**Difference is where/how it runs:**
- **Plain React**: Only runs in browser (CSR)
- **Next.js**: Can run on server OR browser (SSR/CSR/SSG)

**So:**
You write React components once, and Next.js decides **where** to render them (server or client). The component code itself is identical React.