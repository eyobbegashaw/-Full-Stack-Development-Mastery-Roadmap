# **Process Management & Concurrency in Node.js - Detailed Notes & Code**

## **Process Management**

### **Keep app Running**
Keeping applications running ensures high availability and reliability. Node.js processes can crash due to unhandled exceptions, memory limits, or external failures. Use process managers like PM2, Forever, or systemd to automatically restart crashed applications. Implement graceful shutdowns, health checks, and monitoring for production deployments. This prevents downtime and maintains service continuity.

```javascript
// Basic keep-alive with auto-restart
const cluster = require('cluster');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
  
  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died. Restarting...`);
    cluster.fork();
  });
} else {
  require('./app.js');
}
```

### **pm2**
PM2 is a **production process manager** for Node.js applications. Features: process management, load balancing, zero-downtime reloads, logging, monitoring, and startup scripts. Manages application lifecycle, auto-restarts on crashes, and provides CLI for process management. Essential for production deployment.

```bash
# Install PM2 globally
npm install -g pm2

# Start application
pm2 start app.js --name "my-app"

# Common commands
pm2 list              # List all processes
pm2 logs my-app       # View logs
pm2 monit             # Monitor dashboard
pm2 restart my-app    # Restart application
pm2 stop my-app       # Stop application
pm2 delete my-app     # Remove from PM2
pm2 save              # Save process list
pm2 startup           # Generate startup script

# With ecosystem file
pm2 start ecosystem.config.js
```

```javascript
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'my-app',
    script: './app.js',
    instances: 'max',           // Use all CPU cores
    exec_mode: 'cluster',       // Cluster mode
    watch: false,               // Don't watch for changes in production
    max_memory_restart: '1G',   // Restart if memory exceeds 1GB
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
    time: true                  // Add timestamp to logs
  }]
};
```

## **Concurrency Models**

### **Child Process**
Child processes spawn new operating system processes to run shell commands or other programs. Use for CPU-intensive tasks or running external commands. Methods: `spawn()` (streaming), `exec()` (buffered), `execFile()`, `fork()` (Node-specific). Communicate via IPC (Inter-Process Communication).

```javascript
const { spawn, exec, fork } = require('child_process');

// spawn - for streaming large outputs
const ls = spawn('ls', ['-la']);
ls.stdout.on('data', (data) => {
  console.log(`stdout: ${data}`);
});

// exec - for small outputs
exec('ls -la', (error, stdout, stderr) => {
  if (error) {
    console.error(`exec error: ${error}`);
    return;
  }
  console.log(`stdout: ${stdout}`);
});

// fork - for Node.js modules
const child = fork('./child-script.js');
child.on('message', (msg) => {
  console.log('Message from child:', msg);
});
child.send({ hello: 'world' });
```

### **Cluster**
Cluster module creates **multiple Node.js processes sharing the same server port**. Uses multiple CPU cores for better performance. Master process distributes incoming connections to worker processes via round-robin (except on Windows). Improves throughput for I/O-bound applications like web servers.

```javascript
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  console.log(`Master ${process.pid} is running`);
  
  // Fork workers
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
  
  // Handle worker events
  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
    // Restart worker
    cluster.fork();
  });
  
  cluster.on('online', (worker) => {
    console.log(`Worker ${worker.process.pid} is online`);
  });
  
} else {
  // Workers share the same port
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end(`Hello from worker ${process.pid}\n`);
  }).listen(8000);
  
  console.log(`Worker ${process.pid} started`);
}
```

### **Worker Threads**
Worker Threads enable **running JavaScript in parallel threads** within the same process. Share memory via `SharedArrayBuffer`. Unlike child processes, they're lighter weight and share the same memory space. Ideal for CPU-intensive JavaScript operations. Each thread has its own V8 instance and event loop.

```javascript
const { Worker, isMainThread, parentPort, workerData } = require('worker_threads');

