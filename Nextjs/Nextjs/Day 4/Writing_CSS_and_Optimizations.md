**Global CSS (100 words)**
Global CSS applies styles universally across your entire application. Use it sparingly for reset/normalize styles, typography defaults, and CSS custom properties (variables). Place in `index.css` or `globals.css`. Avoid excessive global styles that cause specificity conflicts. Prefer scoped approaches for component styling. Use `@import` for external fonts/icons. Modern frameworks support CSS imports in JS/TS files. Global styles load first and affect performance; keep them minimal. Use PostCSS for vendor prefixing and optimization. Consider CSS containment for performance isolation.

---

**Ways to Write CSS (100 words)**
Multiple approaches exist: Global CSS, CSS Modules (scoped), CSS-in-JS (styled-components, Emotion), Utility-first (Tailwind), Preprocessors (Sass/SCSS), and CSS frameworks (Bootstrap). Each has tradeoffs: CSS Modules offer scoping without runtime overhead; CSS-in-JS enables dynamic styles but adds bundle size; Utility classes speed development but can become verbose. Choose based on project scale, team preference, and performance requirements. Many projects combine approaches: global for base styles, modules/components for UI, utilities for layouts. Modern frameworks support all methods.

---

**CSS Modules (100 words)**
CSS Modules provide locally scoped CSS by automatically generating unique class names. Import styles as objects: `import styles from './Button.module.css'`. Use with `className={styles.button}`. Supports composition via `composes`. Styles don't leak globally; conflicts are impossible. Works with Sass/PostCSS. Ideal for component-based architectures. Build tools (Webpack, Vite) transform class names during bundling. Enable with `[name].module.css` naming convention. Supports TypeScript with `*.module.css.d.ts` files. Offers great performance with zero runtime overhead. Combines CSS's power with JavaScript's modularity.

---


**Global CSS**
```css
/* global.css */
:root {
  --primary: #007bff;
  --spacing: 1rem;
}

* { margin: 0; padding: 0; box-sizing: border-box; }

body { font-family: system-ui; }
```

---

**CSS Modules**
```css
/* Button.module.css */
.button {
  background: var(--primary);
  padding: 12px 24px;
}

.error { border: 2px solid red; }
```
```jsx
import styles from './Button.module.css';
<button className={`${styles.button} ${styles.error}`}>
```

---

**Tailwind CSS**
```html
<button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">
  Click
</button>
```

---

**Sass**
```scss
// _variables.scss
$primary: #333;

// button.scss
@import 'variables';

.button {
  background: $primary;
  
  &:hover { opacity: 0.8; }
}
```

---

**CSS in JS**
```jsx
import styled from 'styled-components';

const Button = styled.button`
  background: ${props => props.primary ? 'blue' : 'gray'};
  padding: 1rem;
`;

<Button primary>Save</Button>
```

---

**Images**
```jsx
// Next.js Image component
import Image from 'next/image';

<Image
  src="/photo.jpg"
  alt="Description"
  width={500}
  height={300}
  priority={true}
/>
```

---

**Videos**
```html
<video 
  controls 
  width="600"
  poster="preview.jpg"
  preload="metadata"
>
  <source src="video.mp4" type="video/mp4">
</video>
```

---

**Fonts**
```css
@font-face {
  font-family: 'CustomFont';
  src: url('/fonts/custom.woff2') format('woff2');
  font-display: swap;
}

body { font-family: 'CustomFont', sans-serif; }
```

---

**Metadata**
```jsx
// Next.js
import Head from 'next/head';

<Head>
  <title>Page Title</title>
  <meta name="description" content="Page description" />
</Head>
```

---

**Package Bundling**
```javascript
// webpack.config.js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
    },
  },
};
```

---

**Lazy Loading**
```jsx
import { lazy, Suspense } from 'react';

const HeavyComponent = lazy(() => import('./HeavyComponent'));

<Suspense fallback={<Spinner />}>
  <HeavyComponent />
</Suspense>
```

