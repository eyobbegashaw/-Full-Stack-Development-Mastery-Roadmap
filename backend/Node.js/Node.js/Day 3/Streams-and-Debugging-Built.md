# **Advanced Node.js: Streams, Memory & Debugging - Detailed Notes & Code**

## **Streams**

### **Streams**
Streams are collections of data that might not be available all at once. They process data in chunks, enabling efficient handling of large files, network communication, or real-time data. Types: Readable (source), Writable (destination), Duplex (both), Transform (modify). Streams prevent memory overload by processing data incrementally. Use for file I/O, HTTP requests/responses, compression, and data transformation.

```javascript
const fs = require('fs');

// Read large file in chunks
const readable = fs.createReadStream('largefile.txt', { highWaterMark: 64 * 1024 });
const writable = fs.createWriteStream('output.txt');

readable.pipe(writable); // Pipe data from source to destination

// Or process chunks manually
readable.on('data', (chunk) => {
  console.log(`Received ${chunk.length} bytes`);
  writable.write(chunk);
});
```

### **Stream Types & Patterns**
```javascript
const { Readable, Writable, Transform, pipeline } = require('stream');
const { createGzip } = require('zlib');

// Custom Readable stream
class CounterStream extends Readable {
  constructor(max, options) {
    super(options);
    this.max = max;
    this.index = 0;
  }

  _read() {
    this.index++;
    if (this.index > this.max) {
      this.push(null); // End stream
    } else {
      const buf = Buffer.from(`${this.index}\n`);
      this.push(buf);
    }
  }
}

// Custom Transform stream
class UpperCaseTransform extends Transform {
  _transform(chunk, encoding, callback) {
    this.push(chunk.toString().toUpperCase());
    callback();
  }
}

// Custom Writable stream
class LoggerStream extends Writable {
  _write(chunk, encoding, callback) {
    console.log(`Log: ${chunk.toString()}`);
    callback();
  }
}

// Usage
const counter = new CounterStream(5);
const upper = new UpperCaseTransform();
const logger = new LoggerStream();

counter.pipe(upper).pipe(logger);

// Pipeline with error handling
pipeline(
  fs.createReadStream('input.txt'),
  createGzip(),
  fs.createWriteStream('input.txt.gz'),
  (err) => {
    if (err) {
      console.error('Pipeline failed:', err);
    } else {
      console.log('Pipeline succeeded');
    }
  }
);
```

## **Memory Management**

### **Garbage Collection**
Garbage Collection (GC) automatically frees memory occupied by unused objects. Node.js uses V8's generational GC with two main phases: Scavenge (young generation) and Mark-Sweep/Compact (old generation). GC runs when memory allocation fails or periodically. Monitor with `--trace-gc` flag. Tune with `--max-old-space-size`. Avoid creating many short-lived objects.

```javascript
// Monitor GC events
const v8 = require('v8');

// Get heap statistics
const heapStats = v8.getHeapStatistics();
console.log('Heap Statistics:', heapStats);

// Force garbage collection (development only)
if (global.gc) {
  global.gc();
} else {
  console.log('Run with --expose-gc flag');
}

// Memory usage
const used = process.memoryUsage();
console.log(`RSS: ${Math.round(used.rss / 1024 / 1024)}MB`);
console.log(`Heap Total: ${Math.round(used.heapTotal / 1024 / 1024)}MB`);
console.log(`Heap Used: ${Math.round(used.heapUsed / 1024 / 1024)}MB`);
```

### **Memory Leaks**
Memory leaks occur when memory is allocated but never freed, causing gradual memory growth. Common causes: global variables, closures, timers, event listeners, caching without limits. Detect with heap snapshots, memory profiling, and monitoring tools. Fix by removing references and cleaning up resources.

