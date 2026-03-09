# **Express.js: Complete Guide from Basic to Advanced**

## **Table of Contents**
1. [Introduction & Setup](#introduction)
2. [Basic Routing](#basic-routing)
3. [Middleware System](#middleware-system)
4. [Request & Response Objects](#request-response)
5. [Advanced Routing](#advanced-routing)
6. [Error Handling](#error-handling)
7. [Authentication & Security](#authentication-security)
8. [Performance & Optimization](#performance-optimization)
9. [Testing Express Apps](#testing)
10. [Production Deployment](#production)
11. [Real-World Architecture](#architecture)

---

## **1. Introduction & Setup** <a name="introduction"></a>

### **What is Express.js?**
Express.js is a minimal, flexible, and unopinionated web application framework for Node.js. It provides a robust set of features for building web and mobile applications, including:
- Routing
- Middleware support
- Template engines
- Error handling
- HTTP utilities

### **Installation & Setup**
```bash
# Create a new project
mkdir express-app
cd express-app
npm init -y

# Install Express
npm install express

# For development (optional)
npm install -D nodemon
```

### **Basic Server**
```javascript
// app.js
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

```json
// package.json scripts
{
  "scripts": {
    "start": "node app.js",
    "dev": "nodemon app.js"
  }
}
```

---

## **2. Basic Routing** <a name="basic-routing"></a>

### **HTTP Methods**
```javascript
app.get('/', (req, res) => {
  res.send('GET request to homepage');
});

app.post('/', (req, res) => {
  res.send('POST request to homepage');
});

app.put('/user', (req, res) => {
  res.send('PUT request to /user');
});

app.delete('/user', (req, res) => {
  res.send('DELETE request to /user');
});

app.patch('/user', (req, res) => {
  res.send('PATCH request to /user');
});

// All HTTP methods
app.all('/secret', (req, res, next) => {
  console.log('Accessing the secret section...');
  next(); // Pass control to the next handler
});
```

### **Route Parameters**
```javascript
// Basic parameters
app.get('/users/:userId', (req, res) => {
  res.send(`User ID: ${req.params.userId}`);
});

// Multiple parameters
app.get('/books/:bookId/authors/:authorId', (req, res) => {
  res.send(`Book: ${req.params.bookId}, Author: ${req.params.authorId}`);
});

// Optional parameters
app.get('/users/:userId?', (req, res) => {
  const userId = req.params.userId || 'all';
  res.send(`Showing user: ${userId}`);
});

// Regex in parameters
app.get('/user/:id(\\d+)', (req, res) => { // Only numeric IDs
  res.send(`User ID (numeric only): ${req.params.id}`);
});
```

### **Query Strings**
```javascript
app.get('/search', (req, res) => {
  const { q, page = 1, limit = 10 } = req.query;
  res.json({
    query: q,
    page: parseInt(page),
    limit: parseInt(limit),
    results: []
  });
});
```

---

## **3. Middleware System** <a name="middleware-system"></a>

### **What is Middleware?**
Middleware functions have access to request object (req), response object (res), and the next middleware function (next). They can:
- Execute any code
- Modify request/response objects
- End request-response cycle
- Call next middleware

### **Built-in Middleware**
```javascript
// Parse JSON bodies
app.use(express.json());

// Parse URL-encoded bodies
app.use(express.urlencoded({ extended: true }));

// Serve static files
app.use(express.static('public'));

// Serve static files with options
app.use('/static', express.static('public', {
  maxAge: '1d', // Cache for 1 day
  setHeaders: (res, path) => {
    if (path.endsWith('.css')) {
      res.set('Content-Type', 'text/css');
    }
  }
}));
```

### **Custom Middleware**
```javascript
// Simple logging middleware
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url} - ${new Date().toISOString()}`);
  next();
});

// Authentication middleware
const authenticate = (req, res, next) => {
  const token = req.headers.authorization;
  
  if (!token || token !== 'valid-token') {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  
  req.user = { id: 1, name: 'John Doe' }; // Attach user to request
  next();
};

app.get('/profile', authenticate, (req, res) => {
  res.json({ user: req.user });
});

// Error handling middleware (special - 4 parameters)
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Something broke!');
});
```

### **Third-Party Middleware**
```javascript
// Install: npm install helmet cors morgan compression
const helmet = require('helmet');
const cors = require('cors');
const morgan = require('morgan');
const compression = require('compression');

// Security headers
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      scriptSrc: ["'self'", "'unsafe-inline'"]
    }
  }
}));

// CORS
app.use(cors({
  origin: ['https://example.com', 'http://localhost:3000'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE']
}));

// HTTP request logging
app.use(morgan('combined')); // or 'dev', 'common', 'tiny'

// Response compression
app.use(compression({
  level: 6, // Compression level (0-9)
  threshold: 1024 // Only compress responses > 1KB
}));
```

### **Middleware Execution Order**
```javascript
// Order matters!
app.use(express.json()); // 1. Parse JSON first
app.use(express.urlencoded({ extended: true })); // 2. Parse URL-encoded
app.use(cors()); // 3. Enable CORS
app.use(helmet()); // 4. Security headers
app.use(morgan('dev')); // 5. Logging

// Custom middleware
app.use((req, res, next) => {
  req.requestTime = Date.now(); // 6. Add timestamp
  next();
});

// Route-specific middleware
app.use('/api', authenticate); // 7. Auth for API routes only
```

---

## **4. Request & Response Objects** <a name="request-response"></a>

### **Request Object (req)**
```javascript
app.get('/api/data', (req, res) => {
  // Request properties
  console.log('Method:', req.method);
  console.log('URL:', req.url);
  console.log('Path:', req.path);
  console.log('Hostname:', req.hostname);
  console.log('IP:', req.ip);
  console.log('Protocol:', req.protocol);
  console.log('Secure:', req.secure);
  console.log('Query:', req.query);
  console.log('Params:', req.params);
  
  // Headers
  console.log('Content-Type:', req.get('Content-Type'));
  console.log('User-Agent:', req.get('User-Agent'));
  
  // Body (for POST/PUT/PATCH)
  console.log('Body:', req.body);
  
  // Cookies
  console.log('Cookies:', req.cookies);
  
  // Files (with multer middleware)
  console.log('Files:', req.files);
  
  // Custom properties (added by middleware)
  console.log('Request time:', req.requestTime);
  console.log('User:', req.user);
});
```

### **Response Object (res)**
```javascript
app.get('/response-examples', (req, res) => {
  // Send different types of responses
  
  // 1. Send plain text
  // res.send('Hello World');
  
  // 2. Send JSON
  // res.json({ message: 'Success', data: [] });
  
  // 3. Send file
  // res.sendFile('/path/to/file.pdf');
  
  // 4. Set status code
  res.status(201).json({ id: 1, created: true });
  
  // 5. Set headers
  res.set('Content-Type', 'text/html');
  res.set('Cache-Control', 'no-cache');
  
  // 6. Set cookie
  res.cookie('sessionId', 'abc123', {
    maxAge: 24 * 60 * 60 * 1000, // 1 day
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production'
  });
  
  // 7. Clear cookie
  // res.clearCookie('sessionId');
  
  // 8. Redirect
  // res.redirect('/new-location');
  // res.redirect(301, '/permanent-redirect');
  
  // 9. Set content type and send
  // res.type('html').send('<h1>Hello</h1>');
  // res.type('json').send({ data: 'json' });
  
  // 10. End response
  // res.end();
  
  // 11. Send status only
  // res.sendStatus(404); // Sends "Not Found"
  
  // 12. Set link header
  // res.links({
  //   next: 'http://api.example.com/users?page=2',
  //   last: 'http://api.example.com/users?page=5'
  // });
});
```

### **Streaming Responses**
```javascript
const fs = require('fs');
const { pipeline } = require('stream');

// Stream large file
app.get('/download', (req, res) => {
  const filePath = './large-file.zip';
  const stat = fs.statSync(filePath);
  
  res.setHeader('Content-Length', stat.size);
  res.setHeader('Content-Type', 'application/zip');
  res.setHeader('Content-Disposition', 'attachment; filename=large-file.zip');
  
  const readStream = fs.createReadStream(filePath);
  
  // Handle stream errors
  readStream.on('error', (err) => {
    console.error('Stream error:', err);
    res.status(500).end();
  });
  
  readStream.pipe(res);
});

// Streaming JSON response
app.get('/stream-data', (req, res) => {
  res.setHeader('Content-Type', 'application/json');
  res.write('[');
  
  // Stream data chunks
  for (let i = 0; i < 10; i++) {
    res.write(JSON.stringify({ id: i, data: 'chunk' }) + (i < 9 ? ',' : ''));
    // Simulate delay
    if (i % 3 === 0) {
      res.flush(); // Flush the buffer
    }
  }
  
  res.write(']');
  res.end();
});
```

---

## **5. Advanced Routing** <a name="advanced-routing"></a>

### **Router Module**
```javascript
// routes/users.js
const express = require('express');
const router = express.Router();

// Middleware specific to this router
router.use((req, res, next) => {
  console.log('Time:', Date.now());
  next();
});

// Define routes
router.get('/', (req, res) => {
  res.send('List of users');
});

router.get('/:id', (req, res) => {
  res.send(`User ${req.params.id}`);
});

router.post('/', (req, res) => {
  res.status(201).send('User created');
});

module.exports = router;
```

```javascript
// app.js - Mount the router
const userRouter = require('./routes/users');
app.use('/users', userRouter);

// Multiple routers
const productRouter = require('./routes/products');
const orderRouter = require('./routes/orders');

app.use('/api/v1/products', productRouter);
app.use('/api/v1/orders', orderRouter);
```

### **Route Chaining**
```javascript
router.route('/books/:bookId')
  .get((req, res) => {
    res.send(`Get book ${req.params.bookId}`);
  })
  .put((req, res) => {
    res.send(`Update book ${req.params.bookId}`);
  })
  .delete((req, res) => {
    res.send(`Delete book ${req.params.bookId}`);
  });
```

### **Parameter Validation**
```javascript
const { param, query, body, validationResult } = require('express-validator');

app.post('/users', 
  // Validation middleware
  body('email').isEmail().normalizeEmail(),
  body('password').isLength({ min: 6 }),
  body('age').optional().isInt({ min: 0 }),
  
  // Route handler
  (req, res) => {
    // Check validation errors
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    
    // Process valid data
    res.status(201).json({ message: 'User created' });
  }
);
```

### **Wildcard & Regex Routes**
```javascript
// Wildcard route
app.get('/files/*', (req, res) => {
  res.send(`Requested file: ${req.params[0]}`);
});

// Regex route
app.get(/.*fly$/, (req, res) => {
  res.send(`Matches any path ending with "fly"`);
});

// Multiple patterns
app.get(['/apple', '/banana'], (req, res) => {
  res.send('Apple or Banana');
});
```

---

## **6. Error Handling** <a name="error-handling"></a>

### **Synchronous Error Handling**
```javascript
// Express catches synchronous errors automatically
app.get('/error-sync', (req, res) => {
  throw new Error('Something went wrong!'); // Will be caught
});
```

### **Asynchronous Error Handling**
```javascript
// Async errors need to be passed to next()
app.get('/error-async', async (req, res, next) => {
  try {
    const data = await someAsyncOperation();
    res.json(data);
  } catch (err) {
    next(err); // Pass error to error handler
  }
});

// Or use express-async-errors package
require('express-async-errors'); // Import at top

app.get('/error-async-easy', async (req, res) => {
  const data = await someAsyncOperation(); // Errors auto-caught
  res.json(data);
});
```

### **Custom Error Classes**
```javascript
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.status = `${statusCode}`.startsWith('4') ? 'fail' : 'error';
    this.isOperational = true;
    
    Error.captureStackTrace(this, this.constructor);
  }
}

class NotFoundError extends AppError {
  constructor(message = 'Resource not found') {
    super(message, 404);
  }
}

class ValidationError extends AppError {
  constructor(errors) {
    super('Validation failed', 400);
    this.errors = errors;
  }
}

// Usage
app.get('/users/:id', (req, res, next) => {
  const user = users.find(u => u.id === req.params.id);
  
  if (!user) {
    return next(new NotFoundError());
  }
  
  if (!user.isActive) {
    return next(new AppError('User is inactive', 403));
  }
  
  res.json(user);
});
```

### **Global Error Handler**
```javascript
// Development error handler
const sendErrorDev = (err, res) => {
  res.status(err.statusCode).json({
    status: err.status,
    error: err,
    message: err.message,
    stack: err.stack
  });
};

// Production error handler
const sendErrorProd = (err, res) => {
  // Operational, trusted error: send message to client
  if (err.isOperational) {
    res.status(err.statusCode).json({
      status: err.status,
      message: err.message
    });
  } 
  // Programming or unknown error: don't leak error details
  else {
    console.error('ERROR 💥', err);
    res.status(500).json({
      status: 'error',
      message: 'Something went very wrong!'
    });
  }
};

// Global error handling middleware
app.use((err, req, res, next) => {
  err.statusCode = err.statusCode || 500;
  err.status = err.status || 'error';
  
  if (process.env.NODE_ENV === 'development') {
    sendErrorDev(err, res);
  } else {
    sendErrorProd(err, res);
  }
});
```

### **404 Handler**
```javascript
// Catch-all for undefined routes (must be after all routes)
app.all('*', (req, res, next) => {
  next(new NotFoundError(`Can't find ${req.originalUrl} on this server!`));
});
```

### **Unhandled Rejections & Exceptions**
```javascript
// Handle unhandled promise rejections
process.on('unhandledRejection', (err) => {
  console.error('UNHANDLED REJECTION! 💥 Shutting down...');
  console.error(err.name, err.message);
  // Gracefully close server
  server.close(() => {
    process.exit(1);
  });
});

// Handle uncaught exceptions
process.on('uncaughtException', (err) => {
  console.error('UNCAUGHT EXCEPTION! 💥 Shutting down...');
  console.error(err.name, err.message);
  process.exit(1);
});
```

---

## **7. Authentication & Security** <a name="authentication-security"></a>

### **JWT Authentication**
```javascript
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');

// Generate token
const generateToken = (userId) => {
  return jwt.sign(
    { userId },
    process.env.JWT_SECRET,
    { expiresIn: process.env.JWT_EXPIRES_IN }
  );
};

// Authentication middleware
const protect = async (req, res, next) => {
  try {
    // 1. Get token
    let token;
    if (req.headers.authorization?.startsWith('Bearer')) {
      token = req.headers.authorization.split(' ')[1];
    } else if (req.cookies?.jwt) {
      token = req.cookies.jwt;
    }
    
    if (!token) {
      throw new Error('Not authorized');
    }
    
    // 2. Verify token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    
    // 3. Get user from database
    const user = await User.findById(decoded.userId);
    if (!user) {
      throw new Error('User no longer exists');
    }
    
    // 4. Check if user changed password after token was issued
    if (user.changedPasswordAfter(decoded.iat)) {
      throw new Error('Password changed recently');
    }
    
    // 5. Grant access
    req.user = user;
    next();
  } catch (error) {
    res.status(401).json({
      status: 'fail',
      message: 'Please log in to get access'
    });
  }
};

// Routes
app.post('/login', async (req, res, next) => {
  try {
    const { email, password } = req.body;
    
    // 1. Check if email and password exist
    if (!email || !password) {
      throw new AppError('Please provide email and password', 400);
    }
    
    // 2. Check if user exists && password is correct
    const user = await User.findOne({ email }).select('+password');
    
    if (!user || !(await bcrypt.compare(password, user.password))) {
      throw new AppError('Incorrect email or password', 401);
    }
    
    // 3. Generate token
    const token = generateToken(user._id);
    
    // 4. Send token in cookie
    res.cookie('jwt', token, {
      expires: new Date(
        Date.now() + process.env.JWT_COOKIE_EXPIRES_IN * 24 * 60 * 60 * 1000
      ),
      httpOnly: true,
      secure: req.secure || req.headers['x-forwarded-proto'] === 'https'
    });
    
    // 5. Remove password from output
    user.password = undefined;
    
    res.status(200).json({
      status: 'success',
      token,
      user
    });
  } catch (error) {
    next(error);
  }
});
```

### **Role-Based Authorization**
```javascript
const restrictTo = (...roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return next(
        new AppError('You do not have permission to perform this action', 403)
      );
    }
    next();
  };
};

// Usage
app.get(
  '/admin/users',
  protect,
  restrictTo('admin', 'super-admin'),
  async (req, res, next) => {
    const users = await User.find();
    res.json(users);
  }
);
```

### **Rate Limiting**
```javascript
const rateLimit = require('express-rate-limit');

// Global rate limiter
const globalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP, please try again after 15 minutes',
  standardHeaders: true, // Return rate limit info in `RateLimit-*` headers
  legacyHeaders: false, // Disable `X-RateLimit-*` headers
});

app.use('/api', globalLimiter);

// Stricter rate limiter for auth endpoints
const authLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 5, // 5 attempts per hour
  message: 'Too many login attempts, please try again after an hour'
});

app.use('/api/v1/auth/login', authLimiter);
```

### **Security Headers**
```javascript
const helmet = require('helmet');
const mongoSanitize = require('express-mongo-sanitize');
const xss = require('xss-clean');
const hpp = require('hpp');

// Security middleware stack
app.use(helmet()); // Set security HTTP headers

// Data sanitization against NoSQL query injection
app.use(mongoSanitize());

// Data sanitization against XSS
app.use(xss());

// Prevent parameter pollution
app.use(hpp({
  whitelist: [
    'duration',
    'ratingsQuantity',
    'ratingsAverage',
    'maxGroupSize',
    'difficulty',
    'price'
  ]
}));
```

---

## **8. Performance & Optimization** <a name="performance-optimization"></a>

### **Caching Strategies**
```javascript
const redis = require('redis');
const { promisify } = require('util');

// Redis client setup
const client = redis.createClient({
  host: process.env.REDIS_HOST,
  port: process.env.REDIS_PORT,
  password: process.env.REDIS_PASSWORD
});

client.on('error', (err) => console.error('Redis Client Error:', err));

const getAsync = promisify(client.get).bind(client);
const setAsync = promisify(client.set).bind(client);

// Redis caching middleware
const cacheMiddleware = (duration = 3600) => { // 1 hour default
  return async (req, res, next) => {
    // Only cache GET requests
    if (req.method !== 'GET') {
      return next();
    }
    
    const key = `cache:${req.originalUrl}`;
    
    try {
      const cachedData = await getAsync(key);
      
      if (cachedData) {
        console.log('Serving from cache');
        return res.json(JSON.parse(cachedData));
      }
      
      // Store original send function
      const originalSend = res.json.bind(res);
      
      // Override res.json to cache response
      res.json = async (data) => {
        await setAsync(key, JSON.stringify(data), 'EX', duration);
        originalSend(data);
      };
      
      next();
    } catch (error) {
      console.error('Cache error:', error);
      next();
    }
  };
};

// Usage
app.get('/api/products', cacheMiddleware(1800), async (req, res) => {
  const products = await Product.find();
  res.json(products); // Will be cached automatically
});
```

### **Database Query Optimization**
```javascript
// Advanced query features
app.get('/api/tours', async (req, res, next) => {
  try {
    // BUILD QUERY
    const queryObj = { ...req.query };
    const excludedFields = ['page', 'sort', 'limit', 'fields'];
    excludedFields.forEach(el => delete queryObj[el]);
    
    // Advanced filtering: gte, gt, lte, lt
    let queryStr = JSON.stringify(queryObj);
    queryStr = queryStr.replace(/\b(gte|gt|lte|lt)\b/g, match => `$${match}`);
    
    let query = Tour.find(JSON.parse(queryStr));
    
    // Sorting
    if (req.query.sort) {
      const sortBy = req.query.sort.split(',').join(' ');
      query = query.sort(sortBy);
    } else {
      query = query.sort('-createdAt');
    }
    
    // Field limiting
    if (req.query.fields) {
      const fields = req.query.fields.split(',').join(' ');
      query = query.select(fields);
    } else {
      query = query.select('-__v'); // Exclude version key
    }
    
    // Pagination
    const page = parseInt(req.query.page, 10) || 1;
    const limit = parseInt(req.query.limit, 10) || 100;
    const skip = (page - 1) * limit;
    
    query = query.skip(skip).limit(limit);
    
    // Execute query
    const tours = await query;
    
    // Send response
    res.status(200).json({
      status: 'success',
      results: tours.length,
      data: { tours }
    });
  } catch (error) {
    next(error);
  }
});
```

### **Compression & Minification**
```javascript
const compression = require('compression');
const minify = require('express-minify');

// Enable compression
app.use(compression({
  level: 6,
  threshold: 1024,
  filter: (req, res) => {
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  }
}));

// Minify responses (for HTML, CSS, JS)
app.use(minify({
  cache: path.join(__dirname, 'cache'),
  uglifyJS: require('uglify-js'),
  cssmin: require('cssmin')
}));
```

### **Connection Pooling**
```javascript
const mongoose = require('mongoose');

// MongoDB connection with pooling
mongoose.connect(process.env.MONGODB_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
  poolSize: 10, // Maintain up to 10 socket connections
  serverSelectionTimeoutMS: 5000,
  socketTimeoutMS: 45000,
});

// PostgreSQL with pooling
const { Pool } = require('pg');
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20, // Maximum number of clients in pool
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});
```

---

## **9. Testing Express Apps** <a name="testing"></a>

### **Unit Testing**
```javascript
// math.js
const add = (a, b) => a + b;
module.exports = { add };

