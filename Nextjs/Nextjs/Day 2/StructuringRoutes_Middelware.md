# **Next.js Advanced Routing Concepts**

## **Structuring Routes**
Route structure in Next.js determines both URL paths and code organization. The App Router uses folder-based routing where each folder represents a route segment and files like `page.js`, `layout.js`, and `route.js` define specific behaviors. Nested folders create nested routes, while special naming conventions enable advanced patterns.

**Key Structure Patterns:**
- Flat structure for simple apps
- Nested folders for complex hierarchies
- Route groups `(folder)` for logical organization without affecting URLs
- Dynamic segments `[param]` for data-driven routes
- Parallel routes `@folder` for simultaneous rendering
- Intercepting routes `(.)folder` for modal-like behavior

```jsx
app/
├── (auth)/           ← Route group (not in URL)
│   ├── login/
│   │   └── page.js   → /login
│   └── register/
│       └── page.js   → /register
├── shop/
│   ├── [category]/   ← Dynamic segment
│   │   └── page.js   → /shop/electronics
│   └── cart/
│       └── page.js   → /shop/cart
├── blog/
│   ├── @sidebar/     ← Parallel route slot
│   │   └── page.js
│   └── page.js       → /blog (with @sidebar)
└── api/
    └── users/
        └── route.js  → API endpoint
```

## **Middleware**
Middleware runs before a request is completed, allowing you to modify the request/response. It executes on every request, making it perfect for cross-cutting concerns like authentication, logging, or redirects. Middleware runs on the Edge Runtime, close to users for maximum performance.

**How it works:**
1. Request comes to your Next.js app
2. Middleware intercepts it
3. Middleware can: redirect, rewrite, modify headers, or continue
4. If continued, request reaches your page/API

```javascript
// middleware.js at project root
import { NextResponse } from 'next/server';
import { NextRequest } from 'next/server';

export function middleware(request) {
  // Check authentication
  const token = request.cookies.get('auth-token');
  
  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    // Redirect unauthenticated users
    return NextResponse.redirect(new URL('/login', request.url));
  }
  
  // Add custom header to all responses
  const response = NextResponse.next();
  response.headers.set('X-Custom-Header', 'my-value');
  
  return response;
}

// Configure which paths middleware runs on
export const config = {
  matcher: '/dashboard/:path*', // Only run on dashboard routes
};
```

## **Use Cases for Middleware**
Middleware solves problems that need to run before page rendering:

1. **Authentication/Authorization**: Protect routes before they load
2. **A/B Testing**: Route users to different versions
3. **Bot Detection**: Block or log bot traffic
4. **Geo-Redirecting**: Send users to region-specific content
5. **Feature Flags**: Enable/disable features per user
6. **Logging**: Track requests and performance
7. **Maintenance Mode**: Show maintenance pages
8. **Language Detection**: Redirect to localized versions

```javascript
// Advanced middleware with multiple use cases
export function middleware(request) {
  const url = request.nextUrl;
  const country = request.geo?.country || 'US';
  const userAgent = request.headers.get('user-agent');
  
  // 1. Bot protection
  if (userAgent?.includes('bot')) {
    return NextResponse.rewrite(new URL('/api/bot-response', request.url));
  }
  
  // 2. Geo-redirecting
  if (country === 'GB' && url.pathname === '/') {
    url.pathname = '/uk';
    return NextResponse.redirect(url);
  }
  
  // 3. Feature flagging
  const showBeta = request.cookies.get('beta-user');
  if (showBeta && url.pathname === '/dashboard') {
    url.pathname = '/dashboard/beta';
    return NextResponse.rewrite(url);
  }
  
  return NextResponse.next();
}
```

## **Route Matcher**
Route matchers control where middleware runs, optimizing performance by avoiding unnecessary execution. Use glob patterns to define which paths should or shouldn't trigger middleware.

**Matcher Patterns:**
- `/:path*` = match all paths
- `/dashboard/:path*` = match dashboard and subpaths
- `/((?!api|_next/static|_next/image|favicon.ico).*)` = exclude certain paths
- Array of matchers for complex conditions

```javascript
// middleware.js - Advanced matcher configuration
export const config = {
  matcher: [
    // Match specific paths
    '/dashboard/:path*',
    '/profile/:path*',
    
    // Match but exclude certain subpaths
    '/api/:path*',
    
    // Complex regex to exclude static files
    '/((?!_next/static|_next/image|favicon.ico|public/).*)',
  ],
  
  // Alternative: skip specific paths
  // skipMiddleware: true for static optimization
};
```