```javascript
// Common memory leak patterns and fixes

// LEAK: Global variables
function leaky() {
  leakedArray = new Array(1000000).fill('*'); // Missing var/let/const
}
// FIX: Always declare variables
function fixed() {
  const localArray = new Array(1000000).fill('*');
}

// LEAK: Unremoved event listeners
const EventEmitter = require('events');
class LeakyEmitter extends EventEmitter {}
const emitter = new LeakyEmitter();

function createListener() {
  const obj = { data: new Array(10000).fill('*') };
  emitter.on('event', () => console.log(obj.data.length));
}
// FIX: Remove listeners
function createFixedListener() {
  const obj = { data: new Array(10000).fill('*') };
  const handler = () => console.log(obj.data.length);
  emitter.on('event', handler);
  // Later remove it
  // emitter.removeListener('event', handler);
}

// LEAK: Timers not cleared
function startLeakyTimer() {
  setInterval(() => {
    const data = new Array(10000).fill('*');
    console.log('Tick');
  }, 1000);
}
// FIX: Store timer ID and clear
function startFixedTimer() {
  const timerId = setInterval(() => {
    const data = new Array(10000).fill('*');
    console.log('Tick');
  }, 1000);
  // Later clear it
  // clearInterval(timerId);
}

// LEAK: Caching without limits
const cache = {};
function getData(key) {
  if (!cache[key]) {
    cache[key] = fetchExpensiveData(key);
  }
  return cache[key];
}
// FIX: Use LRU cache
const LRU = require('lru-cache');
const lruCache = new LRU({ max: 100, maxAge: 1000 * 60 * 60 });
```

## **Debugging & Monitoring**

### **Debugging**
Debugging identifies and fixes bugs in Node.js applications. Techniques: console logging, debugger statements, Chrome DevTools, IDE debuggers, and Node's inspect flag. Use `console.trace()` for stack traces, `util.inspect()` for object inspection. Structured logging helps track down issues in production.

```javascript
const util = require('util');

// Debug logging
console.log('Standard log');
console.debug('Debug info');
console.info('Information');
console.warn('Warning');
console.error('Error');

// Object inspection
const obj = { 
  name: 'John', 
  address: { city: 'NYC', zip: '10001' },
  nested: { deep: { data: 'value' } }
};
console.log(util.inspect(obj, { 
  depth: null,           // Show all levels
  colors: true,          // Colorized output
  compact: false         // Multi-line
}));

// Stack traces
function a() { b(); }
function b() { c(); }
function c() { 
  console.trace('Call stack:'); 
  // Alternative: console.log(new Error().stack);
}
a();

// Debugger statement
function buggyFunction() {
  const x = 10;
  const y = 0;
  debugger; // Execution pauses here in debugger
  return x / y;
}
```

### **node --inspect**
`node --inspect` enables debugging via Chrome DevTools. Opens a WebSocket server for debugging. Use with `--inspect-brk` to break on first line. Connect via `chrome://inspect` in Chrome. Supports breakpoints, stepping, variable inspection, and profiling. Alternative: `ndb` package for enhanced debugging.

```bash
# Basic inspect
node --inspect app.js

# Break on first line
node --inspect-brk app.js

# Specify port
node --inspect=9229 app.js

# With custom debugger (like VS Code)
node --inspect --inspect-brk app.js
```

```json
// VS Code launch.json configuration
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Launch Program",
      "skipFiles": ["<node_internals>/**"],
      "program": "${workspaceFolder}/app.js"
    },
    {
      "type": "node",
      "request": "attach",
      "name": "Attach to Process",
      "port": 9229,
      "skipFiles": ["<node_internals>/**"]
    }
  ]
}
```

### **Using APM**
APM (Application Performance Monitoring) tracks application performance, errors, and metrics in production. Tools: New Relic, Datadog, AppDynamics, Elastic APM. Features: request tracing, error tracking, database query monitoring, custom metrics. Essential for identifying bottlenecks and issues in production.