// math.test.js
const { add } = require('./math');

describe('Math functions', () => {
  test('adds two numbers correctly', () => {
    expect(add(2, 3)).toBe(5);
    expect(add(-1, 1)).toBe(0);
    expect(add(0, 0)).toBe(0);
  });
});
```

### **Integration Testing**
```javascript
// app.js
const express = require('express');
const app = express();
app.use(express.json());

let items = [];

app.get('/api/items', (req, res) => {
  res.json(items);
});

app.post('/api/items', (req, res) => {
  const item = { id: items.length + 1, ...req.body };
  items.push(item);
  res.status(201).json(item);
});

module.exports = app;

// server.js (for production)
const app = require('./app');
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

```javascript
// app.test.js
const request = require('supertest');
const app = require('./app');

describe('Items API', () => {
  beforeEach(() => {
    // Reset items array before each test
    require('./app').items = [];
  });

  test('GET /api/items returns empty array initially', async () => {
    const response = await request(app).get('/api/items');
    expect(response.status).toBe(200);
    expect(response.body).toEqual([]);
  });

  test('POST /api/items creates a new item', async () => {
    const newItem = { name: 'Test Item', price: 100 };
    
    const response = await request(app)
      .post('/api/items')
      .send(newItem);
    
    expect(response.status).toBe(201);
    expect(response.body).toMatchObject(newItem);
    expect(response.body.id).toBeDefined();
  });

  test('POST /api/items returns 400 for invalid data', async () => {
    const response = await request(app)
      .post('/api/items')
      .send({}); // Empty body
      
    expect(response.status).toBe(400);
  });
});
```