---

**Analytics**
```html
<script async src="https://www.googletagmanager.com/gtag/js?id=GA_ID"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'GA_ID');
</script>
```

---

**Instrumentation**
```javascript
// Error tracking
import * as Sentry from '@sentry/react';

Sentry.init({ dsn: 'YOUR_DSN' });

try {
  riskyOperation();
} catch (error) {
  Sentry.captureException(error);
}
```

---

**OpenTelemetry**
```javascript
import { trace } from '@opentelemetry/api';

const tracer = trace.getTracer('app');
tracer.startActiveSpan('operation', span => {
  // Your code
  span.end();
});
```

---

**Static Assets**
```javascript
// Import in JavaScript
import logo from './logo.png';

function Header() {
  return <img src={logo} alt="Logo" />;
}
```

---

**Scripts**
```html
<!-- Load after DOM, non-blocking -->
<script defer src="app.js"></script>

<!-- Load immediately, non-blocking -->
<script async src="analytics.js"></script>
```

---

**Third Party Libraries**
```javascript
// Dynamic import for code splitting
import('lodash').then(({ debounce }) => {
  const search = debounce(() => {
    // search logic
  }, 300);
});
```

---

**Memory Usage**
```jsx
// Cleanup to prevent memory leaks
useEffect(() => {
  const controller = new AbortController();
  
  fetch('/api', { signal: controller.signal })
    .then(/* ... */);
  
  return () => controller.abort(); // Cleanup
}, []);
```
**Tailwind CSS (100 words)**
Utility-first CSS framework using predefined classes for rapid UI development. Write HTML with classes like `flex`, `p-4`, `bg-blue-500`. No custom CSS needed for most cases. Highly customizable via `tailwind.config.js`. Purges unused styles in production. Supports responsive design with prefixes (`md:`, `lg:`). Enforces consistency through design system. Integrates with JS frameworks via PostCSS. Reduces context switching between files. Critics cite verbose HTML; proponents value development speed. Use with component libraries for production. Includes JIT compiler for on-demand styles. Popular in modern web development.

---

**Sass (100 words)**
CSS preprocessor extending CSS with features like variables, nesting, mixins, functions, and imports. Write `.scss` or `.sass` files compiled to CSS. Variables store values (`$primary-color: #333`). Nesting mirrors HTML structure. Mixins reuse style blocks. Functions perform calculations. Partials/modularize code. Modern CSS now includes many Sass features (custom properties, nesting), but Sass retains advantages: better tooling, math functions, and mixin libraries. Works with module systems. Use with `node-sass` or `sass` npm packages. Integrates with build tools. Still valuable for large codebases.

---

**CSS in JS (100 words)**
Write CSS directly in JavaScript components using libraries like styled-components, Emotion, or Stitches. Enables dynamic styles based on props/state. Scopes styles to components automatically. Supports themes and design tokens. Eliminates class name conflicts. Enables code splitting with components. Drawbacks: runtime overhead, larger bundles, and devtool complexity. Popular in React ecosystems. styled-components example: `const Button = styled.button`color: ${props => props.primary ? 'white' : 'blue'}`;`. Supports SSR with server-side style extraction. Use for interactive UIs needing dynamic styling. Not ideal for static sites.

---

**Optimizations (100 words)**
CSS optimizations improve performance: minification removes whitespace/comments; purging eliminates unused styles (Tailwind, PurgeCSS); code splitting loads CSS per-route; critical CSS inlines above-fold styles; font-display controls font loading; contain property isolates layout; will-change hints animations; compress with Brotli/Gzip; use CSS containment for complex components; reduce specificity for faster matching; avoid expensive selectors (`*`, `[class*="..."]`); use transform/opacity for animations; implement lazy loading; leverage CDN caching. Measure with Lighthouse, WebPageTest. Prioritize Largest Contentful Paint (LCP) improvements.

---