if (isMainThread) {
  // Main thread
  const worker = new Worker(__filename, {
    workerData: { numbers: [1, 2, 3, 4, 5] }
  });
  
  worker.on('message', (result) => {
    console.log('Result from worker:', result);
  });
  
  worker.on('error', (err) => {
    console.error('Worker error:', err);
  });
  
  worker.on('exit', (code) => {
    if (code !== 0) {
      console.error(`Worker stopped with exit code ${code}`);
    }
  });
  
} else {
  // Worker thread
  const heavyComputation = (numbers) => {
    return numbers.map(n => n * n).reduce((a, b) => a + b, 0);
  };
  
  const result = heavyComputation(workerData.numbers);
  parentPort.postMessage(result);
}
```

### **Threads**
Threads refer to **parallel execution units** within a process. Node.js is single-threaded for JavaScript execution but uses thread pool for I/O operations (via libuv). Worker Threads API allows true multi-threading. Use threads for CPU-bound tasks while keeping the main thread responsive for I/O operations.

```javascript
const { ThreadPool } = require('worker_threads');

class ThreadPool {
  constructor(poolSize, workerFile) {
    this.pool = [];
    for (let i = 0; i < poolSize; i++) {
      this.pool.push(new Worker(workerFile));
    }
    this.index = 0;
  }
  
  execute(data) {
    const worker = this.pool[this.index];
    this.index = (this.index + 1) % this.pool.length;
    
    return new Promise((resolve, reject) => {
      worker.once('message', resolve);
      worker.once('error', reject);
      worker.postMessage(data);
    });
  }
}
```

## **Complete Examples**

### **PM2 with Cluster Mode**
```javascript
// app.js - Express server optimized for clustering
const express = require('express');
const cluster = require('cluster');
const os = require('os');

const app = express();

// Health check endpoint
app.get('/health', (req, res) => {
  res.json({ 
    status: 'healthy',
    pid: process.pid,
    memory: process.memoryUsage(),
    uptime: process.uptime()
  });
});

// Heavy computation endpoint (use worker threads in real app)
app.get('/compute', (req, res) => {
  let sum = 0;
  for (let i = 0; i < 1e7; i++) {
    sum += Math.sqrt(i);
  }
  res.json({ 
    result: sum,
    pid: process.pid 
  });
});

// Normal route
app.get('/', (req, res) => {
  res.send(`Hello from process ${process.pid} on core`);
});

// Graceful shutdown
process.on('SIGINT', () => {
  console.log(`Process ${process.pid} shutting down gracefully`);
  server.close(() => {
    process.exit(0);
  });
});

const PORT = process.env.PORT || 3000;
const server = app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}, PID: ${process.pid}`);
});
```

```javascript
// ecosystem.config.js for PM2
module.exports = {
  apps: [{
    name: 'my-cluster-app',
    script: './app.js',
    instances: 'max',               // Use all CPU cores
    exec_mode: 'cluster',           // Enable clustering
    autorestart: true,
    watch: false,
    max_memory_restart: '500M',     // Restart if memory > 500MB
    env: {
      NODE_ENV: 'development'
    },
    env_production: {
      NODE_ENV: 'production',
      PORT: 80
    },
    // Advanced logging
    log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
    merge_logs: true,
    error_file: '/var/log/node-app/err.log',
    out_file: '/var/log/node-app/out.log',
    pid_file: '/var/run/node-app.pid',
    // Monitoring
    min_uptime: '60s',
    max_restarts: 10
  }]
};
```