### **E2E Testing with Cypress**
```javascript
// cypress/e2e/auth.cy.js
describe('Authentication', () => {
  beforeEach(() => {
    cy.visit('/');
  });

  it('successfully logs in', () => {
    cy.get('[data-testid=email]').type('test@example.com');
    cy.get('[data-testid=password]').type('password123');
    cy.get('[data-testid=submit]').click();
    
    cy.url().should('include', '/dashboard');
    cy.get('[data-testid=welcome]').should('contain', 'Welcome');
  });

  it('shows error for invalid credentials', () => {
    cy.get('[data-testid=email]').type('wrong@example.com');
    cy.get('[data-testid=password]').type('wrong');
    cy.get('[data-testid=submit]').click();
    
    cy.get('[data-testid=error]').should('be.visible');
  });
});
```

### **Load Testing**
```javascript
// load-test.js
const autocannon = require('autocannon');
const { PassThrough } = require('stream');

function runLoadTest() {
  const url = 'http://localhost:3000';
  
  const instance = autocannon({
    url: url,
    connections: 100, // Number of concurrent connections
    duration: 30, // Test duration in seconds
    requests: [
      {
        method: 'GET',
        path: '/api/products'
      },
      {
        method: 'POST',
        path: '/api/orders',
        headers: {
          'content-type': 'application/json'
        },
        body: JSON.stringify({ productId: 1, quantity: 2 })
      }
    ]
  });
  
  autocannon.track(instance, { renderProgressBar: false });
  
  const output = new PassThrough();
  autocannon.track(instance, { outputStream: output });
  
  output.on('data', (data) => {
    console.log(data.toString());
  });
  
  instance.on('done', (result) => {
    console.log('Load test complete:', result);
  });
}

runLoadTest();
```