## **Using Cookies in Middleware**
Cookies provide persistent storage between requests. Middleware can read existing cookies, set new cookies, and delete cookies. This is essential for authentication, session management, and personalization.

```javascript
import { NextResponse } from 'next/server';
import { cookies } from 'next/headers';

export function middleware(request) {
  // 1. READING cookies from incoming request
  const theme = request.cookies.get('theme');
  const sessionToken = request.cookies.get('session');
  
  // 2. MODIFYING response to set cookies
  const response = NextResponse.next();
  
  // Set persistent cookie (expires in 30 days)
  response.cookies.set('user-preferences', 'dark-mode', {
    maxAge: 60 * 60 * 24 * 30, // 30 days in seconds
    httpOnly: true, // Not accessible via JavaScript
    secure: process.env.NODE_ENV === 'production', // HTTPS only in prod
    sameSite: 'lax', // CSRF protection
    path: '/', // Available on all paths
  });
  
  // 3. DELETING cookies
  if (request.nextUrl.pathname === '/logout') {
    response.cookies.delete('session');
  }
  
  // 4. Reading from server components (different API)
  // async function getServerSideCookie() {
  //   const cookieStore = cookies();
  //   return cookieStore.get('theme');
  // }
  
  return response;
}
```

## **Setting Headers**
Headers control browser behavior and provide metadata. Middleware can modify both request headers (sent to your server) and response headers (sent to the browser). Common uses include security headers, caching directives, and custom application headers.

```javascript
export function middleware(request) {
  const response = NextResponse.next();
  
  // SECURITY HEADERS (Critical for production)
  response.headers.set('X-Frame-Options', 'DENY'); // Prevent clickjacking
  response.headers.set('X-Content-Type-Options', 'nosniff'); // Prevent MIME sniffing
  response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');
  
  // CACHE CONTROL (Optimize performance)
  if (request.nextUrl.pathname.startsWith('/_next/static')) {
    response.headers.set('Cache-Control', 'public, max-age=31536000, immutable');
  } else if (request.nextUrl.pathname.startsWith('/api/')) {
    response.headers.set('Cache-Control', 'no-cache, no-store, must-revalidate');
  }
  
  // CORS HEADERS (If serving APIs to other domains)
  response.headers.set('Access-Control-Allow-Origin', 'https://trusted-site.com');
  response.headers.set('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
  
  // CUSTOM APPLICATION HEADERS
  response.headers.set('X-Request-ID', crypto.randomUUID());
  response.headers.set('X-App-Version', process.env.APP_VERSION || '1.0.0');
  
  // MODIFYING REQUEST HEADERS (before reaching your pages/APIs)
  const newRequestHeaders = new Headers(request.headers);
  newRequestHeaders.set('x-custom-request-header', 'custom-value');
  
  return NextResponse.next({
    request: {
      headers: newRequestHeaders,
    },
  });
}

// Complete middleware example combining everything
export function middleware(request) {
  // 1. Route matching logic
  if (!request.nextUrl.pathname.startsWith('/protected')) {
    return NextResponse.next();
  }
  
  // 2. Authentication check via cookies
  const session = request.cookies.get('session');
  if (!session) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
  
  // 3. Create response with security headers
  const response = NextResponse.next();
  response.headers.set('X-Powered-By', 'Next.js Middleware');
  
  // 4. Set cookie for tracking (with proper options)
  response.cookies.set('last-visit', new Date().toISOString(), {
    maxAge: 60 * 60 * 24,
    httpOnly: true,
    sameSite: 'strict',
  });
  
  return response;
}
```

## **Best Practices Summary**
1. **Keep middleware lightweight** - It runs on every matched request
2. **Use specific matchers** - Don't run middleware on static assets
3. **Secure cookies properly** - Use httpOnly, secure, and sameSite
4. **Set security headers** - Protect against common vulnerabilities
5. **Test middleware thoroughly** - It can break your entire app if buggy
6. **Use environment variables** - For different behavior in dev/prod
7. **Monitor performance** - Middleware adds overhead; measure impact

Middleware transforms Next.js from a static framework to a dynamic application platform, enabling enterprise-grade features with minimal code.


