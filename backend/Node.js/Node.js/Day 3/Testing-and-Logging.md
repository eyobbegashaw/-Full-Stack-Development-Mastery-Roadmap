# **Testing & Logging in Node.js - Detailed Notes & Code**

## **Testing**

### **Testing**
Testing verifies code works correctly, catches bugs early, and ensures reliability. Types: unit tests (single functions), integration tests (components working together), and end-to-end tests (full user flows). Testing improves code quality, enables refactoring confidence, and documents expected behavior. Use testing frameworks like Jest, Vitest, or Node's built-in test runner.

```javascript
// Basic test example
function sum(a, b) {
  return a + b;
}

// Manual test
console.log(sum(2, 3) === 5); // true
```

### **node:test**
`node:test` is Node.js's **built-in test runner** (available from Node.js 20+). No dependencies needed. Supports test suites, assertions, mocking, and reporting. Use via `import test from 'node:test'`. Integrates with Node.js tooling. Good for simple tests without external dependencies.

```javascript
import test from 'node:test';
import assert from 'node:assert';

test('sum function', () => {
  assert.strictEqual(sum(2, 3), 5);
  assert.strictEqual(sum(-1, 1), 0);
});

test('async test', async () => {
  const result = await asyncFunction();
  assert.ok(result);
});
```

### **Vitest**
Vitest is a **fast testing framework** built on Vite. Works with TypeScript, ES modules, and React/Vue out of the box. Feature-rich: snapshot testing, mocking, coverage, UI mode. Faster than Jest for Vite projects. Great for frontend and full-stack testing.

```javascript
// vitest.config.js
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
  },
});
```

```javascript
import { describe, it, expect } from 'vitest';
import { sum } from './math';

describe('math functions', () => {
  it('adds two numbers', () => {
    expect(sum(2, 3)).toBe(5);
  });

  it('handles negative numbers', () => {
    expect(sum(-1, 1)).toBe(0);
  });
});
```

### **Cypress**
Cypress is an **end-to-end testing framework** for web applications. Runs tests in real browser, provides time-travel debugging, automatic waiting, and visual testing. Tests user interactions, network requests, and UI behavior. Good for testing complete user workflows.

```javascript
// cypress/e2e/login.cy.js
describe('Login', () => {
  it('successfully logs in', () => {
    cy.visit('/login');
    cy.get('[data-cy=username]').type('testuser');
    cy.get('[data-cy=password]').type('password123');
    cy.get('[data-cy=submit]').click();
    cy.url().should('include', '/dashboard');
    cy.get('[data-cy=welcome]').should('contain', 'Welcome testuser');
  });
});
```

### **Playwright**
Playwright is a **cross-browser automation framework** by Microsoft. Supports multiple browsers (Chromium, Firefox, WebKit) and platforms. Features: auto-waiting, network interception, mobile emulation, parallel execution. Used for E2E testing, scraping, and automation. More modern alternative to Selenium.

```javascript
// playwright test
import { test, expect } from '@playwright/test';

test('basic test', async ({ page }) => {
  await page.goto('https://example.com');
  await expect(page).toHaveTitle('Example Domain');
  
  const header = page.locator('h1');
  await expect(header).toContainText('Example');
  
  // Take screenshot
  await page.screenshot({ path: 'example.png' });
});

test('API testing', async ({ request }) => {
  const response = await request.get('/api/users');
  expect(response.ok()).toBeTruthy();
  expect(await response.json()).toHaveLength(10);
});
```

## **Logging**

### **Logging**
Logging records application events for debugging, monitoring, and auditing. Different levels: error, warn, info, debug, trace. Structured logging includes timestamps, levels, and context. Logs help troubleshoot issues, track performance, and understand user behavior. Should be searchable and rotated to manage disk space.

```javascript
// Basic console logging
console.error('Error occurred:', error);
console.warn('Warning: Low memory');
console.info('Server started on port', port);
console.debug('Debug info:', data);
```

### **Winston**
Winston is a **versatile logging library** for Node.js. Supports multiple transports (console, file, HTTP, database), log levels, formatting, and querying. Customizable with formatters and filters. Production-ready with rotation and streaming capabilities. Industry standard for Node.js logging.