---

## **10. Production Deployment** <a name="production"></a>

### **Environment Configuration**
```javascript
// config/config.js
const dotenv = require('dotenv');

// Load environment variables
dotenv.config({ path: `.env.${process.env.NODE_ENV || 'development'}` });

module.exports = {
  env: process.env.NODE_ENV,
  port: process.env.PORT,
  jwtSecret: process.env.JWT_SECRET,
  jwtExpiresIn: process.env.JWT_EXPIRES_IN,
  database: {
    uri: process.env.NODE_ENV === 'production' 
      ? process.env.DATABASE_URL.replace(
        '<password>',
        process.env.DATABASE_PASSWORD
      )
      : process.env.DATABASE_LOCAL
  },
  email: {
    host: process.env.EMAIL_HOST,
    port: process.env.EMAIL_PORT,
    username: process.env.EMAIL_USERNAME,
    password: process.env.EMAIL_PASSWORD
  },
  redis: {
    host: process.env.REDIS_HOST,
    port: process.env.REDIS_PORT,
    password: process.env.REDIS_PASSWORD
  }
};
```

### **PM2 Configuration**
```javascript
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'my-express-app',
    script: './server.js',
    instances: 'max', // Use all CPU cores
    exec_mode: 'cluster', // Enable clustering
    autorestart: true,
    watch: false,
    max_memory_restart: '1G', // Restart if memory exceeds 1GB
    env: {
      NODE_ENV: 'development',
      PORT: 3000
    },
    env_production: {
      NODE_ENV: 'production',
      PORT: 80
    },
    error_file: './logs/err.log',
    out_file: './logs/out.log',
    log_file: './logs/combined.log',
    time: true,
    merge_logs: true,
    log_date_format: 'YYYY-MM-DD HH:mm:ss',
    // Graceful shutdown
    kill_timeout: 5000,
    listen_timeout: 5000
  }],
  
  // Deployment configuration
  deploy: {
    production: {
      user: 'deploy',
      host: ['server1.example.com', 'server2.example.com'],
      ref: 'origin/main',
      repo: 'git@github.com:user/repo.git',
      path: '/var/www/my-express-app',
      'post-deploy': 'npm install && pm2 reload ecosystem.config.js --env production',
      'pre-setup': 'echo "Setting up the server"'
    }
  }
};
```