**Images (100 words)**
Optimize images for web: compress formats (WebP, AVIF), specify dimensions, use `srcset`/`sizes` for responsiveness, lazy load with `loading="lazy"`, implement blur-up placeholders. Modern frameworks (Next.js Image, Gatsby Image) handle optimizations automatically. Use CDNs for transformation/delivery. Prioritize LCP image loading. Include `alt` text for accessibility. Consider SVGs for icons/illustrations. For galleries, use intersection observer for lazy loading. Set appropriate cache headers. Implement art direction with `<picture>` element. Monitor Core Web Vitals. Balance quality vs file size. Use sharp/ImageMagick for processing.

---

**Videos (100 words)**
Optimize video delivery: compress with modern codecs (H.265/VP9/AV1), use streaming formats (HLS/DASH) for adaptive bitrate, implement lazy loading, include poster images, provide captions/subtitles. Use `<video>` attributes: `preload="none"` for offscreen, `playsinline` for mobile. Host on dedicated platforms (YouTube, Vimeo, Mux) for global CDN. Consider self-hosting for control. Implement intersection observer for autoplay. Provide multiple sources for compatibility. Use `MediaSourceExtensions` for streaming. Monitor bandwidth usage. Optimize for mobile with lower resolutions. Include fallbacks for unsupported formats.

---

**Fonts (100 words)**
Web font optimization: subset fonts to required characters, use `font-display: swap` for FOIT prevention, preload critical fonts, host locally for control, or use reliable CDNs (Google Fonts). Choose modern formats (WOFF2). Limit font families/weights. Implement font loading strategies: `fontfaceobserver` or native CSS. Define fallback system fonts. Use `size-adjust` for metric matching. Consider variable fonts for multiple weights/sizes in one file. Monitor Cumulative Layout Shift (CLS) from font loading. Set proper cache headers. Test across browsers. Balance typography design with performance.

---

**Metadata (100 words)**
HTML metadata improves SEO and social sharing: `<title>` for page titles, `<meta name="description">` for snippets, Open Graph tags (`og:title`, `og:image`) for social platforms, Twitter Cards, structured JSON-LD for rich results, canonical URLs to avoid duplicate content, viewport for responsiveness, charset declaration, theme-color for PWA. Use robots.txt and sitemap.xml for crawling. Implement `next/head` in Next.js or `Helmet` in React. Dynamic metadata based on content. Validate with Rich Results Test. Essential for discoverability and user experience.

---

**Package Bundling (100 words)**
Bundlers (Webpack, Vite, esbuild, Rollup) combine modules for browser delivery. Techniques: tree shaking removes unused code, code splitting creates separate chunks, minification reduces size, compression (Gzip/Brotli) improves transfer, hashing enables cache busting, source maps aid debugging. Modern frameworks handle configuration. Vite uses ES modules for fast dev server. Analyze bundles with `webpack-bundle-analyzer`. Set chunking strategy: vendor, common, async. Use dynamic imports for lazy loading. Monitor bundle size budgets. Configure loaders for assets. Optimize for cold/warm cache scenarios.

---

**Lazy Loading (100 words)**
Defer loading non-critical resources until needed: images/videos with `loading="lazy"`, components via `React.lazy()` + `Suspense`, routes with dynamic imports, scripts using intersection observer. Improves initial load time and Time to Interactive. Implement for below-fold content. Use placeholder skeletons. Balance user experience vs perceived performance. Browser support is native for images/iframes. For React: `const Component = lazy(() => import('./Component'))`. Combine with preloading for strategic loading. Monitor LCP elements—avoid lazy loading them. Test on various connections. Essential for large applications.

---

**Analytics (100 words)**
Collect user interaction data: page views, events, performance metrics. Use Google Analytics, Plausible, Mixpanel, or custom solutions. Implement efficiently: load asynchronously, use tag managers sparingly, respect privacy (GDPR/CCPA) with consent banners, anonymize IPs. Capture Core Web Vitals via `web-vitals` library. Monitor custom business metrics. Use A/B testing frameworks. Avoid blocking main thread. Consider client-side vs server-side analytics. Use `sendBeacon` for reliable event sending. Implement error tracking (Sentry). Analyze user flows. Optimize based on data. Balance insights with performance impact.