```javascript
// New Relic setup
require('newrelic');

// newrelic.js configuration
'use strict';
exports.config = {
  app_name: ['My Application'],
  license_key: 'YOUR_LICENSE_KEY',
  logging: {
    level: 'info'
  },
  allow_all_headers: true,
  attributes: {
    exclude: [
      'request.headers.cookie',
      'request.headers.authorization',
      'request.headers.proxyAuthorization',
      'request.headers.setCookie*',
      'request.headers.x*',
      'response.headers.cookie',
      'response.headers.authorization',
      'response.headers.proxyAuthorization',
      'response.headers.setCookie*',
      'response.headers.x*'
    ]
  }
};

// Custom metrics
const newrelic = require('newrelic');
newrelic.recordMetric('Custom/Users/Registered', 150);
newrelic.incrementMetric('Custom/PageViews');
newrelic.recordCustomEvent('UserAction', {
  action: 'login',
  userId: '12345'
});
```

```javascript
// Elastic APM setup
const apm = require('elastic-apm-node').start({
  serviceName: 'my-service',
  secretToken: 'YOUR_SECRET_TOKEN',
  serverUrl: 'http://localhost:8200',
  environment: 'production'
});

// Custom transactions
const transaction = apm.startTransaction('my-transaction');
try {
  // Your code here
  const result = await someAsyncOperation();
  transaction.result = 'success';
} catch (err) {
  apm.captureError(err);
  transaction.result = 'failure';
} finally {
  transaction.end();
}

// Custom spans
const span = apm.startSpan('my-operation');
// Perform operation
span.end();
```

## **Common Built-in Modules**

### **Essential Built-in Modules**
```javascript
// 1. fs - File System
const fs = require('fs').promises;
async function fileOps() {
  await fs.writeFile('test.txt', 'Hello');
  const data = await fs.readFile('test.txt', 'utf8');
  await fs.unlink('test.txt');
}

// 2. path - Path Utilities
const path = require('path');
const fullPath = path.join(__dirname, 'data', 'file.json');
const ext = path.extname(fullPath); // .json
const basename = path.basename(fullPath); // file.json

// 3. http/https - HTTP Servers/Clients
const http = require('http');
const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello World');
});
server.listen(3000);

// 4. crypto - Cryptography
const crypto = require('crypto');
const hash = crypto.createHash('sha256');
hash.update('password');
const digest = hash.digest('hex');

// 5. events - Event Emitter
const EventEmitter = require('events');
class MyEmitter extends EventEmitter {}
const myEmitter = new MyEmitter();
myEmitter.on('event', () => console.log('Event fired'));
myEmitter.emit('event');

// 6. util - Utilities
const util = require('util');
const sleep = util.promisify(setTimeout);
await sleep(1000);

// 7. os - Operating System
const os = require('os');
console.log('CPUs:', os.cpus().length);
console.log('Free mem:', os.freemem() / 1024 / 1024, 'MB');

// 8. child_process - Child Processes
const { exec } = require('child_process');
exec('ls -la', (error, stdout, stderr) => {
  console.log(stdout);
});

// 9. cluster - Clustering
const cluster = require('cluster');
if (cluster.isMaster) {
  cluster.fork();
} else {
  // Worker code
}

// 10. stream - Streams (as covered above)
```

### **Lesser-Known Useful Modules**
```javascript
// 1. assert - Assertions
const assert = require('assert');
assert.strictEqual(1 + 1, 2);

// 2. buffer - Binary Data
const { Buffer } = require('buffer');
const buf = Buffer.from('Hello', 'utf8');

// 3. url - URL Parsing
const { URL } = require('url');
const myUrl = new URL('https://example.com/path?query=1');

// 4. querystring - Query String Parsing
const querystring = require('querystring');
const parsed = querystring.parse('name=John&age=30');

// 5. zlib - Compression
const zlib = require('zlib');
const gzip = zlib.createGzip();

// 6. readline - Read Input
const readline = require('readline');
const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
});

// 7. perf_hooks - Performance Timing
const { performance, PerformanceObserver } = require('perf_hooks');
const obs = new PerformanceObserver((items) => {
  console.log(items.getEntries()[0].duration);
});
obs.observe({ entryTypes: ['measure'] });

performance.mark('start');
// ... code to measure
performance.mark('end');
performance.measure('My Operation', 'start', 'end');

// 8. worker_threads - Worker Threads
const { Worker } = require('worker_threads');
```