### **Docker Configuration**
```dockerfile
# Dockerfile
FROM node:18-alpine

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
COPY package*.json ./
RUN npm ci --only=production

# Bundle app source
COPY . .

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js || exit 1

# Run the app
CMD ["node", "server.js"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://user:pass@db:5432/dbname
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    volumes:
      - ./logs:/usr/src/app/logs
    restart: unless-stopped
    networks:
      - app-network

  db:
    image: postgres:14-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=dbname
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    command: redis-server --requirepass redispass
    volumes:
      - redis-data:/data
    networks:
      - app-network

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/ssl
    depends_on:
      - app
    networks:
      - app-network

volumes:
  postgres-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```

### **NGINX Configuration**
```nginx
# nginx.conf
events {
    worker_connections 1024;
}

http {
    upstream nodejs_backend {
        least_conn;
        server app:3000;
        
        # Health check
        check interval=3000 rise=2 fall=3 timeout=1000;
    }
    
    server {
        listen 80;
        server_name example.com;
        return 301 https://$server_name$request_uri;
    }
    
    server {
        listen 443 ssl http2;
        server_name example.com;
        
        ssl_certificate /etc/ssl/example.com.crt;
        ssl_certificate_key /etc/ssl/example.com.key;
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers HIGH:!aNULL:!MD5;
        
        # Security headers
        add_header X-Frame-Options DENY;
        add_header X-Content-Type-Options nosniff;
        add_header X-XSS-Protection "1; mode=block";
        
        # Compression
        gzip on;
        gzip_vary on;
        gzip_min_length 1024;
        gzip_types text/plain text/css application/json application/javascript;
        
        # Proxy settings
        location / {
            proxy_pass http://nodejs_backend;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_cache_bypass $http_upgrade;
            
            # Timeouts
            proxy_connect_timeout 60s;
            proxy_send_timeout 60s;
            proxy_read_timeout 60s;
        }
        
        # Static files
        location /static/ {
            alias /var/www/static/;
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    }
}
```