```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: process.env.NODE_ENV === 'production' ? 'info' : 'debug',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  transports: [
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple()
      )
    }),
    new winston.transports.File({ 
      filename: 'logs/error.log', 
      level: 'error' 
    }),
    new winston.transports.File({ 
      filename: 'logs/combined.log' 
    }),
  ],
});

// Usage
logger.info('Server started on port 3000');
logger.error('Database connection failed', { error: err });
```

### **Morgan**
Morgan is **HTTP request logger middleware** for Express.js. Logs request details: method, URL, status, response time, IP. Configurable formats: combined, common, dev, short, tiny. Useful for monitoring API usage, debugging requests, and security auditing.

```javascript
const express = require('express');
const morgan = require('morgan');

const app = express();

// Different formats
app.use(morgan('dev')); // Concise output colored by response status
// GET /api/users 200 12.356 ms - 2345

app.use(morgan('combined')); // Apache combined log format
// ::1 - - [15/Feb/2024:10:30:45 +0000] "GET /api/users HTTP/1.1" 200 2345

app.use(morgan(':method :url :status :res[content-length] - :response-time ms'));

// Custom token
morgan.token('host', (req) => req.headers['host']);
app.use(morgan(':host :method :url'));

// Write to file
const fs = require('fs');
const accessLogStream = fs.createWriteStream('logs/access.log', { flags: 'a' });
app.use(morgan('combined', { stream: accessLogStream }));
```

## **Complete Testing & Logging Setup**

### **Full Test Suite Example**
```javascript
// math.js
function sum(a, b) {
  if (typeof a !== 'number' || typeof b !== 'number') {
    throw new Error('Arguments must be numbers');
  }
  return a + b;
}

async function fetchData(url) {
  const response = await fetch(url);
  if (!response.ok) throw new Error('Network error');
  return response.json();
}

module.exports = { sum, fetchData };
```

```javascript
// math.test.js with Vitest
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest';
import { sum, fetchData } from './math';

describe('sum function', () => {
  it('adds two positive numbers', () => {
    expect(sum(2, 3)).toBe(5);
  });

  it('adds negative numbers', () => {
    expect(sum(-1, -1)).toBe(-2);
  });

  it('throws error for non-numbers', () => {
    expect(() => sum('2', 3)).toThrow('Arguments must be numbers');
  });

  it('handles decimal numbers', () => {
    expect(sum(0.1, 0.2)).toBeCloseTo(0.3);
  });
});

describe('fetchData', () => {
  let mockFetch;

  beforeEach(() => {
    mockFetch = vi.fn();
    global.fetch = mockFetch;
  });

  afterEach(() => {
    vi.restoreAllMocks();
  });

  it('fetches data successfully', async () => {
    const mockData = { id: 1, name: 'John' };
    mockFetch.mockResolvedValue({
      ok: true,
      json: () => Promise.resolve(mockData)
    });

    const result = await fetchData('https://api.example.com/data');
    expect(result).toEqual(mockData);
    expect(mockFetch).toHaveBeenCalledWith('https://api.example.com/data');
  });

  it('throws error on network failure', async () => {
    mockFetch.mockResolvedValue({ ok: false });
    
    await expect(fetchData('https://api.example.com/data'))
      .rejects.toThrow('Network error');
  });
});
```

### **Integration Test Example**
```javascript
// api.test.js - Testing Express API
import request from 'supertest';
import express from 'express';

const app = express();
app.use(express.json());

let users = [];

app.get('/api/users', (req, res) => {
  res.json(users);
});

app.post('/api/users', (req, res) => {
  const { name, email } = req.body;
  if (!name || !email) {
    return res.status(400).json({ error: 'Name and email required' });
  }
  
  const newUser = { id: users.length + 1, name, email };
  users.push(newUser);
  res.status(201).json(newUser);
});

app.delete('/api/users/:id', (req, res) => {
  const id = parseInt(req.params.id);
  users = users.filter(user => user.id !== id);
  res.status(204).send();
});

describe('User API', () => {
  beforeEach(() => {
    users = []; // Reset before each test
  });

  it('GET /api/users returns empty array initially', async () => {
    const response = await request(app).get('/api/users');
    expect(response.status).toBe(200);
    expect(response.body).toEqual([]);
  });

  it('POST /api/users creates a new user', async () => {
    const userData = { name: 'John Doe', email: 'john@example.com' };
    const response = await request(app)
      .post('/api/users')
      .send(userData);
    
    expect(response.status).toBe(201);
    expect(response.body).toMatchObject(userData);
    expect(response.body.id).toBeDefined();
    
    // Verify user was added
    const getResponse = await request(app).get('/api/users');
    expect(getResponse.body).toHaveLength(1);
  });

  it('POST /api/users returns 400 for invalid data', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ name: 'John' }); // Missing email
    
    expect(response.status).toBe(400);
    expect(response.body.error).toBeDefined();
  });

  it('DELETE /api/users/:id removes a user', async () => {
    // First create a user
    const createResponse = await request(app)
      .post('/api/users')
      .send({ name: 'John', email: 'john@example.com' });
    const userId = createResponse.body.id;
    
    // Then delete it
    const deleteResponse = await request(app).delete(`/api/users/${userId}`);
    expect(deleteResponse.status).toBe(204);
    
    // Verify it's gone
    const getResponse = await request(app).get('/api/users');
    expect(getResponse.body).toHaveLength(0);
  });
});
```