### **Worker Pool for CPU-Intensive Tasks**
```javascript
// worker-pool.js
const { Worker } = require('worker_threads');
const path = require('path');

class WorkerPool {
  constructor(poolSize, workerScript) {
    this.poolSize = poolSize;
    this.workerScript = workerScript;
    this.workers = [];
    this.tasks = [];
    this.init();
  }

  init() {
    for (let i = 0; i < this.poolSize; i++) {
      const worker = new Worker(this.workerScript);
      
      worker.on('message', (result) => {
        const { taskId, data } = result;
        const task = this.tasks.find(t => t.id === taskId);
        if (task) {
          task.resolve(data);
          this.tasks = this.tasks.filter(t => t.id !== taskId);
        }
        // Mark worker as available
        worker.isBusy = false;
        this.processQueue();
      });

      worker.on('error', (error) => {
        console.error(`Worker error: ${error}`);
        // Handle worker failure
        this.replaceWorker(worker);
      });

      worker.on('exit', (code) => {
        if (code !== 0) {
          console.error(`Worker exited with code ${code}`);
          this.replaceWorker(worker);
        }
      });

      this.workers.push({ worker, isBusy: false });
    }
  }

  execute(taskData) {
    return new Promise((resolve, reject) => {
      const taskId = Date.now() + Math.random();
      this.tasks.push({ id: taskId, resolve, reject, data: taskData });
      this.processQueue();
    });
  }

  processQueue() {
    const availableWorker = this.workers.find(w => !w.isBusy);
    if (availableWorker && this.tasks.length > 0) {
      const task = this.tasks.shift();
      availableWorker.isBusy = true;
      availableWorker.worker.postMessage({
        taskId: task.id,
        data: task.data
      });
    }
  }

  replaceWorker(failedWorker) {
    const index = this.workers.findIndex(w => w.worker === failedWorker);
    if (index !== -1) {
      const newWorker = new Worker(this.workerScript);
      this.workers[index] = { worker: newWorker, isBusy: false };
      console.log('Worker replaced');
    }
  }

  async terminate() {
    await Promise.all(
      this.workers.map(({ worker }) => worker.terminate())
    );
  }
}

module.exports = WorkerPool;
```

```javascript
// image-processor-worker.js
const { parentPort } = require('worker_threads');

parentPort.on('message', async ({ taskId, data }) => {
  try {
    // Simulate CPU-intensive image processing
    const result = processImage(data);
    parentPort.postMessage({ taskId, data: result });
  } catch (error) {
    parentPort.postMessage({ taskId, error: error.message });
  }
});

function processImage(imageData) {
  // CPU-intensive image processing
  const start = Date.now();
  let processedPixels = 0;
  
  // Simulate pixel processing
  for (let i = 0; i < 10000000; i++) {
    processedPixels += Math.sqrt(i) * Math.random();
  }
  
  return {
    processed: true,
    pixels: processedPixels,
    duration: Date.now() - start,
    threadId: require('worker_threads').threadId
  };
}
```

```javascript
// app.js using worker pool
const express = require('express');
const WorkerPool = require('./worker-pool');

const app = express();
const workerPool = new WorkerPool(4, './image-processor-worker.js');

app.use(express.json({ limit: '10mb' }));

app.post('/process-image', async (req, res) => {
  try {
    const imageData = req.body;
    
    // Use worker pool for CPU-intensive task
    const result = await workerPool.execute(imageData);
    
    res.json({
      success: true,
      result,
      processedBy: `Worker ${result.threadId}`
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.get('/stats', (req, res) => {
  res.json({
    workers: workerPool.workers.length,
    tasksInQueue: workerPool.tasks.length,
    busyWorkers: workerPool.workers.filter(w => w.isBusy).length
  });
});

// Graceful shutdown
process.on('SIGTERM', async () => {
  console.log('SIGTERM received, shutting down gracefully');
  await workerPool.terminate();
  process.exit(0);
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### **Advanced Child Process Management**
```javascript
// process-manager.js
const { spawn } = require('child_process');
const EventEmitter = require('events');

class ProcessManager extends EventEmitter {
  constructor(command, args = [], options = {}) {
    super();
    this.command = command;
    this.args = args;
    this.options = options;
    this.process = null;
    this.restartCount = 0;
    this.maxRestarts = options.maxRestarts || 5;
    this.restartDelay = options.restartDelay || 1000;
  }

  start() {
    this.process = spawn(this.command, this.args, {
      stdio: ['pipe', 'pipe', 'pipe'],
      ...this.options
    });

    this.process.stdout.on('data', (data) => {
      this.emit('stdout', data.toString());
    });

    this.process.stderr.on('data', (data) => {
      this.emit('stderr', data.toString());
    });

    this.process.on('close', (code) => {
      this.emit('close', code);
      if (code !== 0 && this.restartCount < this.maxRestarts) {
        this.restartCount++;
        setTimeout(() => this.start(), this.restartDelay);
      } else {
        this.emit('maxRestartsExceeded');
      }
    });

    this.process.on('error', (error) => {
      this.emit('error', error);
    });

    this.emit('started', this.process.pid);
  }