---

**Instrumentation (100 words)**
Code instrumentation collects runtime data: error tracking, performance monitoring, user behavior. Implement with tools like Sentry, Datadog, New Relic. Capture JavaScript errors, API request times, component render durations. Use Performance API for measurements. Add custom metrics for business logic. Implement logging with structured data. Use OpenTelemetry for distributed tracing. Avoid overhead: sample data, throttle collection. Use `console` methods strategically (remove in production). Monitor memory usage, CPU. Correlate frontend/backend traces. Set up alerts for anomalies. Essential for debugging and performance optimization in production.

---

**OpenTelemetry (100 words)**
Open standard for observability: traces, metrics, logs. Instrument applications for distributed tracing across services. Implement with `@opentelemetry/api` in JavaScript. Auto-instrument frameworks (Express, React). Capture spans for operations. Export to backends (Jaeger, Zipkin, vendor solutions). Context propagation connects frontend/backend requests. Measure performance bottlenecks. Low overhead with sampling. Standardizes telemetry across languages. Use for error tracking, latency analysis, dependency monitoring. Configure with environment variables. Combine with existing monitoring. Growing adoption in microservices architectures. Essential for complex distributed systems.

---

**Static Assets (100 words)**
Static files: images, fonts, icons, PDFs, JSON data. Serve from CDN for global performance. Use version hashing for cache invalidation. Compress with Brotli/Gzip. Set long cache headers for immutable assets. Organize in `public/` or `static/` folder. Modern frameworks copy to build output. Use import for JavaScript bundling. Consider base64 encoding for small assets. SVG sprites for icons. Optimize delivery: HTTP/2, CDN caching, proper MIME types. Monitor with asset size budgets. Lazy load non-critical assets. Use `preload`/`prefetch` for priority assets.

---

**Scripts (100 words)**
JavaScript loading strategies: `<script async>` for independent scripts, `<script defer>` for DOM-dependent scripts, module type for ES modules, dynamic `import()` for code splitting. Place critical scripts in `<head>`, others before `</body>`. Use `preload` for early fetching. Minimize render-blocking. Third-party scripts can impact performance; lazy load if possible. Use `nomodule` for legacy browsers. Implement error handling. Monitor with Resource Timing API. Consider using `<script type="speculationrules">` for prerendering. Optimize execution with web workers for heavy tasks. Balance functionality with page load time.

---

**Third Party Libraries (100 words)**
External dependencies increase functionality but impact bundle size. Evaluate: bundle cost, maintenance, compatibility, licensing. Use npm/yarn/pnpm. Regularly update for security/features. Audit with `npm audit`. Consider lightweight alternatives. Import only needed features (lodash-es). Monitor with bundle analyzers. Lazy load non-critical libraries. Check browser support. Some increase memory usage (maps, charts). Use CDN for common libraries (jQuery, React) but consider version locking. Implement fallbacks for CDN failures. Tree shaking eliminates unused code. Limit dependencies to reduce vulnerability surface. Essential for rapid development.

---

**Memory Usage (100 words)**
Frontend memory management prevents leaks and slowdowns. Monitor via DevTools Memory tab. Common issues: detached DOM elements, event listeners not removed, closure references, large datasets in state, cache without limits. Fix: cleanup in `useEffect`, weak references, pagination, debouncing. Use `useMemo`/`useCallback` wisely. Profile with Heap Snapshots. Limit history state. Avoid memory-intensive operations in loops. Dispose WebGL/Media resources. Test long sessions. Implement virtual lists for large data. Set up monitoring with Performance Observer. Memory impacts user experience, especially on low-end devices. Regular audits prevent issues.