## **Complete Monitoring & Debugging Setup**

### **Memory Leak Detection Script**
```javascript
// memory-monitor.js
const v8 = require('v8');
const heapdump = require('heapdump');
const fs = require('fs').promises;

class MemoryMonitor {
  constructor(interval = 60000, threshold = 0.1) {
    this.interval = interval;
    this.threshold = threshold; // 10% growth threshold
    this.previousHeapUsed = 0;
    this.leakCount = 0;
    this.maxLeaks = 3;
  }

  start() {
    console.log('Starting memory monitor...');
    this.timer = setInterval(() => this.checkMemory(), this.interval);
  }

  stop() {
    if (this.timer) {
      clearInterval(this.timer);
      console.log('Memory monitor stopped');
    }
  }

  async checkMemory() {
    const currentHeapUsed = process.memoryUsage().heapUsed;
    const heapGrowth = currentHeapUsed - this.previousHeapUsed;
    
    console.log(`Memory: ${(currentHeapUsed / 1024 / 1024).toFixed(2)}MB`);
    
    if (this.previousHeapUsed > 0) {
      const growthPercentage = heapGrowth / this.previousHeapUsed;
      
      if (growthPercentage > this.threshold) {
        this.leakCount++;
        console.warn(`⚠️  Possible memory leak detected! Growth: ${(growthPercentage * 100).toFixed(2)}%`);
        
        if (this.leakCount >= this.maxLeaks) {
          console.error('🚨 Multiple memory leaks detected! Taking heap snapshot...');
          await this.takeHeapSnapshot();
          this.leakCount = 0;
        }
      }
    }
    
    this.previousHeapUsed = currentHeapUsed;
  }

  async takeHeapSnapshot() {
    const filename = `heapdump-${Date.now()}.heapsnapshot`;
    return new Promise((resolve, reject) => {
      heapdump.writeSnapshot(filename, (err, filename) => {
        if (err) {
          console.error('Failed to take heap snapshot:', err);
          reject(err);
        } else {
          console.log(`Heap snapshot saved to: ${filename}`);
          resolve(filename);
        }
      });
    });
  }

  async analyzeHeap(filepath) {
    try {
      const snapshot = require(filepath);
      const nodesByConstructor = {};
      
      snapshot.nodes.forEach(node => {
        const constructor = node.class_name;
        nodesByConstructor[constructor] = (nodesByConstructor[constructor] || 0) + 1;
      });
      
      console.log('Heap analysis:');
      Object.entries(nodesByConstructor)
        .sort((a, b) => b[1] - a[1])
        .slice(0, 10)
        .forEach(([constructor, count]) => {
          console.log(`  ${constructor}: ${count} instances`);
        });
    } catch (error) {
      console.error('Failed to analyze heap:', error);
    }
  }
}

module.exports = MemoryMonitor;
```