---

## **11. Real-World Architecture** <a name="architecture"></a>

### **Project Structure**
```
my-express-app/
├── src/
│   ├── app.js                 # Express app configuration
│   ├── server.js              # Server entry point
│   ├── config/                # Configuration files
│   │   ├── config.js
│   │   ├── database.js
│   │   └── redis.js
│   ├── controllers/           # Route controllers
│   │   ├── authController.js
│   │   ├── userController.js
│   │   └── productController.js
│   ├── models/                # Database models
│   │   ├── userModel.js
│   │   └── productModel.js
│   ├── routes/                # Route definitions
│   │   ├── authRoutes.js
│   │   ├── userRoutes.js
│   │   └── productRoutes.js
│   ├── middleware/            # Custom middleware
│   │   ├── auth.js
│   │   ├── errorHandler.js
│   │   └── validation.js
│   ├── services/              # Business logic
│   │   ├── authService.js
│   │   ├── emailService.js
│   │   └── paymentService.js
│   ├── utils/                 # Utility functions
│   │   ├── appError.js
│   │   ├── catchAsync.js
│   │   └── logger.js
│   └── public/                # Static files
│       ├── css/
│       ├── js/
│       └── images/
├── tests/                     # Test files
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── logs/                      # Application logs
├── .env.example               # Environment variables example
├── .env.production            # Production environment
├── .eslintrc.js               # ESLint configuration
├── .prettierrc                # Prettier configuration
├── package.json
├── Dockerfile
├── docker-compose.yml
└── README.md
```

