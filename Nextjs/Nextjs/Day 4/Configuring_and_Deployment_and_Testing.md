**TypeScript (100 words)**
TypeScript adds static typing to JavaScript, catching errors during development. It compiles to plain JavaScript. Use interfaces for object shapes, types for unions/intersections, generics for reusable code. Provides better IDE support and documentation. Configure via `tsconfig.json`. Supports modern ECMAScript features. Essential for large applications and team collaboration. Reduces runtime bugs and improves code maintainability. Works with React through `@types/react`. Type checking happens at compile time, not runtime. Gradual adoption possible with `.js` to `.ts` migration.

```typescript
interface User {
  id: number;
  name: string;
}

const getUser = (id: number): User => ({ id, name: "John" });
```

---

**ESLint (100 words)**
ESLint statically analyzes JavaScript/TypeScript code to find problems and enforce coding standards. Configure rules in `.eslintrc.js`. Use plugins for frameworks (React, Vue). Auto-fix many issues. Integrates with IDEs and CI/CD. Prevents common bugs and maintains consistency. Popular configs: Airbnb, Standard. Use with Prettier for formatting. Extensible with custom rules. Run via CLI or npm scripts. Essential for team projects and code reviews.

```json
// .eslintrc.json
{
  "extends": ["eslint:recommended"],
  "rules": {
    "no-console": "warn"
  }
}
```

---

**Prettier (100 words)**
Prettier automatically formats code for consistent style. Supports JavaScript, TypeScript, CSS, HTML, JSON. Configure via `.prettierrc`. Ignores files in `.prettierignore`. Integrates with ESLint using `eslint-config-prettier`. Runs on save or commit. Eliminates formatting debates. Ensures uniform codebase. Works with most editors.

```json
// .prettierrc
{
  "semi": true,
  "singleQuote": true
}
```

---

**Environment Variables (100 words)**
Environment variables store configuration outside code. Use `.env` files (never commit `.env` with secrets). Access via `process.env.VAR_NAME`. Different environments: `.env.development`, `.env.production`. Framework-specific loaders (Next.js, Create React App). Validate with libraries like `dotenv-safe`. Use for API keys, database URLs, feature flags.

```javascript
// .env
API_KEY=secret123

// app.js
const apiKey = process.env.API_KEY;
```

---

**Markdown and MDX (100 words)**
Markdown is lightweight markup language for documentation. MDX extends Markdown to include JSX components. Write React components within Markdown. Used for blogs, docs, and content sites. Supports frontmatter for metadata. Process with remark/rehype plugins. Render in React with `next-mdx-remote` or MDX libraries.

```mdx
# Title

Regular markdown content.

<MyComponent prop="value" />
```

---

**Preparing for Production (100 words)**
Production preparation includes: minifying code, compressing assets, setting cache headers, enabling HTTPS, configuring CDN, setting up monitoring/error tracking, optimizing database queries, implementing logging, securing environment variables, and load testing. Build optimizations: tree shaking, code splitting, lazy loading. Ensure security headers and CORS.

```bash
# Build command
npm run build

# Production start
npm start
```

---

**Custom Server (100 words)**
Custom server allows custom routing, middleware, and server logic. Use Express with Next.js for API routes or SSR customization. Handle WebSockets, authentication middleware, custom logging. Not needed for static sites. Increases complexity but offers flexibility.

```javascript
// server.js
const express = require('express');
const next = require('next');

const app = next({ dev: false });
app.prepare().then(() => {
  const server = express();
  server.get('/custom', (req, res) => res.send('Custom'));
});
```

---

**Node.js Server (100 words)**
Node.js server runs JavaScript on server. Handle HTTP requests, file operations, database connections. Use Express, Koa, or Fastify frameworks. Implement REST/GraphQL APIs. Middleware for auth, logging, compression. Cluster mode for multi-core utilization. PM2 for process management.

```javascript
const http = require('http');

http.createServer((req, res) => {
  res.writeHead(200);
  res.end('Hello');
}).listen(3000);
```

---

**Testing Frameworks (100 words)**
Testing frameworks verify code correctness. Unit tests check individual functions; integration tests check components working together; E2E tests simulate user flows. Use Vitest/Jest for unit, React Testing Library for components, Cypress/Playwright for E2E. Mock dependencies, test edge cases. Continuous Integration runs tests automatically.

```javascript
// Jest/Vitest example
test('adds numbers', () => {
  expect(1 + 2).toBe(3);
});
```

---