### **Advanced Winston Configuration**
```javascript
// logger.js - Production-ready logger
const winston = require('winston');
require('winston-daily-rotate-file');

const enumerateErrorFormat = winston.format((info) => {
  if (info instanceof Error) {
    Object.assign(info, { message: info.stack });
  }
  return info;
});

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    enumerateErrorFormat(),
    process.env.NODE_ENV === 'production' 
      ? winston.format.json()
      : winston.format.combine(
          winston.format.colorize(),
          winston.format.timestamp({ format: 'YYYY-MM-DD HH:mm:ss' }),
          winston.format.printf(({ level, message, timestamp, ...meta }) => {
            let metaStr = '';
            if (Object.keys(meta).length > 0) {
              metaStr = JSON.stringify(meta, null, 2);
            }
            return `${timestamp} ${level}: ${message} ${metaStr}`;
          })
        )
  ),
  transports: [
    new winston.transports.Console({
      stderrLevels: ['error']
    }),
    new winston.transports.DailyRotateFile({
      filename: 'logs/application-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      maxFiles: '14d', // Keep logs for 14 days
      maxSize: '20m', // Rotate when file reaches 20MB
      level: 'info',
      format: winston.format.uncolorize()
    }),
    new winston.transports.DailyRotateFile({
      filename: 'logs/error-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      maxFiles: '30d',
      level: 'error'
    })
  ],
  exceptionHandlers: [
    new winston.transports.File({ filename: 'logs/exceptions.log' })
  ],
  rejectionHandlers: [
    new winston.transports.File({ filename: 'logs/rejections.log' })
  ]
});

// Custom logger methods
logger.requestLogger = (req, res, next) => {
  const startTime = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - startTime;
    logger.info('HTTP Request', {
      method: req.method,
      url: req.url,
      status: res.statusCode,
      duration: `${duration}ms`,
      ip: req.ip,
      userAgent: req.get('User-Agent')
    });
  });
  
  next();
};

module.exports = logger;
```

### **E2E Testing with Playwright**
```javascript
// tests/e2e/auth.spec.js
import { test, expect } from '@playwright/test';

test.describe('Authentication', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
  });

  test('successful login', async ({ page }) => {
    // Navigate to login
    await page.click('text=Login');
    await expect(page).toHaveURL('/login');
    
    // Fill form
    await page.fill('[data-testid=email]', 'test@example.com');
    await page.fill('[data-testid=password]', 'password123');
    
    // Submit
    await Promise.all([
      page.waitForNavigation(),
      page.click('[data-testid=submit]')
    ]);
    
    // Verify redirect and welcome message
    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('[data-testid=welcome]')).toContainText('Welcome');
  });

  test('failed login shows error', async ({ page }) => {
    await page.goto('/login');
    
    // Fill with invalid credentials
    await page.fill('[data-testid=email]', 'wrong@example.com');
    await page.fill('[data-testid=password]', 'wrong');
    await page.click('[data-testid=submit]');
    
    // Verify error message
    await expect(page.locator('[data-testid=error]')).toBeVisible();
    await expect(page.locator('[data-testid=error]')).toContainText('Invalid credentials');
  });

  test('logout works', async ({ page }) => {
    // First login
    await page.goto('/login');
    await page.fill('[data-testid=email]', 'test@example.com');
    await page.fill('[data-testid=password]', 'password123');
    await page.click('[data-testid=submit]');
    await page.waitForURL('/dashboard');
    
    // Then logout
    await page.click('[data-testid=logout]');
    await expect(page).toHaveURL('/');
    await expect(page.locator('text=Login')).toBeVisible();
  });
});

// Global setup for authentication state
import { chromium } from '@playwright/test';

async function globalSetup() {
  const browser = await chromium.launch();
  const page = await browser.newPage();
  
  await page.goto('/login');
  await page.fill('[data-testid=email]', 'admin@example.com');
  await page.fill('[data-testid=password]', 'admin123');
  await page.click('[data-testid=submit]');
  await page.waitForURL('/dashboard');
  
  // Save authentication state
  await page.context().storageState({ path: 'storageState.json' });
  await browser.close();
}

export default globalSetup;
```