  stop() {
    if (this.process) {
      this.process.kill('SIGTERM');
    }
  }

  restart() {
    this.stop();
    setTimeout(() => this.start(), this.restartDelay);
  }

  send(message) {
    if (this.process && this.process.stdin.writable) {
      this.process.stdin.write(message + '\n');
    }
  }
}

// Usage
const pythonProcess = new ProcessManager('python', ['script.py'], {
  maxRestarts: 3,
  restartDelay: 2000
});

pythonProcess.on('stdout', (data) => {
  console.log('Python output:', data);
});

pythonProcess.on('stderr', (data) => {
  console.error('Python error:', data);
});

pythonProcess.on('started', (pid) => {
  console.log(`Python process started with PID: ${pid}`);
});

pythonProcess.start();
```

## **Best Practices & Comparison**

### **When to Use Which Concurrency Model**
```javascript
/*
1. Child Process:
   - Run shell commands or other programs
   - Heavy computation in other languages (Python, Rust)
   - Isolate risky operations

2. Cluster:
   - Web servers (Express, Fastify)
   - I/O-bound applications
   - Utilize multiple CPU cores

3. Worker Threads:
   - CPU-intensive JavaScript operations
   - Image/video processing
   - Complex mathematical computations
   - When you need shared memory

4. PM2:
   - Production deployment
   - Process management and monitoring
   - Zero-downtime deployments
   - Log management
*/
```

### **Performance Comparison Table**
```
| Method         | Memory | Startup | IPC     | Use Case                |
|----------------|--------|---------|---------|-------------------------|
| Child Process  | High   | Slow    | Slow    | External programs       |
| Cluster        | Medium | Fast    | N/A     | HTTP servers            |
| Worker Threads | Low    | Fast    | Fast    | CPU-intensive JS        |
| PM2 Cluster    | Medium | Fast    | N/A     | Production deployment   |
```

### **Memory Management Tips**
```javascript
// Monitor memory usage
setInterval(() => {
  const memoryUsage = process.memoryUsage();
  console.log(`Memory: ${Math.round(memoryUsage.heapUsed / 1024 / 1024)}MB`);
}, 60000);

// Force garbage collection (development only)
if (global.gc) {
  global.gc();
}

// Limit worker memory
const worker = new Worker('./worker.js', {
  resourceLimits: {
    maxOldGenerationSizeMb: 512,
    maxYoungGenerationSizeMb: 256,
    codeRangeSizeMb: 128
  }
});

// PM2 memory limit restart
// In ecosystem.config.js:
max_memory_restart: '1G'  // Restart if > 1GB
```

### **Error Handling & Monitoring**
```javascript
// PM2 with error tracking
const pm2 = require('pm2');

pm2.connect((err) => {
  if (err) {
    console.error(err);
    process.exit(2);
  }
  
  pm2.start({
    script: 'app.js',
    name: 'my-app',
    error_file: './logs/err.log',
    out_file: './logs/out.log',
    merge_logs: true,
    log_date_format: 'YYYY-MM-DD HH:mm:ss'
  }, (err, apps) => {
    pm2.disconnect();
    if (err) throw err;
  });
});

// Process events for monitoring
process.on('uncaughtException', (error) => {
  console.error('Uncaught Exception:', error);
  // Don't exit immediately in production
  if (process.env.NODE_ENV === 'production') {
    // Log to external service
    // Continue running if possible
  } else {
    process.exit(1);
  }
});

process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection at:', promise, 'reason:', reason);
});
```

These tools provide robust solutions for managing Node.js processes in production, handling CPU-intensive workloads, and maximizing server resources. PM2 simplifies deployment and monitoring, while Worker Threads and Cluster enable true parallelism for different use cases.