### **Service Layer Architecture**
```javascript
// services/userService.js
class UserService {
  constructor(userModel) {
    this.userModel = userModel;
  }
  
  async createUser(userData) {
    // Business logic here
    const existingUser = await this.userModel.findOne({ 
      email: userData.email 
    });
    
    if (existingUser) {
      throw new AppError('Email already exists', 400);
    }
    
    const user = await this.userModel.create(userData);
    
    // Send welcome email
    await this.emailService.sendWelcomeEmail(user.email, user.name);
    
    // Create audit log
    await this.auditService.log('USER_CREATED', user._id);
    
    return user;
  }
  
  async updateUser(userId, updateData) {
    // More business logic
    const user = await this.userModel.findById(userId);
    
    if (!user) {
      throw new AppError('User not found', 404);
    }
    
    Object.assign(user, updateData);
    await user.save();
    
    return user;
  }
  
  async deleteUser(userId) {
    // Soft delete
    const user = await this.userModel.findById(userId);
    
    if (!user) {
      throw new AppError('User not found', 404);
    }
    
    user.deleted = true;
    user.deletedAt = new Date();
    await user.save();
    
    return { message: 'User deleted successfully' };
  }
}

module.exports = UserService;
```

```javascript
// controllers/userController.js
const UserService = require('../services/userService');
const UserModel = require('../models/userModel');

const userService = new UserService(UserModel);

exports.createUser = async (req, res, next) => {
  try {
    const user = await userService.createUser(req.body);
    
    res.status(201).json({
      status: 'success',
      data: { user }
    });
  } catch (error) {
    next(error);
  }
};

exports.updateUser = async (req, res, next) => {
  try {
    const user = await userService.updateUser(req.params.id, req.body);
    
    res.status(200).json({
      status: 'success',
      data: { user }
    });
  } catch (error) {
    next(error);
  }
};
```

### **Event-Driven Architecture**
```javascript
// events/eventEmitter.js
const EventEmitter = require('events');
class AppEventEmitter extends EventEmitter {}
const appEvents = new AppEventEmitter();

// Set max listeners
appEvents.setMaxListeners(20);

module.exports = appEvents;

// services/notificationService.js
const appEvents = require('../events/eventEmitter');

class NotificationService {
  constructor() {
    this.setupListeners();
  }
  
  setupListeners() {
    appEvents.on('user.registered', async (user) => {
      await this.sendWelcomeEmail(user);
      await this.sendAdminNotification(user);
    });
    
    appEvents.on('order.placed', async (order) => {
      await this.sendOrderConfirmation(order);
      await this.updateInventory(order);
    });
    
    appEvents.on('payment.received', async (payment) => {
      await this.sendPaymentReceipt(payment);
      await this.fulfillOrder(payment.orderId);
    });
  }
  
  async sendWelcomeEmail(user) {
    // Send email logic
  }
  
  async sendOrderConfirmation(order) {
    // Send confirmation logic
  }
}

module.exports = NotificationService;
```