**Vitest (100 words)**
Vitest is fast testing framework built on Vite. Compatible with Jest API. Native ES modules support. Watch mode by default. Works with TypeScript out-of-box. Lower configuration than Jest. Great for Vite projects. Built-in coverage reporting. Parallel test execution.

```javascript
import { test, expect } from 'vitest';

test('works', () => {
  expect(true).toBe(true);
});
```

---

**Jest (100 words)**
Jest is popular JavaScript testing framework. Includes test runner, assertion library, mocking. Snapshot testing for UI. Configure via `jest.config.js`. Used with React Testing Library. Mock modules with `jest.mock()`. Setup/teardown hooks. CI-friendly.

```javascript
// sum.test.js
test('adds 1+2 to equal 3', () => {
  expect(sum(1, 2)).toBe(3);
});
```

---

**Playwright (100 words)**
Playwright is E2E testing for modern web apps. Cross-browser (Chromium, Firefox, WebKit). Auto-wait for elements. Network interception. Mobile emulation. Generate tests via codegen. Trace viewer for debugging. Supports multiple languages.

```javascript
import { test } from '@playwright/test';

test('homepage', async ({ page }) => {
  await page.goto('https://example.com');
  await expect(page).toHaveTitle('Example');
});
```

---

**Cypress (100 words)**
Cypress is E2E testing framework running in browser. Real-time reloads. Time travel debugging. Automatic waiting. Screenshot/video recording. Dashboard service for results. Component testing available. Mocha/Chai syntax.

```javascript
describe('Login', () => {
  it('successfully logs in', () => {
    cy.visit('/login');
    cy.get('[data-cy=email]').type('user@example.com');
    cy.contains('Login').click();
  });
});
```

---

**Deployment Options (100 words)**
Deploy to various platforms: Vercel/Netlify for frontend, AWS/GCP/Azure for full-stack, Heroku/Railway for simplicity, Docker for containers, static hosting (GitHub Pages). Consider: scalability, cost, DevOps complexity, region. Automatic deployments from Git. Set up CI/CD pipelines. Environment-specific configurations.

```yaml
# GitHub Actions deploy
- name: Deploy
  run: npm run deploy
```

---

**Static Export (100 words)**
Static export generates HTML/CSS/JS files for hosting on any static server. No Node.js required. Use `next export` or similar. Pre-renders all pages. CDN-cacheable. Limited dynamic functionality. Good for blogs, marketing sites. Combine with client-side fetching for dynamic data.

```json
// package.json
{
  "scripts": {
    "export": "next build && next export"
  }
}
```

---

**Adapters (100 words)**
Adapters convert framework output for different hosting environments. In Next.js/SvelteKit, adapters target Node.js, Vercel, Netlify, etc. Handle serverless functions, edge functions, static files. Configure in framework config. Simplifies deployment across platforms.

```javascript
// sveltekit.config.js
import adapter from '@sveltejs/adapter-vercel';

export default {
  kit: { adapter: adapter() }
};
```

---

**Docker Container (100 words)**
Docker containers package app with dependencies for consistent environments. Define with `Dockerfile`. Build image, run container. Orchestrate with Docker Compose/Kubernetes. Multi-stage builds reduce size. .dockerignore excludes files. Portable across machines. Cloud deployment to ECS, GKE.

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

---

**Configuring / Setting things Up (100 words)**
Project setup involves: initializing package.json, installing dependencies, configuring TypeScript/ESLint/Prettier, setting up Git, creating folder structure, adding scripts, environment setup, editor config, CI/CD config. Use templates/starters. Document setup process. Ensure team consistency.

```bash
# Typical setup
npm init -y
npm install react react-dom
npm install -D typescript @types/node
npx tsc --init
```

---

**Testing (100 words)**
Testing ensures software quality and prevents regressions. Types: unit, integration, E2E, snapshot, performance, accessibility. Arrange-Act-Assert pattern. Mock external services. Test critical paths first. Aim for good coverage but not 100%. Fast feedback loops. Continuous testing in CI.

```javascript
// React Testing Library
import { render, screen } from '@testing-library/react';
test('renders button', () => {
  render(<Button>Click</Button>);
  expect(screen.getByText('Click')).toBeInTheDocument();
});
```

---

**Deployment (100 words)**
Deployment process: build app, run tests, create production bundle, deploy to server/CDN, run migrations, warm caches, health checks, monitor. Strategies: blue-green, canary, rolling updates. Zero-downtime deployments. Rollback plans. Post-deployment verification. Automate with CI/CD tools.

```bash
# Simple deploy script
npm run build
scp -r build/* user@server:/var/www/html
```