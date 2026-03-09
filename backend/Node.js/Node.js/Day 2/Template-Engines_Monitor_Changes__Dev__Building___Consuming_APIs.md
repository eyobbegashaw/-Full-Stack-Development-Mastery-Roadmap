# **Building & Consuming APIs in Node.js - Detailed Notes & Code**

## **Web Frameworks**

### **Express.js**
Express.js is a **minimal, flexible Node.js web framework** for building web applications and APIs. It provides routing, middleware, template engines, and HTTP utilities. Middleware functions handle requests and responses. Express simplifies server creation, request handling, and response sending. It's the most popular Node.js framework with vast middleware ecosystem.

```javascript
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

### **fastify**
fastify is a **fast, low-overhead web framework** focused on performance. It claims up to 2x faster than Express. Features include schema-based validation, logging, dependency injection, and async/await support. Uses JSON Schema for validation and serialization. Great for high-performance APIs and microservices.

```javascript
const fastify = require('fastify')({ logger: true });

fastify.get('/', async (request, reply) => {
  return { hello: 'world' };
});

fastify.listen({ port: 3000 }, (err) => {
  if (err) {
    fastify.log.error(err);
    process.exit(1);
  }
});
```

## **Development Tools**

### **Monitor Changes (Dev)**
Monitoring changes during development automatically restarts the server when files change. This improves workflow by eliminating manual restarts. Tools like `nodemon` and Node's built-in `--watch` flag watch file changes and restart the process. Essential for efficient development.

```bash
# Using nodemon
nodemon server.js

# Using Node.js built-in watch mode
node --watch server.js
```

### **--watch (Node.js built-in)**
Node.js 18+ includes **built-in file watching** with `--watch` flag. It restarts the process when imported files change. No additional dependencies needed. Works with ES modules and CommonJS. Similar to nodemon but built into Node.js runtime.

```bash
# Watch for changes and restart
node --watch server.js

# With inspect for debugging
node --watch --inspect server.js
```

### **nodemon**
nodemon is a **tool that monitors file changes** and automatically restarts Node.js applications. More feature-rich than built-in `--watch`. Supports configuration files, ignoring files/directories, watching extensions, and running non-Node scripts.

```bash
# Install globally
npm install -g nodemon

# Basic usage
nodemon app.js

# With custom config in package.json
"scripts": {
  "dev": "nodemon server.js"
}
```

```json
// nodemon.json configuration
{
  "watch": ["src", "config"],
  "ignore": ["node_modules", "*.test.js"],
  "ext": "js,json",
  "exec": "node server.js"
}
```

## **Template Engines**

### **Template Engines**
Template engines generate HTML dynamically by injecting data into templates. They separate presentation from logic, enable reuse, and support layouts/partials. Popular engines: EJS, Pug, Handlebars. Configure in Express with `app.set('view engine', 'ejs')`.

```javascript
// Express with EJS
app.set('view engine', 'ejs');
app.get('/', (req, res) => {
  res.render('index', { title: 'Home', user: req.user });
});
```

### **ejs**
EJS (Embedded JavaScript) embeds JavaScript code in HTML templates with `<% %>` tags. Simple syntax similar to plain HTML. Supports includes, layouts via partials. Renders on server-side.

```html
<!-- views/index.ejs -->
<!DOCTYPE html>
<html>
<head>
  <title><%= title %></title>
</head>
<body>
  <h1>Welcome <%= user.name %></h1>
  <% if (user.isAdmin) { %>
    <p>Admin Dashboard</p>
  <% } %>
  <ul>
    <% users.forEach(user => { %>
      <li><%= user.name %></li>
    <% }) %>
  </ul>
</body>
</html>
```

### **pug**
Pug (formerly Jade) uses **indentation-based syntax** (no closing tags). Clean, concise templates. Supports mixins, includes, inheritance. Converts to HTML. Steeper learning curve but less verbose.

```pug
//- views/layout.pug
doctype html
html
  head
    title= title
    block styles
  body
    block content

//- views/index.pug  
extends layout.pug