## **Best Practices**

### **Testing Best Practices**
```javascript
// 1. Test organization
describe('User Service', () => {
  describe('createUser', () => {
    it('creates user with valid data', () => { /* ... */ });
    it('throws error for duplicate email', () => { /* ... */ });
  });
  
  describe('updateUser', () => {
    it('updates existing user', () => { /* ... */ });
    it('returns null for non-existent user', () => { /* ... */ });
  });
});

// 2. Use test fixtures
function createTestUser(overrides = {}) {
  return {
    name: 'Test User',
    email: 'test@example.com',
    age: 25,
    ...overrides
  };
}

// 3. Clean up after tests
afterEach(async () => {
  await cleanupDatabase();
});

// 4. Test edge cases
it('handles empty input', () => { /* ... */ });
it('handles extremely large input', () => { /* ... */ });
it('handles special characters', () => { /* ... */ });
```

### **Logging Best Practices**
```javascript
// 1. Structured logging
logger.info('User registered', {
  userId: user.id,
  email: user.email,
  timestamp: new Date().toISOString(),
  source: 'registration-service',
  duration: 245 // ms
});

// 2. Different log levels appropriately
logger.error('Database connection failed', { error: err.stack });
logger.warn('API response slow', { endpoint: '/api/data', duration: 5000 });
logger.info('User logged in', { userId: 123 });
logger.debug('Processing data', { dataSize: data.length });

// 3. Log HTTP requests with correlation IDs
app.use((req, res, next) => {
  req.correlationId = uuid.v4();
  logger.info('Request started', {
    correlationId: req.correlationId,
    method: req.method,
    url: req.url,
    ip: req.ip
  });
  
  res.on('finish', () => {
    logger.info('Request completed', {
      correlationId: req.correlationId,
      status: res.statusCode,
      duration: Date.now() - req.startTime
    });
  });
  
  next();
});

// 4. Rotate logs to prevent disk filling
const { createLogger, transports } = require('winston');
require('winston-daily-rotate-file');

const transport = new transports.DailyRotateFile({
  filename: 'logs/application-%DATE%.log',
  datePattern: 'YYYY-MM-DD',
  maxSize: '20m',
  maxFiles: '14d'
});
```

### **Package.json Scripts Setup**
```json
{
  "scripts": {
    "test": "vitest",
    "test:watch": "vitest --watch",
    "test:coverage": "vitest --coverage",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:e2e:headed": "playwright test --headed",
    "test:ci": "vitest run && playwright test",
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "format": "prettier --write .",
    "dev": "nodemon server.js",
    "start": "node server.js"
  },
  "devDependencies": {
    "vitest": "^1.0.0",
    "@playwright/test": "^1.40.0",
    "supertest": "^6.3.0",
    "winston": "^3.11.0",
    "morgan": "^1.10.0",
    "nodemon": "^3.0.0",
    "eslint": "^8.0.0",
    "prettier": "^3.0.0"
  }
}
```

## **Monitoring & Alerting Integration**

```javascript
// logger with monitoring integration
const logger = winston.createLogger({
  transports: [
    // ... other transports
    new winston.transports.Http({
      host: 'log-aggregator.example.com',
      port: 12345,
      path: '/logs',
      ssl: true,
      format: winston.format.json()
    })
  ]
});

// Error tracking with Sentry integration
const Sentry = require('@sentry/node');

logger.errorStream = {
  write: (message) => {
    const log = JSON.parse(message);
    if (log.level === 'error') {
      Sentry.captureException(new Error(log.message), {
        extra: log
      });
    }
  }
};

app.use(morgan('combined', { stream: logger.errorStream }));
```

These testing and logging tools provide comprehensive coverage for building reliable, maintainable Node.js applications. Proper testing catches bugs early, while effective logging provides visibility into production systems.