### **Production Debugging Endpoints**
```javascript
// debug-endpoints.js
const express = require('express');
const router = express.Router();
const v8 = require('v8');
const os = require('os');
const { performance } = require('perf_hooks');

// Security middleware - restrict to internal IPs
router.use((req, res, next) => {
  const ip = req.ip || req.connection.remoteAddress;
  const internalIps = ['127.0.0.1', '::1', '192.168.', '10.'];
  
  if (process.env.NODE_ENV === 'production' && 
      !internalIps.some(internal => ip.startsWith(internal))) {
    return res.status(403).json({ error: 'Access denied' });
  }
  next();
});

// Memory info endpoint
router.get('/memory', (req, res) => {
  const memoryUsage = process.memoryUsage();
  const heapStats = v8.getHeapStatistics();
  
  res.json({
    rss: `${(memoryUsage.rss / 1024 / 1024).toFixed(2)}MB`,
    heapTotal: `${(memoryUsage.heapTotal / 1024 / 1024).toFixed(2)}MB`,
    heapUsed: `${(memoryUsage.heapUsed / 1024 / 1024).toFixed(2)}MB`,
    external: `${(memoryUsage.external / 1024 / 1024).toFixed(2)}MB`,
    heapStats: {
      totalHeapSize: `${(heapStats.total_heap_size / 1024 / 1024).toFixed(2)}MB`,
      usedHeapSize: `${(heapStats.used_heap_size / 1024 / 1024).toFixed(2)}MB`,
      heapSizeLimit: `${(heapStats.heap_size_limit / 1024 / 1024).toFixed(2)}MB`,
      totalPhysicalSize: `${(heapStats.total_physical_size / 1024 / 1024).toFixed(2)}MB`
    }
  });
});

// System info endpoint
router.get('/system', (req, res) => {
  res.json({
    platform: os.platform(),
    arch: os.arch(),
    cpus: os.cpus().length,
    totalMemory: `${(os.totalmem() / 1024 / 1024 / 1024).toFixed(2)}GB`,
    freeMemory: `${(os.freemem() / 1024 / 1024 / 1024).toFixed(2)}GB`,
    uptime: `${os.uptime()} seconds`,
    loadAvg: os.loadavg(),
    processUptime: `${process.uptime()} seconds`
  });
});

// GC endpoint (trigger garbage collection)
router.post('/gc', (req, res) => {
  if (global.gc) {
    const before = process.memoryUsage().heapUsed;
    global.gc();
    const after = process.memoryUsage().heapUsed;
    const freed = before - after;
    
    res.json({
      success: true,
      freed: `${(freed / 1024 / 1024).toFixed(2)}MB`,
      before: `${(before / 1024 / 1024).toFixed(2)}MB`,
      after: `${(after / 1024 / 1024).toFixed(2)}MB`
    });
  } else {
    res.status(400).json({
      error: 'Garbage collection not exposed. Run with --expose-gc flag'
    });
  }
});

// Active handles/requests endpoint
router.get('/handles', (req, res) => {
  const handles = process._getActiveHandles ? process._getActiveHandles() : [];
  const requests = process._getActiveRequests ? process._getActiveRequests() : [];
  
  res.json({
    handles: handles.length,
    requests: requests.length,
    sampleHandles: handles.slice(0, 10).map(h => h.constructor.name),
    sampleRequests: requests.slice(0, 10).map(r => r.constructor.name)
  });
});

// Performance metrics endpoint
router.get('/performance', (req, res) => {
  const entries = performance.getEntriesByType('measure');
  const metrics = entries.reduce((acc, entry) => {
    acc[entry.name] = `${entry.duration.toFixed(2)}ms`;
    return acc;
  }, {});
  
  res.json({
    metrics,
    totalEntries: entries.length
  });
});

module.exports = router;
```