block content
  h1 Welcome #{user.name}
  if user.isAdmin
    p Admin Dashboard
  ul
    each user in users
      li= user.name
```

### **marko**
Marko is a **high-performance template engine** by eBay. Supports streaming, async rendering, and components. Used for server-side rendering with minimal overhead. Integrates with Express and other frameworks.

```javascript
// Express with Marko
app.get('/', (req, res) => {
  res.marko(require('./views/index.marko'), {
    name: 'World'
  });
});
```

```html
<!-- views/index.marko -->
<h1>Hello ${input.name}!</h1>
<ul>
  <for|user| of=input.users>
    <li>${user.name}</li>
  </for>
</ul>
```

## **Making API Calls**

### **Making API Calls**
Making HTTP requests to external APIs is common in Node.js for data integration, microservices, or third-party services. Multiple libraries: `axios`, `fetch` (Node 18+), `got`, `node-fetch`. Handle errors, timeouts, retries, and response parsing.

```javascript
// Basic API call pattern
async function fetchData(url) {
  try {
    const response = await axios.get(url);
    return response.data;
  } catch (error) {
    console.error('API call failed:', error.message);
    throw error;
  }
}
```

### **axios**
axios is a **Promise-based HTTP client** for Node.js and browsers. Features: interceptors, request/response transformation, cancel tokens, timeout, automatic JSON parsing. Most popular HTTP client with comprehensive features.

```javascript
const axios = require('axios');

// GET request
const response = await axios.get('https://api.example.com/data', {
  params: { page: 1, limit: 10 },
  headers: { 'Authorization': `Bearer ${token}` },
  timeout: 5000
});

// POST request
await axios.post('https://api.example.com/users', {
  name: 'John',
  email: 'john@example.com'
});

// Interceptors
axios.interceptors.request.use(config => {
  config.headers['X-Request-ID'] = uuid.v4();
  return config;
});
```

### **fetch (Node.js built-in)**
Node.js 18+ includes **built-in fetch API** (same as browser fetch). No dependencies needed. Modern, promise-based, supports streams. Less features than axios but lightweight. Polyfill needed for older Node versions.

```javascript
// Using built-in fetch
async function getData() {
  try {
    const response = await fetch('https://api.example.com/data');
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Fetch failed:', error);
  }
}

// POST with fetch
const response = await fetch('https://api.example.com/users', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: 'John' })
});
```

### **got package**
`got` is a **human-friendly, lightweight HTTP client**. Feature-rich with streams, retries, pagination, hooks. Designed specifically for Node.js (not browsers). Used by many popular packages.

```javascript
const got = require('got');

// Simple GET
const data = await got.get('https://api.example.com/data').json();

// With options
const response = await got('https://api.example.com/users', {
  method: 'POST',
  json: { name: 'John' },
  responseType: 'json',
  retry: { limit: 3 },
  timeout: { request: 5000 }
});
```

## **Authentication & Security**

### **Authentication**
Authentication verifies user identity in web applications. Common methods: JWT (JSON Web Tokens), sessions, OAuth. Implement login, registration, password reset, and protected routes. Always hash passwords, use HTTPS, and validate inputs.

```javascript
// Basic authentication middleware
function authenticate(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'No token provided' });
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' });
  }
}
```

### **jsonwebtoken**
`jsonwebtoken` handles **JWT creation and verification**. JWTs are compact, URL-safe tokens containing claims. Used for authentication and information exchange. Consists of header, payload, signature. Store in HTTP-only cookies or Authorization header.

```javascript
const jwt = require('jsonwebtoken');

// Create token
const token = jwt.sign(
  { userId: user.id, role: user.role },
  process.env.JWT_SECRET,
  { expiresIn: '1h' }
);

// Verify token
try {
  const decoded = jwt.verify(token, process.env.JWT_SECRET);
  console.log('User ID:', decoded.userId);
} catch (error) {
  console.error('Invalid token:', error.message);
}