### **Advanced Error Handling Pattern**
```javascript
// utils/catchAsync.js
module.exports = (fn) => {
  return (req, res, next) => {
    fn(req, res, next).catch(next);
  };
};

// Usage in controllers
exports.createUser = catchAsync(async (req, res, next) => {
  const user = await User.create(req.body);
  res.status(201).json({
    status: 'success',
    data: { user }
  });
});

// utils/apiFeatures.js - Advanced querying
class APIFeatures {
  constructor(query, queryString) {
    this.query = query;
    this.queryString = queryString;
  }
  
  filter() {
    const queryObj = { ...this.queryString };
    const excludedFields = ['page', 'sort', 'limit', 'fields'];
    excludedFields.forEach(el => delete queryObj[el]);
    
    // Advanced filtering
    let queryStr = JSON.stringify(queryObj);
    queryStr = queryStr.replace(/\b(gte|gt|lte|lt)\b/g, match => `$${match}`);
    
    this.query = this.query.find(JSON.parse(queryStr));
    return this;
  }
  
  sort() {
    if (this.queryString.sort) {
      const sortBy = this.queryString.sort.split(',').join(' ');
      this.query = this.query.sort(sortBy);
    } else {
      this.query = this.query.sort('-createdAt');
    }
    return this;
  }
  
  limitFields() {
    if (this.queryString.fields) {
      const fields = this.queryString.fields.split(',').join(' ');
      this.query = this.query.select(fields);
    } else {
      this.query = this.query.select('-__v');
    }
    return this;
  }
  
  paginate() {
    const page = this.queryString.page * 1 || 1;
    const limit = this.queryString.limit * 1 || 100;
    const skip = (page - 1) * limit;
    
    this.query = this.query.skip(skip).limit(limit);
    return this;
  }
}

// Usage in controller
exports.getAllUsers = catchAsync(async (req, res, next) => {
  const features = new APIFeatures(User.find(), req.query)
    .filter()
    .sort()
    .limitFields()
    .paginate();
    
  const users = await features.query;
  
  res.status(200).json({
    status: 'success',
    results: users.length,
    data: { users }
  });
});
```

## **Final Production-Ready Server**
```javascript
// server.js - Complete production server
const express = require('express');
const helmet = require('helmet');
const cors = require('cors');
const morgan = require('morgan');
const compression = require('compression');
const rateLimit = require('express-rate-limit');
const mongoSanitize = require('express-mongo-sanitize');
const xss = require('xss-clean');
const hpp = require('hpp');
const cookieParser = require('cookie-parser');

const AppError = require('./utils/appError');
const globalErrorHandler = require('./middleware/errorHandler');
const userRouter = require('./routes/userRoutes');
const productRouter = require('./routes/productRoutes');
const authRouter = require('./routes/authRoutes');

const app = express();

// 1) GLOBAL MIDDLEWARE

// Security HTTP headers
app.use(helmet());

// Development logging
if (process.env.NODE_ENV === 'development') {
  app.use(morgan('dev'));
}

// Rate limiting
const limiter = rateLimit({
  max: 100,
  windowMs: 60 * 60 * 1000,
  message: 'Too many requests from this IP, please try again in an hour!'
});
app.use('/api', limiter);

// Body parser, reading data from body into req.body
app.use(express.json({ limit: '10kb' }));
app.use(express.urlencoded({ extended: true, limit: '10kb' }));
app.use(cookieParser());

// Data sanitization against NoSQL query injection
app.use(mongoSanitize());

// Data sanitization against XSS
app.use(xss());

// Prevent parameter pollution
app.use(hpp({
  whitelist: ['price', 'rating', 'duration']
}));

// Compression
app.use(compression());

// CORS
app.use(cors({
  origin: process.env.CLIENT_URL,
  credentials: true
}));

// Test middleware
app.use((req, res, next) => {
  req.requestTime = new Date().toISOString();
  next();
});

// 2) ROUTES
app.use('/api/v1/auth', authRouter);
app.use('/api/v1/users', userRouter);
app.use('/api/v1/products', productRouter);

// 3) ERROR HANDLING
app.all('*', (req, res, next) => {
  next(new AppError(`Can't find ${req.originalUrl} on this server!`, 404));
});

app.use(globalErrorHandler);

// 4) START SERVER
const PORT = process.env.PORT || 3000;
const server = app.listen(PORT, () => {
  console.log(`App running on port ${PORT}...`);
});

// Handle unhandled rejections
process.on('unhandledRejection', (err) => {
  console.error('UNHANDLED REJECTION! 💥 Shutting down...');
  console.error(err.name, err.message);
  server.close(() => {
    process.exit(1);
  });
});

// Handle uncaught exceptions
process.on('uncaughtException', (err) => {
  console.error('UNCAUGHT EXCEPTION! 💥 Shutting down...');
  console.error(err.name, err.message);
  process.exit(1);
});
```

## **Key Takeaways**

1. **Middleware First**: Understand middleware execution order
2. **Error Handling**: Implement comprehensive error handling
3. **Security**: Always use security middleware (helmet, CORS, rate limiting)
4. **Separation of Concerns**: Use controllers, services, and models
5. **Testing**: Write tests for all critical paths
6. **Monitoring**: Implement logging and monitoring
7. **Performance**: Use caching, compression, and connection pooling
8. **Scalability**: Design for horizontal scaling from the start
9. **Documentation**: Document your API endpoints
10. **Production Ready**: Always prepare for production deployment

Express.js provides the foundation, but how you structure and secure your application determines its success in production environments.