### **Application with Monitoring**
```javascript
// app-with-monitoring.js
const express = require('express');
const helmet = require('helmet');
const compression = require('compression');
const MemoryMonitor = require('./memory-monitor');
const debugRoutes = require('./debug-endpoints');

const app = express();

// Security and performance middleware
app.use(helmet());
app.use(compression());

// Debug routes (protected)
app.use('/_debug', debugRoutes);

// Health check
app.get('/health', (req, res) => {
  res.json({ 
    status: 'healthy', 
    timestamp: new Date().toISOString(),
    uptime: process.uptime()
  });
});

// Main application routes
app.get('/', (req, res) => {
  res.send('Hello World');
});

// Simulate memory leak endpoint
app.get('/leak', (req, res) => {
  // DON'T DO THIS IN PRODUCTION - just for testing
  if (!global.leakyArray) {
    global.leakyArray = [];
  }
  
  for (let i = 0; i < 10000; i++) {
    global.leakyArray.push(new Array(1000).fill('*'));
  }
  
  res.json({ 
    message: 'Added to leaky array',
    length: global.leakyArray.length 
  });
});

// Stream endpoint
app.get('/stream', (req, res) => {
  const { Readable } = require('stream');
  
  class DataStream extends Readable {
    constructor(options) {
      super(options);
      this.count = 0;
      this.maxCount = 100;
    }
    
    _read() {
      this.count++;
      if (this.count > this.maxCount) {
        this.push(null);
      } else {
        const data = `Chunk ${this.count}\n`;
        this.push(data);
      }
    }
  }
  
  const stream = new DataStream();
  res.setHeader('Content-Type', 'text/plain');
  stream.pipe(res);
});

// Start memory monitor
const monitor = new MemoryMonitor(30000); // Check every 30 seconds
if (process.env.NODE_ENV === 'production') {
  monitor.start();
}

// Graceful shutdown
process.on('SIGTERM', () => {
  console.log('SIGTERM received, shutting down gracefully');
  monitor.stop();
  server.close(() => {
    console.log('Server closed');
    process.exit(0);
  });
});

const PORT = process.env.PORT || 3000;
const server = app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
  console.log(`Debug endpoints available at http://localhost:${PORT}/_debug`);
  console.log(`Health check at http://localhost:${PORT}/health`);
});
```

## **Best Practices Summary**

### **Streams Best Practices**
```javascript
// 1. Always handle errors
readableStream
  .pipe(transformStream)
  .pipe(writableStream)
  .on('error', (err) => console.error('Stream error:', err));

// 2. Use pipeline for proper cleanup
const { pipeline } = require('stream');
pipeline(
  readable,
  transform,
  writable,
  (err) => {
    if (err) console.error('Pipeline failed:', err);
  }
);

// 3. Set appropriate highWaterMark
fs.createReadStream('file.txt', { highWaterMark: 64 * 1024 });

// 4. Destroy streams on completion
stream.destroy();

// 5. Use stream.pipeline for backpressure handling
```

### **Memory Management Best Practices**
```javascript
// 1. Monitor memory regularly
setInterval(() => {
  const used = process.memoryUsage();
  if (used.heapUsed > 500 * 1024 * 1024) { // 500MB
    console.warn('High memory usage!');
  }
}, 60000);

// 2. Use weak references for caches
const { WeakRef, FinalizationRegistry } = require('weakref');
const registry = new FinalizationRegistry((heldValue) => {
  console.log(`${heldValue} was garbage collected`);
});

// 3. Limit cache sizes
const lru = require('lru-cache');
const cache = new lru({ max: 1000, maxAge: 1000 * 60 * 60 });

// 4. Clean up event listeners
emitter.removeAllListeners();
// Or store reference and remove individually
const handler = () => {};
emitter.on('event', handler);
// Later...
emitter.off('event', handler);

// 5. Use streams for large data
// Instead of: const data = fs.readFileSync('hugefile.txt');
// Use: fs.createReadStream('hugefile.txt').pipe(processData);
```

### **Debugging Best Practices**
```javascript
// 1. Use structured logging
const logger = require('./logger');
logger.info('User action', { 
  userId: user.id, 
  action: 'login',
  ip: req.ip 
});

// 2. Add correlation IDs
app.use((req, res, next) => {
  req.correlationId = require('crypto').randomUUID();
  next();
});

// 3. Use APM in production
if (process.env.NODE_ENV === 'production') {
  require('newrelic');
}

// 4. Enable heap snapshots on OOM
node --max-old-space-size=512 --heapsnapshot-on-signal app.js

// 5. Profile CPU usage
node --cpu-prof app.js
// Generates .cpuprofile file for Chrome DevTools
```

These tools and techniques provide comprehensive solutions for building robust, efficient, and maintainable Node.js applications that can handle real-world production workloads while being debuggable and monitorable.