// Middleware example
function authMiddleware(req, res, next) {
  const token = req.cookies.token || req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).send('Access denied');
  
  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch (error) {
    res.status(400).send('Invalid token');
  }
}
```

### **passport.js**
Passport is **authentication middleware** with 500+ strategies (local, JWT, OAuth, OpenID). Simplifies authentication implementation. Uses strategies, sessions, and serialization. Flexible and modular.

```javascript
const passport = require('passport');
const LocalStrategy = require('passport-local').Strategy;
const JwtStrategy = require('passport-jwt').Strategy;

// Local strategy (username/password)
passport.use(new LocalStrategy(
  async (username, password, done) => {
    try {
      const user = await User.findOne({ username });
      if (!user) return done(null, false);
      
      const isValid = await user.validatePassword(password);
      return isValid ? done(null, user) : done(null, false);
    } catch (error) {
      return done(error);
    }
  }
));

// JWT strategy
passport.use(new JwtStrategy({
  jwtFromRequest: req => req.cookies.jwt,
  secretOrKey: process.env.JWT_SECRET
}, (payload, done) => {
  User.findById(payload.sub, (err, user) => {
    if (err) return done(err, false);
    return user ? done(null, user) : done(null, false);
  });
}));

// In Express routes
app.post('/login', 
  passport.authenticate('local', { 
    failureRedirect: '/login',
    failureFlash: true 
  }),
  (req, res) => {
    res.redirect('/dashboard');
  }
);
```

## **Complete API Example**

```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');
const axios = require('axios');
require('dotenv').config();

const app = express();
app.use(express.json());

// Mock database
const users = [];

// Middleware
function authenticateToken(req, res, next) {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];
  
  if (!token) return res.sendStatus(401);
  
  jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
    if (err) return res.sendStatus(403);
    req.user = user;
    next();
  });
}

// Routes
app.post('/register', async (req, res) => {
  try {
    const { username, password } = req.body;
    const hashedPassword = await bcrypt.hash(password, 10);
    const user = { id: users.length + 1, username, password: hashedPassword };
    users.push(user);
    res.status(201).json({ message: 'User created', userId: user.id });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.post('/login', async (req, res) => {
  const { username, password } = req.body;
  const user = users.find(u => u.username === username);
  
  if (!user || !await bcrypt.compare(password, user.password)) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  const token = jwt.sign(
    { userId: user.id, username: user.username },
    process.env.JWT_SECRET,
    { expiresIn: '1h' }
  );
  
  res.json({ token });
});

app.get('/profile', authenticateToken, (req, res) => {
  const user = users.find(u => u.id === req.user.userId);
  res.json({ username: user.username, id: user.id });
});

// External API integration
app.get('/github/:username', async (req, res) => {
  try {
    const response = await axios.get(
      `https://api.github.com/users/${req.params.username}`,
      { headers: { 'User-Agent': 'MyApp' } }
    );
    res.json({
      username: response.data.login,
      name: response.data.name,
      repos: response.data.public_repos
    });
  } catch (error) {
    res.status(404).json({ error: 'GitHub user not found' });
  }
});

// Template rendering
app.set('view engine', 'ejs');
app.get('/dashboard', authenticateToken, (req, res) => {
  const user = users.find(u => u.id === req.user.userId);
  res.render('dashboard', { 
    title: 'Dashboard', 
    user: { username: user.username },
    lastLogin: new Date().toLocaleDateString()
  });
});

// Start server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
  console.log(`Development mode: ${process.env.NODE_ENV || 'development'}`);
});
```

## **Best Practices**

1. **Always validate input** on API endpoints
2. **Use environment variables** for secrets and configuration
3. **Implement proper error handling** with status codes
4. **Rate limiting** to prevent abuse
5. **CORS configuration** for web clients
6. **Input sanitization** to prevent injection attacks
7. **HTTPS in production** for security
8. **Logging and monitoring** for debugging and analytics

```javascript
// Security middleware example
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const cors = require('cors');

app.use(helmet()); // Security headers
app.use(cors({ origin: process.env.CLIENT_URL }));
app.use(rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
}));
```

These tools and patterns enable building secure, efficient, and maintainable APIs and web applications in Node.js, from simple servers to complex enterprise applications.